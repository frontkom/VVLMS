# LMS Platform Architecture

## Document Status

| Field | Value |
|-------|-------|
| Status | Proposed |
| Date | 2026-03-22 |
| Scope | Full platform architecture for Norwegian public sector LMS SaaS |
| First target tenant | Statens vegvesen (~5,300 internal + ~15,000 external users) |

---

## 1. Tech Stack Selection

### 1.1 Frontend: Next.js 15 (React 19) + TypeScript

**Decision**: Next.js with App Router and React Server Components.

**Rationale**:
- Server-side rendering for WCAG/universal design compliance (accessibility requires content without JS where feasible)
- React has the largest talent pool in Norway and the broadest component ecosystem
- Next.js provides file-based routing, API routes (for BFF patterns), and built-in i18n support (Norwegian bokmaal/nynorsk)
- Server Components reduce client bundle size, improving performance on lower-end devices used by external users (construction workers, contractors)
- Strong TypeScript support for type safety across the full stack

**UI component library**: Radix UI primitives + Tailwind CSS. Radix provides unstyled, accessible-by-default components (WCAG 2.1 AA). Tailwind enables rapid consistent styling without a heavy design system framework.

**SCORM player UI**: Iframe-based content player embedded within the Next.js shell. The SCORM runtime API is injected into the iframe context.

**What we're giving up**: SvelteKit would give us smaller bundles and simpler reactivity. Vue/Nuxt would be equally viable. But React/Next.js is the pragmatic choice for hiring in Norway and long-term ecosystem support.

### 1.2 Backend: Node.js with NestJS + TypeScript

**Decision**: NestJS framework on Node.js.

**Rationale**:
- Full-stack TypeScript: shared types and validation schemas between frontend and backend eliminate an entire class of integration bugs
- NestJS provides opinionated structure (modules, guards, interceptors, pipes) that scales well for a multi-tenant SaaS
- First-class support for OpenAPI/Swagger generation from decorators
- Mature ecosystem for OIDC/SAML (passport.js), SCIM, and webhook handling
- Good performance for I/O-bound workloads (which LMS is: database queries, file serving, API proxying)
- Easy to hire for in Norway

**Key NestJS modules**:
- `@nestjs/passport` + `openid-client` for OIDC/SAML
- `@nestjs/bull` for job queues (backed by Redis)
- `@nestjs/terminus` for health checks
- Custom SCORM runtime module
- Custom SCIM 2.0 server module

**What we're giving up**: Go or Rust would give better raw throughput and lower memory usage. Python/FastAPI would be simpler for data-heavy operations. But shared TypeScript across the stack reduces total system complexity and team size requirements.

### 1.3 Database: PostgreSQL 16 + Redis + Meilisearch

#### Primary database: PostgreSQL 16

**Rationale**:
- Row-level security (RLS) for multi-tenancy (see Section 2)
- JSONB columns for flexible metadata (SCORM CMI data, xAPI statements, custom fields per tenant)
- Full-text search with `tsvector` for Norwegian language (built-in Norwegian stemmer)
- Partitioning for large tables (completions, xAPI statements) by tenant and date
- Mature, battle-tested, excellent tooling
- Available as managed service on all major cloud providers

#### Cache and job queue backing store: Redis 7

- Session cache (JWT token blacklist for logout)
- Rate limiting counters
- Bull queue backing store for async jobs
- Cached tenant configuration (avoid DB lookup on every request)
- SCORM runtime state cache (suspend data, bookmark) with write-through to PostgreSQL

#### Search engine: Meilisearch

**Rationale**:
- Typo-tolerant, instant search for course catalog (critical UX requirement from tender)
- Simple to operate compared to Elasticsearch (single binary, minimal config)
- Multi-tenant search via tenant-scoped indexes
- Norwegian language support
- Faceted search for filtering by category, competency area, format, duration

**What we're giving up**: Elasticsearch would give us more powerful analytics and aggregation. But Meilisearch is dramatically simpler to operate and more than sufficient for course catalog search. If we later need log analytics or complex aggregations, we add Elasticsearch for that specific use case.

### 1.4 File Storage: S3-Compatible Object Storage

- SCORM/xAPI/cmi5 packages (ZIP files, extracted to folder structure)
- Video content (with CDN for streaming)
- Certificate PDFs (generated and archived)
- Course thumbnails and media assets
- User profile images

Storage is organized by tenant: `s3://{bucket}/{tenant_id}/scorm/{package_id}/`, etc.

Presigned URLs for direct upload/download to avoid proxying large files through the application server.

### 1.5 Message Queue: BullMQ (Redis-backed)

**Decision**: BullMQ for async job processing, not a standalone message broker.

