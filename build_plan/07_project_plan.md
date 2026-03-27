# Project Plan: LMS Product Build
## Team, Timeline, Sprint Structure, and Effort Estimation

**Date**: 2026-03-22
**Context**: Building an LMS SaaS product from scratch targeting Norwegian public sector. First customer target: Statens vegvesen (SVV). SVV demo dates: 11-12.05.2026. Qualification deadline: 07.04.2026.

---

## 1. TIMELINE REALITY CHECK

### 1.1 Can We Demo by May 2026?

**Short answer: No, not a production-quality demo.**

Working backwards from SVV's demonstration dates (11-12.05.2026):

| Date | Event | Days from Today |
|---|---|---|
| 22.03.2026 | Today | 0 |
| 07.04.2026 | Qualification deadline | 16 days |
| 06.05.2026 | Bid 1 deadline | 45 days |
| 07.05.2026 | Test environment due | 46 days |
| 11-12.05.2026 | Demonstrations | 50-51 days |

We have approximately 7 weeks to build something demonstrable. This is not enough time to build a credible LMS platform that can compete against established vendors like Cornerstone, Docebo, or Totara in a live demo evaluated by 5+ SVV evaluators using ISO 9241-11 and Nielsen heuristics.

**What IS possible in 7 weeks**: A clickable prototype or limited-scope demo showing core navigation, user flows, and 2-3 key differentiators (e.g., AI-powered learning recommendations, Norwegian public sector-specific competency management). This would not pass hands-on testing by evaluators.

### 1.2 Honest Assessment: SVV Bid Viability

| Scenario | Feasibility | Risk |
|---|---|---|
| Bid SVV with "product in development" | Very low chance of qualification | SVV explicitly requires a "proven SaaS standard solution" -- pre-qualification demands comparable LMS references (65% weight) |
| Build demo-only prototype for SVV | Possible but poor ROI | Demo will not withstand hands-on evaluator testing; no references to pass qualification |
| Skip SVV, build product for next opportunity | Realistic | Lose first-mover timing but build something credible |
| Hybrid: bid SVV with partner platform, build own product in parallel | Complex but viable | Use Cornerstone/Docebo bid to win SVV; build own product for next tenders |

**Recommendation**: The SVV tender is better served by the partner-platform approach (see delivery_plan/). The build plan should target the NEXT public sector LMS procurement, giving us 12-18 months to build a competitive product. However, lessons from the SVV bid process (requirements, evaluation criteria, competitor analysis) directly inform what we build.

### 1.3 Realistic Timelines

| Milestone | Best Case | Expected | Worst Case |
|---|---|---|---|
| Development start | April 2026 | April 2026 | May 2026 |
| Core platform MVP (internal demo) | August 2026 | October 2026 | December 2026 |
| Demo-ready MVP (can show to prospects) | October 2026 | January 2027 | March 2027 |
| Pilot-ready (real users, limited scope) | January 2027 | April 2027 | July 2027 |
| Production v1 (full tender-ready) | April 2027 | July 2027 | October 2027 |
| First production customer live | July 2027 | October 2027 | January 2028 |

**Key assumption**: A small, focused team (4-6 people) starting April 2026.

---

## 2. TEAM COMPOSITION

### 2.1 Roles and Sizing

The team must be small to start. We cannot justify a large team before product-market fit is validated. The build has four parallel workstreams: platform, content engine, integrations, and AI.

#### Phase 1: Core Build Team (Month 1-6)

| Role | Count | FTE | Workstream | Non-Negotiable Skills |
|---|---|---|---|---|
| **Tech Lead / Architect** | 1 | 1.0 | All (oversight) | Full-stack senior, cloud architecture, identity/auth (OIDC/SAML/SCIM), Norwegian public sector compliance |
| **Backend Developer** | 2 | 2.0 | Platform + Integrations | API design, multi-tenancy, event-driven architecture, PostgreSQL, one must know SCIM/IAM |
| **Frontend Developer** | 1 | 1.0 | Platform + UX | Modern JS/TS framework (React/Next.js or similar), accessibility (WCAG 2.1 AA), responsive design |
| **Product Manager / PM** | 1 | 0.5 | All (coordination) | Public sector procurement knowledge, LMS domain understanding, agile delivery |
| **UX/UI Designer** | 1 | 0.5 | Platform + Content | Enterprise SaaS design, WCAG compliance, Norwegian language UI, design system creation |

**Total Phase 1**: 5 people, 5.0 FTE

