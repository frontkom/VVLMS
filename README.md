# Veileder

**Norwegian-first LMS for the public sector.**

Veileder is a Learning Management System (LMS) built specifically for Norway's public sector -- state agencies, health trusts, and municipalities. It combines strong data sovereignty, native Norwegian language support, and seamless integration with Norwegian identity infrastructure (ID-porten, ForgeRock, BankID).

> **Status**: Planning phase complete. Development not yet started.

## Why Veileder?

Norway's public sector (750,000-830,000 workers) currently depends on international LMS platforms that don't meet Norwegian requirements for data residency, accessibility, or identity management. Veileder is purpose-built to fill this gap.

**Key differentiators:**
- Built on [Designsystemet](https://designsystemet.no), the official Norwegian government design system
- EU/EEA data residency (AWS Stockholm) -- Schrems II compliant
- Native ID-porten and ForgeRock SSO integration
- SCIM 2.0 for automated user provisioning
- Full SCORM 1.2/2004 and xAPI support
- AI-powered course recommendations and semantic search in Norwegian
- WCAG 2.1 AA accessibility (Norwegian law)
- Schema-per-tenant multi-tenancy for strong data isolation

## Tech Stack

| | Technology |
|---|---|
| **Backend** | Python 3.11+ / FastAPI |
| **Frontend** | Next.js 15 / React 19 |
| **Database** | PostgreSQL 16 + pgvector |
| **Search** | Meilisearch + pgvector |
| **Infrastructure** | AWS ECS Fargate (eu-north-1) |
| **SCORM** | scorm-again (browser runtime) |
| **Auth** | Authlib (OIDC/SAML), ID-porten, SCIM |

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  CloudFront                      │
│              app.veileder.no                     │
├──────────────┬──────────────┬───────────────────┤
│  /app/*      │  /api/*      │  /scorm/*         │
│  Next.js SSR │  FastAPI     │  S3 (SCORM pkgs)  │
└──────┬───────┴──────┬───────┴──────┬────────────┘
       │              │              │
       v              v              v
┌──────────┐  ┌──────────────┐  ┌─────────┐
│ Next.js  │  │   FastAPI    │  │  S3     │
│ (ECS)    │  │   (ECS)      │  │ (OAC)  │
└──────────┘  └──────┬───────┘  └─────────┘
                     │
              ┌──────┴───────┐
              │ PostgreSQL   │
              │ (per-tenant  │
              │  schemas)    │
              └──────────────┘
```

## Product Scope

12 epics covering the full LMS lifecycle:

| # | Epic | Description |
|---|------|-------------|
| E1 | Platform Foundation | RBAC, org hierarchy, notifications, audit logging |
| E2 | Authentication | OIDC, SCIM, ID-porten, SAML, ForgeRock |
| E3 | Content Engine | SCORM player, course management, quiz builder, catalog |
| E4 | Learning Management | Learner dashboard, enrollment, progress, learning paths, events |
| E5 | Competency Framework | Skills taxonomy, gap analysis, certifications |
| E6 | External Portal | Self-registration, external catalog, certificate verification |
| E7 | Integrations | Kursoppslag API, webhooks, ServiceNow, Power BI |
| E8 | AI & Personalization | Recommendations, semantic search, content assist |
| E9 | Admin & Reporting | Analytics dashboards, reports, tenant branding |
| E10 | Compliance & Security | WCAG, encryption at rest/transit, GDPR, DDoS protection |
| E11 | Infrastructure | CI/CD, monitoring, backups, disaster recovery |
| E12 | Workplace Learning | 70/20/10 model, learning areas, landing page CMS |

## Project Documentation

All planning documents are in the `build_plan/` directory:

| Document | Description |
|----------|-------------|
| `BUILD_DELIVERY_PLAN.md` | Master build plan with full strategy and timeline |
| `01b_architecture_revised.md` | Architecture rationale (Python/FastAPI decision) |
| `02_ux_design.md` | UX strategy, wireframes, user flows |
| `03_ai_architecture.md` | AI/ML strategy and build-vs-buy analysis |
| `04_reusable_components.md` | Open-source component research |
| `05_epics_and_stories.md` | 12 epics with 66+ MVP user stories |
| `06_market_opportunity.md` | Norwegian public sector market sizing |
| `07_project_plan.md` | Team structure, sprint plan, timeline |
| `08_risk_feasibility.md` | Build feasibility assessment |
| `09_database_schema.md` | Complete PostgreSQL schema (50 tables) |
| `10_data_access_layer.md` | Repository pattern for tenant-aware data access |
| `11_scorm_content_architecture.md` | SCORM content serving and sandboxing |
| `GAP_ANALYSIS.md` | Requirements gap analysis vs SVV tender |

## Target Market

| Segment | Size | Examples |
|---------|------|---------|
| State agencies | ~170,000 workers | Statens vegvesen, Skatteetaten, NAV |
| Health trusts | ~150,000 workers | Helse Sor-Ost, Helse Vest |
| Municipalities | ~450,000 workers | Oslo, Bergen, Trondheim |

First target customer: **Statens vegvesen** (5,300 employees + 15,000 external certificate holders).

## Timeline

| Milestone | Duration | Target |
|-----------|----------|--------|
| MVP (demo-ready) | ~8 months | Internal demo + first prospect meetings |
| Production v1 | 12 months | First pilot customer live |
| Market ready | 18 months | Tender-competitive with references |

## License

Proprietary. All rights reserved.