**Rationale**:
- We already need Redis for caching; BullMQ uses the same infrastructure
- Job types: course completion processing, certificate generation (PDF), notification dispatch, SCIM sync, PowerBI data export, xAPI statement forwarding
- Supports delayed jobs (e.g., send reminder 7 days before certification expires)
- Supports rate limiting (e.g., don't overwhelm ServiceNow API)
- Dashboard UI (Bull Board) for operations monitoring
- If we outgrow BullMQ, we can introduce RabbitMQ or AWS SQS for specific high-throughput flows without replacing the entire system

**What we're giving up**: Kafka or RabbitMQ would give us stronger delivery guarantees and better throughput. But BullMQ is operationally simpler and sufficient for our scale (tens of thousands of events/day, not millions).

---

## 2. Multi-Tenancy Architecture

### Decision: Schema-per-tenant with shared application layer

**Architecture**:

```
                    ┌─────────────────────────┐
                    │   Load Balancer / CDN    │
                    │  tenant1.lms.no          │
                    │  tenant2.lms.no          │
                    └──────────┬──────────────┘
                               │
                    ┌──────────▼──────────────┐
                    │   Application Layer      │
                    │   (shared NestJS)         │
                    │   Tenant resolved from:   │
                    │   - subdomain             │
                    │   - JWT claim             │
                    │   - API key               │
                    └──────────┬──────────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
     ┌────────▼───────┐ ┌─────▼──────┐ ┌───────▼──────┐
     │ schema: svv     │ │ schema: t2  │ │ schema: t3   │
     │ (vegvesen)      │ │ (tenant 2)  │ │ (tenant 3)   │
     │ PostgreSQL      │ │ PostgreSQL  │ │ PostgreSQL   │
     └────────────────┘ └────────────┘ └──────────────┘
              │                │                │
              └────────────────┼────────────────┘
                               │
                    ┌──────────▼──────────────┐
                    │  schema: shared          │
                    │  (platform config,       │
                    │   tenant registry,       │
                    │   billing)               │
                    └─────────────────────────┘
```

**How it works**:

1. **Tenant resolution**: Middleware extracts tenant from subdomain (`svv.lms.no`), JWT `tenant_id` claim, or API key. Sets `search_path` on the database connection for the request.

2. **Schema isolation**: Each tenant gets its own PostgreSQL schema with identical table structure. Migrations run against all tenant schemas. A `shared` schema holds platform-level data (tenant registry, feature flags, platform admin accounts).

3. **Connection pooling**: PgBouncer in front of PostgreSQL. Connection pool per tenant schema to prevent noisy-neighbor issues. Each request gets a connection with `SET search_path TO {tenant_schema}, shared;`.

4. **Row-level security as defense-in-depth**: Even with schema separation, RLS policies on the `shared` schema tables prevent cross-tenant data access if a bug bypasses schema switching.

**Why schema-per-tenant over alternatives**:

| Approach | Pros | Cons | Verdict |
|----------|------|------|---------|
| Shared tables + RLS only | Simple deployment, easy cross-tenant queries | One bug leaks all tenant data; Norwegian public sector will not accept this risk | Rejected |
| Schema-per-tenant | Strong isolation, easy per-tenant backup/restore, satisfies security audits | More complex migrations, limited cross-tenant analytics | **Selected** |
| Database-per-tenant | Strongest isolation | Operational nightmare at scale, connection overhead | Overkill |

**Trade-off acknowledged**: Schema-per-tenant makes cross-tenant analytics harder. We solve this with a read-only analytics database that aggregates anonymized data across tenants for platform-level insights.

### Tenant configuration

Each tenant has a configuration record in the `shared` schema:

```typescript
interface TenantConfig {
  id: string;                          // UUID
  slug: string;                        // 'svv' - used in subdomain
  name: string;                        // 'Statens vegvesen'
  schema_name: string;                 // 'tenant_svv'
  iam_provider: 'forgerock' | 'azure_ad' | 'entra_id';
  iam_config: {                        // OIDC/SAML endpoints
    issuer: string;
    client_id: string;
    scim_endpoint?: string;
    // ...
  };
  features: {                          // Feature flags
    external_portal: boolean;
    id_porten: boolean;
    scorm_player: boolean;
    xapi_lrs: boolean;
    ai_recommendations: boolean;
    certificate_generation: boolean;
    payment: boolean;
  };
  integrations: IntegrationConfig[];   // ServiceNow, PowerBI, etc.
  branding: {                          // White-label
    logo_url: string;
    primary_color: string;
    favicon_url: string;
  };
  locale: 'nb' | 'nn' | 'en';         // Default locale
  data_retention_days: number;         // GDPR retention policy
}
```

---

## 3. API Design

### Decision: REST with OpenAPI 3.1, plus targeted GraphQL for reporting

**Primary API: REST**

- OpenAPI 3.1 specification generated from NestJS decorators (via `@nestjs/swagger`)
- Versioned via URL path: `/api/v1/courses`, `/api/v2/courses`
- JSON:API-inspired response format for consistency (but not full JSON:API to avoid complexity)
- HATEOAS links for discoverability where useful (pagination, related resources)

**Why not GraphQL everywhere**: Most LMS operations are well-modeled as REST resources (courses, enrollments, completions). GraphQL adds complexity without proportional benefit for CRUD-heavy domains. However, the reporting/analytics surface benefits from GraphQL's flexible querying.

### API structure

```
/api/v1/
├── auth/
│   ├── POST   /login                    # Local login (dev/test only)
│   ├── GET    /oidc/callback            # OIDC callback
│   ├── GET    /saml/callback            # SAML callback
│   └── POST   /logout
│
├── users/
│   ├── GET    /                         # List users (admin)
│   ├── GET    /:id                      # Get user profile
│   ├── PATCH  /:id                      # Update profile
│   └── GET    /:id/enrollments          # User's enrollments
│
├── scim/v2/                             # SCIM 2.0 server endpoints
│   ├── GET    /Users                    # List/filter users
│   ├── POST   /Users                    # Provision user
│   ├── GET    /Users/:id
│   ├── PUT    /Users/:id               # Replace user
│   ├── PATCH  /Users/:id              # Update user (SCIM PATCH)
│   ├── DELETE /Users/:id              # Deprovision user
│   ├── GET    /Groups                  # List groups
│   ├── POST   /Groups                  # Create group
│   └── ...
│
├── courses/
│   ├── GET    /                         # List/search courses
│   ├── POST   /                         # Create course (admin)
│   ├── GET    /:id                      # Course detail
│   ├── PATCH  /:id                      # Update course
│   ├── DELETE /:id                      # Archive course
│   ├── POST   /:id/enroll              # Enroll user(s)
│   └── GET    /:id/completions          # Course completions
│
├── content/
│   ├── POST   /scorm/upload             # Upload SCORM package
│   ├── GET    /scorm/:id/launch         # Get SCORM launch URL
│   ├── POST   /xapi/statements          # xAPI statement inbox
│   └── GET    /xapi/statements          # xAPI statement query
│
├── learning-paths/
│   ├── GET    /
│   ├── POST   /
│   ├── GET    /:id
│   └── GET    /:id/progress/:userId
│
├── competencies/
│   ├── GET    /                         # List competency frameworks
│   ├── GET    /:id/gaps/:userId         # Competency gap analysis
│   └── GET    /:id/certificates         # Certificates for competency
│
├── certificates/
│   ├── GET    /:id                      # Get certificate
│   ├── GET    /:id/pdf                  # Download PDF
│   └── POST   /:id/verify              # Verify certificate (public)
│
├── reports/                             # GraphQL endpoint for flexible reporting
│   └── POST   /graphql
│
├── kursoppslag/                         # Proxy API (maintains existing contract)
│   ├── GET    /courses
│   ├── GET    /courses/:id
│   ├── GET    /courses/:id/participants
│   ├── GET    /learning-plans
│   └── GET    /learning-plans/:id/participants
│
└── webhooks/
    ├── POST   /                         # Register webhook
    └── GET    /                          # List webhooks
```

### Rate limiting

- Per-tenant rate limits (configurable): default 1,000 req/min for API, 100 req/min for SCIM
- Per-user rate limits: 200 req/min
- Sliding window algorithm backed by Redis
- Rate limit headers in responses: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- Maskinporten-authenticated machine clients get higher limits (5,000 req/min)

### Kursoppslag compatibility layer

The tender requires maintaining the existing Kursoppslag proxy API contract. We implement this as a dedicated NestJS module that maps internal data models to the legacy API response format. This is a conformist pattern: we accept the upstream contract as-is rather than negotiating changes.

---

## 4. SCORM/xAPI/cmi5 Runtime

This is the hardest technical challenge in the platform. There are three viable approaches:

### Options analysis

| Option | Cost | Effort | Risk | Vendor lock-in |
|--------|------|--------|------|----------------|
| A: Rustici Engine (cloud) | ~$30-50K/yr | Low (weeks) | Low (proven) | High |
| B: Open-source player (e.g., SCORM Again) | $0 license | Medium (months) | Medium | None |
| C: Custom runtime from scratch | $0 license | High (6+ months) | High | None |

### Decision: Option B - Open-source foundation with custom enhancements

**Rationale**:
- Rustici Engine is proven but creates a critical vendor dependency and ongoing cost that makes our SaaS pricing less competitive
- Building from scratch for SCORM 2004 4th Edition is a 6+ month effort with high risk of edge-case bugs
- Open-source SCORM players (like `scorm-again` or similar JS libraries) handle the SCORM API spec correctly and are well-tested against real content
- We wrap the open-source player in our own runtime shell that handles multi-tenancy, storage, and reporting

**Architecture**:

```
┌──────────────────────────────────────────────────────┐
│                  LMS Frontend Shell                   │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Content Player Component            │ │
│  │  ┌───────────────────────────────────────────┐  │ │
│  │  │          SCORM API Adapter Layer           │  │ │
│  │  │  ┌─────────┐ ┌──────────┐ ┌───────────┐  │  │ │
│  │  │  │SCORM 1.2│ │SCORM 2004│ │  xAPI/cmi5 │  │  │ │
│  │  │  │ Runtime │ │ Runtime  │ │  Runtime   │  │  │ │
│  │  │  └────┬────┘ └────┬─────┘ └─────┬─────┘  │  │ │
│  │  │       └───────────┼─────────────┘         │  │ │
│  │  │              API Bridge                    │  │ │
│  │  └───────────────────┼───────────────────────┘  │ │
│  │                      │                           │ │
│  │            ┌─────────▼─────────┐                │ │
│  │            │   Content iframe   │                │ │
│  │            │  (SCORM package)   │                │ │
│  │            └───────────────────┘                │ │
│  └─────────────────────────────────────────────────┘ │
│                         │                             │
│              WebSocket / REST                         │
│                         │                             │
└─────────────────────────┼────────────────────────────┘
                          │
┌─────────────────────────▼────────────────────────────┐
│              Content Runtime Service (Backend)         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐│
│  │ CMI Data Store│  │ xAPI LRS     │  │ Package     ││
│  │ (PostgreSQL + │  │ (conformant  │  │ Manager     ││
│  │  Redis cache) │  │  statement   │  │ (extract,   ││
│  │              │  │  store)      │  │  validate,  ││
│  │              │  │              │  │  serve)     ││
│  └──────────────┘  └──────────────┘  └─────────────┘│
└──────────────────────────────────────────────────────┘
```

**How it works**:

1. **Package upload**: Admin uploads SCORM ZIP. Backend extracts, validates manifest (imsmanifest.xml for SCORM, tincan.xml for xAPI, cmi5.xml for cmi5), stores files in S3, records metadata in database.

2. **Launch**: User clicks "Start course". Frontend loads Content Player Component. Backend generates a signed launch URL with session token. Content iframe loads the SCORM entry point from S3 (via CDN).

3. **Runtime communication**: The SCORM content calls `API.LMSSetValue()` / `API_1484_11.SetValue()`. Our JavaScript adapter intercepts these calls, validates data, and persists to backend via WebSocket (for real-time saves) or REST (for batch commits).

4. **Suspend/resume**: CMI suspend data and bookmarks are cached in Redis for fast retrieval and persisted to PostgreSQL. When a learner resumes, the cached state is loaded instantly.

5. **xAPI/cmi5**: For xAPI content, we implement a conformant Learning Record Store (LRS) endpoint. For cmi5, we implement the cmi5 launch mechanism (fetch URL, AU launch) and map cmi5 defined statements to our completion model.

6. **Completion detection**: The adapter monitors `cmi.core.lesson_status` (SCORM 1.2), `cmi.completion_status` + `cmi.success_status` (SCORM 2004), or xAPI completion/passed verbs. On completion, an event is emitted to the job queue for downstream processing (certificates, notifications, ServiceNow sync).

**Risk mitigation**: We build a SCORM content test suite using the ADL SCORM test content packages. Every SCORM package upload runs through a manifest validator. We maintain a compatibility matrix of tested authoring tools (Articulate Storyline/Rise, Adobe Captivate, iSpring, Lectora).

### Fallback plan

If the open-source approach proves insufficient for edge cases in SCORM 2004 4th Edition sequencing and navigation, we can integrate Rustici Engine for complex SCORM 2004 content while keeping our lightweight player for SCORM 1.2 and xAPI. This hybrid approach limits Rustici costs to only the content that needs it.

---

## 5. Authentication Architecture

This is the most integration-heavy part of the system. SVV uses ForgeRock for IAM, and external users authenticate via ID-porten (Norwegian national identity provider).

### Authentication flow overview

```
                    ┌─────────────┐
                    │   User      │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────────┐
              │            │                │
    ┌─────────▼──────┐ ┌──▼────────┐ ┌─────▼────────┐
    │ Internal (SSO)  │ │ External  │ │ Machine-to-  │
    │ ForgeRock OIDC  │ │ ID-porten │ │ Machine      │
    │ or SAML 2.0     │ │ OIDC      │ │ Maskinporten │
    └─────────┬──────┘ └──┬────────┘ └─────┬────────┘
              │            │                │
              └────────────┼────────────────┘
                           │
                    ┌──────▼──────┐
                    │  LMS Auth   │
                    │  Gateway    │
                    │             │
                    │ - Validate  │
                    │   tokens    │
                    │ - Map to    │
                    │   internal  │
                    │   user      │
                    │ - Issue LMS │
                    │   session   │
                    │   (JWT)     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ LMS Platform│
                    └─────────────┘
```

### 5.1 OIDC/SAML support

**Implementation**: NestJS Passport strategies for both OIDC and SAML 2.0.

- **OIDC flow** (primary for ForgeRock, ID-porten):
  - Authorization Code Flow with PKCE
  - Discovery via `.well-known/openid-configuration`
  - Token validation: signature verification, issuer, audience, expiry, nonce
  - Claims mapping: `sub` -> internal user ID, `email`, `name`, `groups`

- **SAML 2.0 flow** (fallback for legacy IdPs):
  - SP-initiated SSO
  - Assertion validation: signature, conditions, audience restriction
  - Attribute mapping configurable per tenant

**Per-tenant IdP configuration**: Each tenant can configure multiple IdPs (e.g., ForgeRock for internal users, ID-porten for external users). The login page shows relevant options based on the tenant and user type.

### 5.2 SCIM 2.0 server

We implement a SCIM 2.0 server endpoint that ForgeRock (or other IdPs) pushes user lifecycle events to.

**Supported operations**:
- `POST /scim/v2/Users` - Create user (provisioning)
- `PUT /scim/v2/Users/:id` - Replace user
- `PATCH /scim/v2/Users/:id` - Update user attributes
- `DELETE /scim/v2/Users/:id` - Deactivate user (soft delete, preserve learning history)
- `GET /scim/v2/Users` - List/filter users (for reconciliation)
- `POST /scim/v2/Groups` - Create group/role
- `PATCH /scim/v2/Groups/:id` - Update group membership

**Schema mapping**: SCIM core schema (`urn:ietf:params:scim:schemas:core:2.0:User`) is mapped to our internal user model. Custom enterprise extension attributes (department, division, job title, manager) are mapped to SVV-specific fields via configurable mapping per tenant.

**Conflict resolution**: If a SCIM push conflicts with a local edit, SCIM wins (IdP is source of truth for identity attributes). Learning data is never modified by SCIM.

### 5.3 ID-porten integration

**Protocol**: OIDC Authorization Code Flow with PKCE.

**Scopes**: `openid`, `profile`, `pid` (personal identification number).

**User matching**: External users are matched by `pid` (Norwegian national identity number, encrypted at rest). If no existing user, a self-registration flow is triggered.

**Security levels**: ID-porten supports multiple authentication levels (Level 3: MinID, Level 4: BankID). For issuing competency certificates (KKP portal), we require Level 4 (BankID) to ensure identity assurance.

### 5.4 BankID / MinID

These are authentication methods within ID-porten, not separate integrations. ID-porten handles the BankID/MinID authentication and returns the result to us via OIDC. We configure the required `acr_values` (authentication context class reference) to demand the appropriate security level:

- `idporten-loa-substantial` (Level 3) - MinID, for general course access
- `idporten-loa-high` (Level 4) - BankID, for certificate issuance and sensitive operations

### 5.5 Maskinporten

**Purpose**: Machine-to-machine authentication for API access (e.g., Kursoppslag, ServiceNow integration, PowerBI data pull).

**Implementation**:
- Our platform registers as an API provider in Maskinporten
- External systems request access tokens using their own Maskinporten client (JWT grant)
- We validate Maskinporten tokens, checking `scope` claims against our API permission model
- Scopes are defined per integration: `svv:lms.courses.read`, `svv:lms.completions.read`, etc.

### 5.6 Multi-factor authentication

- For internal users: MFA is handled by ForgeRock (not our responsibility, but we support `amr` claim verification)
- For external users: ID-porten provides MFA natively (BankID = something you have + something you know)
- For admin accounts without federated IdP: TOTP-based MFA (RFC 6238) as a fallback

### 5.7 Session management

- **Internal session**: Short-lived JWT access tokens (15 min) + refresh tokens (8 hours, rotating)
- **Token storage**: Access token in memory, refresh token in httpOnly secure cookie
- **Session termination**: Revocation list in Redis for immediate logout. Back-channel logout support for OIDC.
- **Concurrent sessions**: Configurable per tenant (default: allow multiple devices)

---

## 6. Database Schema

### Entity-Relationship Overview

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    User       │────<│  Enrollment  │>────│    Course     │
│              │     │              │     │              │
│ id           │     │ id           │     │ id           │
│ external_id  │     │ user_id      │     │ title        │
│ email        │     │ course_id    │     │ description  │
│ display_name │     │ status       │     │ type         │
│ pid_hash     │     │ enrolled_at  │     │ format       │
│ user_type    │     │ completed_at │     │ duration_min │
│ locale       │     │ score        │     │ status       │
│ is_active    │     │ progress_pct │     │ is_mandatory │
│ created_at   │     │ due_date     │     │ created_by   │
│ updated_at   │     │ source       │     │ published_at │
└──────┬───────┘     └──────────────┘     └──────┬───────┘
       │                                         │
       │    ┌──────────────┐                     │
       │───<│  UserRole     │              ┌─────▼────────┐
       │    │              │              │ CourseContent │
       │    │ user_id      │              │              │
       │    │ role_id      │              │ course_id    │
       │    │ org_unit_id  │              │ content_type │
       │    │ granted_at   │              │ package_id   │
       │    └──────────────┘              │ scorm_version│
       │                                  │ entry_point  │
       │    ┌──────────────┐              │ storage_path │
       │───<│  Completion   │              └──────────────┘
       │    │              │
       │    │ id           │     ┌──────────────┐
       │    │ user_id      │     │  LearningPath │
       │    │ course_id    │     │              │
       │    │ completed_at │     │ id           │
       │    │ score        │     │ title        │
       │    │ certificate_id│    │ description  │
       │    │ valid_until  │     │ is_sequential│
       │    └──────────────┘     └──────┬───────┘
       │                                │
       │    ┌──────────────┐     ┌──────▼───────┐
       │───<│  Certificate  │     │ LP_Item      │
       │    │              │     │              │
       │    │ id           │     │ path_id      │
       │    │ user_id      │     │ course_id    │
       │    │ competency_id│     │ position     │
       │    │ issued_at    │     │ is_required  │
       │    │ valid_until  │     └──────────────┘
       │    │ pdf_url      │
       │    │ verification │
       │    │ revoked      │
       │    └──────────────┘
       │
       │    ┌──────────────┐     ┌──────────────┐
       │───<│UserCompetency│>────│  Competency   │
            │              │     │              │
            │ user_id      │     │ id           │
            │ competency_id│     │ name         │
            │ level        │     │ framework_id │
            │ acquired_at  │     │ parent_id    │
            │ expires_at   │     │ level_scale  │
            │ source       │     │ description  │
            └──────────────┘     └──────────────┘
```

### Core table definitions

```sql
-- All tables below exist per tenant schema

-- Users (provisioned via SCIM or self-registration)
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id     VARCHAR(255) UNIQUE,         -- IdP subject ID
    scim_id         VARCHAR(255) UNIQUE,         -- SCIM resource ID
    email           VARCHAR(255),
    display_name    VARCHAR(255) NOT NULL,
    given_name      VARCHAR(100),
    family_name     VARCHAR(100),
    pid_hash        VARCHAR(64),                 -- Hashed national ID (external users)
    user_type       VARCHAR(20) NOT NULL DEFAULT 'internal',
                    -- 'internal', 'external', 'consultant', 'admin'
    locale          VARCHAR(5) DEFAULT 'nb',
    org_unit_id     UUID REFERENCES org_units(id),
    manager_id      UUID REFERENCES users(id),
    is_active       BOOLEAN DEFAULT true,
    last_login_at   TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT now(),
    updated_at      TIMESTAMPTZ DEFAULT now()
);

