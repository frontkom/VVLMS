# DELIVERY PLAN v1.0
## Statens vegvesen LMS (Laeringsplattform) - Case 25/223727

**Date**: 2026-03-22
**Status**: Draft for internal review
**Next Review**: Risk Assessor + Bid Manager

---

## 1. EXECUTIVE SUMMARY

This delivery plan covers the end-to-end approach for delivering a cloud-based Learning Management System (LMS) to Statens vegvesen (SVV) - the Norwegian Public Roads Administration. The procurement is valued at 20-35 MNOK over a maximum 6-year period (3+1+1+1) under the SSA-L contract framework.

**Core challenge**: SVV does not need a custom LMS built. They need a proven SaaS standard platform integrated into a tightly governed Norwegian government IT ecosystem (ForgeRock IAM, ServiceNow, PowerBI, ID-porten) while consolidating multiple disconnected learning systems (Kilden, KKP, Excel tracking) into one unified platform serving 5,300 internal employees, 1,100 consultants, and 15,000+ external certificate holders.

**Recommended platform**: Cornerstone Galaxy as primary recommendation, with Docebo as alternative. Both meet the critical requirements: EU data centers, Norwegian language, SCIM/SAML/OAuth2, extended enterprise portals, strong AI capabilities, PowerBI integration, and enterprise-grade competency management.

**Win themes**:
1. One Platform for the Entire Competency Lifecycle
2. Security Architecture Built for Norwegian Public Sector
3. Built for How SVV Actually Learns -- the 70/20/10 Model
4. Data-Driven Competency Intelligence for Leaders
5. Future-Ready Platform with Responsible AI

---

## 2. TIMELINE AND CRITICAL MILESTONES

### 2.1 Bid Process Timeline

| Phase | Date | Activity |
|---|---|---|
| **Pre-Qualification** | 07.04.2026 | Qualification submission deadline |
| **Bid 1** | 06.05.2026 | First bid submission |
| **Demonstration** | 11-12.05.2026 | POC demonstration (80 min) |
| **Negotiations R1** | 26-27.05.2026 | First negotiation round |
| **Bid 2** | 08.06.2026 | Revised bid submission |
| **Negotiations R2** | 16-17.06.2026 | Second negotiation round |
| **Final Bid** | 01.07.2026 | Final bid submission |
| **Award** | 21.08.2026 | Contract award decision |
| **Signing** | 01.09.2026 | Contract signing |

**URGENT**: Question deadline for qualification is 23.03.2026 (tomorrow). Qualification submission is 16 days away.

### 2.2 Delivery Timeline (Post Contract Signing)

Based on technical architecture analysis and risk assessment, we propose a **6-phase, 28-week delivery** from contract signing to go-live:

| Phase | Duration | Weeks | Deliverables |
|---|---|---|---|
| **Phase 1: Startup** | 4 weeks | W1-W4 | L01 (startup, test plan, acceptance criteria) |
| **Phase 2: Needs Analysis** | 4 weeks | W3-W6 | L02 (behovsanalyse, learning architecture) |
| **Phase 3: Core Platform** | 6 weeks | W5-W10 | L03 (configuration), L04 (IAM/SSO) |
| **Phase 4: Integrations** | 8 weeks | W7-W14 | L05 (ServiceNow), L06 (PowerBI) |
| **Phase 5: Content & Migration** | 6 weeks | W11-W16 | L07 (learning surfaces), L08 (migration), L09 (training) |
| **Phase 6: Testing & Go-Live** | 6 weeks | W15-W20 | L10 (acceptance test), go-live, hypercare |

**Note**: Phases overlap. Critical path runs through IAM integration (L04) which blocks everything else.

**Estimated go-live**: Q1/Q2 2027 (approximately 5-6 months after contract signing, assuming Sept 2026 signing).

**Risk flag**: The risk assessment identifies go-live by 01.01.2027 as CRITICAL risk (L-01). We recommend communicating a realistic Q2 2027 target with an MVP scope for Q1 2027.

---

## 3. TECHNICAL ARCHITECTURE

### 3.1 Integration Architecture

The LMS sits at the center of a hub-and-spoke topology with 6 surrounding systems:

