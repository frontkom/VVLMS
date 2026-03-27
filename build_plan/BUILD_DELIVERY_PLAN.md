# BUILD DELIVERY PLAN
## Norwegian Public Sector LMS SaaS Product
## Codename: "Veileder" (Guide)

**Date**: 2026-03-22
**Vision**: Purpose-built, Norwegian-first LMS SaaS for the public sector
**Strategy**: Option B -- Build a product, SVV is first target customer (not first delivery)

---

## 1. EXECUTIVE SUMMARY

### The Opportunity
- 750,000-830,000 Norwegian public sector workers need learning management
- No Norwegian-built modern LMS exists for state/health sectors
- No centralized state-level rammeavtale for LMS -- greenfield
- TAM: 400-600M NOK/year; realistic Year 5 ARR: 180M NOK
- Data sovereignty (post-Schrems II) is the killer differentiator

### The Strategy: Two-Track
- **Track A** (immediate): Bid SVV tender with Cornerstone Galaxy as implementation partner. Win revenue, references, and deep domain knowledge.
- **Track B** (18-month horizon): Build our own Norwegian-first LMS product. Target smaller public sector orgs first, then compete for larger tenders (2028+).

Track A funds and informs Track B. They are complementary, not competing.

### SVV Timeline Reality
Building a custom product for the SVV May 2026 demo is **not feasible** (7 weeks, zero references, "proven SaaS" requirement). The build targets the broader market with a realistic 12-18 month timeline.

### Investment Summary (Revised -- AI-built, Python backend)
| Phase | Duration | Team | Cost | Milestone |
|---|---|---|---|---|
| MVP (demo-ready) | 6-7 months | 3-4 FTE + AI | ~3.5-4.0 MNOK | Internal demo + first prospect meetings |
| Production v1 | 12 months | 6-7 FTE + AI | ~8-9 MNOK | First pilot customer live |
| Market ready | 18 months | 8 FTE + AI | ~12-14 MNOK | Tender-competitive with references |

---

## 2. TECH STACK (Revised -- AI-built)

**Design principle**: AI does the building, so tech choices optimize for product quality, performance, and architectural simplicity -- not hiring pool or developer productivity.

| Layer | Choice | Rationale |
|---|---|---|
| **Frontend** | Next.js 15 (App Router, React 19) | Best SSR + accessibility ecosystem. Designsystemet and Radix are React-native. |
| **UI Components** | Designsystemet (Norwegian gov) + Radix UI | Official Norwegian government design system = instant public sector credibility |
| **Backend** | **FastAPI (Python)** | AI-native: ML/recommendations/embeddings in same codebase (no separate AI service). AI generates excellent Python. Async performance sufficient for LMS scale. |
| **Database** | PostgreSQL 16 + pgvector | Multi-tenant (schema-per-tenant), vector embeddings for AI search built-in |
| **Cache** | Redis 7 | Sessions, job queues, real-time features |
| **Search** | Meilisearch | Fast, typo-tolerant, Norwegian language support |
| **File Storage** | S3-compatible (AWS S3) | SCORM packages, videos, certificates |
| **Job Queue** | **Celery or ARQ** (Redis-backed) | Python async task processing |
| **SCORM Runtime** | scorm-again (MIT, browser JS) | Full SCORM 1.2/2004 -- runs in browser regardless of backend language |
| **xAPI LRS** | SQL LRS or **built into main app** | PostgreSQL-based, can be integrated directly |
| **Auth** | **Authlib** (Python) | Mature OIDC/SAML/OAuth2 library |
| **SCIM Server** | **scim2-api** or custom FastAPI | SCIM 2.0 is simple REST -- straightforward in any language |
| **Quiz Engine** | SurveyJS (MIT, browser JS) | 20+ question types, JSON schema |
| **Certificates** | **ReportLab or WeasyPrint** (Python) | PDF/A generation with templates |
| **Video** | Video.js + xAPI profile (browser JS) | Tracking-enabled video player |
| **Notifications** | Novu (MIT) or **custom Python** | Multi-channel notifications |
| **Interactive Content** | H5P via Lumi (GPL-3.0, isolated) | 50+ content types, xAPI native |
| **AI Embeddings** | pgvector + Multilingual E5 | Semantic search in same database -- no separate vector DB |
| **AI LLM** | Azure OpenAI (Sweden) or Claude EU | Content generation, chatbot -- called from same Python codebase |
| **Infrastructure** | AWS eu-north-1 (Stockholm), **ECS Fargate** | EU data residency. Simpler than Kubernetes -- no EKS overhead |
| **CI/CD** | GitHub Actions | Simple deployment pipeline |

### Why Python over TypeScript (when AI builds)

1. **AI is the #1 differentiator** -- having ML capabilities in the same codebase means faster iteration on recommendation engine, semantic search, content generation, and competency analytics. No service boundary, no serialization, no deployment coordination.
2. **2 services instead of 3** -- Next.js frontend + FastAPI backend. No separate AI microservice.
3. **AI generates more correct Python** than any other language (largest training corpus).
4. **Simpler infrastructure** -- ECS Fargate instead of Kubernetes. FastAPI + uvicorn + Celery workers.
5. **SCORM is unaffected** -- `scorm-again` runs in the browser (JavaScript) regardless of backend. Backend only receives data model commits via REST endpoint.
6. **Performance is sufficient** -- FastAPI async handles 5,000-50,000 users per tenant comfortably. Upgrade path to Go/Rust exists if needed at 100K+ users.

---

## 3. ARCHITECTURE OVERVIEW

### 3.1 Multi-Tenancy (ADR-005)