#### Phase 2: Scale Team (Month 7-12)

| Role | Count | FTE | Added When |
|---|---|---|---|
| **AI/ML Engineer** | 1 | 1.0 | Month 4-5 (start part-time Month 3) |
| **Backend Developer** | 1 | 1.0 | Month 6 (integrations ramp-up) |
| **Frontend Developer** | 1 | 1.0 | Month 7 (content authoring UI) |
| **QA / Test Engineer** | 1 | 0.5 | Month 6 (pre-pilot testing) |
| **DevOps / Platform Engineer** | 1 | 0.5 | Month 4 (CI/CD, infrastructure) |

**Total Phase 2 peak**: 10 people, 8.5 FTE (Month 8-12)

#### Phase 3: Production Readiness (Month 13-18)

| Role | Count | FTE | Focus |
|---|---|---|---|
| **Security Engineer** | 1 | 0.5 | Penetration testing, ROS-analyse, GDPR compliance verification |
| **Content Migration Specialist** | 1 | 0.5 | SCORM/xAPI import tooling, Kilden-format handling |
| **Technical Writer** | 1 | 0.3 | Admin documentation, API docs, user guides |

**Total Phase 3 peak**: 13 people, 10.3 FTE

### 2.2 Non-Negotiable Skills (Across Team)

These skills MUST exist in the team from day one. If they are missing, hiring or contracting for them is a blocker.

1. **Norwegian public sector IAM**: ForgeRock, ID-porten, Maskinporten, BankID integration experience. Without this, the IAM integration (which is the hardest technical challenge) will stall.
2. **Multi-tenant SaaS architecture**: Building for multiple customers from the start. Data isolation, tenant configuration, per-tenant branding.
3. **WCAG 2.1 AA accessibility**: Norwegian law requires universal design. This is not optional and cannot be retrofitted.
4. **Norwegian language proficiency**: At least PM and designer must be fluent. UX copy, error messages, and admin interfaces must be natively Norwegian.
5. **SCORM/xAPI/cmi5 standards**: At least one developer must understand e-learning content standards deeply.

### 2.3 Scaling Strategy

Start with the 5-person core team. Validate the architecture and core user flows before adding people. Adding developers to a project that lacks clear architecture creates waste, not speed.

| Trigger | Action |
|---|---|
| Architecture validated (Month 2-3) | Add AI/ML engineer, DevOps |
| First internal demo works (Month 4-5) | Add 3rd backend developer for integrations |
| Pilot customer confirmed (Month 6-8) | Add QA, 2nd frontend developer |
| Production readiness (Month 12+) | Add security, migration, tech writer |

---

## 3. SPRINT STRUCTURE

### 3.1 Sprint Length: 2 Weeks

**Rationale**: 1-week sprints create too much ceremony overhead for a small team. 2-week sprints allow meaningful progress while maintaining short feedback loops. Once the product is more mature (post-v1), consider switching to 1-week sprints for faster iteration.

### 3.2 Sprint Calendar

| Day | Activity | Duration | Participants |
|---|---|---|---|
| **Monday W1** | Sprint Planning | 1.5 hours | Full team |
| **Daily** | Standup | 15 min | Full team |
| **Wednesday W1** | Backlog Refinement | 1 hour | PM, Tech Lead, Designer + relevant devs |
| **Thursday W2** | Sprint Demo | 45 min | Full team + stakeholders |
| **Thursday W2** | Retrospective | 30 min | Full team |
| **Friday W2** | Sprint Review / Planning prep | 1 hour | PM, Tech Lead |

**Total ceremony time**: ~6 hours per sprint per person (out of 80 working hours = 7.5%). This is lean enough for a startup-speed team.

### 3.3 Definition of Done

A story/task is "Done" when ALL of the following are true:

1. **Code complete**: Implementation matches acceptance criteria
2. **Tests pass**: Unit tests + integration tests covering the change
3. **Accessibility verified**: WCAG 2.1 AA compliance checked for any UI change
4. **Code reviewed**: At least one other developer has reviewed and approved
5. **Documentation updated**: API docs, admin guides if applicable
6. **Deployed to staging**: Change is running in the staging environment
7. **No regressions**: Existing functionality still works (CI pipeline green)
8. **Norwegian language**: All user-facing text is in Norwegian (bokmaal)

### 3.4 Handling Parallel Workstreams

Four workstreams run in parallel. They must stay loosely coupled to avoid blocking each other.