-- Organizational structure
CREATE TABLE org_units (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    parent_id       UUID REFERENCES org_units(id),
    external_id     VARCHAR(255),                -- Maps to ServiceNow org structure
    level           INT NOT NULL,                -- 0=org, 1=division, 2=section, 3=team
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Roles and permissions
CREATE TABLE roles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(100) NOT NULL UNIQUE,
    description     TEXT,
    permissions     JSONB NOT NULL DEFAULT '[]'
);

CREATE TABLE user_roles (
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    role_id         UUID REFERENCES roles(id) ON DELETE CASCADE,
    org_unit_id     UUID REFERENCES org_units(id),  -- Scoped to org unit
    granted_at      TIMESTAMPTZ DEFAULT now(),
    granted_by      UUID REFERENCES users(id),
    PRIMARY KEY (user_id, role_id, org_unit_id)
);

-- Courses
CREATE TABLE courses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    short_desc      VARCHAR(255),
    type            VARCHAR(50) NOT NULL,
                    -- 'e-learning', 'classroom', 'webinar', 'blended',
                    -- 'assignment', 'external', 'video'
    format          VARCHAR(50),                 -- 'scorm_12', 'scorm_2004', 'xapi', 'cmi5', 'native'
    duration_min    INT,
    status          VARCHAR(20) DEFAULT 'draft',
                    -- 'draft', 'published', 'archived'
    is_mandatory    BOOLEAN DEFAULT false,
    category_id     UUID REFERENCES categories(id),
    competency_id   UUID REFERENCES competencies(id),
    created_by      UUID REFERENCES users(id),
    published_at    TIMESTAMPTZ,
    archived_at     TIMESTAMPTZ,
    metadata        JSONB DEFAULT '{}',          -- Flexible: tags, target audience, prerequisites
    search_vector   TSVECTOR,                    -- Norwegian full-text search
    created_at      TIMESTAMPTZ DEFAULT now(),
    updated_at      TIMESTAMPTZ DEFAULT now()
);

