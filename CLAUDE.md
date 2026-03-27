# Veileder LMS - Claude Code Context

## What This Is

Norwegian public sector LMS SaaS product (codename: "Veileder"). First target customer is Statens vegvesen (SVV). Product play targeting entire Norwegian public sector (750K-830K workers, TAM 400-600M NOK/year).

**Current phase**: Planning complete, development not started.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.11+ / FastAPI |
| Frontend | Next.js 15 (App Router) / React 19 |
| UI Components | Designsystemet (Norwegian gov design system) + Radix UI |
| Database | PostgreSQL 16 + pgvector |
| Multi-tenancy | Schema-per-tenant (MVP), repository pattern abstracts isolation |
| Cache / Queue | Redis 7, Celery or ARQ |
| Search | Meilisearch (text) + pgvector (semantic) |
| Auth | Authlib (OIDC/SAML), ID-porten, ForgeRock, SCIM |
| SCORM Runtime | scorm-again (MIT, browser JS) |
| File Storage | AWS S3 (eu-north-1) |
| Infrastructure | AWS ECS Fargate, CloudFront, eu-north-1 (Stockholm) |
| CI/CD | GitHub Actions |

## Architecture Decisions (Locked)

- **ADR-001**: Python/FastAPI backend (not TypeScript) -- AI builds the code, ML in same codebase
- **ADR-002**: Next.js 15 frontend with Designsystemet
- **ADR-003**: PostgreSQL with pgvector, no MongoDB
- **ADR-004**: scorm-again for SCORM runtime (browser-side)
- **ADR-005**: Schema-per-tenant multi-tenancy, repository pattern for data access
- **ADR-006**: SCORM content served via CloudFront multi-origin (same domain), signed cookies for auth

## Data Access Rules

All database access goes through the repository pattern:

1. `TenantContext` middleware resolves tenant from subdomain/JWT
2. `get_tenant_session()` is the ONLY way to get a DB session
3. `BaseRepository` provides CRUD; domain repos extend it
4. Services contain business logic, have zero DB/tenant awareness
5. FastAPI routes wire via dependency injection
6. **Never** write raw SQL with schema names
7. **Never** access sessions directly outside `get_tenant_session()`

## Project Structure

```
/
├── build_plan/                    # Architecture & planning docs (14 files)
│   ├── BUILD_DELIVERY_PLAN.md     # Master build plan
│   ├── 01b_architecture_revised.md # Python/FastAPI rationale
│   ├── 05_epics_and_stories.md    # 12 epics, 66+ MVP stories
│   ├── 09_database_schema.md      # 50-table PostgreSQL DDL (3,156 lines)
│   ├── 10_data_access_layer.md    # Repository pattern design
│   ├── 11_scorm_content_architecture.md # SCORM serving & sandboxing
│   └── GAP_ANALYSIS.md            # 11 gaps vs SVV tender
├── delivery_plan/                 # SVV bid documents (Track A)
│   └── DELIVERY_PLAN_FINAL.md     # Approved Cornerstone Galaxy bid
├── CONTINUATION_PROMPT.md         # Full context for new sessions
├── TENDER_BRIEFING.md             # SVV tender summary
└── [SVV tender source documents]  # .docx, .xlsx, .pdf from SVV
```

## Key Documents to Read

When starting work, read in this order:

1. **This file** (CLAUDE.md) -- quick orientation
2. **build_plan/BUILD_DELIVERY_PLAN.md** -- master plan with full tech stack and epics
3. **build_plan/09_database_schema.md** -- 50-table schema (the source of truth for data model)
4. **build_plan/10_data_access_layer.md** -- repository pattern and tenant isolation rules
5. **build_plan/11_scorm_content_architecture.md** -- SCORM content serving (highest-risk area)

For specific epics/stories: `build_plan/05_epics_and_stories.md`
For UX/wireframes: `build_plan/02_ux_design.md`

## Epics (12 total)

| Epic | Name | Phase |
|------|------|-------|
| E1 | Platform Foundation (RBAC, org hierarchy, notifications, audit) | MVP |
| E2 | Authentication & Identity (OIDC, SCIM, ID-porten, SAML) | MVP |
| E3 | Content Engine (SCORM player, course management, quiz, catalog) | MVP |
| E4 | Learning Management (dashboard, enrollment, progress, paths, events) | MVP |
| E5 | Competency Framework (skills, gap analysis, certifications) | MVP |
| E6 | External Portal (self-registration, external catalog, certificates) | MVP |
| E7 | Integrations (Kursoppslag API, webhooks) | MVP (partial) |
| E8 | AI & Personalization (recommendations, semantic search) | MVP (basic) |
| E9 | Admin & Reporting (dashboards, reports, branding) | MVP |
| E10 | Compliance & Security (WCAG, encryption, GDPR) | MVP (partial) |
| E11 | Infrastructure & DevOps (cloud, CI/CD, monitoring) | MVP |
| E12 | Workplace Learning & CMS (70/20/10, learning areas, CMS) | MVP |

## Open Architecture Decisions (3 remaining)

1. **Content Authoring**: Build editor vs import-only (MVP is import-only)
2. **Offline/PWA**: Field workers need offline SCORM playback? Binary decision, retrofitting is hard
3. **Certificate Legal Validity**: eIDAS qualified signatures for arkivpliktige certificates?

## Norwegian Public Sector Context

- **Language**: All UI in Norwegian (Bokmaal). Code/comments in English.
- **Accessibility**: WCAG 2.1 AA mandatory (Norwegian law, uu-tilsynet enforces)
- **Data residency**: EU/EEA required (Schrems II). AWS eu-north-1 (Stockholm).
- **Auth**: ID-porten for citizen login (OIDC since Jan 2026), ForgeRock for enterprise SSO
- **Design system**: Designsystemet (digdir.no) is the official Norwegian gov component library
- **Terminology**: "Kurs" (course), "Kompetanse" (competency), "Sertifisering" (certification), "Laeringssti" (learning path)

## Conventions

- Backend: Python with type hints, async/await, Pydantic models
- Frontend: TypeScript strict mode, React Server Components where possible
- API: RESTful, versioned (`/api/v1/`), OpenAPI schema auto-generated
- Testing: pytest (backend), Vitest + Playwright (frontend)
- Database migrations: Alembic
- Commit messages: Conventional commits