| Workstream | Primary Owner | Dependencies |
|---|---|---|
| **Platform Core** | Tech Lead + Backend Dev 1 | None (foundational) |
| **Content Engine** | Backend Dev 2 + Frontend Dev | Platform Core (auth, data model) |
| **Integrations** | Backend Dev (from Month 6) | Platform Core (API layer, auth) |
| **AI/ML** | AI Engineer | Content Engine (data), Platform Core (API) |

**Coordination mechanism**:
- Workstreams share a single backlog, prioritized by PM
- Cross-workstream dependencies are tracked as blockers in the backlog
- Weekly 30-min architecture sync (Tech Lead + workstream leads) to catch coupling issues early
- Shared API contracts defined early and versioned -- workstreams code against contracts, not implementations

---

## 4. MILESTONE PLAN

### 4.1 Epic-to-Milestone Mapping

#### M1: Foundation (Sprints 1-3, Weeks 1-6)
**Gate: Architecture validated, core data model stable**

| Epic | Key Deliverables |
|---|---|
| E1: Multi-Tenant Platform | Tenant isolation, configuration framework, database schema, deployment pipeline |
| E2: Authentication & Authorization | OIDC/SAML integration framework, role-based access control, session management |
| E3: User Management | User model, organizational hierarchy, SCIM endpoint (receive), profile management |
| E10: DevOps & Infrastructure | CI/CD pipeline, staging environment, EU-hosted cloud infrastructure, monitoring |

**Decision point**: Is the architecture sound? Can we build the rest on this foundation? If not, pivot now.

#### M2: Core Learning (Sprints 4-6, Weeks 7-12)
**Gate: Internal users can browse and complete a course**

| Epic | Key Deliverables |
|---|---|
| E4: Course Catalog & Search | Course listing, search (basic), filtering, course detail pages |
| E5: Content Player | SCORM 1.2/2004 player, xAPI support, cmi5 support, progress tracking |
| E6: Learning Paths | Sequenced course collections, prerequisites, progress visualization |
| E7: Enrollment & Assignment | Self-enrollment, admin assignment, mandatory course flagging, due dates |

**Decision point**: Does the learning experience feel credible? Can we demo this to a prospect?

#### M3: Admin & Reporting (Sprints 7-9, Weeks 13-18)
**Gate: Admins can manage courses, users, and view reports**

| Epic | Key Deliverables |
|---|---|
| E8: Admin Dashboard | User management, course management, organization management, bulk operations |
| E9: Reporting & Analytics | Completion reports, compliance dashboards, user activity, export to CSV/Excel |
| E12: Notifications & Communication | Email notifications, in-app notifications, reminders for due courses |

**Decision point**: Is the admin experience efficient enough for 400+ administrators?

#### M4: Demo-Ready MVP (Sprints 10-12, Weeks 19-24)
**Gate: Can demonstrate all 5 POC scenarios from SVV-type tender**

| Epic | Key Deliverables |
|---|---|
| E11: External User Portal | Self-registration, ID-porten login, competency certificates, public course catalog |
| E13: Content Authoring (Basic) | Simple course creation, quiz builder, SCORM upload, metadata management |
| E14: AI Recommendations (v1) | Content-based recommendations, semantic search (Norwegian NLP) |
| Polish & Integration | UX polish, performance optimization, demo data setup |

**Decision point**: Can we bid a real tender with this product? What gaps remain vs. competitor platforms?

#### M5: Integration Suite (Sprints 13-16, Weeks 25-32)
**Gate: Can integrate with Norwegian public sector identity and HR systems**

| Epic | Key Deliverables |
|---|---|
| E15: ForgeRock IAM Integration | Full SCIM provisioning, OIDC/SAML federation, MFA support |
| E16: ID-porten / BankID | External user authentication via national eID |
| E17: ServiceNow Integration | Competency register sync, learning log, bidirectional REST API |
| E18: PowerBI / Data Export | Structured data export, API for BI tools, pre-built report templates |
| E19: Kursoppslag API | Proxy-compatible REST API matching SVV's existing API contract |

#### M6: Production v1 (Sprints 17-20, Weeks 33-40)
**Gate: Ready for pilot customer deployment**

| Epic | Key Deliverables |
|---|---|
| E20: Security Hardening | Penetration testing, ROS-analyse framework, vulnerability management |
| E21: GDPR Compliance | DPA framework, data export/deletion, consent management, sub-processor docs |
| E22: Migration Tooling | SCORM/xAPI import, user data migration, Kilden-format adapter |
| E23: Archiving & Certificates | PDF/A certificate generation, Noark-5 export, digital validation |
| E24: Performance & Scale | Load testing (5,000+ concurrent), caching, CDN, database optimization |