| System | Direction | Protocol | Priority | Deliverable |
|---|---|---|---|---|
| ForgeRock AM/IDM (IAM) | Inbound | SCIM, OAuth2/OIDC, SAML 2.0 | Mandatory | L04 (fixed price) |
| Kursoppslag (proxy API) | LMS exposes data | REST API (JSON) | Mandatory | Part of L03 |
| ServiceNow | Bidirectional | REST API | Mandatory | L05 (100h T&M) |
| PowerBI | Outbound | REST API / file export | Mandatory | L06 (100h T&M) |
| eTeori | Bidirectional | REST API (OAuth2) | Option | L22 (70h T&M) |
| Mime (archive) | Outbound | SOAP/REST | Option | L24 (150h T&M) |

### 3.2 Key Architecture Decisions

1. **ADR-001**: OAuth2/OIDC as primary authentication (SAML 2.0 as secondary)
2. **ADR-002**: SVV owns Kursoppslag mapping layer; LMS exposes standard APIs
3. **ADR-003**: Event-driven sync with nightly batch reconciliation for ServiceNow
4. **ADR-004**: Direct point-to-point integrations (no middleware needed for 4 mandatory integrations)

### 3.3 Security Architecture

- **Authentication**: OAuth2/OIDC for internal (SSO), ID-porten (OIDC since Jan 2026) for external, MFA for all admin access
- **User provisioning**: SCIM 2.0 from ForgeRock IDM - no manual user creation for internal users
- **Data sovereignty**: EU/EEA data centers only (Frankfurt/Ireland for Cornerstone, equivalent for Docebo)
- **Encryption**: TLS 1.3 in transit, AES-256 at rest, backup to 2 separate locations
- **Compliance**: ISO 27001, SOC 2 Type II, GDPR, arkivloven, WCAG 2.1 AA

### 3.4 Migration Strategy

**From Kilden (Storyboard AS)**:
- 297 active e-learning courses (Articulate Rise/Storyline, SCORM packages)
- Historical completion data (18,500 completions in 2025)
- User data, metadata, access structures
- Approach: Audit > Classify > Pilot (20-30 courses) > Full migration > Validation
- Estimated effort: 150h budgeted, **recommend 250h realistic** (risk flag T-03)
- Dependency: Cooperation from Storyboard AS required

**From KKP Portal (Option L21)**:
- 350,000 competency certificates
- 15,000 certificates issued annually
- Seasonal usage patterns
- Depends on L22 (eTeori integration)

---

## 4. PLATFORM RECOMMENDATION

### 4.1 Primary: Cornerstone Galaxy

**Score: 4.7/5.0** - Best overall enterprise fit

| Strength | Detail |
|---|---|
| AI | Galaxy AI with 40TB+ labor market data, adaptive learning agent |
| EU hosting | France/Germany data centers |
| Norwegian | 50+ languages including Norwegian |
| Extended enterprise | Portal Builder for KKP scenario |
| PowerBI | Official integration utility (open source) |
| Competency mgmt | Skills graph, gap analysis, workforce planning |
| API | Edge APIs, OData, DEAPI, RAPI |

**Budget fit**: Estimated 20-30 MNOK over 6 years (within 20-35M envelope)

### 4.2 Alternative: Docebo

**Score: 4.4/5.0** - Best AI-powered learning experience

Strengths: Excellent AI, strong extended enterprise, 400+ integrations, Learn Data for PowerBI.
Risks: Norwegian language needs confirmation, Canadian parent (GDPR DPA scrutiny needed).

**Budget fit**: Estimated 16-26 MNOK over 6 years.

### 4.3 Not Recommended
- Microsoft Viva Learning (not a full LMS)
- SAP SuccessFactors (overkill standalone, weak external portal)
- Absorb (no confirmed Norwegian)
- Totara (weak AI, no PowerBI connector -- but strong competency management)

---

## 5. DELIVERABLES PLAN

### 5.1 Fixed-Price Deliverables (Pre-Delivery)

| # | Deliverable | Description | Phase | Duration |
|---|---|---|---|---|
| L01 | Startup phase | Test plan, acceptance criteria, project setup, initial training | Phase 1 | 4 weeks |
| L02 | Needs analysis | Workshops with 6 divisions, learning architecture design | Phase 2 | 4 weeks |
| L03 | Platform configuration | Configure per SVV requirements and solution description | Phase 3 | 6 weeks |
| L04 | IAM/SSO integration | ForgeRock SCIM, OAuth2/OIDC, SAML, ID-porten, BankID | Phase 3 | 6 weeks |
| L07 | Learning surfaces | Configure learning areas per approved architecture | Phase 5 | 4 weeks |
| L09 | Documentation & training | Admin/superuser training, Norwegian user guides | Phase 5 | 3 weeks |
| L10 | Acceptance testing | Execute per agreed test plan and acceptance criteria | Phase 6 | 4 weeks |

