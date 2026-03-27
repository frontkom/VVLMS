# Statens vegvesen LMS Tender Briefing

## 1. CLIENT OVERVIEW
- **Organization**: Statens vegvesen (Norwegian Public Roads Administration)
- **Under**: Samferdselsdepartementet (Ministry of Transport)
- **Employees**: ~5,000 across Vegdirektoratet + 6 divisions (Trafikant og kjøretøy, Transport og samfunn, Utbygging, Drift og vedlikehold, Fellesfunksjoner og HR, IT)
- **Key figures**: 10,600 km roads, 5,600 bridges, 610 tunnels, 72 traffic stations, 55.3B NOK budget
- **Mission**: Safe and efficient transport for people and goods
- **Strategy pillars**: Accessibility, Traffic safety, Climate/environment, Digitalization, Efficient resource use, Resilience

## 2. PROCUREMENT SUMMARY
- **What**: Cloud-based LMS (Learning Management System) - SaaS standard solution
- **Contract type**: SSA-L (Statens standardavtale - Løsningskontrakt / ongoing service purchase)
- **Duration**: 3 years + 1+1+1 year options = max 6 years
- **Estimated value**: 20-35 MNOK excl. VAT over full period
- **Procedure**: Competition with negotiation (FOA §13-1), two phases
- **Case number**: 25/223727
- **Published**: Doffin and TED

## 3. TIMELINE
| Milestone | Date |
|---|---|
| Published | 25.02.2026 |
| Bidder conference | 12.03.2026 |
| Question deadline (qualification) | 23.03.2026 |
| **Qualification deadline** | **07.04.2026** |
| Qualification result | 10.04.2026 |
| Invitation to bid | 10.04.2026 |
| Question deadline (bid) | 29.04.2026 |
| **Bid 1 deadline** | **06.05.2026** |
| Test environment available | 07.05.2026 |
| **Demonstrations** | **11-12.05.2026** |
| Negotiation invitation | 20.05.2026 |
| **Negotiations round 1** | **26-27.05.2026** |
| Invitation to bid 2 | 01.06.2026 |
| Bid 2 deadline | 08.06.2026 |
| Negotiations round 2 | 16-17.06.2026 |
| Invitation to final bid | 23.06.2026 |
| **Final bid deadline** | **01.07.2026** |
| Contract award | 21.08.2026 |
| Contract signing | 01.09.2026 |

## 4. SELECTION CRITERIA (Pre-qualification)
3-6 suppliers will be selected from qualified applicants.
| Criterion | Weight |
|---|---|
| Experience from comparable LMS engagements | 65% |
| Capacity to execute the contract | 35% |

## 5. AWARD CRITERIA (Bid evaluation)
| Criterion | Weight |
|---|---|
| **Quality** | **80%** |
| - Functional requirements | 50% |
| - Non-functional requirements | 20% |
| - Usability (UX) | 12.5% |
| - Establishment & implementation capability | 12.5% |
| - Business principles & standard terms | 5% |
| **Price** | **20%** |

### Usability evaluation based on:
- UI/UX descriptions and user stories in Bilag 2
- Demonstration of selected functional requirements (POC tasks)
- Hands-on testing by evaluators in demo environment
- ISO 9241-11:2018 (Efficacy, Efficiency, Satisfaction) and Nielsen's 10 heuristics

## 6. USERS & SCALE
| User type | Count |
|---|---|
| Internal SVV employees (SSO) | ~5,300 |
| Administrators (SSO) | ~400 |
| Leaders with personnel responsibility (SSO) | ~400 |
| Consultants/contractors (SSO) | ~1,100 |
| External users - Kilden (ID-porten) | ~1,100 |
| External users - KKP portal (ID-porten) | ~15,000 competency certificates/year |

## 7. CURRENT STATE
- **Kilden**: Main LMS by Storyboard AS - contract expires 2027. 297 active e-learning courses, 18,500 completions, 318 classroom courses/events in 2025
- **KKP Portal**: "Kurs- og kompetanseportal for arbeidsvarsling og vinterdrift" by Apropos/Dossier. Contract 2023-2029. 350,000 competency certificates total, 15,000 issued annually. Primarily for external users.
- **Content tools**: Articulate 360 (Storyline/Rise), Junglemap, Vyond - not integrated
- **Pain points**: Multiple disconnected systems, manual Excel-based competency tracking, no unified data model

## 8. VISION & KEY OBJECTIVES
**Vision**: Establish a modern, user-friendly, future-oriented learning platform that strengthens competency development, promotes knowledge sharing, and supports strategic goals.

**Key objectives**:
1. Accessible and engaging learning for all - regardless of role, location, or technical competence
2. Support the 70/20/10 learning model (on-the-job/social/formal)
3. Continuous learning and development with personalized content
4. Knowledge sharing and collaboration via forums, groups, professional networks
5. HMS/mandatory course oversight for leaders and safety units
6. Data-driven development - competency gap analysis, analytics for strategic decisions
7. Efficient administration and reporting integrated with HR systems
8. External competency portal for partners
9. AI-powered personalized learning and content generation
10. Employee development tracking and goal-setting

## 9. FUNCTIONAL REQUIREMENTS AREAS
(From Bilag 1 vedlegg 1 - detailed spreadsheet)
- User profiles and access control
- Usability, customization and support
- Data and analytics
- External competency portal
- Content overview, search and recommendations
- Communication and collaboration
- Production, content and metadata
- Assignment, enrollment, progression and history

