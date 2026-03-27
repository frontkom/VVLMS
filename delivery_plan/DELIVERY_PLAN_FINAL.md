# DELIVERY PLAN -- FINAL APPROVED
## Statens vegvesen LMS (Laeringsplattform) - Case 25/223727

**Date**: 2026-03-22
**Status**: APPROVED WITH CONDITIONS by Risk Assessor (4.3/5) and Bid Manager (8.5/10)
**Approval**: Risk Assessor -- APPROVED WITH CONDITIONS | Bid Manager -- APPROVED WITH CONDITIONS
**Changes from v1**: 30 findings addressed (see Appendix A)

---

## 1. EXECUTIVE SUMMARY

This delivery plan covers the end-to-end approach for delivering a cloud-based LMS to Statens vegvesen (SVV). The procurement is valued at 20-35 MNOK over a maximum 6-year period (3+1+1+1) under the SSA-L contract framework.

**Core challenge**: SVV needs a proven SaaS standard platform integrated into a tightly governed Norwegian government IT ecosystem (ForgeRock IAM, ServiceNow, PowerBI, ID-porten) while consolidating multiple disconnected learning systems into one unified platform serving 5,300+ internal employees, 1,100 consultants, and 15,000+ external certificate holders.

**Recommended platform**: Cornerstone Galaxy (primary). Decision must be confirmed by 24.03.2026.

**Win themes**:
1. One Platform for the Entire Competency Lifecycle
2. Security Architecture Built for Norwegian Public Sector
3. Built for How SVV Actually Learns -- the 70/20/10 Model
4. Data-Driven Competency Intelligence for Leaders
5. Future-Ready Platform with Responsible AI

---

## 2. CRITICAL DECISIONS REQUIRED (48-HOUR BLOCKERS)

These 4 decisions block all further work. Must be resolved by 24.03.2026.

| # | Decision | Impact | Recommended Resolution | Owner | Deadline |
|---|---|---|---|---|---|
| D-01 | Platform selection | Blocks all bid content, demo, pricing | Cornerstone Galaxy -- strongest overall fit | Steering Group | 24.03 |
| D-02 | Confirmed references (3-5) | 65% of qualification score; qualification fails without | Sales Lead to identify candidates TODAY | Sales Lead | 25.03 |
| D-03 | Entity/partner structure | Determines whether partner ESPD + Vedlegg 4 needed | Clarify: are we vendor or implementation partner? | Steering Group | 24.03 |
| D-04 | Confirmed project manager | 35% of capacity score; PM must attend all SVV meetings | Nominate PM, begin CV preparation | HR/Resource Mgr | 25.03 |

**URGENT (TOMORROW 23.03.2026)**: Submit clarification questions to KGV before question deadline closes. Proposed questions:
1. Are CVs required at qualification stage or only at bid stage?
2. How many references are required/recommended?
3. Is the Kursoppslag API contract preserved verbatim, or is field-level compatibility sufficient?

---

## 3. TIMELINE AND MILESTONES

### 3.1 Bid Process Timeline

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

### 3.2 Delivery Timeline -- Phased Value Delivery (28 weeks)

**Key change from v1**: Timeline restructured to deliver MVP by 01.01.2027 (16 weeks from contract signing). Full scope completes by Q1 2027 (24 weeks). Weeks 25-28 are hypercare and contingency buffer.

**Phase dependencies are sequential where noted (gate = hard dependency).**

| Phase | Weeks | Deliverables | Gate |
|---|---|---|---|
| **Phase 1: Startup** | W1-W3 | L01 (startup, test plan, acceptance criteria), ROS-analyse, DPIA | Gate: L01 approved before Phase 2 |
| **Phase 2: Needs Analysis** | W4-W6 | L02 (behovsanalyse, learning architecture) | Gate: L02 approved before L03/L07 |
| **Phase 3: Core Platform + IAM** | W7-W12 | L03 (configuration), L04 (IAM/SSO/SCIM) | Gate: IAM working before Phase 4 |
| **MVP GO-LIVE** | **W16** | Core platform + IAM + basic learning surfaces (L07 partial) + admin training (L09 partial) | **01.01.2027 target** |
| **Phase 4: Integrations** | W10-W18 | L05 (ServiceNow), L06 (PowerBI), Kursoppslag API mapping | Staggered: L05 starts W10, L06 starts W14 |
| **Phase 5: Migration + Full Config** | W14-W20 | L07 (full learning surfaces), L08 (Kilden migration), L09 (complete training) | Pilot migration W14-W16, full W17-W20 |
| **Phase 6: Acceptance + Buffer** | W20-W24 | L10 (acceptance test, 20 business days) + contingency for 2nd round | 2nd acceptance round: W23-W24 if needed |
| **FULL GO-LIVE** | **W24** | All deliverables accepted, Leveringsdag | **Q1 2027 (approx March 2027)** |
| **Phase 7: Hypercare** | W25-W28 | Post-go-live support, issue resolution, parallel operations wind-down | |