**MVP**: Schema-per-tenant in PostgreSQL. Each government agency gets its own schema with complete data isolation. Satisfies Norwegian public sector audit requirements.

**Future**: When we scale beyond ~100 tenants (municipalities), add a shared-schema + RLS tier for small organizations. The application code will NOT need rewriting.

**How this is achieved**: All database access goes through a tenant-aware repository layer. No raw SQL, no hardcoded schema names, no direct session access outside repositories. The `TenantContext` middleware resolves tenant and sets the isolation mode. Repositories and services are completely unaware of which isolation strategy is active.

See `10_data_access_layer.md` for full implementation design.

### 3.2 System Architecture (Revised -- 2 services, not 3)
```
[Browser/Mobile]
       |
   [CDN/WAF (CloudFront)]
       |
   [Next.js Frontend] -- SSR + static assets (Vercel or S3+CloudFront)
       |
   [FastAPI Backend] -- REST + WebSocket + AI (single Python service)
       |
   +---+---+---+---+---+
   |   |   |   |   |   |
  [PG] [Redis] [S3] [Meilisearch] [Celery Workers]
   |                                (async jobs: notifications,
   +-- pgvector (AI embeddings)      integration sync, cert generation,
   +-- xAPI statements (built-in)    SCORM data processing)
```

**Key simplification**: No separate AI microservice. Recommendations, semantic search, LLM calls, and competency analytics all run in the FastAPI application. Celery workers handle async processing.

### 3.3 Integration Architecture
```
[ForgeRock] --SCIM/OIDC--> [Veileder API]
[ID-porten] --OIDC-------> [Veileder API]
[Maskinporten] --JWT------> [Veileder API]
[ServiceNow] <--REST------> [Veileder API]
[PowerBI] <---OData/REST--- [Veileder API]
[Kursoppslag] <--REST------ [Veileder API]
```

### 3.4 SCORM Player Architecture
```
[Content Manager uploads .zip]
       |
   [Backend: extract, validate imsmanifest.xml, store in S3]
       |
   [Learner launches course]
       |
   [Frontend: SCORM Player Component]
   - Instantiates scorm-again API on window
   - Renders content in sandboxed iframe
   - Captures cmi data model commits via API events
   - Sends completion/progress to backend
       |
   [Backend: persists to DB + emits xAPI to LRS]
```

---

## 4. EPIC BREAKDOWN (Revised with Gap Analysis)

### Summary: 12 Epics, ~104 User Stories

Gap analysis against SVV tender documents identified 11 missing features (see GAP_ANALYSIS.md). These are now incorporated.