-- Index for full-text search
CREATE INDEX idx_courses_search ON courses USING GIN(search_vector);

-- Course content (SCORM packages, videos, etc.)
CREATE TABLE course_contents (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID REFERENCES courses(id) ON DELETE CASCADE,
    content_type    VARCHAR(50) NOT NULL,        -- 'scorm_package', 'video', 'document', 'quiz'
    scorm_version   VARCHAR(20),                 -- 'scorm_12', 'scorm_2004_4th', 'xapi', 'cmi5'
    entry_point     VARCHAR(500),                -- Relative path to launch file
    storage_path    VARCHAR(500) NOT NULL,       -- S3 path
    file_size_bytes BIGINT,
    manifest_data   JSONB,                       -- Parsed imsmanifest.xml / tincan.xml
    version         INT DEFAULT 1,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Enrollments
CREATE TABLE enrollments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    course_id       UUID REFERENCES courses(id) ON DELETE CASCADE,
    status          VARCHAR(20) DEFAULT 'enrolled',
                    -- 'enrolled', 'in_progress', 'completed', 'failed', 'expired', 'withdrawn'
    enrolled_at     TIMESTAMPTZ DEFAULT now(),
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    due_date        DATE,
    score           DECIMAL(5,2),                -- Percentage 0-100
    progress_pct    SMALLINT DEFAULT 0,          -- 0-100
    attempts        INT DEFAULT 0,
    source          VARCHAR(50) DEFAULT 'manual',
                    -- 'manual', 'auto_assign', 'manager', 'self', 'learning_path'
    assigned_by     UUID REFERENCES users(id),
    UNIQUE (user_id, course_id)
);