#### M7: Market Ready (Sprints 21-24, Weeks 41-48)
**Gate: Can onboard customers independently**

| Epic | Key Deliverables |
|---|---|
| E25: Tenant Onboarding | Self-service tenant setup, configuration wizard, branding customization |
| E26: AI v2 | Collaborative filtering, learning path optimization, competency gap analysis |
| E27: Social Learning | Forums, professional networks, peer content sharing, mentoring tools |
| E28: Advanced Content | Video hosting, Articulate integration, content versioning, approval workflows |
| E29: Documentation & Training | Admin guides, API documentation, onboarding materials |

### 4.2 Key Decision Points

| When | Decision | Impact |
|---|---|---|
| End of M1 (Week 6) | Architecture Go/No-Go | If architecture is wrong, cost of change is still low |
| End of M2 (Week 12) | Product viability assessment | Can this compete? Pivot, persevere, or partner? |
| End of M4 (Week 24) | First tender bid decision | Do we bid the next public sector LMS tender? |
| End of M5 (Week 32) | Integration validation | Are Norwegian public sector integrations working? |
| End of M6 (Week 40) | Pilot customer Go/No-Go | Ready for real users? |
| End of M7 (Week 48) | GA release decision | Ready for general availability? |

---

## 5. EFFORT ESTIMATION

### 5.1 Person-Months by Phase

| Phase | Duration | Avg FTE | Person-Months |
|---|---|---|---|
| M1: Foundation (W1-6) | 1.5 months | 5.0 | 7.5 |
| M2: Core Learning (W7-12) | 1.5 months | 5.5 | 8.25 |
| M3: Admin & Reporting (W13-18) | 1.5 months | 6.5 | 9.75 |
| M4: Demo-Ready MVP (W19-24) | 1.5 months | 7.0 | 10.5 |
| **MVP Total** | **6 months** | **6.0 avg** | **36 person-months** |
| M5: Integration Suite (W25-32) | 2 months | 8.0 | 16.0 |
| M6: Production v1 (W33-40) | 2 months | 9.0 | 18.0 |
| **Production v1 Total** | **10 months** | **7.4 avg** | **70 person-months** |
| M7: Market Ready (W41-48) | 2 months | 10.0 | 20.0 |
| **Full Product Total** | **12 months** | **7.5 avg** | **90 person-months** |

### 5.2 Cost Estimation

#### Salary Costs (Norwegian market rates, 2026)

| Role | Monthly Cost (employer total) | Notes |
|---|---|---|
| Senior Developer / Tech Lead | 110,000 NOK | Includes social costs, pension, insurance |
| Mid-Senior Developer | 90,000 NOK | |
| AI/ML Engineer | 100,000 NOK | |
| UX/UI Designer | 85,000 NOK | |
| Product Manager | 95,000 NOK | |
| QA Engineer | 80,000 NOK | |
| DevOps Engineer | 95,000 NOK | |

**Note**: These are fully loaded employer costs (bruttolonn + arbeidsgiveravgift + pension + forsikring). If using contractors/consultants, expect 30-50% premium (130,000-165,000 NOK/month for senior roles).

#### Total Cost by Phase (Employees)

| Phase | Person-Months | Avg Cost/PM | Total |
|---|---|---|---|
| MVP (M1-M4) | 36 | 95,000 | 3,420,000 NOK |
| Production v1 (M5-M6) | 34 | 93,000 | 3,162,000 NOK |
| Market Ready (M7) | 20 | 92,000 | 1,840,000 NOK |
| **Total Labor** | **90** | | **8,422,000 NOK** |

#### Infrastructure Costs

| Item | Monthly Cost | Annual Cost | Notes |
|---|---|---|---|
| Cloud hosting (EU, dev+staging+prod) | 15,000-30,000 NOK | 180,000-360,000 NOK | AWS/Azure EU region, scales with usage |
| CI/CD and tooling | 5,000 NOK | 60,000 NOK | GitHub, monitoring, error tracking |
| Design tools | 3,000 NOK | 36,000 NOK | Figma, prototyping |
| AI/ML infrastructure | 10,000-20,000 NOK | 120,000-240,000 NOK | GPU instances for Norwegian NLP, embeddings |
| Security tooling | 5,000 NOK | 60,000 NOK | SAST, DAST, dependency scanning |
| Domain, certificates, CDN | 2,000 NOK | 24,000 NOK | |
| **Total Infrastructure** | **40,000-65,000** | **480,000-780,000 NOK** | |