### 5.2 Time & Materials Deliverables (Pre-Delivery)

| # | Deliverable | Est. Hours | Realistic Hours | Risk |
|---|---|---|---|---|
| L05 | ServiceNow integration | 100h | 120-150h | HIGH - skills framework not finalized |
| L06 | PowerBI integration | 100h | 80-100h | MEDIUM - well-defined data structure |
| L08 | Kilden migration | 150h | 200-250h | HIGH - format uncertainty, Storyboard dependency |

### 5.3 Post-Delivery

| # | Deliverable | Est. Hours | Description |
|---|---|---|---|
| L11 | Support & helpdesk | Ongoing | SLA-defined support in contract period |
| L12 | Change requests | 200h | Enhancements and modifications at discounted rate |

### 5.4 Options

| # | Deliverable | Est. Hours | Dependency |
|---|---|---|---|
| L21 | KKP portal migration | 250h | Requires L22 |
| L22 | eTeori integration | 70h | - |
| L23 | Payment solution | TBD | Not evaluated |
| L24 | Mime archive integration | 150h | - |

---

## 6. UX AND DEMONSTRATION STRATEGY

### 6.1 Evaluation Framework

Usability weighs 12.5% of the 80% quality score. Evaluated via:
- Written UI/UX descriptions (Bilag 2)
- Live POC demonstration (80 min: 40+40 with 5-min break)
- Hands-on evaluator testing in demo environment
- Scored against ISO 9241-11:2018 (Efficacy, Efficiency, Satisfaction) and Nielsen's 10 heuristics

### 6.2 POC Demonstration Strategy (80 minutes)

**Session 1 (40 min)**: User-facing scenarios
| # | Scenario | Role | Key UX Moments |
|---|---|---|---|
| 1.1-1.2 | Internal user | Employee | Personalized landing page, AI recommendations, search, learning path progression |
| 2.1 | External user | Contractor | ID-porten registration, first login, onboarding wizard (max 3 steps) |
| 3.1-3.3 | Leader | Manager | Traffic light dashboard, certification expiry alerts, bulk assignment, activity approval |

**Session 2 (40 min)**: Admin-facing scenarios
| # | Scenario | Role | Key UX Moments |
|---|---|---|---|
| 4.1-4.4 | Content manager | Instructional designer | SCORM upload, learning path builder, course lifecycle, mandatory vs optional config |
| 5.1 | Division admin | HR admin | Dashboards, reports, data export, PowerBI integration preview |

### 6.3 Key UX Differentiators

- Netflix-like content recommendations with Norwegian explanations ("Anbefalt fordi...")
- Traffic light leader dashboard (green/yellow/red compliance)
- Mobile-first design for field workers (tunnels, bridges, highways)
- Visual drag-and-drop learning path builder
- 3-step onboarding wizard for external users
- WCAG 2.1 AA compliance with Norwegian universal design law

---

## 7. AI STRATEGY

### 7.1 Phased AI Roadmap

**Phase 1 (Launch)**: Content-based recommendations, semantic search (Norwegian), competency gap dashboard, certification alerts, basic AI content assist
**Phase 2 (Year 1)**: Collaborative filtering, learning path optimization, AI course generator, admin intelligence, chatbot
**Phase 3 (Years 2-3)**: Predictive models, training ROI, trend forecasting, advanced content generation

### 7.2 AI Compliance

- All AI processing within EU/EEA (Azure OpenAI EU regions or equivalent)
- EU AI Act classification: all features Limited/Minimal risk (no high-risk systems)
- Explainability: every recommendation includes human-readable Norwegian reason
- GDPR: legal basis documented per feature, DPIA planned, right to opt out
- Safety override: hard-coded rules for safety-critical certifications (arbeidsvarsling, HMS) - AI cannot override mandatory requirements
- Graceful degradation: system works fully without AI

### 7.3 AI Infrastructure Cost

Estimated 400,000-950,000 NOK/year absorbed into SaaS pricing (not billed separately).

---

## 8. RESOURCE PLAN

### 8.1 Delivery Team