| Epic | Name | Phase | Stories | Complexity | Gap Additions |
|---|---|---|---|---|---|
| E1 | Platform Foundation | MVP | 5 MVP + 3 P2 | XL | +1: Custom role configuration (GAP #8) |
| E2 | Authentication & Identity | MVP | 4 MVP + 3 P2 | XL | -- |
| E3 | Content Engine | MVP | 5 MVP + 3 P2 | XL | -- |
| E4 | Learning Management | MVP | 10 MVP + 4 P2 | XL | +2: Assignment submissions to MVP (GAP #10), event registration forms (GAP #2) |
| E5 | Competency Framework | MVP (partial) | 5 MVP + 4 P2 | L | +1: Per-group cert validity (GAP #3) |
| E6 | External Portal | MVP (partial) | 4 MVP + 3 P2 | L | -- |
| E7 | Integrations | Phase 2 | 2 MVP + 6 P2 | XL | -- |
| E8 | AI & Personalization | MVP (basic) | 3 MVP + 5 P2 | L | -- |
| E9 | Admin & Reporting | MVP | 4 MVP + 3 P2 | L | +2: Division admin scoped mgmt (GAP #4), granular report access (GAP #5), target group filter (GAP #6) |
| E10 | Compliance & Security | MVP (partial) | 4 MVP + 4 P2 | L | -- |
| E11 | Infrastructure & DevOps | MVP | 4 MVP + 2 P2 | M | -- |
| **E12** | **Workplace Learning & CMS** | **MVP** | **8 MVP + 2 P2** | **L** | **NEW: Workplace activities (GAP #11), learning areas (GAP #7), landing page CMS (GAP #9), external activity registration (GAP #1)** |
| | **TOTAL** | | **~66 MVP + 40 P2** | | |

### NEW EPIC 12: Workplace Learning & Content Management (from Gap Analysis)

This epic addresses the **biggest gap** -- SVV's 70/20/10 model requires tracking the 70% of learning that happens on the job. No competitor LMS does this well. This is our key differentiator.

**MVP Stories**:

**WL-01**: Register external/workplace activities
> As a learner or leader, I want to register a learning activity completed outside the platform (field inspection, mentoring, external course, emergency drill), so that all competency-relevant activities are documented.
- Activity type selector (configurable: befaring, hospitering, beredskapsovelse, simulatortrening, jobbobservasjon, egentrening, veiledning, praktisk prove)
- Registration form: type, date, description, evidence upload (photo/PDF), observer name
- Appears in learning history alongside formal courses
- Links to competency framework

**WL-02**: Approval workflow for workplace activities
> As a leader, I want to review and approve workplace activities registered by my team members.
- Approval queue: pending submissions with evidence
- Approve/reject with comment
- Sensor confirmation for practical exams (separate role)

**WL-03**: Checklists and assessment forms
> As a supervisor, I want to fill out a structured checklist when observing a colleague's work performance, so that job observation is documented consistently.
- Configurable checklist templates per activity type
- Fill out on mobile (field use)
- Score/pass/fail per checklist item
- Completed checklist attached to activity record

**WL-04**: Learning areas (omrader/fagsider)
> As an agency admin, I want to create learning areas with configurable read/edit access per user group, so that divisions and competency areas have their own managed spaces.
- Named area with description and branding
- Access control: read group, edit group
- Area contains: courses, documents, links, announcements
- Navigation: areas visible to authorized users

**WL-05**: Landing page CMS
> As an agency admin, I want to edit landing page content (hero image, text, featured courses) for internal and external users without developer help.
- WYSIWYG editor for page sections
- Upload/replace images
- Pin/feature courses on landing page
- Different content for internal vs external
- Delegatable editing permissions

**WL-06**: Event registration forms
> As a content manager, I want to add custom registration forms to classroom events (dietary needs, allergies, accommodation).
- Custom form builder (text, dropdown, checkbox fields)
- Registration data visible to event admin, exportable as CSV

**WL-07**: Re-authorization tracking
> As a system, I want to track re-authorization requirements with validity periods and automatic notifications for safety-critical functions.
- Gyldighetsperioder (validity periods) per competency
- Automatic expiry alerts
- Re-enrollment workflow

**WL-08**: Self-training registration
> As a learner, I want to register self-directed learning (external courses, reading, conferences) and optionally have my leader approve it.
- Self-registration form: activity name, provider, date, hours, description, evidence upload
- Optional leader approval workflow
- Appears in competency profile

---

## 5. DETAILED EPICS WITH USER STORIES AND TASKS

### EPIC 1: Platform Foundation
**Phase**: MVP | **Complexity**: XL | **Dependencies**: None

#### PF-01: Tenant Provisioning
> As a platform operator, I want to provision a new tenant with isolated data and branding, so that each government agency operates independently.

**Tasks**:
- [ ] Design tenant data model (tenant config table, schema creation script)
- [ ] Implement schema-per-tenant PostgreSQL middleware in NestJS
- [ ] Build tenant resolution from subdomain + JWT claims
- [ ] Create tenant seed script (demo data for SVV-like org)
- [ ] Implement tenant-scoped branding (logo, colors, name in config)
- [ ] Write tenant creation CLI tool

**Estimate**: 2 sprints (4 weeks)

#### PF-02: User Management & RBAC
> As an administrator, I want role-based access control, so that users see only what their role permits.

**Tasks**:
- [ ] Design user data model (users, roles, permissions, org_units)
- [ ] Implement RBAC middleware (role + permission guards on API)
- [ ] Define default roles: Learner, Leader, Content Manager, Division Admin, System Admin
- [ ] Build user CRUD API endpoints
- [ ] Build user management UI (list, search, edit, deactivate)
- [ ] Implement role assignment UI
- [ ] Create demo users for all 5 POC roles

**Estimate**: 2 sprints

#### PF-03: Organization Hierarchy
> As an administrator, I want to model my organizational structure, so that reporting and access follow the org chart.

**Tasks**:
- [ ] Design org_units table (tree structure with parent_id)
- [ ] Build org tree API (CRUD, tree traversal)
- [ ] Build org tree UI (collapsible tree view, drag-and-drop reorganization)
- [ ] Implement "my team" scope resolution for leaders
- [ ] CSV import for initial org structure setup

**Estimate**: 1 sprint

#### PF-04: Notification System
> As a user, I want notifications about assignments, deadlines, and completions.

**Tasks**:
- [ ] Integrate Novu for multi-channel notifications
- [ ] Define notification templates (Norwegian): assignment, deadline-7d, deadline-1d, completion, cert-expiry
- [ ] Build in-app notification center component (bell icon, dropdown, unread count)
- [ ] Implement email notification channel
- [ ] Build notification preferences UI (opt-out per category)
- [ ] Implement leader notifications for team cert expiry

**Estimate**: 1.5 sprints

#### PF-05: Audit Logging
> As a system admin, I want all user actions logged for compliance (LOG-04).

**Tasks**:
- [ ] Design audit_log table (timestamp, user_id, action, resource_type, resource_id, details JSONB)
- [ ] Implement audit logging middleware (auto-capture on API mutations)
- [ ] Build audit log viewer UI (filterable by user, action, date range)
- [ ] Log auth events (login, logout, failed attempts)

**Estimate**: 1 sprint

---

### EPIC 2: Authentication & Identity
**Phase**: MVP | **Complexity**: XL | **Dependencies**: E1

#### AU-01: OIDC SSO
> As an internal employee, I want to log in via SSO, so that I access the LMS with my existing credentials.

**Tasks**:
- [ ] Implement OIDC authorization code flow with PKCE (Passport.js strategy)
- [ ] Build IdP configuration UI (tenant-scoped: issuer URL, client ID, client secret)
- [ ] Implement token validation and session creation
- [ ] Build login/logout flows with proper redirect handling
- [ ] Test with Keycloak (dev) and ForgeRock (target)
- [ ] Implement session management (timeout, refresh)

**Estimate**: 1.5 sprints

#### AU-02: SCIM 2.0 Server
> As an IT admin, I want users provisioned automatically via SCIM, so that the LMS stays in sync with our identity system.

**Tasks**:
- [ ] Integrate SCIMMY library as Express middleware
- [ ] Map SCIM User attributes to internal user model (name, email, org_unit, role, status)
- [ ] Implement SCIM Group -> org_unit mapping
- [ ] Handle user lifecycle: create, update, deactivate (never hard delete)
- [ ] Implement bearer token auth for SCIM endpoint
- [ ] Build SCIM provisioning log/dashboard for admins
- [ ] Test with ForgeRock IDM SCIM client (or equivalent)

**Estimate**: 2 sprints

#### AU-03: ID-porten for External Users
> As an external contractor, I want to log in via ID-porten using my national eID.

**Tasks**:
- [ ] Implement ID-porten OIDC flow (openid-client library)
- [ ] Configure acr_values for security level (Level3 = BankID, Level4 = MinID)
- [ ] Auto-create external user profile on first login
- [ ] Map pid (person identifier) to internal user
- [ ] Separate external user portal routing
- [ ] Test with ID-porten test environment

**Estimate**: 1.5 sprints

#### AU-04: SAML 2.0 Support
> As a tenant, I want SAML SSO as a fallback federation option.

**Tasks**:
- [ ] Implement SP-initiated SAML flow (passport-saml)
- [ ] Build SAML metadata exchange UI
- [ ] Map SAML assertions to user attributes

**Estimate**: 1 sprint

---

### EPIC 3: Content Engine
**Phase**: MVP | **Complexity**: XL | **Dependencies**: E1

#### CE-01: SCORM Player
> As a content manager, I want to upload and play SCORM packages, so that existing e-learning courses work.

**Tasks**:
- [ ] Build SCORM package upload endpoint (accept .zip, validate imsmanifest.xml)
- [ ] Implement package extraction and S3 storage
- [ ] Build SCORM Player React component using scorm-again
- [ ] Implement SCORM 1.2 API adapter (LMSInitialize through LMSFinish)
- [ ] Implement SCORM 2004 RTE adapter (Initialize through Terminate)
- [ ] Build cmi data model persistence endpoint (commit handler)
- [ ] Track: completion_status, success_status, score, suspend_data, session_time
- [ ] Implement course resume (suspend_data restoration)
- [ ] Build player UI chrome (progress bar, module navigation, exit button)
- [ ] Test with 10+ real Articulate Storyline/Rise packages

**Estimate**: 3 sprints

#### CE-02: Course Management
> As a content manager, I want to create and manage courses with metadata.

**Tasks**:
- [ ] Design courses table (title, description, thumbnail, duration, language, difficulty, tags, status)
- [ ] Build course CRUD API
- [ ] Build course creation wizard UI (multi-step: details > content > metadata > publish)
- [ ] Implement course status workflow (Draft > Published > Archived)
- [ ] Build course versioning (update content without losing history)
- [ ] Implement rich text editor for descriptions (Tiptap)
- [ ] Thumbnail upload with auto-resize

**Estimate**: 2 sprints

#### CE-03: Quiz Engine
> As a content manager, I want to build quizzes within the platform.

**Tasks**:
- [ ] Integrate SurveyJS for quiz rendering
- [ ] Build quiz creation UI (question types: MCQ, multi-select, true/false, fill-blank, ordering)
- [ ] Implement question bank (tagged, reusable)
- [ ] Build quiz settings (pass score, attempts, time limit, randomization)
- [ ] Implement quiz results tracking and display
- [ ] Build quiz review mode (show correct answers with explanations)

**Estimate**: 2 sprints

#### CE-04: Course Catalog
> As a learner, I want to browse courses by category and filter.

**Tasks**:
- [ ] Build category management (hierarchical: e.g., HMS > Arbeidsvarsling > Grunnkurs)
- [ ] Build catalog UI (grid/list toggle, filters, search)
- [ ] Implement Meilisearch indexing for courses
- [ ] Build course detail page (description, metadata, enroll button, reviews)

**Estimate**: 1.5 sprints

#### CE-05: Course Lifecycle
> As a content manager, I want to update and archive courses while preserving history.

**Tasks**:
- [ ] Implement version management (new version replaces active, old versions archived)
- [ ] Handle in-progress enrollments on version change
- [ ] Build archive flow with data retention
- [ ] Implement "require re-completion" flag for major updates

**Estimate**: 1 sprint

---

### EPIC 4: Learning Management
**Phase**: MVP | **Complexity**: XL | **Dependencies**: E1, E3

#### LM-01: Learner Dashboard
> As a learner, I want a personalized landing page showing my courses and deadlines.

**Tasks**:
- [ ] Build dashboard layout (responsive: desktop + tablet + mobile)
- [ ] "My courses" widget (in-progress, with progress bars)
- [ ] "Assigned to me" widget (mandatory courses with deadlines)
- [ ] "Recently completed" widget
- [ ] "Upcoming deadlines" widget (sorted by urgency)
- [ ] Quick-launch buttons (resume course)
- [ ] Recommendation widget stub (content-based for MVP)

**Estimate**: 1.5 sprints

#### LM-02: Search & Discovery
> As a learner, I want to search and filter the catalog.

**Tasks**:
- [ ] Implement full-text search via Meilisearch
- [ ] Build search UI with filters (category, format, duration, mandatory/optional)
- [ ] Implement Norwegian typo tolerance
- [ ] Add semantic search via pgvector embeddings (E8 dependency)
- [ ] Sort options: relevance, newest, popularity

**Estimate**: 1 sprint

#### LM-03: Enrollment
> As a learner, I want to enroll in courses (self or assigned).

**Tasks**:
- [ ] Build enrollment model (user_id, course_id, enrolled_at, deadline, source: self/admin/leader)
- [ ] Self-enrollment (one-click for optional courses)
- [ ] Admin/leader assignment (individual + bulk by org unit)
- [ ] Enrollment notifications
- [ ] Enrollment history visible to learner + leader

**Estimate**: 1 sprint

#### LM-04: Progress Tracking
> As a learner, I want my progress tracked automatically.

**Tasks**:
- [ ] SCORM suspend_data enables resume
- [ ] Progress percentage display (dashboard + course player)
- [ ] Time spent tracking (per session + total)
- [ ] Completion event handling (SCORM status -> DB -> notification)
- [ ] Certificate generation on completion (if configured)

**Estimate**: 1 sprint

#### LM-05: Leader Dashboard
> As a leader, I want to see my team's learning status at a glance.

**Tasks**:
- [ ] Build leader dashboard (separate view from learner)
- [ ] Team overview: enrolled, in-progress, completed, overdue per member
- [ ] Traffic light compliance indicators (green/yellow/red)
- [ ] Certification expiry timeline (next 30/60/90 days)
- [ ] Drill-down to individual learner profile
- [ ] Filter by course, time period, status
- [ ] Export to CSV

**Estimate**: 2 sprints

#### LM-06: Learning Paths
> As a content manager, I want to create ordered sequences of courses.

**Tasks**:
- [ ] Design learning_paths and learning_path_items tables
- [ ] Build learning path creation UI (drag-and-drop ordering)
- [ ] Implement prerequisite enforcement (must complete A before B)
- [ ] Build learner view: visual progress timeline
- [ ] Path-level progress tracking (X of Y completed)
- [ ] Assign learning paths like courses

**Estimate**: 2 sprints

#### LM-07: Course Assignment
> As a leader, I want to assign courses to my team.

**Tasks**:
- [ ] Build assignment UI (select course/path > select employees > set deadline > confirm)
- [ ] Bulk assignment by org unit
- [ ] Assignment notification with personal message
- [ ] Assignment tracking dashboard

**Estimate**: 1 sprint

#### LM-08: Classroom Events
> As a content manager, I want to manage physical and virtual events.

**Tasks**:
- [ ] Build event model (title, instructor, location/link, datetime, capacity)
- [ ] Registration with capacity limits + waitlist
- [ ] Attendance marking UI (instructor/admin)
- [ ] Calendar view (upcoming events)
- [ ] Teams/Zoom link integration for virtual events

**Estimate**: 1.5 sprints

#### LM-09: Activity Approval
> As a leader, I want to approve workplace-based learning completions.

**Tasks**:
- [ ] Build approval workflow (learner submits > leader reviews > approve/reject)
- [ ] Approval queue UI for leaders
- [ ] Comment on approval/rejection
- [ ] Notification chain (submission, approved, rejected)

**Estimate**: 1 sprint

#### LM-10: Mandatory vs Optional Configuration
> As a content manager, I want to configure courses as mandatory per org unit.

**Tasks**:
- [ ] Mandatory flag per course
- [ ] Scope mandatory to specific org units
- [ ] Mandatory courses highlighted on learner dashboard
- [ ] Overdue escalation notifications

**Estimate**: 0.5 sprint

---

### EPIC 5: Competency Framework
**Phase**: MVP (basic) + Phase 2 (full) | **Complexity**: L | **Dependencies**: E1, E4

#### CF-01: Competency Definitions
> As an admin, I want to define competencies and map them to roles.

**Tasks**:
- [ ] Design competency model (competencies, levels, role_competency_requirements)
- [ ] Build competency CRUD API + UI
- [ ] Role-to-competency mapping UI

**Estimate**: 1 sprint

#### CF-02: Individual Competency Profile
> As a learner, I want to see my competency profile with gaps highlighted.

**Tasks**:
- [ ] Build competency profile page (current vs required for role)
- [ ] Gap visualization (what's missing, suggested courses to close gaps)
- [ ] Competency history (how skills developed over time)

**Estimate**: 1 sprint

#### CF-03: Certification Management
> As a system, I want to issue, track, and manage certifications with expiry dates.

**Tasks**:
- [ ] Design certifications table (user, competency, issued_at, valid_until, status)
- [ ] Auto-issue certificate on learning path completion
- [ ] Expiry tracking with notifications (90/60/30 days)
- [ ] Re-enrollment workflow for expired certifications
- [ ] Certificate PDF generation (pdfme with templates)
- [ ] QR code for certificate verification

**Estimate**: 2 sprints

#### CF-04: Competency Dashboard for Leaders
> As a leader, I want to see my team's competency status.

**Tasks**:
- [ ] Team competency matrix (people x competencies)
- [ ] Traffic light status per person per competency
- [ ] Expiry alerts aggregated by team
- [ ] Export capability

**Estimate**: 1 sprint

#### CF-05: Competency Certificates (Archival)
> As a system, I want certificates to comply with arkivloven.

**Tasks**:
- [ ] PDF/A generation with digital signature
- [ ] Noark-5 compatible metadata export (XML + PDF)
- [ ] Batch export capability for Mime import
- [ ] Certificate validation URL (public endpoint with QR code)

**Estimate**: 1 sprint

---

### EPIC 6: External Portal
**Phase**: MVP (basic) + Phase 2 (full) | **Complexity**: L | **Dependencies**: E1, E2, E3

#### EP-01: External User Registration
> As an external user, I want to register via ID-porten.

**Tasks**:
- [ ] External landing page (separate subdomain/path)
- [ ] ID-porten login flow (AU-03 dependency)
- [ ] 3-step onboarding wizard (confirm details, select employer, view courses)
- [ ] Pre-fill from Folkeregisteret where available

**Estimate**: 1 sprint

#### EP-02: External Course Catalog
> As an external user, I want to see courses available to me.

**Tasks**:
- [ ] Filtered catalog (only externally-available courses)
- [ ] Enrollment for external users
- [ ] External user dashboard (my courses, certificates)

**Estimate**: 1 sprint

#### EP-03: External Certificates
> As an external user, I want to download my competency certificates.

**Tasks**:
- [ ] Certificate wallet view (all issued certificates)
- [ ] PDF download
- [ ] Certificate sharing (URL with verification)
- [ ] Employer notification on completion

**Estimate**: 0.5 sprint

#### EP-04: Branded Sub-Portal
> As a tenant admin, I want a branded external portal.

**Tasks**:
- [ ] Custom subdomain per tenant
- [ ] Tenant-specific branding on external portal
- [ ] Separate content configuration for external vs internal

**Estimate**: 1 sprint

---

### EPIC 7: Integrations
**Phase**: Phase 2 (primarily) | **Complexity**: XL | **Dependencies**: E1, E2

#### IN-01: Integration Framework
> As a platform, I want a pluggable integration framework.

**Tasks**:
- [ ] Design IntegrationHandler interface (init, sync, push, pull, health)
- [ ] Tenant-scoped integration configuration (API keys, endpoints, mapping)
- [ ] Integration health dashboard
- [ ] Webhook outbound framework (configurable events)

**Estimate**: 1.5 sprints

#### IN-02: Kursoppslag-Compatible API
> As SVV's internal systems, I want to consume the same API contract.

**Tasks**:
- [ ] Implement all 6 Kursoppslag endpoints with exact field compatibility
- [ ] Delta query support (endretFor/endretEtter parameters)
- [ ] Maskinporten authentication
- [ ] API documentation (OpenAPI spec)

**Estimate**: 1.5 sprints

#### IN-03: ServiceNow Connector (Phase 2)
> As SVV, I want learning data synced to my ServiceNow competency register.

**Tasks**:
- [ ] Inbound: Pull job families, job roles, skills from ServiceNow REST API
- [ ] Outbound: Push learning completions to ServiceNow learning log
- [ ] Bidirectional mapping configuration UI
- [ ] Sync scheduling (event-driven + nightly reconciliation)
- [ ] Sopra Steria coordination interface

**Estimate**: 3 sprints

#### IN-04: PowerBI Data Export (Phase 2)
> As an analyst, I want structured data available for PowerBI.

**Tasks**:
- [ ] OData-compatible API endpoints for reporting data
- [ ] Data model: users, enrollments, completions, certifications, skills
- [ ] Scheduled data export (CSV/Parquet to S3 for PowerBI pickup)
- [ ] PowerBI template dashboard

**Estimate**: 2 sprints

#### IN-05 to IN-08: eTeori, Mime, LTI, Webhooks (Phase 2-3)

**Estimate**: 4 sprints combined

---

### EPIC 8: AI & Personalization
**Phase**: MVP (basic) + Phase 2 (advanced) | **Complexity**: L | **Dependencies**: E3, E4

#### AI-01: Content Recommendations (MVP)
> As a learner, I want course recommendations based on my role and activity.

**Tasks**:
- [ ] Rule-based recommendations (role-based, competency-gap-based, popularity)
- [ ] "Anbefalt for deg" widget on dashboard with Norwegian explanations
- [ ] Recommendation API endpoint
- [ ] A/B testing framework for recommendation quality

**Estimate**: 1 sprint

#### AI-02: Semantic Search (MVP)
> As a learner, I want Norwegian-language intelligent search.

**Tasks**:
- [ ] Generate embeddings for all courses using Multilingual E5 model
- [ ] Store in pgvector
- [ ] Hybrid search: BM25 (Meilisearch) + vector similarity (pgvector)
- [ ] Norwegian-specific: handle Bokmal/Nynorsk, abbreviations (SVV, HMS, KKP)
- [ ] Search result ranking considering user role/division

**Estimate**: 1.5 sprints

#### AI-03: AI Content Assistant (MVP basic)
> As a content manager, I want AI help generating quiz questions and summaries.

**Tasks**:
- [ ] AI Gateway (PII filtering via Presidio, audit logging, cost controls)
- [ ] Quiz question generation from course content (LLM API call)
- [ ] Course summary generation
- [ ] Bokmal/Nynorsk translation assistance
- [ ] "AI-assistert" watermark on generated content
- [ ] Human review required before publish

**Estimate**: 1.5 sprints

#### AI-04 to AI-08: Chatbot, predictive analytics, advanced recommendations (Phase 2-3)

**Estimate**: 6 sprints combined

---

### EPIC 9: Admin & Reporting
**Phase**: MVP | **Complexity**: L | **Dependencies**: E1, E4

#### AR-01: Admin Dashboard
> As an admin, I want KPI overview and system health.

**Tasks**:
- [ ] KPI cards: active users, courses, completions this month, compliance rate
- [ ] Trend charts (30/90/365 day views)
- [ ] Recent activity feed
- [ ] System health indicators

**Estimate**: 1 sprint

#### AR-02: Reports
> As a division admin, I want standard reports.

**Tasks**:
- [ ] Pre-built reports: completion rates, compliance status, training hours, cert expiry
- [ ] Filter by: period, division, role, course
- [ ] Export to CSV/Excel
- [ ] Scheduled report delivery (email)

**Estimate**: 1.5 sprints

#### AR-03: Tenant Configuration
> As an admin, I want to configure branding, settings, and features.

**Tasks**:
- [ ] Branding settings (logo, colors, favicon, portal name)
- [ ] Feature toggles (enable/disable AI, external portal, etc.)
- [ ] Email template customization
- [ ] Integration configuration panel

**Estimate**: 1 sprint

#### AR-04: User Import/Export
> As an admin, I want to bulk import/export users.

**Tasks**:
- [ ] CSV import with validation and error handling
- [ ] CSV/Excel export of user data
- [ ] GDPR data export (all data for a specific user)
- [ ] User deletion with data anonymization

**Estimate**: 1 sprint

---

### EPIC 10: Compliance & Security
**Phase**: MVP (partial) + Phase 2 | **Complexity**: L

#### CS-01: GDPR Tools
- [ ] Data export per user (right of access)
- [ ] Data deletion with anonymization (right to erasure)
- [ ] Consent management for optional features (AI, analytics)
- [ ] Data processing log

**Estimate**: 1.5 sprints

#### CS-02: WCAG 2.1 AA
- [ ] axe-core in CI pipeline (automated accessibility testing)
- [ ] Keyboard navigation for all interactions
- [ ] Screen reader testing (NVDA/VoiceOver) per sprint
- [ ] Color contrast compliance (Designsystemet handles most)
- [ ] Focus indicators, ARIA labels, alt text

**Estimate**: Ongoing (built-in, not bolt-on -- +30% per UI task)

#### CS-03: Security Hardening
- [ ] HTTPS everywhere (TLS 1.3)
- [ ] CSRF/XSS protection (NestJS helmet, CSP headers)
- [ ] SQL injection prevention (parameterized queries, ORM)
- [ ] Rate limiting on API endpoints
- [ ] DDoS protection (AWS Shield)
- [ ] Secrets management (AWS Secrets Manager)

**Estimate**: 1 sprint + ongoing

#### CS-04: Archiving (Noark-5)
- [ ] PDF/A certificate generation
- [ ] XML metadata export per Noark-5 standard
- [ ] Batch export tool for manual Mime import
- [ ] Certificate verification endpoint

**Estimate**: 1 sprint

---

### EPIC 11: Infrastructure & DevOps
**Phase**: MVP | **Complexity**: M

#### ID-01: Cloud Infrastructure
- [ ] AWS eu-north-1 Terraform/CDK setup
- [ ] EKS cluster with auto-scaling
- [ ] RDS PostgreSQL (multi-AZ)
- [ ] ElastiCache Redis
- [ ] S3 buckets (SCORM packages, certificates, uploads)
- [ ] CloudFront CDN

**Estimate**: 1.5 sprints

#### ID-02: CI/CD Pipeline
- [ ] GitHub Actions: lint, test, build, security scan
- [ ] ArgoCD for GitOps deployment
- [ ] Staging environment (auto-deploy on merge to main)
- [ ] Production environment (manual approval deploy)
- [ ] Database migration automation (Prisma or Drizzle)

**Estimate**: 1 sprint

#### ID-03: Monitoring & Observability
- [ ] Application metrics (Prometheus + Grafana)
- [ ] Log aggregation (CloudWatch or Loki)
- [ ] Error tracking (Sentry)
- [ ] Uptime monitoring (synthetic checks)
- [ ] Alerting (PagerDuty or similar)

**Estimate**: 1 sprint

#### ID-04: Backup & DR
- [ ] Daily automated database backups
- [ ] Cross-region backup replication (eu-west-1 as secondary)
- [ ] Backup restoration testing (quarterly)
- [ ] Disaster recovery playbook

**Estimate**: 0.5 sprint

---

## 6. SPRINT PLAN (MVP -- 17 Sprints / 34 Weeks, revised with gaps + Python stack)

| Sprint | Weeks | Focus | Key Deliverables |
|---|---|---|---|
| **S1** | W1-W2 | Foundation | Tenant model, DB schema (SQLAlchemy), Next.js scaffold, FastAPI scaffold, CI/CD, ECS Fargate setup |
| **S2** | W3-W4 | Foundation + Auth | RBAC with **custom role configuration (GAP #8)**, user model, OIDC integration (Authlib), org hierarchy |
| **S3** | W5-W6 | Auth + Content | SCIM server (Python), ID-porten OIDC, SCORM upload + extraction to S3 |
| **S4** | W7-W8 | Content Engine | SCORM Player v1 (scorm-again in browser + FastAPI commit endpoint), course CRUD, metadata |
| **S5** | W9-W10 | Content + Learning | Quiz engine (SurveyJS), course catalog, enrollment, **event registration forms (GAP #2)** |
| **S6** | W11-W12 | Learning Core | Learner dashboard, progress tracking, Meilisearch integration |
| **S7** | W13-W14 | Learning Paths + Leader | Learning path builder, leader dashboard (traffic light), team view |
| **S8** | W15-W16 | Leader + Competency | Assignment, approval workflow, competency model, **per-group cert validity (GAP #3)** |
| **S9** | W17-W18 | Certifications + External | Cert management + PDF generation, external portal (ID-porten, catalog, certs) |
| **S10** | W19-W20 | **Workplace Learning (GAP #11)** | Activity types, external activity registration (GAP #1), approval workflow, checklists, re-authorization |
| **S11** | W21-W22 | **CMS + Admin Areas (GAP #7, #9)** | Learning areas with access control, landing page CMS editor, **assignment submissions (GAP #10)** |
| **S12** | W23-W24 | Admin + Reporting | Admin dashboard, reports, **division admin scoped management (GAP #4)**, **granular report access (GAP #5)**, **target group filtering (GAP #6)** |
| **S13** | W25-W26 | AI | Recommendations (in-process Python), semantic search (pgvector), AI content assist (LLM API) |
| **S14** | W27-W28 | Compliance + Security | GDPR tools, WCAG audit, security hardening, DDoS, Noark-5 export |
| **S15** | W29-W30 | Polish + Events | Classroom events, notifications polish, mobile optimization, self-training registration (GAP #1) |
| **S16** | W31-W32 | Integration + Testing | Kursoppslag API compatibility, webhook framework, E2E testing, performance testing |
| **S17** | W33-W34 | Demo Prep | Demo data, bug fixing, demo rehearsals, documentation |

**MVP complete at Week 34 (November/December 2026)** with demo-ready product covering ALL SVV POC scenarios (1.1 through 6.3) and workplace-based activities.

---

## 7. TEAM AND COST (Revised -- AI-built, Python backend)

### 7.1 Team Ramp

AI does the heavy lifting on code generation. Humans focus on architecture decisions, domain modeling, UX design, security review, and QA. Smaller team, higher leverage.

| Month | Team Size | Monthly Burn | Roles |
|---|---|---|---|
| 1-4 | 3-4 FTE | ~500-600K NOK | Tech Lead/Architect, 1 Full-stack (Python+React), 0.5 PM, 0.5 Designer |
| 5-8 | 5-6 FTE | ~700-800K NOK | + 1 Full-stack, + 0.5 QA/DevOps |
| 9-12 | 6-7 FTE | ~850-950K NOK | + Security review, + Migration specialist |
| 13-18 | 8 FTE | ~1.0M NOK | + Customer success, + Sales/presales |

### 7.2 Investment to Key Milestones

| Milestone | Months | Cumulative Cost |
|---|---|---|
| MVP (demo-ready) | 8 | ~3.5-4.0 MNOK |
| Pilot-ready | 10 | ~5.5-6.0 MNOK |
| Production v1 | 12 | ~8-9 MNOK |
| Tender-competitive (with reference) | 18 | ~12-14 MNOK |

### 7.3 Revenue Path

| Year | Customers | Users | ARR |
|---|---|---|---|
| Year 1 | 3-5 pilot agencies | ~5,000 | ~3.6M NOK |
| Year 2 | 15-20 agencies | ~30,000 | ~22M NOK |
| Year 3 | 40+ agencies + health | ~80,000 | ~58M NOK |
| Year 5 | 120+ entities | ~250,000 | ~180M NOK |

---

## 8. RISK SUMMARY (Revised)

| Risk | Level | Mitigation |
|---|---|---|
| SCORM compliance complexity | HIGH | Use scorm-again (proven), test with real Articulate packages early |
| Timeline to market (now 34 weeks MVP) | HIGH | AI-accelerated development, ruthless prioritization, demo-first |
| Zero references vs established vendors | CRITICAL | Start with smaller agencies, build references before large tenders |
| Investment (12-14 MNOK to market) | HIGH | Hybrid strategy: Track A revenue funds Track B. Lower than original estimate due to AI building + Python simplification |
| ISO 27001 takes 9-18 months | MEDIUM | Start certification process at Month 3 |
| Norwegian IAM complexity | MEDIUM | Tech Lead must have ForgeRock/ID-porten experience |
| Multi-tenant data isolation | MEDIUM | Schema-per-tenant is proven pattern |
| **Workplace activities feature scope (GAP #11)** | **MEDIUM** | **This is the differentiator. Invest in it. No competitor does this well.** |
| Python ecosystem gaps (SCIM, some LMS-specific libraries) | LOW | SCIM is simple REST. AI generates adapters quickly. Browser-side JS libraries (scorm-again, SurveyJS) are unaffected by backend language. |

### Go/No-Go Verdict

**CONDITIONAL GO** for the product build, with conditions:
1. Secure 12-14 MNOK funding (or self-fund from Track A revenue) -- lower than original estimate
2. Confirm Tech Lead with Norwegian public sector IAM experience (ForgeRock, ID-porten)
3. Validate with 5+ potential customers before full commitment
4. Start with smaller agencies (500-3,000 users) for first references
5. Workplace-based learning (70/20/10 model) must be in MVP -- it's the key differentiator

---

## 9. RECOMMENDED NEXT STEPS

1. **This week**: Confirm Two-Track strategy (Track A: SVV with Cornerstone, Track B: build product)
2. **April 2026**: Confirm Tech Lead with Norwegian IAM experience (ForgeRock, ID-porten)
3. **April 2026**: Validate product concept with 5+ Norwegian government agencies
4. **May 2026**: Start Sprint 1 with 3-4 person team + AI building
5. **Nov/Dec 2026**: MVP demo to first prospects (34 weeks, all POC scenarios + workplace learning)
6. **Q1 2027**: First pilot customers (smaller agencies, 500-3000 users)
7. **Q3 2027**: Production v1, first tender bid with own product

---

## 10. REFERENCE DOCUMENTS

| Document | Content |
|---|---|
| `01_architecture.md` | Original TypeScript architecture (superseded) |
| `01b_architecture_revised.md` | **Current**: Python/FastAPI architecture with AI-built rationale |
| `02_ux_design.md` | Product UX design, user flows, wireframes, design system |
| `03_ai_architecture.md` | AI/ML architecture, build-vs-buy, Norwegian NLP |
| `04_reusable_components.md` | Open-source building blocks (SCORM, SCIM, ID-porten, etc.) |
| `05_epics_and_stories.md` | Original 11 epics with 92 stories (before gap analysis) |
| `06_market_opportunity.md` | Norwegian public sector LMS market sizing |
| `07_project_plan.md` | Original team/timeline plan (revised in this doc) |
| `08_risk_feasibility.md` | Build feasibility and risk assessment |
| `09_database_schema.md` | Complete PostgreSQL schema: 50 tables, partitioning, indexes, seed data (3,156 lines) |
| `10_data_access_layer.md` | **ADR-005**: Repository pattern for tenant-agnostic data access (schema now, RLS later) |
| `GAP_ANALYSIS.md` | 11 gaps found cross-referencing against SVV tender docs |

---

*This plan synthesizes inputs from: System Architect, UX Designer, AI Specialist, LMS Expert (open-source research), Product Strategist, Market Researcher, Project Manager, and Risk Assessor. Revised with architecture update (Python/FastAPI) and gap analysis (11 missing features from SVV requirements).*