#### Total Build Cost Summary

| Category | MVP (6 months) | Production v1 (10 months) | Full Product (12 months) |
|---|---|---|---|
| Labor | 3,420,000 | 6,582,000 | 8,422,000 |
| Infrastructure | 240,000-390,000 | 400,000-650,000 | 480,000-780,000 |
| Contingency (15%) | 549,000-571,500 | 1,047,300-1,084,800 | 1,335,300-1,380,300 |
| **Total** | **4,209,000-4,381,500** | **8,029,300-8,316,800** | **10,237,300-10,582,300** |

### 5.3 Burn Rate

| Phase | Monthly Burn Rate | Team Size |
|---|---|---|
| Month 1-3 (Foundation) | ~500,000 NOK | 5 people |
| Month 4-6 (Core Learning + MVP) | ~650,000 NOK | 6-7 people |
| Month 7-10 (Integrations + Production) | ~850,000 NOK | 8-9 people |
| Month 11-12 (Market Ready) | ~950,000 NOK | 10 people |
| Post-launch (steady state) | ~700,000 NOK | 7-8 people (support + development) |

### 5.4 Revenue Breakeven Analysis

Using SVV contract as reference (20-35 MNOK over 6 years = 3.3-5.8 MNOK/year):

| Scenario | Annual Revenue per Customer | Customers to Break Even (monthly) | Time to Breakeven |
|---|---|---|---|
| Small contracts | 1.0 MNOK/year | 8-10 | 24+ months post-launch |
| Medium contracts (SVV-size) | 4.0 MNOK/year | 2-3 | 12-18 months post-launch |
| Mix | 2.0 MNOK/year avg | 4-5 | 18-24 months post-launch |

**Total investment before first revenue**: 8-10 MNOK (12 months build + 3-6 months sales cycle).

---

## 6. RISK-ADJUSTED TIMELINE

### 6.1 Schedule Scenarios

| Milestone | Best Case | Expected | Worst Case |
|---|---|---|---|
| M1: Foundation complete | Week 5 | Week 6 | Week 8 |
| M2: Core Learning complete | Week 10 | Week 12 | Week 16 |
| M4: Demo-Ready MVP | Week 20 | Week 24 | Week 30 |
| M6: Production v1 | Week 34 | Week 40 | Week 50 |
| M7: Market Ready | Week 42 | Week 48 | Week 60 |

### 6.2 Schedule Risks

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| **SCORM/xAPI player complexity** | High | 3-4 week delay | Use open-source SCORM player (rustici-software/scormcloud or similar) rather than building from scratch |
| **ForgeRock integration unknowns** | High | 4-6 week delay | Start with generic OIDC/SCIM; specialize for ForgeRock only when a customer requires it |
| **Norwegian NLP for search/AI** | Medium | 2-4 week delay | Use existing Norwegian language models (NB-BERT, multilingual embeddings); don't train from scratch |
| **WCAG compliance retrofitting** | Medium | 3-5 week delay | Build accessible from Sprint 1; do not defer. Include in Definition of Done |
| **Key person departure** | Medium | 6-8 week delay | Document architecture decisions (ADRs); no single-person knowledge silos |
| **Multi-tenancy complexity** | Medium | 2-4 week delay | Design for multi-tenancy from day one; do not retrofit |
| **Underestimated integration scope** | High | 4-8 week delay | Each integration (ForgeRock, ServiceNow, PowerBI, ID-porten, Kursoppslag, Mime) is a mini-project. Budget generously. |
| **Hiring delays** | High | 4-8 week delay | Start recruiting immediately; have contractor backup plan |
| **Regulatory/compliance surprises** | Low | 2-6 week delay | Engage legal/compliance advisor early; do not assume requirements are stable |

### 6.3 Risk-Adjusted Budget

| Scenario | Total Cost (12 months) | Notes |
|---|---|---|
| Best case | 9.5 MNOK | Minimal hiring delays, architecture works first time |
| Expected | 11.5 MNOK | Some delays, one major pivot, typical hiring timeline |
| Worst case | 14.0 MNOK | Multiple delays, architecture rework, difficult hiring market |

---

## 7. BUILD vs. BUY DECISION CONTEXT