-- Completions (immutable audit log, separate from enrollment state)
CREATE TABLE completions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id),
    course_id       UUID REFERENCES courses(id),
    completed_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    score           DECIMAL(5,2),
    time_spent_sec  INT,
    attempt_number  INT,
    certificate_id  UUID REFERENCES certificates(id),
    valid_until     DATE,                        -- For certifications that expire
    metadata        JSONB DEFAULT '{}',          -- SCORM CMI data snapshot
    created_at      TIMESTAMPTZ DEFAULT now()
) PARTITION BY RANGE (completed_at);

-- Partition by year for performance
CREATE TABLE completions_2026 PARTITION OF completions
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
CREATE TABLE completions_2027 PARTITION OF completions
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');

-- SCORM runtime data (CMI model)
CREATE TABLE scorm_runtime_data (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    enrollment_id   UUID REFERENCES enrollments(id) ON DELETE CASCADE,
    cmi_data        JSONB NOT NULL,              -- Full CMI data model
    suspend_data    TEXT,                         -- SCORM suspend_data (can be large)
    bookmark        VARCHAR(500),                -- cmi.core.lesson_location
    total_time      VARCHAR(20),                 -- cmi.core.total_time (SCORM format)
    attempt_number  INT DEFAULT 1,
    last_updated    TIMESTAMPTZ DEFAULT now(),
    UNIQUE (enrollment_id, attempt_number)
);