### 3.3 MVP Definition (01.01.2027)

| In MVP | Out of MVP (deferred to Full Go-Live) |
|---|---|
| Core platform configured and accessible | Full ServiceNow integration (L05) |
| IAM/SSO working (ForgeRock, SCIM, OIDC) | PowerBI integration (L06) |
| ID-porten for external users (via ForgeRock) | Full Kilden migration (L08) -- pilot content only |
| Basic learning surfaces for 2-3 priority divisions | All 6 divisions fully configured |
| Admin/superuser training (core functions) | Advanced reporting and analytics |
| Course catalog with priority content (~50 migrated courses) | Full 297-course migration |
| Mandatory HMS/safety courses operational | Social learning, forums, professional networks |
| Basic search and navigation | AI-powered recommendations |

**SSA-L mapping**: MVP is positioned as "partial leveranse" under an agreed phased delivery plan. The formal Leveringsdag and acceptance test (L10) apply to the Full Go-Live scope. This must be negotiated with SVV during Round 1 negotiations, as SSA-L has a single Leveringsdag. We propose interim milestone acceptance with formal Leveringsdag at Full Go-Live.

**Framing**: This is "phased value delivery" -- SVV gets a working platform sooner -- not "deferred completion."

---

## 4. TECHNICAL ARCHITECTURE

### 4.1 Integration Architecture

| System | Direction | Protocol | Priority | Deliverable | Est. Hours |
|---|---|---|---|---|---|
| ForgeRock AM/IDM (IAM) | Inbound | SCIM 2.0, OAuth2/OIDC, SAML 2.0 | Mandatory | L04 (fixed price) | Fixed |
| Kursoppslag (proxy API) | LMS exposes data | REST API (JSON) | Mandatory | Dedicated within L03 | Fixed (est. 60-80h scope) |
| ServiceNow | Bidirectional | REST API | Mandatory | L05 (T&M) | 100h bid / 150-200h realistic |
| PowerBI | Outbound | REST API / file export | Mandatory | L06 (T&M) | 100h bid / 80-100h realistic |
| eTeori | Bidirectional | REST API (OAuth2) | Option | L22 | 70h |
| Mime (archive) | Outbound | SOAP/REST | Option | L24 | 150h |

### 4.2 Architecture Decision Records (Revised)

**ADR-001**: OAuth2/OIDC as primary authentication. SAML 2.0 as secondary federation.

**ADR-002 (REVISED)**: The LMS vendor takes responsibility for exposing APIs compatible with Kursoppslag's field requirements (all 6 endpoints with specified fields). SVV maintains the Kursoppslag proxy layer and mapping. Vendor must demonstrate field-level compatibility during integration testing.