This project plan assumes a full build. For context, here is how build compares to the partner-platform approach documented in `delivery_plan/`:

| Factor | Build Our Own | Partner Platform (Cornerstone/Docebo) |
|---|---|---|
| Time to first customer | 12-18 months | 6-9 months (SVV timeline) |
| Total cost to first customer | 8-10 MNOK | 2-3 MNOK (bid prep + integration) |
| Long-term margin | Higher (own IP, no license fees) | Lower (pass-through licensing, vendor dependency) |
| Competitive differentiation | Full control, Norwegian-first | Limited to configuration and integration |
| Risk | High (unproven product) | Lower (proven platform) |
| SVV tender viability | Cannot bid May 2026 | Can bid May 2026 |
| Scalability to other customers | Own product, full control | Constrained by vendor terms |

**The two approaches are not mutually exclusive.** The recommended strategy:

1. **Short-term (now)**: Bid SVV with partner platform (Cornerstone Galaxy) per delivery_plan
2. **Medium-term (April 2026+)**: Begin building own LMS product in parallel
3. **Long-term (2027+)**: Transition to own product for new customers; potentially migrate SVV if/when own product is mature enough

---

## 8. TECH STACK RECOMMENDATIONS

For the build plan, the following stack is recommended based on the requirements:

| Layer | Technology | Rationale |
|---|---|---|
| **Backend** | Python (Django/FastAPI) or Node.js (NestJS) | Rapid development, strong ecosystem for REST APIs, good AI/ML integration |
| **Frontend** | React + Next.js or SvelteKit | Component-based, strong accessibility tooling, SSR for performance |
| **Database** | PostgreSQL | Multi-tenant support, JSON columns for flexible metadata, proven at scale |
| **Search** | Elasticsearch or Meilisearch | Full-text search with Norwegian language support |
| **AI/ML** | Python + HuggingFace + Azure OpenAI (EU) | Norwegian NLP models, RAG architecture, EU-hosted LLM |
| **Auth** | Keycloak or custom OIDC/SAML | Standards-based, ForgeRock-compatible, ID-porten ready |
| **Content Player** | SCORM Cloud API or open-source (scorm-again) | SCORM 1.2/2004/xAPI/cmi5 without building from scratch |
| **Storage** | S3-compatible (EU region) | SCORM packages, certificates, user uploads |
| **Infrastructure** | Kubernetes on AWS EKS or Azure AKS (EU) | EU data sovereignty, auto-scaling, multi-tenant isolation |
| **CI/CD** | GitHub Actions | Standard, well-integrated |
| **Monitoring** | Grafana + Prometheus + Sentry | Full observability stack |

**Final tech stack decision should be made by the Tech Lead in Sprint 0 (Week 1).**

---

## 9. SPRINT 0: PRE-DEVELOPMENT (Week 0-1)

Before Sprint 1, the following must be completed:

| Task | Owner | Output |
|---|---|---|
| Finalize tech stack decision | Tech Lead | ADR document |
| Set up development environment | Tech Lead + DevOps | Docker-compose local dev, CI pipeline |
| Set up cloud infrastructure (dev) | DevOps | EU-hosted cloud account, staging environment |
| Create design system foundation | Designer | Component library, color palette, typography, Norwegian UI patterns |
| Define data model v1 | Tech Lead + PM | Entity relationship diagram for users, courses, enrollments, organizations |
| Set up project tooling | PM | Issue tracker, documentation wiki, communication channels |
| Define API contract v1 | Tech Lead | OpenAPI spec for core endpoints |
| Recruit/confirm team | PM | Signed contracts or commitment from all Phase 1 team members |

---

## 10. SUMMARY

| Parameter | Value |
|---|---|
| **Team size (start)** | 5 people (5.0 FTE) |
| **Team size (peak)** | 10-13 people (8.5-10.3 FTE) |
| **Time to demo-ready MVP** | 6 months (expected) |
| **Time to production v1** | 10 months (expected) |
| **Time to market ready** | 12 months (expected) |
| **Total effort (MVP)** | 36 person-months |
| **Total effort (full product)** | 90 person-months |
| **Total cost (expected, 12 months)** | 10.2-10.6 MNOK |
| **Monthly burn rate (average)** | 725,000 NOK |
| **Sprint length** | 2 weeks |
| **Can demo for SVV May 2026?** | No |
| **Recommended strategy** | Bid SVV with partner platform; build own product in parallel |
| **First realistic tender bid with own product** | Q1-Q2 2027 |