## 10. NON-FUNCTIONAL REQUIREMENTS (Summary)
Categories with absolute and evaluable requirements:
- **Logging & monitoring** (LOG-04): All activity logging for traceability
- **Identity & access management** (IAM-03 to IAM-20): SSO, MFA, SCIM provisioning, ForgeRock integration, role-based access, centralized user admin
- **Application security** (APP-08, APP-12, APP-13): End-to-end data flow security, mobile encryption
- **Privacy/GDPR** (PEO-13 to PEO-29): Data processing agreement, EU/EEA storage mandatory, sub-processor controls, transfer documentation
- **Data protection** (POD-02 to POD-10): EU processing, encryption at rest and in transit, backup to 2 locations, tested recovery
- **Network security** (NET-13): DDoS protection
- **Migration** (MIG-01, 03.011): Data migration from existing systems, API integration capabilities
- **Architecture** (05.xxx): Document standards, archiving compliance, log integration
- **Usability** (01.013, 01.015): WCAG compliance, universal design, responsive design
- **AI** (KI-01): Describe AI technology, training data, GDPR compliance
- **Availability** (TI-01 to TI-05): Response times, uptime SLA, no maintenance during exams, responsive UI, update processes
- **Ownership** (EI-01 to EI-03): Data remains SVV property, data return/deletion on termination
- **Security** (RO-01 to RO-05): ROS (risk assessment), vulnerability disclosure, penetration testing, minimal data collection

## 11. INTEGRATIONS (Critical)
1. **IAM (ForgeRock AM/IDM)**: SCIM provisioning, OAuth2/OIDC, SAML 2.0, SSO, MFA, ID-porten, BankID, Maskinporten
2. **Kursoppslag**: Proxy API abstracting LMS - must maintain existing API contract for internal consumers. REST API with endpoints for course participants, course info, learning plan participants/info
3. **ServiceNow**: Competency register - job families, job roles, skills. Learning log synchronization. REST APIs
4. **PowerBI**: Data export for SVV's reporting platform. Requires structured, compatible data
5. **eTeori** (option): Exam registration and results. REST APIs
6. **Mime** (option): Archive and journal system integration for archival-mandatory competency certificates

## 12. DELIVERABLES
### Pre-delivery (L01-L10):
- L01: Startup phase with test plan and acceptance criteria
- L02: Needs analysis and stakeholder alignment across divisions
- L03: Solution configuration per SVV requirements
- L04: Role-based access with SSO integration
- L05: ServiceNow integration (time & materials, 100h estimated)
- L06: PowerBI integration (time & materials, 100h estimated)
- L07: Learning surfaces configured per approved architecture
- L08: Migration from Kilden (time & materials, 150h estimated)
- L09: Documentation and training for admins/superusers
- L10: Acceptance testing per agreed test plan

### Post-delivery (L11-L12):
- L11: Ongoing support and helpdesk
- L12: Change requests and enhancements (200h estimated, discounted rate)

### Options (L21-L24):
- L21: KKP portal migration (250h) - depends on L22
- L22: eTeori integration (70h)
- L23: Integrated payment solution
- L24: Mime archive integration (150h)

## 13. PRICING STRUCTURE
- **1A**: Fixed price for pre-delivery deliverables (L01-L04, L07, L09, L10)
- **1B**: Time & materials for pre-delivery integrations (L05, L06, L08)
- **1C**: Time & materials for post-delivery changes (L12)
- **1D/1E**: Options pricing (not evaluated)
- **1F**: Annual licensing/SaaS costs 2027-2032 (site license or per-user)
- **1G**: Annual support and maintenance
- **1H/1I**: Hourly rates for pre/post-delivery work

## 14. KEY RISKS & CONSIDERATIONS
- **Data sovereignty**: GDPR strict - EU/EEA data storage mandatory, transfers documented
- **Integration complexity**: 6+ system integrations, SCIM/OAuth2/SAML required
- **Migration risk**: 297 courses + historical data from Kilden, 350,000 certificates from KKP
- **External users**: Must support ID-porten, self-registration, competency certificates at scale
- **AI ethics**: AI must be explainable, reliable, and ethical per Norwegian standards
- **Archiving**: Competency certificates are archival-mandatory (arkivplikt)
- **Accessibility**: WCAG and Norwegian universal design law compliance
- **Seasonal patterns**: KKP has seasonal usage peaks
- **Budget reservation**: SVV reserves right to cancel if budget not approved
- **Contract regime**: SSA-L with specific SLA, penalty clauses, key personnel requirements

## 15. LEARNING ACTIVITIES TO SUPPORT
Broad range including: classroom courses, webinars, digital courses, e-learning (SCORM/xAPI/cmi5), quizzes, assignment submissions, learning paths, workplace-based activities (inspections, internships, emergency drills, simulator training, theoretical/practical exams, job observation, re-authorization, self-training, coaching/mentoring)

## 16. POC DEMONSTRATION SCENARIOS
5 role-based scenarios with specific tasks:
1. **Internal user**: Landing page, recommendations, search, notifications, learning path progression
2. **External user**: Registration, first login, onboarding
3. **Leader**: Team learning status, certification expiry, course assignment, activity approval
4. **Content manager**: Course creation (SCORM), learning path setup, course updates/archiving, mandatory vs optional configuration
5. **Division administrator**: Reports, dashboards, data export capabilities