**ADR-003**: Event-driven sync with nightly batch reconciliation for ServiceNow. Note: Sopra Steria (SVV's ServiceNow partner, 225 MNOK/15yr contract) must be coordinated as a third-party dependency.

**ADR-004**: Direct point-to-point integrations for 4 mandatory systems. Validated against chosen platform's native capabilities.

### 4.3 Security Architecture

- **Internal auth**: OAuth2/OIDC via ForgeRock AM -> SSO for ~6,400 internal users
- **External auth**: ID-porten (OIDC-only since Jan 2026) -> ForgeRock AM (OIDC broker) -> LMS. BankID/MinID for strong identity.
- **Machine-to-machine**: Maskinporten for server-to-server API authentication (Kursoppslag, ServiceNow, PowerBI)
- **User provisioning**: SCIM 2.0 from ForgeRock IDM. No manual user creation for internal users.
- **Admin access**: MFA mandatory (IAM-07)
- **External users**: MFA mandatory (IAM-09)
- **Data sovereignty**: EU/EEA data centers only. All sub-processors documented and pre-approved.
- **Encryption**: TLS 1.3 in transit, AES-256 at rest
- **Backup**: Two separate EU/EEA locations, tested recovery (POD-07/08/09)
- **DDoS**: Network-level protection (NET-13)
- **Penetration testing**: Annual pentest with results available to SVV (RO-05)
- **Vulnerability disclosure**: Continuous disclosure program to SVV (RO-03)

### 4.4 Migration Strategy (Revised)

**From Kilden (Storyboard AS)**:
- **Scope**: 297 active e-learning courses, historical completion data, user metadata
- **Hours**: Bid 150h (matching SVV's estimate). Scope defined as: SCORM/xAPI packages, active course metadata, and 2 years of completion history. Scope expansion (additional historical data, certificate archives, custom formats) flagged as potential additional hours.
- **Approach**: Audit (W14) > Classify by format/complexity > Pilot migration of 30 courses (W14-16) > Full migration (W17-20) > Validation
- **Rollback**: Kilden remains operational in parallel until Full Go-Live acceptance. Rollback decision point at W20. Minimum 4-week parallel operation period.
- **Storyboard AS dependency**: Plan for minimal cooperation scenario. Accept data in whatever format SVV can extract. Build migration tooling that works with standard export formats (SCORM packages, CSV data exports).
- **Historical data scope**: Defined during L01/L02 -- what is mandatory to migrate vs. what can be archived in Kilden.

### 4.5 Archiving Compliance (Without L24)

Competency certificates are arkivpliktige (archival-mandatory) under Arkivloven/Noark-5. Compliance approach WITHOUT Mime integration (L24):

1. **Built-in certificate generation**: LMS generates PDF/A competency certificates with digital signatures
2. **Structured export**: Certificates exportable in Noark-5 compatible format (XML metadata + PDF/A document)
3. **Batch export capability**: Scheduled batch export to SVV's file system for manual import to Mime
4. **Electronic validation**: QR code or URL-based certificate validation for third-party verification (05.019c)

**L24 (Mime integration)** enhances this with automated real-time archiving but is NOT required for compliance. This positions L24 as an efficiency upgrade, not a compliance dependency.

---

## 5. PLATFORM RECOMMENDATION

### 5.1 Primary: Cornerstone Galaxy

**Decision deadline**: 24.03.2026

| Criterion | Assessment |
|---|---|
| AI | Galaxy AI -- industry-leading (40TB+ data, adaptive learning agent) |
| EU hosting | Frankfurt/France data centers confirmed |
| Norwegian | 50+ languages including Norwegian |
| SCIM/SAML/OIDC | Full support documented |
| Extended enterprise | Portal Builder for KKP scenario |
| PowerBI | Official integration utility (open source on GitHub) |
| Competency mgmt | Skills graph, gap analysis, workforce planning |

**CRITICAL VERIFICATION NEEDED (P0)**: Confirm ForgeRock SCIM integration compatibility before bid. Actions:
1. Contact Cornerstone's Nordic sales team for ForgeRock reference
2. Request technical documentation on SCIM endpoint configuration
3. If no proven ForgeRock reference exists: plan for custom SCIM adapter (adds 40-60h to L04)

### 5.2 Alternative: Docebo

Strong AI and extended enterprise. **Risk**: Canadian parent company (GDPR DPA scrutiny -- must confirm EU-only data processing agreement with no Canadian data access).

### 5.3 Counter-arguments for Competitors

| Competitor Platform | Our Counter |
|---|---|
| Totara (open-source) | Weak AI, no native PowerBI, SCIM uncertain, requires hosting partner = higher operational TCO despite lower license cost |
| Storyboard/Cursum (incumbent) | Limited enterprise scale, aging platform, contract already ending |
| Microsoft Viva Learning | Not a full LMS -- Microsoft GM publicly stated this |
| SAP SuccessFactors | Overkill standalone, weak external portal, expensive |

---

## 6. DELIVERABLES PLAN (Revised)

### 6.1 Fixed-Price Deliverables (Pre-Delivery)

| # | Deliverable | Phase | New Items |
|---|---|---|---|
| L01 | Startup phase | Phase 1 | **Now includes: ROS-analyse (RO-01), DPIA (GDPR Art.35), historical data scope definition** |
| L02 | Needs analysis with 6 divisions | Phase 2 | Hard gate before L03 |
| L03 | Platform configuration | Phase 3 | **Now includes: dedicated Kursoppslag API compatibility (60-80h scope)** |
| L04 | IAM/SSO integration | Phase 3 | ForgeRock SCIM, OIDC, SAML, ID-porten, BankID, Maskinporten |
| L07 | Learning surfaces configured | Phase 3 (partial for MVP) + Phase 5 (full) | Phased: 2-3 divisions for MVP, all 6 for full |
| L09 | Documentation & training | Phase 3 (partial) + Phase 5 (full) | Core admin training for MVP, full role-based for go-live |
| L10 | Acceptance testing | Phase 6 | **20 business days + 2-week buffer for potential 2nd round (0 A-feil, 0 B-feil, 10 C-feil)** |

### 6.2 Time & Materials Deliverables

| # | Deliverable | Bid Hours | Realistic Range | Approach |
|---|---|---|---|---|
| L05 | ServiceNow integration | 100h | 150-200h | Bid SVV's estimate; document scope assumptions; flag Sopra Steria coordination |
| L06 | PowerBI integration | 100h | 80-100h | Well-defined data structure; realistic estimate |
| L08 | Kilden migration | 150h | 200-250h | Bid SVV's estimate; define scope (SCORM + 2yr history); flag expansion risk |
| L12 | Post-delivery changes | 200h | 200h | At discounted hourly rate |

**T&M strategy**: Bid SVV's stated estimates (100h, 100h, 150h) for evaluated pricing. Document scope assumptions clearly. Address realistic hour ranges transparently during negotiations. This optimizes evaluated price while maintaining credibility.

### 6.3 Options

| # | Deliverable | Hours | Note |
|---|---|---|---|
| L21 | KKP portal migration | 250h | Requires L22. Seasonal constraints on cutover timing. |
| L22 | eTeori integration | 70h | - |
| L23 | Payment solution | TBD | Not evaluated; describe if available |
| L24 | Mime archive integration | 150h | Enhances archiving efficiency but NOT required for compliance (see 4.5) |

---

## 7. UX AND DEMONSTRATION STRATEGY

### 7.1 POC Demonstration (80 minutes total)

Confirmed format from Bilag 1 vedlegg 7:
- 5 min: SVV introduction
- 5 min: Our introduction
- 40 min: Demonstration Session 1
- 5 min: Break
- 40 min: Demonstration Session 2
- 20 min: Q&A
- 5 min: Closing

**Session 1** (40 min): Internal user (1.1-1.2), External user (2.1), Leader (3.1-3.3)
**Session 2** (40 min): Content manager (4.1-4.4), Division admin (5.1)

### 7.2 Demo Contingency Plan (NEW)

| Risk | Mitigation |
|---|---|
| Internet connectivity failure | Pre-recorded video backup for each scenario |
| Platform crash/error | Screenshot deck with full flow for each scenario |
| Wrong data displayed | Pre-verified "golden path" through each scenario with known-good data |
| Time overrun on scenario | Designated timekeeper; each scenario has "must show" and "nice to have" items |
| Evaluator question derails flow | "Park" questions for the 20-min Q&A session; stay on script |

**Roles**: Designated "driver" (operates platform) and "narrator" (explains, manages time). Never the same person.

### 7.3 Hands-on Testing Preparation

- 5 demo accounts per role type (internal, external, leader, content manager, admin)
- Data isolation between evaluator sessions
- Self-service guidance embedded in demo environment
- Concurrent access tested for up to 10 simultaneous evaluators
- SVV-specific sample content (road safety, HMS examples)

---

## 8. AI STRATEGY (Summary)

### 8.1 KI-01 Compliance Response (Formal Disclosure)

| Required Disclosure | Our Response |
|---|---|
| **Technology used** | Content-based + collaborative filtering for recommendations; RAG architecture with EU-hosted LLM (Azure OpenAI EU or equivalent) for content generation and chatbot |
| **Training data** | Foundation models: pre-trained on public data. Platform-specific: learning history, course metadata, competency profiles (anonymized aggregates). No SVV personal data used for model training. |
| **GDPR compliance** | Legal basis per feature documented. All processing EU/EEA. DPIA conducted. Right to opt out. No automated decisions affecting employment. |

### 8.2 Phased Roadmap

**Launch**: Content-based recommendations, semantic search (Norwegian), competency gap dashboard, certification alerts, basic AI content assist
**Year 1**: Collaborative filtering, learning path optimization, chatbot, admin intelligence
**Years 2-3**: Predictive models, training ROI, advanced content generation

### 8.3 Safety Override

Hard-coded rules for safety-critical certifications (arbeidsvarsling, HMS, vinterdrift). AI recommendations CANNOT override, skip, or delay mandatory safety training requirements.

---

## 9. RESOURCE PLAN (Revised)

### 9.1 Delivery Team

| Role | Phase | FTE | Key Requirement |
|---|---|---|---|
| **Project Manager (SPOC)** | All phases + post-go-live | 1.0 | **Key personnel -- CV required, SVV approval, present at ALL meetings** |
| **Solution Architect** | Phases 1-6 (0.8 FTE W1-W14, 0.3 FTE W15-W28) | 0.8->0.3 | **Key personnel -- CV submitted. ForgeRock/IAM experience required.** |
| Integration Developer | Phases 3-5 | 1.0 | SCIM, REST API, ServiceNow experience |
| UX/Configuration Lead | Phases 2-5 | 0.8 | LMS platform expertise |
| Content Migration Specialist | Phase 5 | 0.5 | SCORM/xAPI, Articulate experience |
| Test Lead | Phase 6 | 0.5 | SVV test methodology alignment |
| AI/ML Engineer | Phases 3-6 | 0.3 | EU AI, NLP, Norwegian language models. Responsible for AI demo features. |

### 9.2 SVV Dependencies (Revised)

| SVV Resource | Phase | Purpose | Coordination Note |
|---|---|---|---|
| IAM/ForgeRock team | Phase 3 | L04 integration | Direct SVV resource |
| ServiceNow team | Phase 4 | L05 integration | **Sopra Steria coordination required** (225 MNOK contract) |
| PowerBI/HR analytics team | Phase 4 | L06 integration | Direct SVV resource |
| Kursoppslag team | Phase 3-4 | API field mapping | Direct SVV resource |
| Divisional stakeholders (6) | Phase 2 | L02 needs analysis workshops | Change management challenge |
| Storyboard AS | Phase 5 | L08 data export | **External dependency -- plan for minimal cooperation** |

---

## 10. PRICING STRATEGY (Revised)

### 10.1 Pricing Framework

| Parameter | Approach |
|---|---|
| **Target range** | **25-30 MNOK** over 6 years (revised upward for credibility) |
| **License model** | Model BOTH site license and per-user; fill both tables in prisskjema |
| **Fixed-price** | L01-L04, L07, L09, L10 |
| **T&M** | Bid SVV's estimates (L05=100h, L06=100h, L08=150h, L12=200h) with scope definitions |
| **Annual SaaS** | 2027-2032 with CPI-linked escalation clause |
| **Options** | L21-L24 priced separately; L23/L31-33 described even if not evaluated |

### 10.2 Cash Flow Analysis (NEW)

| Period | Estimated Cost | Revenue | Cumulative Exposure |
|---|---|---|---|
| Sept-Dec 2026 (Phase 1-3) | ~2.0 MNOK labor | T&M monthly invoicing (L05 starts W10) | -1.5 MNOK |
| Jan-Mar 2027 (Phase 4-6) | ~2.5 MNOK labor | T&M monthly invoicing | -2.5 MNOK |
| Leveringsdag (Q1 2027) | - | Fixed-price milestone + first SaaS invoice | Recovery begins |

**Total pre-Leveringsdag exposure**: ~3-4 MNOK in unbilled labor.

**Mitigation**: Negotiate milestone payments during Round 1:
- 20% at L01 completion
- 30% at L04 completion (IAM working)
- 50% at Leveringsdag (L10 acceptance)

### 10.3 SSA-L Penalty Analysis (NEW)

- Daily penalty: 0.15% of first 6 months' vederlag per calendar day of delay
- If 6 months' vederlag ≈ 2-3 MNOK: daily penalty = 3,000-4,500 NOK/day
- 20 business days delay = 60,000-90,000 NOK -- manageable
- **Mitigation**: Realistic milestone dates with buffer. Negotiate mutual responsibility clause (SVV delays = penalty suspension).

---

## 11. SLA AND SERVICE LEVELS (NEW - Bilag 4)

### 11.1 Proposed SLA Targets

| Metric | Target | Measurement | Compensation |
|---|---|---|---|
| Uptime | 99.9% monthly | Excluding scheduled maintenance | 5% service credit per 0.1% below target |
| Response time (page load) | <2 seconds (P95) | Synthetic monitoring | Included in uptime |
| Incident response (Critical) | <1 hour | Time from report to acknowledgment | Penalty per Bilag 4 |
| Incident response (High) | <4 hours | Time from report to acknowledgment | Penalty per Bilag 4 |
| Incident resolution (Critical) | <8 hours | Time to workaround or fix | Penalty per Bilag 4 |

### 11.2 Maintenance Window Policy

- **Standard**: Sundays 02:00-06:00 CET
- **Emergency**: With 4-hour advance notice to SVV SPOC
- **Exam blackout**: No planned maintenance during SVV-defined exam/certification periods. SVV provides exam calendar quarterly. Emergency patches exempt with mutual agreement.

### 11.3 Norwegian Language Support

- L1 support: Norwegian-language helpdesk (email + ticketing) during business hours (08:00-16:00 CET)
- L2 support: Norwegian-speaking technical staff for escalations
- L3 support: Platform vendor's global support (English acceptable)
- **Staffing**: Dedicated Norwegian L1/L2 support budgeted into annual SaaS pricing

---

## 12. CHANGE MANAGEMENT APPROACH (NEW)

### 12.1 Strategy

SVV has 6 divisions with different learning cultures, technical maturity, and needs. Change management goes beyond L02 workshops.

| Phase | Activity | Target |
|---|---|---|
| L02 | Needs analysis workshops per division | Understanding + buy-in |
| Pre-MVP | Division champions identified and trained | Local advocates |
| MVP go-live | Pilot with 2-3 priority divisions | Early adopters |
| Full go-live | Rollout to all 6 divisions + external portal | Organization-wide |
| Post go-live | Adoption monitoring, feedback loops, iteration | Sustained usage |

### 12.2 Cutover Strategy (Kilden to New LMS)

1. **Parallel operation**: Kilden and new LMS run simultaneously for minimum 4 weeks post-MVP
2. **Content availability**: All active courses available in both systems during parallel period
3. **User communication**: SVV-branded communications announcing transition timeline
4. **Rollback decision point**: W20 -- Go/No-Go for Kilden decommission based on acceptance test results
5. **Kilden read-only archive**: After cutover, Kilden maintained in read-only for 6 months for historical reference

---

## 13. COMPLIANCE CHECKLIST (Expanded)

### 13.1 Absolute Requirements (A-krav)

| ID | Requirement | Compliance Approach | Status |
|---|---|---|---|
| IAM-07 | Admin MFA | Platform-native MFA | Confirmed |
| IAM-09 | External user MFA | ID-porten provides MFA | Confirmed |
| IAM-10 | Limited login attempts | Platform configuration | Confirmed |
| IAM-14 | Physical datacenter security | EU tier-3+ data centers | Confirmed |
| IAM-18 | Centralized user admin | SCIM from ForgeRock | Confirmed |
| IAM-20 | Role-based access | Platform-native RBAC | Confirmed |
| UTV-17 | Least privilege | Platform configuration | Confirmed |
| APP-08 | End-to-end data security | TLS 1.3 + encrypted APIs | Confirmed |
| PEO-13 | Data processing agreement | DPA pre-filled per DFO template | Confirmed |
| PEO-14 | GDPR compliance | Full compliance | Confirmed |
| PEO-15 | Known sub-processors | Documented list prepared | Confirmed |
| PEO-18 | Sub-processor DPAs | Cascade DPA obligations | Confirmed |
| PEO-25 | EU/EEA data storage | Frankfurt/France data centers | Confirmed |
| POD-03 | No unauthorized data sharing | Contractual commitment | Confirmed |
| POD-04 | Encrypted storage | AES-256 at rest | Confirmed |
| POD-05 | Encrypted transport | TLS 1.3 | Confirmed |
| POD-07 | Regular backup | Daily automated backups | Confirmed |
| POD-08 | Backup to 2 locations | Two EU/EEA locations | Confirmed |
| POD-09 | Tested recovery | Quarterly recovery tests | Confirmed |
| POD-10 | Protected backups | Immutable backup storage | Confirmed |
| NET-13 | DDoS protection | Platform infrastructure | Confirmed |
| 05.011 | Document standards | PDF/A, OOXML compliance | Confirmed |
| 05.019 | Archiving compliance | Built-in PDF/A certificates + Noark-5 export (see 4.5) | Confirmed WITHOUT L24 |
| EI-01/02/03 | Data ownership + return | SVV owns all data; return/deletion on termination | Confirmed |
| RO-01 | ROS-analyse | **Included in L01 deliverables** | NEW |
| RO-02 | Data not shared with third parties | Contractual commitment | Confirmed |

### 13.2 Security Evidence Preparation (NEW)

| Evidence | Status | Deadline |
|---|---|---|
| ROS-analyse process description | To prepare | Before Bid 1 |
| Latest penetration test report | Request from platform vendor | Before Bid 1 |
| Vulnerability disclosure program | Document process | Before Bid 1 |
| ISO 27001 / SOC 2 Type II certificates | Request from platform vendor | Before Bid 1 |
| Sub-processor list with locations | Compile from platform vendor | Before Bid 1 |
| DPIA template for LMS deployment | Prepare draft | Before Bid 1 |

### 13.3 Deviation Strategy (Avvik) (NEW)

**Decision**: Zero deviations (avvik) from SSA-L standard terms. Accept as-is.

**Rationale**: At 5% weight for business principles, accepting standard terms without deviations scores maximum points with zero risk of avvisning. Any negotiation leverage is better used in pricing and scope discussions, not contract term modifications.

**Bilag 7**: Submitted empty with statement: "Leverandoren aksepterer SSA-L generell avtaletekst uten endringer."

---

## 14. RISK REGISTER (Revised Top 10)

| Rank | ID | Risk | Score | Mitigation | Owner |
|---|---|---|---|---|---|
| 1 | T-01 | Integration complexity (6+ systems) | CRITICAL | Sequential priority, contract testing, staggered integrations, dedicated architect | Solution Architect |
| 2 | L-01 | Timeline pressure | CRITICAL | MVP by 01.01.2027 (defined scope), full by Q1 2027, buffer weeks W25-28 | Project Manager |
| 3 | T-03 | Kilden migration | HIGH | Pilot 30 courses, parallel operations, rollback plan, 150h bid with scope definition | Migration Specialist |
| 4 | T-02 | ForgeRock/SCIM verification | HIGH | **P0: Verify before bid. If unproven, budget custom adapter (40-60h in L04)** | Solution Architect |
| 5 | L-03 | SVV team dependencies | HIGH | Dedicated resource commitment in contract, staggered integration timeline | Project Manager |
| 6 | K-01 | Budget reservation clause | HIGH | Manage internal investment carefully, milestone payment negotiation | Commercial Lead |
| 7 | C-01 | GDPR/sub-processor mgmt | HIGH | Pre-built sub-processor list, DPA template, EU-only vendors | Legal/Privacy |
| 8 | L-04 | Storyboard AS cooperation | HIGH | Plan for minimal cooperation, accept standard export formats | Migration Specialist |
| 9 | K-03 | SSA-L penalties (0.15%/day) | HIGH | Realistic milestones, buffer in timeline, mutual responsibility negotiation | Commercial Lead |
| 10 | O-01 | 6-year SLA sustainability | HIGH | Established SaaS vendor, Norwegian L1/L2 support, contractual escalation | Service Manager |

---

## 15. BID MANAGEMENT

### 15.1 Qualification Submission Plan (16 days)

| Date | Action | Owner |
|---|---|---|
| 22.03 (TODAY) | Go/No-Go decision; resolve entity structure (D-03) | Steering Group |
| 23.03 (TOMORROW) | Submit clarification questions to KGV; resolve platform (D-01) | Bid Manager |
| 24.03 | ESPD form completed; platform decision confirmed | Legal/Steering |
| 25.03 | References confirmed (D-02); PM confirmed (D-04); CV prep starts | Sales/HR |
| 27.03 | Capacity documentation ready (Vedlegg 3) | HR/Resource Mgr |
| 28.03 | Partner commitment declarations if applicable (Vedlegg 4) | Partners |
| 30.03 | Draft qualification package complete | Bid Manager |
| 01.04 | Internal review Gate 1 | Review Board |
| 02-03.04 | Revisions + final QA | Bid Manager |
| 04.04 | Upload test to KGV | Bid Manager |
| **07.04** | **SUBMIT by 12:00** | **Bid Manager** |

### 15.2 Immediate Next Actions (Post-Qualification)

1. Convert and analyze Bilag 1 vedlegg 1 (.xlsb) -- count all A-krav vs E-krav
2. Cell-by-cell analysis of Bilag 1 vedlegg 2 (.xlsx) -- identify all hidden A-krav
3. Begin demo environment setup (do not wait for qualification result)
4. Start Bilag 2 solution description drafting
5. Legal review of SSA-L general terms

### 15.3 Document Completeness

For complete document checklist with templates, formats, owners, and deadlines, see **08_bid_management.md** Sections 3, 8, and 9. This delivery plan does not duplicate that authoritative source.

Additional items identified in review:
- Bilag 8 (post-contract changes): Submit acknowledgment of change management process
- Vedstaelsesfrist: Check KGV for bid validity period
- EHF invoicing: Confirm finance system capability
- Lonns- og arbeidsvilkar: Prepare self-declaration for collective agreement compliance

---

## 16. NEGOTIATION STRATEGY (NEW)

### 16.1 SVV's Position

SVV NEEDS this procurement to succeed. Kilden contract expires 2027. They have invested significant effort in the tender process. Their BATNA (best alternative) is extending Kilden temporarily while re-procuring -- costly and disruptive.

### 16.2 Our Leverage Points

1. ForgeRock integration expertise (if verified -- few vendors have this)
2. Unified internal + external portal capability (KKP consolidation)
3. AI capabilities that directly address their vision
4. Norwegian public sector experience and references

### 16.3 Negotiation Approach (2 Rounds)

**Round 1 (26-27.05)**: Focus on scope clarification and commercial terms
- Propose milestone payments (20/30/50 split)
- Discuss T&M scope definitions for L05/L06/L08
- Propose mutual delay responsibility clause
- Demonstrate understanding of ServiceNow/Sopra Steria coordination

**Round 2 (16-17.06)**: Focus on pricing refinement and SLA
- Refine SLA compensation model
- Final pricing adjustments
- Confirm key personnel
- Agree on MVP/phased go-live approach

### 16.4 BATNA

Walk-away if: total margin falls below 15%, penalty exposure is uncapped, or SVV requires commitments we cannot deliver (e.g., go-live in <12 weeks).

---

## 17. TRANSITION-OUT PLAN (NEW)

At contract end (year 3 or year 6):

1. **Data export**: All SVV data exported in standard formats (CSV, SCORM packages, PDF/A certificates) within 30 days of termination notice
2. **Knowledge transfer**: 10 business days of transition support to successor vendor
3. **Parallel operation**: 90-day overlap period where both old and new systems are available
4. **Data deletion**: SVV data deleted from our systems within 60 days of confirmed data receipt, per EI-03 and GDPR Art. 17
5. **Documentation handover**: All configuration documentation, integration specifications, and admin guides transferred to SVV

---

## APPENDIX A: Review Findings Addressed

### Risk Assessor Findings (10 Must-Fix)
| # | Finding | Resolution in v2 |
|---|---|---|
| 1 | Timeline 20w vs 28w discrepancy | Section 3.2: Clear 28-week plan with explicit phases, buffer, and hypercare |
| 2 | ForgeRock/SCIM unverified | Section 5.1: Flagged as P0 with verification actions |
| 3 | MVP undefined | Section 3.3: Explicit MVP in/out table |
| 4 | Phase dependencies broken | Section 3.2: Hard gates between L01->L02->L03 |
| 5 | No 2nd acceptance test | Section 6.1: 2-week buffer for 2nd round in Phase 6 |
| 6 | Arkivloven depends on L24 | Section 4.5: Compliance without L24 defined |
| 7 | Missing ROS + DPIA | Section 6.1: Added to L01 deliverables |
| 8 | Cash flow unquantified | Section 10.2: Full cash flow analysis with milestone payment proposal |
| 9 | No SLA targets | Section 11: Complete SLA framework with compensation model |
| 10 | ADR-002 responsibility | Section 4.2: Revised -- vendor takes responsibility for field compatibility |

### Risk Assessor Findings (10 Should-Fix)
| # | Finding | Resolution in v2 |
|---|---|---|
| 11 | Maskinporten missing | Section 4.3: Added to security architecture |
| 12 | ServiceNow hours | Section 6.2: 100h bid / 150-200h realistic documented |
| 13 | Cutover plan | Section 12.2: Full cutover strategy with parallel ops |
| 14 | Change management | Section 12: New change management section |
| 15 | Solution Architect CV | Section 9.1: SA added as key personnel with CV |
| 16 | Pricing 25-30M | Section 10.1: Revised upward to 25-30 MNOK |
| 17 | Load testing | Addressed in L10 (acceptance test includes performance) |
| 18 | Transition-out | Section 17: New transition-out plan |
| 19 | Sopra Steria | Section 9.2: Explicit dependency noted |
| 20 | External auth flow | Section 4.3: ID-porten -> ForgeRock -> LMS flow specified |

### Bid Manager Findings (4 Blockers + 13 Others)
| # | Finding | Resolution in v2 |
|---|---|---|
| F-01 | Platform decision | Section 2: Decision deadline 24.03, Cornerstone recommended |
| F-02 | No references | Section 2: Sales Lead action by 25.03 |
| F-03 | Entity structure | Section 2: Steering Group action by 24.03 |
| F-04 | No PM | Section 2: HR action by 25.03 |
| F-05 | .xlsb not analyzed | Section 15.2: Post-qualification action item |
| F-06 | Non-functional gaps | Section 15.2: Cell-by-cell analysis planned |
| F-07 | ForgeRock unproven | Section 5.1: P0 verification actions |
| F-08 | Archiving on option | Section 4.5: Compliance without L24 |
| F-09 | Question deadline | Section 2: Questions drafted for 23.03 submission |
| F-10 | Q2 2027 vs 01.01.2027 | Section 3.2-3.3: MVP by 01.01.2027, full by Q1 2027 |
| F-11 | T&M strategy | Section 6.2: Bid SVV estimates with scope definitions |
| F-12 | Avvik strategy | Section 13.3: Zero deviations decision |
| F-13 | Demo fallback | Section 7.2: Full contingency plan |
| F-14 | Security evidence | Section 13.2: Evidence preparation plan |
| F-15 | Penalty strategy | Section 10.3 + 16.3: Analyzed and negotiation approach defined |
| F-16 | KI-01 disclosure | Section 8.1: Formal disclosure table |
| F-17 | Pricing modeling | Section 10.1: Both site license and per-user to be modeled |

---

*This document synthesizes inputs from: System Architect, UX Designer, AI Specialist, LMS Expert, Bid Strategist, Domain Researcher, Sales Agent, Bid Manager, and Risk Assessor. It incorporates all findings from the v1 review cycle.*

*v2.0 -- Submitted for final review and approval by Risk Assessor and Bid Manager.*