-- Learning paths
CREATE TABLE learning_paths (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title           VARCHAR(500) NOT NULL,
    description     TEXT,
    is_sequential   BOOLEAN DEFAULT true,        -- Must complete in order?
    is_mandatory    BOOLEAN DEFAULT false,
    competency_id   UUID REFERENCES competencies(id),
    created_by      UUID REFERENCES users(id),
    status          VARCHAR(20) DEFAULT 'draft',
    created_at      TIMESTAMPTZ DEFAULT now(),
    updated_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE learning_path_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    path_id         UUID REFERENCES learning_paths(id) ON DELETE CASCADE,
    course_id       UUID REFERENCES courses(id),
    position        INT NOT NULL,
    is_required     BOOLEAN DEFAULT true,        -- Optional enrichment items
    UNIQUE (path_id, position)
);

CREATE TABLE learning_path_enrollments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    path_id         UUID REFERENCES learning_paths(id) ON DELETE CASCADE,
    status          VARCHAR(20) DEFAULT 'enrolled',
    enrolled_at     TIMESTAMPTZ DEFAULT now(),
    completed_at    TIMESTAMPTZ,
    UNIQUE (user_id, path_id)
);

-- Competency framework
CREATE TABLE competency_frameworks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    external_id     VARCHAR(255),                -- ServiceNow framework ID
    created_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE competencies (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    framework_id    UUID REFERENCES competency_frameworks(id),
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    parent_id       UUID REFERENCES competencies(id),
    level_scale     JSONB,                       -- e.g., [{"level": 1, "name": "Basic"}, ...]
    external_id     VARCHAR(255),                -- ServiceNow skill ID
    created_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE user_competencies (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    competency_id   UUID REFERENCES competencies(id),
    level           INT,
    acquired_at     DATE,
    expires_at      DATE,
    source          VARCHAR(50),                 -- 'course_completion', 'manual', 'servicenow', 'assessment'
    evidence_id     UUID,                        -- References completion or certificate
    created_at      TIMESTAMPTZ DEFAULT now(),
    UNIQUE (user_id, competency_id)
);

-- Certificates
CREATE TABLE certificates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id),
    competency_id   UUID REFERENCES competencies(id),
    course_id       UUID REFERENCES courses(id),
    template_id     UUID REFERENCES certificate_templates(id),
    certificate_no  VARCHAR(50) UNIQUE NOT NULL,  -- Human-readable number
    issued_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    valid_until     DATE,
    pdf_storage_path VARCHAR(500),
    verification_code VARCHAR(64) UNIQUE,         -- For public verification
    revoked         BOOLEAN DEFAULT false,
    revoked_at      TIMESTAMPTZ,
    revoked_reason  TEXT,
    archived_to     VARCHAR(255),                 -- Mime archive reference
    created_at      TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE certificate_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    html_template   TEXT NOT NULL,                -- Handlebars template for PDF generation
    is_default      BOOLEAN DEFAULT false,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Categories for course organization