| Role | Phase | FTE | Key Requirement |
|---|---|---|---|
| Project Manager (SPOC) | All | 1.0 | CV required, approved by SVV, present in all meetings |
| Solution Architect | Phases 1-4 | 0.8 | ForgeRock/IAM experience |
| Integration Developer | Phases 3-5 | 1.0 | SCIM, REST API, ServiceNow |
| UX/Configuration Lead | Phases 2-5 | 0.8 | LMS platform expertise |
| Content Migration Specialist | Phase 5 | 0.5 | SCORM/xAPI, Articulate |
| Test Lead | Phase 6 | 0.5 | SVV test methodology |
| AI/ML Engineer | Phases 3-6 | 0.3 | EU AI, NLP, Norwegian language models |

### 8.2 Key Personnel Requirements (SSA-L)

- **Project Manager** is mandatory key personnel - CV required, subject to SVV approval including possible interview and case study
- Key personnel replacement: minimum 10-20 business days handover, at vendor's cost
- SPOC continues throughout contract period (post-implementation)
- No invoicing for new personnel until handover complete

### 8.3 SVV Dependencies

Dedicated SVV team members required for:
- IAM/ForgeRock team (L04 integration)
- ServiceNow team (L05 integration)
- PowerBI/HR analytics team (L06 integration)
- Kursoppslag team (API mapping)
- Divisional stakeholders (L02 needs analysis)
- Storyboard AS cooperation (L08 migration)

---

## 9. PRICING STRATEGY

### 9.1 Pricing Framework

| Category | Approach |
|---|---|
| **Target range** | 22-28 MNOK over 6 years (mid-to-lower within 20-35M envelope) |
| **License model** | Site license preferred (predictable cost, no per-user tracking) |
| **Fixed-price deliverables** | L01-L04, L07, L09, L10 as fixed price |
| **T&M deliverables** | L05 (100h), L06 (100h), L08 (150h), L12 (200h) |
| **Annual SaaS** | Years 2027-2032 with price escalation clause |
| **Options** | L21-L24 priced separately (not in evaluation) |

### 9.2 Pricing Strategy Rationale

At 80% quality / 20% price weighting, every NOK invested in quality improvement outperforms the same NOK in price reduction by 4:1. Our strategy:
- **Do not compete primarily on price** -- quality scores dominate
- Price at credible mid-range to avoid "too cheap" perception
- Front-load value, not cost -- emphasize TCO savings from consolidation
- Site license removes user-counting overhead and enables KKP expansion

### 9.3 Cash Flow Considerations

- SSA-L: no invoicing for deliverables until acceptance test passed (L10)
- T&M work (L05, L06, L08) invoiced per agreed schedule
- Annual licensing starts from Leveringsdag (delivery day)
- Budget reservation clause means SVV can cancel if budget not approved

---

## 10. BID MANAGEMENT

### 10.1 Immediate Actions (Next 16 Days to Qualification)

| Priority | Action | Deadline | Owner |
|---|---|---|---|
| P0 | Go/No-Go decision | TODAY (22.03.2026) | Steering Group |
| P0 | Submit clarification questions | 23.03.2026 | Bid Manager |
| P1 | Complete ESPD form | 24.03.2026 | Legal/Compliance |
| P1 | Finalize reference list (Vedlegg 2) | 25.03.2026 | Sales Lead |
| P1 | Prepare capacity documentation (Vedlegg 3) | 27.03.2026 | HR/Resource Mgr |
| P2 | Partner commitment declarations (Vedlegg 4) | 28.03.2026 | Partners |
| P2 | Draft qualification package complete | 30.03.2026 | Bid Manager |
| P3 | Internal review gate 1 | 01.04.2026 | Review Board |
| P3 | Final QA and upload test | 03-04.04.2026 | Bid Manager |
| P0 | **SUBMIT QUALIFICATION** | **07.04.2026 by 12:00** | Bid Manager |

### 10.2 Document Checklist Summary

**Qualification (9 documents)**: ESPD, qualification letter, references, capacity, commitment declarations, tax certificates, company registration, financial statements
**Bid 1 (22 documents)**: Bid letter, functional requirements, non-functional requirements, Bilag 2-7/9, CVs, pricing, DPA, architecture diagrams, redacted version
**Demonstration (5 items)**: Demo environment, POC scripts, test users, backup systems, handouts

### 10.3 Quality Assurance Gates

| Gate | Timing | Review Focus |
|---|---|---|
| Gate 0 | TODAY | Go/No-Go decision |
| Gate 1 | 01.04.2026 | Qualification package review |
| Gate 2 | 25.04.2026 | Complete bid draft review |
| Gate 3 | 04.05.2026 | Final bid QA before submission |
| Gate 4 | 09.05.2026 | Demo rehearsal sign-off |
| Gate 5 | 06.06.2026 | Bid 2 review |
| Gate 6 | 29.06.2026 | Final bid review |

