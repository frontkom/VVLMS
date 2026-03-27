# Continuation Prompt: Norwegian Public Sector LMS Build

Use this prompt to resume work in a fresh context window.

---

## Project Overview

We are building a **Norwegian public sector LMS SaaS product** (codename: "Veileder") from scratch. The first target customer is Statens vegvesen (Norwegian Public Roads Administration, 5,300 employees + 15,000 external certificate holders), but this is a PRODUCT play targeting the entire Norwegian public sector (750,000-830,000 workers, TAM 400-600M NOK/year).

**Two-Track Strategy**:
- **Track A**: Bid the SVV tender (case 25/223727) with Cornerstone Galaxy as implementation partner (immediate revenue + references). Delivery plan completed and approved.
- **Track B**: Build our own Norwegian-first LMS product in parallel (18-month horizon). This is what we're working on.

## Key Decisions Made

### Architecture (ADR-001 through ADR-005)

| Decision | Choice | Rationale |
|---|---|---|
| **Backend** | Python / FastAPI | AI-native (ML in same codebase), AI builds the code so hiring pool irrelevant, 2 services instead of 3 |
| **Frontend** | Next.js 15 / React 19 | Best SSR + accessibility ecosystem, Designsystemet (Norwegian gov design system) is React-native |
| **UI Components** | Designsystemet + Radix UI | Official Norwegian public sector design system = instant credibility |
| **Database** | PostgreSQL 16 + pgvector | Schema-per-tenant multi-tenancy, vector search for AI |
| **Multi-tenancy** | Schema-per-tenant (MVP), RLS-ready for later (ADR-005) | Norwegian public sector requires strong data isolation. Repository pattern abstracts isolation strategy. |
| **Data Access** | Repository pattern, tenant-agnostic | All DB access through repositories. Session factory handles tenant isolation. Services/routes have zero tenant awareness. When we need RLS for small tenants (municipalities), only `get_tenant_session()` changes. |
| **SCORM runtime** | scorm-again (MIT, browser JS) | Full SCORM 1.2/2004 support, runs in browser regardless of backend language |
| **xAPI LRS** | SQL LRS or built into main app | PostgreSQL-based, no MongoDB dependency |
| **Search** | Meilisearch (text) + pgvector (semantic) | Typo-tolerant Norwegian search + AI-powered semantic search |
| **Auth** | Authlib (Python OIDC/SAML) + custom SCIM server | ForgeRock, ID-porten (OIDC-only since Jan 2026), BankID, Maskinporten |
| **AI** | pgvector embeddings, Azure OpenAI Sweden or Claude EU for LLM | Rule-based recommendations for MVP, collaborative filtering Phase 2 |
| **Job Queue** | Celery or ARQ (Redis-backed) | Async: completions, notifications, integrations, cert generation |
| **Certificates** | ReportLab or WeasyPrint (Python) | PDF/A generation with templates, QR verification |
| **Infrastructure** | AWS eu-north-1 (Stockholm), ECS Fargate | EU data residency, simpler than Kubernetes |
| **CI/CD** | GitHub Actions | Simple deployment pipeline |

### Product Scope

**11 epics, ~66 MVP user stories, ~40 Phase 2 stories** (revised after gap analysis):

| Epic | Name | Phase |
|---|---|---|
| E1 | Platform Foundation (RBAC, org hierarchy, notifications, audit) | MVP |
| E2 | Authentication & Identity (OIDC, SCIM, ID-porten, SAML) | MVP |
| E3 | Content Engine (SCORM player, course management, quiz, catalog) | MVP |
| E4 | Learning Management (dashboard, enrollment, progress, learning paths, events) | MVP |
| E5 | Competency Framework (skills, gap analysis, certifications, per-group validity) | MVP |
| E6 | External Portal (self-registration, external catalog, certificates) | MVP |
| E7 | Integrations (Kursoppslag API, webhooks; ServiceNow/PowerBI in Phase 2) | MVP (partial) |
| E8 | AI & Personalization (recommendations, semantic search, content assist) | MVP (basic) |
| E9 | Admin & Reporting (dashboards, reports, branding, scoped management) | MVP |
| E10 | Compliance & Security (WCAG, encryption, GDPR, DDoS) | MVP (partial) |
| E11 | Infrastructure & DevOps (cloud, CI/CD, monitoring, backups) | MVP |
| **E12** | **Workplace Learning & CMS** (70/20/10 activities, learning areas, landing page CMS) | **MVP** |

### Database Schema

**50 tables per tenant schema**, fully designed with:
- Complete SQL DDL in `build_plan/09_database_schema.md` (3,156 lines)
- Unified `learning_records` table for 70/20/10 timeline (formal + social + experiential learning in one table)
- Partitioned tables: completions, audit_log, xapi_statements, learning_records, notifications
- Materialized views for leader dashboards (refreshed every 15 min)
- All 11 gap areas covered (workplace activities, approval workflows, checklists, learning areas, CMS pages, event forms, target groups, object permissions, custom roles, assignments)
- Seed data for Norwegian permissions, roles, and activity types

### Data Access Layer