CREATE TABLE categories (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    parent_id       UUID REFERENCES categories(id),
    sort_order      INT DEFAULT 0,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Classroom/event-specific tables
CREATE TABLE events (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id       UUID REFERENCES courses(id) ON DELETE CASCADE,
    title           VARCHAR(500),
    starts_at       TIMESTAMPTZ NOT NULL,
    ends_at         TIMESTAMPTZ NOT NULL,
    location        VARCHAR(500),
    location_type   VARCHAR(20),                 -- 'physical', 'virtual', 'hybrid'
    virtual_url     VARCHAR(500),                -- Teams/Zoom link
    max_participants INT,
    instructor_id   UUID REFERENCES users(id),
    status          VARCHAR(20) DEFAULT 'scheduled',
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Notifications
CREATE TABLE notifications (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    type            VARCHAR(50) NOT NULL,
    title           VARCHAR(255) NOT NULL,
    body            TEXT,
    link            VARCHAR(500),
    is_read         BOOLEAN DEFAULT false,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Audit log (append-only)
CREATE TABLE audit_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id),
    action          VARCHAR(100) NOT NULL,
    entity_type     VARCHAR(50) NOT NULL,
    entity_id       UUID,
    old_values      JSONB,
    new_values      JSONB,
    ip_address      INET,
    user_agent      TEXT,
    created_at      TIMESTAMPTZ DEFAULT now()
) PARTITION BY RANGE (created_at);

-- xAPI statement store (for LRS functionality)
CREATE TABLE xapi_statements (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    statement_id    UUID UNIQUE NOT NULL,         -- xAPI statement ID
    actor           JSONB NOT NULL,
    verb            JSONB NOT NULL,
    object          JSONB NOT NULL,
    result          JSONB,
    context         JSONB,
    timestamp       TIMESTAMPTZ NOT NULL,
    stored          TIMESTAMPTZ DEFAULT now(),
    authority       JSONB
) PARTITION BY RANGE (timestamp);
```

### Key indexes

```sql
-- Performance-critical queries
CREATE INDEX idx_enrollments_user_status ON enrollments(user_id, status);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
CREATE INDEX idx_completions_user ON completions(user_id, completed_at DESC);
CREATE INDEX idx_certificates_user ON certificates(user_id);
CREATE INDEX idx_certificates_verification ON certificates(verification_code);
CREATE INDEX idx_certificates_valid_until ON certificates(valid_until)
    WHERE revoked = false;
CREATE INDEX idx_user_competencies_user ON user_competencies(user_id);
CREATE INDEX idx_user_competencies_expires ON user_competencies(expires_at)
    WHERE expires_at IS NOT NULL;
CREATE INDEX idx_notifications_user_unread ON notifications(user_id)
    WHERE is_read = false;
CREATE INDEX idx_audit_log_entity ON audit_log(entity_type, entity_id);
CREATE INDEX idx_xapi_actor ON xapi_statements USING GIN(actor);
```

---

## 7. Infrastructure

### 7.1 Cloud Provider: AWS EU (eu-north-1, Stockholm)

**Decision**: AWS with primary region `eu-north-1` (Stockholm).

**Rationale**:
- `eu-north-1` is the closest AWS region to Norway, providing lowest latency
- EU/EEA data residency guaranteed (required by tender: all data processing in EU/EEA)
- AWS has the broadest managed service portfolio (RDS, ElastiCache, S3, EKS, CloudFront)
- Strong compliance posture: SOC 2, ISO 27001, C5, GDPR-compliant DPA
- S3 cross-region replication to `eu-west-1` (Ireland) for backup redundancy (tender requires 2 backup locations)

**Alternative considered**: Azure Norway East (Oslo) would provide Norwegian soil hosting, which some agencies prefer. We design the platform to be cloud-portable (no AWS-specific application code, only infrastructure-level AWS services) so we can offer Azure as an option for tenants that require it. The application layer uses S3-compatible APIs, standard PostgreSQL, and standard Redis -- all available on both clouds.

### 7.2 Container Orchestration: Kubernetes (EKS)

**Architecture**:

```
                        ┌─────────────────────┐
                        │    CloudFront CDN    │
                        │  (static assets,    │
                        │   SCORM packages)   │
                        └──────────┬──────────┘
                                   │
                        ┌──────────▼──────────┐
                        │    ALB (ingress)     │
                        └──────────┬──────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
           ┌────────▼────┐ ┌──────▼─────┐ ┌──────▼─────┐
           │  Frontend   │ │  Backend   │ │  Worker    │
           │  (Next.js)  │ │  (NestJS)  │ │  (BullMQ)  │
           │  2-4 pods   │ │  3-6 pods  │ │  2-4 pods  │
           └─────────────┘ └──────┬─────┘ └──────┬─────┘
                                  │              │
                    ┌─────────────┼──────────────┘
                    │             │
           ┌────────▼────┐ ┌─────▼──────┐
           │  RDS        │ │ ElastiCache│
           │  PostgreSQL │ │ Redis      │
           │  Multi-AZ   │ │ Cluster    │
           └─────────────┘ └────────────┘
                    │
           ┌────────▼────┐ ┌────────────┐
           │  S3         │ │ Meilisearch│
           │  (content)  │ │  (EKS pod) │
           └─────────────┘ └────────────┘
```

**Kubernetes specifics**:
- EKS with managed node groups (Graviton instances for cost efficiency)
- Horizontal Pod Autoscaler (HPA) on CPU/memory and custom metrics (request latency)
- Namespace-per-environment: `production`, `staging`, `development`
- Secrets managed via AWS Secrets Manager with External Secrets Operator
- Network policies for pod-to-pod isolation
- Resource quotas to prevent runaway pods

**Why Kubernetes over serverless**: The SCORM runtime requires WebSocket connections and stateful content serving. Lambda's cold starts and 15-minute timeout are poor fits. Kubernetes gives us predictable performance, WebSocket support, and the ability to run Meilisearch as a sidecar.

### 7.3 CI/CD Pipeline

```
Developer pushes code
         │
         ▼
┌────────────────────┐
│  GitHub Actions    │
│                    │
│  1. Lint + Type    │
│     check          │
│  2. Unit tests     │
│  3. Build Docker   │
│     images         │
│  4. Integration    │
│     tests (testdb) │
│  5. SCORM player   │
│     compatibility  │
│     tests          │
│  6. Security scan  │
│     (Snyk/Trivy)   │
│  7. Push to ECR    │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│  ArgoCD            │
│                    │
│  GitOps deployment │
│  - staging: auto   │
│  - production:     │
│    manual approval │
└────────────────────┘
```

**Key practices**:
- Trunk-based development with short-lived feature branches
- Preview environments per PR (ephemeral Kubernetes namespaces)
- Database migrations via a dedicated migration job (run before app deployment)
- Tenant schema migrations: a migration runner iterates over all tenant schemas
- Blue-green deployments for zero-downtime releases
- Canary releases for high-risk changes (route 5% of traffic first)

### 7.4 Monitoring and Observability

| Layer | Tool | Purpose |
|-------|------|---------|
| Metrics | Prometheus + Grafana | Request latency, error rates, pod resource usage, queue depths |
| Logs | Loki (or CloudWatch) | Structured JSON logs, tenant-tagged, 90-day retention |
| Traces | OpenTelemetry + Tempo | Distributed tracing across frontend -> backend -> database |
| Alerts | Grafana Alerting | PagerDuty/Slack integration for SLA violations |
| Uptime | External probe (e.g., Checkly) | Synthetic monitoring of critical user flows |
| Error tracking | Sentry | Frontend and backend error aggregation with source maps |

**SLA monitoring**: The tender requires specific response time and uptime SLAs. We instrument:
- P50, P95, P99 response times per endpoint
- Availability calculation (uptime minutes / total minutes, excluding planned maintenance)
- SCORM content load time (P95 < 3 seconds)
- Certificate generation time (P95 < 10 seconds)

**Tenant-scoped dashboards**: Each tenant admin gets a Grafana dashboard showing their usage metrics. Platform admins see cross-tenant operational dashboards.

---

## 8. Integration Framework

### Design principle: Integrations are tenant-scoped configuration, not custom code

Each integration is implemented as a NestJS module with a standard interface:

```typescript
interface Integration {
  id: string;
  type: IntegrationType;                 // 'servicenow' | 'powerbi' | 'eteori' | 'mime' | 'webhook'
  tenantId: string;
  config: Record<string, unknown>;       // Type-specific configuration
  enabled: boolean;
  healthStatus: 'healthy' | 'degraded' | 'error';
  lastSyncAt: Date | null;
}

interface IntegrationHandler {
  onCourseCompleted(event: CompletionEvent): Promise<void>;
  onCertificateIssued(event: CertificateEvent): Promise<void>;
  onUserProvisioned(event: UserEvent): Promise<void>;
  sync(): Promise<SyncResult>;           // Scheduled reconciliation
  healthCheck(): Promise<HealthStatus>;
}
```

### 8.1 ServiceNow integration

**Direction**: Bidirectional.
- **Inbound**: Competency framework sync (job families, job roles, skills) from ServiceNow to LMS
- **Outbound**: Learning completions, competency updates from LMS to ServiceNow

**Mechanism**: REST API calls, scheduled sync (every 15 min for completions, daily for framework), with webhook-triggered immediate sync for completions.

**Conflict resolution**: ServiceNow is the source of truth for competency frameworks. LMS is the source of truth for learning completions.

### 8.2 PowerBI integration

**Direction**: Outbound (LMS -> PowerBI).
- Structured data export to a staging area (S3 bucket or Azure Blob) in Parquet format
- Pre-built data model: enrollments, completions, competencies, certificates, user demographics (anonymized where needed)
- Scheduled export: nightly full refresh, hourly incremental
- Direct SQL access option: read replica with tenant-scoped views

### 8.3 Kursoppslag proxy API

**Direction**: Inbound (consumers -> LMS).
- LMS implements the existing Kursoppslag API contract as a compatibility layer
- Internal consumers continue using their existing integrations without changes
- Authenticated via Maskinporten

### 8.4 eTeori integration (option)

**Direction**: Bidirectional.
- **Outbound**: Exam registration from LMS
- **Inbound**: Exam results back to LMS for completion tracking
- REST API, event-driven

### 8.5 Mime archive integration (option)

**Direction**: Outbound.
- Competency certificates that are archival-mandatory (arkivplikt) are submitted to Mime
- Certificate PDFs + metadata in the format required by Mime's API
- Async via job queue (certificates are generated, then archived as a background task)

### 8.6 Custom webhook framework

For integrations not yet defined, tenants can configure webhooks:
- Events: `course.completed`, `certificate.issued`, `user.created`, `enrollment.created`, etc.
- Delivery: HTTP POST with HMAC signature verification
- Retry: Exponential backoff, 3 attempts, dead-letter queue for failed deliveries
- Monitoring: Webhook delivery dashboard in admin UI

### Integration configuration per tenant

```json
{
  "tenant": "svv",
  "integrations": [
    {
      "type": "servicenow",
      "enabled": true,
      "config": {
        "instance_url": "https://svv.service-now.com",
        "auth_type": "oauth2",
        "client_id": "...",
        "scopes": ["read_competency", "write_learning_log"],
        "sync_interval_min": 15,
        "competency_table": "x_custom_competency",
        "learning_log_table": "x_custom_learning_log"
      }
    },
    {
      "type": "powerbi",
      "enabled": true,
      "config": {
        "export_format": "parquet",
        "export_destination": "s3://svv-lms-exports/powerbi/",
        "schedule": "0 2 * * *",
        "include_pii": false
      }
    },
    {
      "type": "kursoppslag",
      "enabled": true,
      "config": {
        "maskinporten_scope": "svv:kursoppslag",
        "rate_limit_rpm": 500
      }
    }
  ]
}
```

---

## 9. Key Architectural Decisions (ADR Summary)

| ADR | Decision | Key trade-off |
|-----|----------|---------------|
| ADR-001 | Next.js + NestJS (full-stack TypeScript) | Giving up raw performance for developer productivity and hiring ease |
| ADR-002 | Schema-per-tenant multi-tenancy | Giving up simple cross-tenant queries for strong data isolation |
| ADR-003 | Open-source SCORM runtime over Rustici Engine | Giving up proven vendor solution for cost control and independence; Rustici as fallback |
| ADR-004 | REST-first API with GraphQL for reporting | Giving up single query language for simplicity in the common case |
| ADR-005 | AWS eu-north-1 primary, cloud-portable design | Giving up AWS-native optimizations for cloud portability |
| ADR-006 | Kubernetes over serverless | Giving up operational simplicity for WebSocket support and predictable performance |
| ADR-007 | BullMQ over Kafka/RabbitMQ | Giving up enterprise messaging guarantees for operational simplicity at current scale |
| ADR-008 | Meilisearch over Elasticsearch | Giving up powerful analytics for simpler operations and faster search UX |

---

## 10. Evolution Strategy

### Phase 1: MVP (Months 1-4)
- Core LMS: courses, enrollments, completions, basic SCORM 1.2 player
- ForgeRock SSO + SCIM provisioning
- Internal user portal
- Basic admin interface
- PostgreSQL + Redis + S3

### Phase 2: Full tender compliance (Months 4-6)
- SCORM 2004 + xAPI support
- ID-porten for external users
- Certificate generation and management
- Competency framework + gap analysis
- Learning paths
- ServiceNow + PowerBI integrations
- Kursoppslag API compatibility

### Phase 3: Differentiation (Months 6-9)
- AI-powered course recommendations
- Advanced analytics and reporting
- Content authoring tools
- Mobile-optimized experience
- Notification engine (email, push, in-app)

### Phase 4: Multi-tenant SaaS (Months 9-12)
- Tenant onboarding automation
- Self-service tenant admin portal
- Billing and usage metering
- Marketplace for shared content
- cmi5 support
- KKP portal migration (option L21)

### Architecture principles for evolution
1. **Modules over rewrites**: Each capability is a NestJS module that can be developed and deployed independently
2. **Feature flags**: New capabilities are deployed behind feature flags, enabled per tenant
3. **API versioning**: Breaking changes go in new API versions; old versions supported for 12 months
4. **Database migrations are additive**: New columns with defaults, new tables -- never drop columns in production without a deprecation cycle
5. **Integration adapters are pluggable**: New integrations follow the `IntegrationHandler` interface without modifying core code