---

## 11. RISK REGISTER (TOP 10)

| Rank | ID | Risk | Score | Mitigation |
|---|---|---|---|---|
| 1 | T-01 | Integration complexity (6+ systems) | CRITICAL | Sequential integration priority, contract testing, dedicated architect |
| 2 | L-01 | Timeline pressure (go-live 01.01.2027 too tight) | CRITICAL | MVP approach for Q1 2027, full scope Q2 2027 |
| 3 | T-03 | Kilden migration (297 courses, unknown formats) | HIGH | Pilot migration of 20-30 courses early, budget 250h realistic |
| 4 | L-03 | Dependency on SVV internal teams | HIGH | Dedicated SVV resource commitment in contract, stagger integrations |
| 5 | K-01 | Budget reservation clause (SVV can cancel) | HIGH | No-cost mitigation possible; manage internal investment carefully |
| 6 | T-02 | SCIM/ForgeRock reliability and edge cases | HIGH | Detailed SCIM mapping workshop, realistic user volume testing |
| 7 | C-01 | GDPR/sub-processor management | HIGH | Pre-built sub-processor list, DPA template alignment, EU-only vendors |
| 8 | L-04 | Storyboard AS cooperation for migration | HIGH | Early engagement, contractual cooperation clause, parallel export path |
| 9 | K-03 | SSA-L penalty clauses (0.15%/day) | HIGH | Realistic milestone commitments, buffer in timeline |
| 10 | O-01 | 6-year SLA sustainability | HIGH | Established SaaS vendor with long track record, contractual escalation paths |

**Critical recommendation**: Present an MVP/incremental delivery approach. A hard go-live deadline of 01.01.2027 creates unacceptable risk for integration quality and data integrity.

---

## 12. COMPETITIVE POSITIONING

### 12.1 Selection Criteria Strategy (Pre-Qualification)

| Criterion | Weight | Our Approach |
|---|---|---|
| Experience | 65% | 3-5 comparable LMS SaaS deployments for large organizations, preferably Nordic/public sector |
| Capacity | 35% | Named team with relevant CVs, partner ecosystem, financial stability |

### 12.2 Award Criteria Strategy (Bid Evaluation)

| Criterion | Weight | Our Scoring Strategy |
|---|---|---|
| Functional requirements | 40% | Map every requirement to specific platform capability with screenshots/examples |
| Non-functional requirements | 16% | Technical precision for IT architect evaluators (Ringstad, Kowalski) |
| Usability | 10% | Win in the 80-min demonstration -- visceral impressions override written responses |
| Implementation | 10% | Phased methodology, division-specific needs analysis, change management |
| Business principles | 4% | Accept SSA-L with minimal avvik, competitive SLA |
| Price | 20% | 22-28 MNOK range, site license, credible mid-range |

### 12.3 Ghost Competitor Analysis

| Competitor | Likely Platform | Strength | Counter |
|---|---|---|---|
| Storyboard AS | Kilden/Cursum | Incumbent knowledge | Limited enterprise scale, contract ending |
| Cornerstone partner | Galaxy | Enterprise features | We may bid this too -- focus on implementation quality |
| Docebo partner | Docebo | AI excellence | Norwegian language confirmation needed |
| Sopra Steria | Various | SVV relationship (ServiceNow) | Not LMS specialists |
| Dossier/Apropos | KKP platform | External portal experience | Niche, limited enterprise LMS |

---

## 13. COMPLIANCE CHECKLIST

### 13.1 Absolute Requirements (Must Pass)

| Category | Key Requirements | Status |
|---|---|---|
| IAM | MFA for admin (IAM-07), MFA for external (IAM-09), limited login attempts (IAM-10) | Platform supports |
| IAM | Centralized user admin (IAM-18), role-based access (IAM-20) | Platform supports |
| Data | EU/EEA storage (PEO-25), encrypted storage/transport (POD-04/05) | EU data centers confirmed |
| Data | Backup to 2 locations (POD-08), tested recovery (POD-09) | Standard SaaS practice |
| Data | No unauthorized data sharing (POD-03), DDoS protection (NET-13) | Platform supports |
| Privacy | DPA required (PEO-13), GDPR compliance (PEO-14), known sub-processors (PEO-15) | DPA template available |
| Security | End-to-end data flow (APP-08), data ownership (EI-01/02/03) | Contractually confirmed |
| Archive | Document archiving compliance (05.019) | Requires Mime integration (option) or built-in |