All database access through tenant-aware repository pattern:
- `TenantContext` middleware resolves tenant from subdomain/JWT
- `get_tenant_session()` is the ONLY way to get a DB session (handles schema switching)
- `BaseRepository` provides CRUD operations
- Domain repositories (CourseRepository, LearningRecordRepository, etc.) extend BaseRepository
- Services receive repositories, contain business logic, have zero DB awareness
- FastAPI routes wire it together via dependency injection
- 7 strict rules: no raw SQL with schema names, no direct session access, etc.

### Timeline & Cost

| Milestone | Duration | Team | Cost |
|---|---|---|---|
| MVP (demo-ready) | 34 weeks (~8 months) | 3-4 FTE + AI | ~3.5-4.0 MNOK |
| Production v1 | 12 months | 6-7 FTE + AI | ~8-9 MNOK |
| Market ready (with reference) | 18 months | 8 FTE + AI | ~12-14 MNOK |

Revenue path: 3.6M NOK year 1 -> 58M NOK year 3 -> 180M NOK ARR year 5.

### Key Risks

- SCORM compliance complexity (mitigated by scorm-again library)
- Zero references vs established vendors (start with smaller agencies)
- Timeline (34 weeks MVP with AI building)
- Norwegian IAM complexity (need ForgeRock/ID-porten experience)

### Open Source Components Identified

| Component | Library | License |
|---|---|---|
| SCORM runtime | scorm-again v3.0.1 | MIT |
| xAPI LRS | SQL LRS v0.9.3 | Apache-2.0 |
| Norwegian design system | Designsystemet v1.13.0 | MIT |
| Interactive content | H5P via Lumi | GPL-3.0 (isolated) |
| Quiz engine | SurveyJS | MIT |
| SCIM server | SCIMMY | MIT |
| Certificate PDFs | pdfme | MIT |
| Video player | Video.js | Apache-2.0 |
| Notifications | Novu | MIT |
| LTI integration | ltijs | Apache-2.0 |

## Documents Produced

### Track A: SVV Bid (in `delivery_plan/`)
- `DELIVERY_PLAN_FINAL.md` -- Approved delivery plan for bidding SVV with Cornerstone Galaxy
- 13 supporting specialist documents (architecture, UX, AI, market research, bid strategy, domain research, sales positioning, bid management, risk assessment, reviews)

### Track B: Build Plan (in `build_plan/`)
- `BUILD_DELIVERY_PLAN.md` -- Master build plan (updated with Python stack, gap analysis, data access layer)
- `01_architecture.md` -- Original TypeScript architecture (superseded)
- `01b_architecture_revised.md` -- Python/FastAPI architecture rationale
- `02_ux_design.md` -- Product UX with wireframes, user flows, design system
- `03_ai_architecture.md` -- AI/ML build-vs-buy decisions
- `04_reusable_components.md` -- Open-source building blocks research
- `05_epics_and_stories.md` -- 11 epics with 92 user stories (pre-gap analysis)
- `06_market_opportunity.md` -- Norwegian public sector LMS market sizing
- `07_project_plan.md` -- Team, timeline, sprint structure
- `08_risk_feasibility.md` -- Build feasibility assessment
- `09_database_schema.md` -- Complete PostgreSQL schema (50 tables, 3,156 lines)
- `10_data_access_layer.md` -- Repository pattern for tenant-agnostic data access
- `GAP_ANALYSIS.md` -- 11 gaps found cross-referencing against SVV tender documents
- `TENDER_BRIEFING.md` -- Full tender summary (in project root)

## Unresolved Architecture Decisions

4 big decisions identified but not yet deeply explored:

### 1. SCORM Sandboxing & Content Serving (HIGHEST RISK)
How to serve and sandbox SCORM packages in iframes. SCORM 2004 requires same-origin for cross-frame API calls. Packages can be 100MB+. Per-tenant content isolation in CDN. This is where most custom LMS projects fail.

### 2. Content Authoring: Build vs Import-Only
MVP is import-only (SCORM upload + SurveyJS quiz builder). Do we need a built-in course editor to compete with Cornerstone/Docebo? Affects product stickiness.

### 3. Offline/PWA for Field Workers
SVV has field workers on tunnels/bridges/highways. Do we need offline SCORM playback? Technically very challenging. Binary decision -- retrofitting is hard.

### 4. Certificate Legal Validity
15,000 certificates/year that are arkivpliktige. Do we need eIDAS qualified electronic signatures? What makes a PDF/A certificate legally valid under Norwegian law?

---

## NEXT TASK: Deep Dive on SCORM Sandboxing

The immediate next task is to research and design the SCORM content serving and sandboxing architecture. This is the highest technical risk in the entire build -- many custom LMS projects fail here.

Key questions to answer:
1. How do we extract, store, and serve SCORM packages (S3 direct with presigned URLs? CloudFront? Same-origin proxy?)
2. How do we sandbox SCORM content in iframes while maintaining the cross-frame API communication that SCORM requires?
3. How does `scorm-again` work with iframe sandboxing in practice?
4. How do we handle SCORM packages that use deprecated APIs, Flash references, or non-standard patterns?
5. How do we isolate content between tenants at the CDN/serving layer?
6. What do Moodle, Canvas, and other open-source LMS platforms do for SCORM serving?
7. Performance: how to handle 100MB+ packages with thousands of concurrent users?

Research online for best practices, then design the architecture.