### 13.2 Norwegian Regulatory Compliance

| Regulation | Requirement | Approach |
|---|---|---|
| Personopplysningsloven/GDPR | EU/EEA data processing, DPA, sub-processor control | Platform EU hosting, DPA pre-filled |
| Arkivloven (2026) / Noark-5 | Competency certificates are archival-mandatory | Option L24 (Mime) or built-in certificate archiving |
| Likestillings- og diskrimineringsloven | WCAG 2.1 AA, universal design | Platform responsive, WCAG compliant |
| ID-porten | OIDC-only since Jan 2026 (SAML deprecated) | Platform supports OIDC, brokered via ForgeRock |

---

## 14. SALES VALUE PROPOSITION

**Core positioning**: We deliver a unified learning and competency platform that consolidates SVV's fragmented systems into a single, future-oriented solution -- enabling data-driven competency management, supporting the 70/20/10 learning model at scale, and ensuring the right competence at the right time.

**Key messages per stakeholder**:
- **HR Leadership (Demarteau)**: "Your 70/20/10 framework becomes operationally real, not just aspirational"
- **Project Leader (Nordin)**: "Proven SaaS with pre-built integrations -- reducing implementation risk"
- **IT Architecture (Ringstad/Kowalski)**: "API-first, EU/EEA, SCIM, zero-trust -- fits your landscape"
- **Procurement (Lidi)**: "Transparent SSA-L pricing, proven references, protected investment"

**ROI drivers**:
- System consolidation: Kilden + KKP + Excel tracking -> one platform
- Admin efficiency: 400 leaders regain hours from manual Excel tracking
- Compliance automation: Zero-gap mandatory training for safety-critical roles
- Strategic analytics: First-ever organization-wide competency gap visibility

---

## 15. DOMAIN CONTEXT

### 15.1 Key Domain Findings

- **SSA-L penalties**: 0.15%/day of first 6 months' compensation for delays
- **ID-porten**: OIDC-only since January 2026 (SAML deprecated) -- critical for external user auth
- **Arkivloven 2026**: New archive law with Noark-5 standard -- competency certificates mandatory
- **SVV ServiceNow**: 225 MNOK/15-year deal with Sopra Steria (Flyt-prosjektet) -- deep ServiceNow investment
- **Similar procurements**: KS Laering won by Task/Valamis (2024), Sikt/UH sector won by Canvas/Instructure (2025), Miljoedirektoratet won by Storyboard AS (2025)
- **Norwegian government cloud direction**: EU/EEA hosting preferred, growing sovereignty requirements

### 15.2 Norwegian Public Sector LMS Landscape

Most Norwegian government agencies use a mix of commercial LMS platforms and DFO's HR infrastructure. The market is fragmented with no dominant player. ForgeRock IAM integration experience is a genuine differentiator -- few LMS vendors have proven Norwegian government IAM integration.

---

## 16. OPEN ITEMS AND DECISIONS NEEDED

| # | Item | Decision Needed | By When |
|---|---|---|---|
| 1 | Go/No-Go on this bid | Steering Group approval | TODAY |
| 2 | Platform selection (Cornerstone vs Docebo) | Vendor alignment and pricing | Before bid writing |
| 3 | Partner/sub-contractor identification | Who delivers implementation? | Before qualification |
| 4 | Reference cases | Confirm 3-5 comparable references | 25.03.2026 |
| 5 | Key personnel | Confirm project manager + technical leads | 25.03.2026 |
| 6 | L08 migration hours | Bid 150h (tender) or realistic 250h? | Before pricing |
| 7 | KKP option (L21) scope | Include or exclude from primary bid? | Before pricing |
| 8 | Docebo Norwegian confirmation | Vendor verification needed | Before platform decision |
| 9 | Cornerstone ForgeRock reference | Has anyone done ForgeRock/Cornerstone? | Before bid |
| 10 | Pricing model | Site license vs per-user? | Before pricing |

---

*This document synthesizes inputs from: System Architect, UX Designer, AI Specialist, LMS Expert, Bid Strategist, Domain Researcher, Sales Agent, Bid Manager, and Risk Assessor.*

*Version 1.0 -- Submitted for review by Risk Assessor and Bid Manager.*
