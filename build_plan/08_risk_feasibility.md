# Risk & Feasibility Assessment: Building an LMS from Scratch

## For: Statens vegvesen Tender (Case 25/223727) -- Build vs Buy Decision
## Date: 2026-03-22
## Status: DRAFT

---

## Executive Summary

This document provides an honest, comprehensive risk and feasibility assessment for building a Norwegian public sector LMS SaaS product from scratch, as an alternative to reselling an established platform (Cornerstone Galaxy, Docebo, or Totara). The assessment covers technical feasibility, timeline, market/business viability, regulatory compliance, financial risks, and a direct build-vs-buy comparison.

**Bottom-line finding**: Building a production-quality LMS from scratch to compete in the SVV tender is **not feasible within the tender timeline** and carries **critical risk across multiple dimensions**. However, a longer-horizon product strategy (18-36 months) targeting Norwegian public sector LMS needs has merit -- provided it is decoupled from the SVV tender and funded independently.

**Recommendation**: **CONDITIONAL NO-GO for the SVV tender as a build-from-scratch product.** Pursue a reseller/integrator strategy for SVV. Consider a parallel build track only if there is independent funding and a 2+ year horizon, targeting the broader Norwegian public sector market (kommuner, fylkeskommuner, direktorater).

---

## 1. Technical Feasibility Risks

### 1.1 SCORM 1.2/2004 Compliance

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | High |
| **Impact if underestimated** | Critical |
| **Estimated effort** | 3-6 months for a reliable SCORM player |

**Reality check**: SCORM is not a clean, well-documented standard. It is a collection of specifications (RTE, CAM, SN) with significant ambiguity, vendor-specific quirks, and edge cases that only emerge when running real-world content packages.

**Key challenges:**
- **SCORM 1.2 Runtime Environment (RTE)**: Requires implementing the API Adapter (window.API), handling 23+ CMI data model elements, managing the communication lifecycle (LMSInitialize, LMSSetValue, LMSGetValue, LMSCommit, LMSFinish), and handling error conditions correctly. The API must be injected into the content frame at the correct time, which varies by content authoring tool.
- **SCORM 2004 (editions 2-4)**: Adds sequencing and navigation (SN), which is dramatically more complex. SN has a 200+ page specification with activity trees, rollup rules, tracking models, and navigation requests. Most LMS vendors took 1-2 years to achieve reasonable SN compliance.
- **Content package parsing**: The imsmanifest.xml parser must handle nested organizations, resource dependencies, metadata schemas (LOM), and prerequisite rules.
- **Cross-browser iframe sandboxing**: SCORM content runs inside iframes with postMessage communication. Content from tools like Articulate Storyline, Adobe Captivate, and iSpring (SVV uses Articulate 360) each have their own quirks in how they call the API, handle window references, and manage session state.
- **ADL conformance testing**: To claim SCORM conformance, you must pass the ADL test suite. Many LMS vendors fail portions of this test even after years of development.

**Mitigation**: Use the **Rustici SCORM Cloud engine** or **Rustici Engine** (commercial license, ~$15-50K/year) as an embedded content player rather than building from scratch. Rustici is the de facto standard and is what Cornerstone, Docebo, and many others use under the hood. This eliminates 80% of the SCORM risk.

**Residual risk with Rustici**: Still requires integration development (API wrapping, grade passback, session management, bookmark/resume handling). Estimated 4-8 weeks of integration work.

**Residual risk without Rustici (building our own)**: Extremely high. SCORM player development is a multi-year endeavor to reach production quality. This alone could consume the entire build timeline.

---

### 1.2 xAPI (Experience API) / cmi5 Support

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | Medium |
| **Impact** | Medium |
| **Estimated effort** | 4-8 weeks for basic xAPI; 8-16 weeks for cmi5 |

**xAPI** is architecturally cleaner than SCORM. It consists of:
- A Learning Record Store (LRS) that accepts xAPI statements (actor-verb-object triples)
- Statement forwarding and querying
- Authentication via OAuth or Basic Auth

**cmi5** is a profile on top of xAPI that adds LMS-launch semantics similar to SCORM. It is relatively well-specified but has limited adoption.

**Options:**
- **Build a basic LRS**: 4-6 weeks. The xAPI spec is straightforward for basic statement storage and retrieval. Use an existing open-source LRS (e.g., Learning Locker -- now Veracity Learning) as a starting point or embed one.
- **cmi5 launch mechanism**: Additional 4-8 weeks. Requires implementing the cmi5 launch sequence, AU (Assignable Unit) management, and session tracking.

**Assessment**: xAPI/cmi5 is manageable and not a showstopper. Can be phased (xAPI in MVP, cmi5 post-launch).

---

### 1.3 Multi-Tenant SaaS at Scale

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | High |
| **Impact** | High |
| **Estimated effort** | Architectural foundation: 8-12 weeks; production hardening: ongoing |

**Requirements**:
- SVV alone: ~5,300 internal + 1,100 consultants + 15,000 external = ~21,400 users
- Multi-tenant: if we want a SaaS product, we need to serve multiple customers
- Each tenant needs: data isolation, branding, custom domains, separate admin hierarchies, independent content libraries

**Key challenges:**
- **Data isolation**: Row-level security (schema-per-tenant or discriminator column) must be airtight. A data leak between tenants in a government SaaS is a career-ending event.
- **Performance isolation**: One tenant's bulk report generation cannot degrade another tenant's user experience. Requires careful queue management, resource quotas, and connection pooling.
- **Tenant-specific configuration**: Each customer will want different IAM providers, different branding, different notification templates, different compliance rules.
- **Database migration management**: Schema changes must work across all tenants simultaneously or be rolled out progressively.
- **Horizontal scaling**: Content delivery (SCORM packages can be 500MB+), concurrent SCORM sessions, and reporting queries all have different scaling profiles.

**Mitigation**: Use a proven multi-tenant architecture pattern (shared database with row-level security + tenant context middleware). Use Kubernetes for compute isolation. Use CDN (CloudFront/Azure CDN) for content delivery. These are well-understood patterns but require disciplined engineering.

**Residual risk**: Medium. Multi-tenancy done right is achievable with an experienced team but adds complexity to every feature.

---

### 1.4 Norwegian Identity Infrastructure (ID-porten, ForgeRock SCIM)

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | Medium-High |
| **Impact** | Critical (blocks go-live) |
| **Estimated effort** | 6-10 weeks |

**ID-porten integration:**
- ID-porten uses OpenID Connect. Integration is well-documented by Digdir.
- However: requires Samarbeidsportalen registration, test environment access, and compliance with Digdir's security requirements.
- SVV's architecture routes ID-porten through ForgeRock AM as an identity broker, so the LMS may not integrate with ID-porten directly but rather with ForgeRock, which federates ID-porten.

**ForgeRock SCIM provisioning:**
- SCIM 2.0 is a standard protocol, but ForgeRock's implementation has specific behaviors (attribute mapping, group provisioning, schema extensions).
- Requires access to SVV's ForgeRock test environment, which is controlled by SVV IT.
- Must handle: user create, update, deactivate, group membership changes, organizational hierarchy.

**Assessment**: This is not technically novel -- it is standard OIDC/SCIM work. The risk is in **dependency on SVV's IAM team** for test environment access and in matching their specific ForgeRock configuration. This is the same risk whether we build or buy.

---

### 1.5 Content Player Reliability

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | High |
| **Impact** | High (user-facing, reputation-damaging) |
| **Time to production quality** | 6-18 months of real-world bug fixing |

**The core problem**: SCORM content packages are authored using dozens of different tools (Articulate Storyline, Rise, Captivate, iSpring, Lectora, Camtasia, custom HTML5). Each tool generates slightly different JavaScript, makes different assumptions about the API adapter lifecycle, and handles edge cases differently. A content player that works perfectly with Articulate content may fail with Captivate content -- and vice versa.

**Real-world failure modes:**
- Content calls LMSGetValue before LMSInitialize
- Content opens in a popup instead of an iframe and loses the API reference
- Content uses window.parent.API instead of window.API
- Bookmarking/resume doesn't work because suspend_data exceeds 4096 characters
- Score normalization differs (raw scores vs. scaled scores vs. percentages)
- Completion status vs. success status semantics differ between content packages
- Multi-SCO packages with sequencing have timing-dependent navigation issues

**Mitigation**: Rustici Engine (see 1.1). Without it, expect 12-18 months of bug reports from users encountering content-specific failures. SVV has 297 active courses authored primarily in Articulate 360 -- we could validate against this toolset first, but edge cases will persist.

---

### 1.6 Performance Under Load (Seasonal Peaks)

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | Medium |
| **Impact** | Medium-High |
| **Estimated effort** | 2-4 weeks for load testing infrastructure; ongoing tuning |

**KKP portal pattern**: 15,000 competency certificates/year with seasonal peaks tied to arbeidsvarsling (road work signage) and vinterdrift (winter operations) seasons. Peaks likely in March-April (pre-summer construction) and September-October (pre-winter operations).

**Scaling challenges:**
- Concurrent SCORM sessions are resource-intensive (each session maintains server-side state)
- Certificate generation at peak (PDF generation, digital signing, archival storage) creates I/O spikes
- Search and reporting queries during enrollment periods compete with content delivery

**Mitigation**: Standard cloud auto-scaling (Kubernetes HPA, database read replicas, CDN for static content). Load test early with realistic scenarios. Use async processing for certificate generation.

**Assessment**: Manageable with proper architecture. Not a differentiating risk compared to other SaaS products.

---

## 2. Timeline Risks

### 2.1 Can We Build a Demo-Ready MVP for the SVV Tender?

| Attribute | Assessment |
|-----------|------------|
| **Deadline** | 11-12 May 2026 (demonstration) |
| **Available time from today** | ~7 weeks |
| **Required for demo** | 5 POC scenarios (see tender Bilag 1 vedlegg 7) |
| **Verdict** | **NO -- not feasible** |

**What the demo must show (from POC oppgaver):**

1. **Internal user flow**: Landing page with recommendations, search, notifications, learning path progression, SCORM course launch and completion tracking
2. **External user flow**: Self-registration via ID-porten, first login, onboarding flow
3. **Leader flow**: Team learning status dashboard, certification expiry alerts, course assignment, activity approval workflows
4. **Content manager flow**: SCORM course upload and publishing, learning path setup, course versioning/archiving, mandatory vs. optional configuration
5. **Division administrator flow**: Reports, dashboards, data export

**What we could realistically build in 7 weeks** (with a 4-6 person team):
- Basic user authentication (local, no ForgeRock/ID-porten)
- Simple course catalog with manual SCORM upload
- Rudimentary SCORM player (using Rustici evaluation license)
- Basic enrollment and completion tracking
- Minimal admin UI

**What we cannot build in 7 weeks:**
- Polished, accessible UI across all 5 scenarios (WCAG 2.1 AA)
- Real ID-porten integration (requires Digdir registration, weeks of lead time)
- Real ForgeRock SCIM provisioning
- Learning path engine with progression rules
- Reporting dashboards with data export
- Notification system
- AI-powered recommendations
- Content versioning and lifecycle management
- Role-based access control with organizational hierarchy
- Any semblance of production quality

**Conclusion**: A 7-week demo build would look like a prototype, not a product. Against Cornerstone Galaxy (7,000 customers, mature UX) and Docebo (AI-native, polished UI), our demo would be embarrassing. The evaluators will use ISO 9241-11 and Nielsen's heuristics -- a rushed MVP will score near zero on usability (12.5% of quality score) and poorly on functional requirements (50% of quality score).

---

### 2.2 Minimum Viable Demo vs. Production-Ready Gap

| Dimension | Demo MVP | Production v1 | Gap |
|-----------|----------|---------------|-----|
| **Users supported** | 10-50 test users | 21,400+ concurrent | 400x scale |
| **Content compatibility** | Works with 5-10 test packages | Works with 297+ real courses from multiple authoring tools | Months of edge-case fixes |
| **IAM** | Local auth or mock OIDC | Full ForgeRock SCIM + ID-porten + BankID + MFA | 6-10 weeks |
| **Accessibility** | Basic responsive | WCAG 2.1 AA audited | 4-8 weeks retrofit |
| **Security** | Basic HTTPS | Penetration tested, SOC 2, ISO 27001 preparation | 6-12 months |
| **Reliability** | Crashes acceptable | 99.5%+ uptime SLA with penalty clauses | Months of hardening |
| **Data migration** | Empty database | 297 courses + history from Kilden, 350K certs from KKP | 4-8 weeks |
| **Integrations** | 0 | 6 (ForgeRock, Kursoppslag, ServiceNow, PowerBI, eTeori, Mime) | 20-30 weeks |

---

### 2.3 What If SCORM Compliance Takes Longer?

**Without Rustici Engine**: SCORM compliance has historically been the longest pole in LMS development. If we attempt to build our own SCORM runtime:

- **Best case**: 3 months for basic SCORM 1.2, 6 months for SCORM 2004 without sequencing
- **Likely case**: 6-9 months for SCORM 1.2 + 2004 basic, 12+ months for sequencing
- **Worst case**: 18+ months before content from diverse authoring tools works reliably

**With Rustici Engine**: SCORM becomes a 4-8 week integration task. However, this adds a commercial dependency ($15-50K/year) and introduces Rustici as a sub-processor under GDPR, which must be declared in the DPA.

**Impact of SCORM delays**: SCORM is the foundation. Without it, there is no course delivery. Everything else (enrollment, completion tracking, reporting, certification) depends on SCORM working. A delay in SCORM delays the entire product.

---

### 2.4 Dependency Risks Between Epics

```
                    ┌─────────────────┐
                    │   IAM / Auth    │  ← Blocks everything
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼──────┐ ┌────▼─────┐ ┌──────▼──────┐
     │ User Mgmt &   │ │ Content  │ │  External   │
     │ Org Hierarchy  │ │ Player   │ │  User Portal│
     └────────┬──────┘ │ (SCORM)  │ │ (ID-porten) │
              │        └────┬─────┘ └──────┬──────┘
              │             │              │
     ┌────────▼─────────────▼──────────────▼──────┐
     │         Enrollment & Learning Paths         │
     └─────────────────────┬──────────────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
     ┌────────▼────┐ ┌────▼─────┐ ┌───▼──────────┐
     │ Completion  │ │Reporting │ │ Certification │
     │ Tracking    │ │& Analytics│ │ & Compliance │
     └─────────────┘ └──────────┘ └──────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
     ┌────────▼────┐ ┌────▼─────┐ ┌───▼──────────┐
     │ ServiceNow  │ │ PowerBI  │ │  Mime/Archive │
     │ Integration │ │ Export   │ │  Integration  │
     └─────────────┘ └──────────┘ └──────────────┘
```

**Critical path**: IAM → User Management → Enrollment → Completion Tracking → Reporting. Any delay in IAM cascades through the entire delivery.

---

## 3. Market & Business Risks

### 3.1 "Established and Proven" Qualification Barrier

| Attribute | Assessment |
|-----------|------------|
| **Probability of disqualification** | Very High (>80%) |
| **Impact** | Fatal -- cannot bid |

**Pre-qualification criteria (65% weight)**: "Experience from comparable LMS engagements." The tender asks for references demonstrating:
- Delivery of cloud-based LMS to organizations with 5,000+ users
- Integration with enterprise IAM systems
- External user portals at scale
- Norwegian or Nordic public sector experience

**Our position with a new build**: Zero references. Zero deployments. Zero production users. We cannot demonstrate "experience from comparable LMS engagements" because the product does not exist. This alone is likely disqualifying.

**Mitigation options:**
- **Partner with an established vendor**: Use Totara or Moodle as a base and present customization references. But this is no longer "build from scratch" -- it is an implementation partnership.
- **Present team experience**: Individual team members may have LMS implementation experience. But the tender asks for organizational/company references, not individual CVs (CVs are for the capacity criterion at 35%).

**Residual risk**: Even with mitigation, we would be the weakest applicant on the 65%-weighted qualification criterion. Against Cornerstone (7,000 customers), Docebo (3,800 customers), and established Nordic integrators, we would likely not pass pre-qualification.

---

### 3.2 Competing Against Established Vendors

| Attribute | Assessment |
|-----------|------------|
| **Probability of losing on functionality** | Very High |
| **Impact** | Cannot win the tender |

**The competitive reality:**

| Capability | Cornerstone Galaxy | Our Build (MVP) |
|------------|-------------------|-----------------|
| Years in market | 25+ | 0 |
| Production customers | 7,000+ | 0 |
| Users on platform | 100M+ | 0 |
| SCORM conformance | Full, battle-tested | Unvalidated |
| AI capabilities | Galaxy AI (40TB data, agentic AI) | Basic recommendations at best |
| Integrations | 100+ marketplace connectors | Custom-built from zero |
| Languages | 50+ | Norwegian + English |
| Accessibility | WCAG audited, remediation program | Untested |
| Mobile app | Native iOS/Android + offline | Responsive web only |
| Extended enterprise | Portal Builder, e-commerce, branded microsites | Basic multi-tenant |

**Functional requirements score (50% of quality = 40% of total)**: SVV's Bilag 1 vedlegg 1 contains hundreds of functional requirements across 8 categories. A new build will meet perhaps 30-40% of these at MVP stage. Cornerstone/Docebo will meet 80-95%. The scoring gap is insurmountable.

---

### 3.3 Customer Acquisition Cost in Norwegian Public Sector

| Attribute | Assessment |
|-----------|------------|
| **Sales cycle** | 6-18 months (public procurement) |
| **Bid cost** | 500K-1.5M NOK per tender response |
| **Win rate for new entrants** | 5-15% |

**Norwegian public sector procurement realities:**
- All significant IT purchases go through Doffin/TED and formal procurement procedures
- Evaluation criteria heavily weight "proven" solutions and reference cases
- First customer is the hardest -- subsequent customers become progressively easier as references accumulate
- Many public entities use framework agreements (rammeavtaler) through DFO or KS, which require pre-qualification

**Path to first customer**: Without the SVV tender, we would need to find a smaller public sector entity willing to take a risk on an unproven platform. Kommuner (municipalities) or smaller direktorater might be candidates, but they have smaller budgets (500K-5M NOK) and still require formal procurement.

**Estimated time to first paying customer**: 12-24 months from product launch.

---

### 3.4 Path to Profitability

| Metric | Estimate |
|--------|----------|
| Break-even customer count | 5-8 tenants at 2-4 MNOK ARR each |
| Time to break-even | 3-5 years from start of development |
| Required investment to break-even | 25-50 MNOK |
| Probability of reaching break-even | 20-30% |

**Revenue model assumptions:**
- Target market: Norwegian public sector organizations with 1,000-10,000 employees
- Addressable market: ~50-100 organizations (direktorater, etater, kommuner, fylkeskommuner)
- Achievable market share after 5 years: 5-10% = 3-10 customers
- Average ARR per customer: 2-4 MNOK
- Total revenue at steady state: 6-40 MNOK/year

**Cost structure:**
- Engineering team (8-12 people): 12-18 MNOK/year
- Infrastructure & SaaS costs: 1-2 MNOK/year
- Sales & marketing: 3-5 MNOK/year
- Total operating cost: 16-25 MNOK/year

**Conclusion**: The unit economics can work at scale (5+ customers), but the path to scale requires significant upfront investment and a long runway.

---

## 4. Regulatory & Compliance Risks

### 4.1 ISO 27001 / SOC 2 Certification

| Attribute | Assessment |
|-----------|------------|
| **Probability of timeline risk** | High |
| **Impact** | May block sales to security-conscious customers |
| **Timeline to certification** | 9-18 months |
| **Cost** | 500K-1.5M NOK |

**ISO 27001:**
- Requires establishing an ISMS (Information Security Management System)
- Minimum 3 months to implement controls, 3 months operating history, then audit
- Fast-track with a consultant: 6-9 months minimum
- The SVV tender does not explicitly require ISO 27001, but it requires a ROS-analyse (risk assessment) and comprehensive security documentation

**SOC 2:**
- Requires 3-12 months of operating evidence for Type II
- Type I (design only) can be achieved in 3-6 months
- Relevant for international credibility but not typically required by Norwegian public sector

**Assessment**: ISO 27001 is achievable within 12 months but not before the SVV tender timeline. For the SVV tender specifically, we would need to demonstrate equivalent security controls without the certification -- possible but weaker than certified competitors.

---

### 4.2 GDPR Compliance

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | Medium |
| **Impact** | Critical (legal liability) |
| **Effort** | 4-8 weeks for DPA framework; ongoing |

**Requirements:**
- Data Processing Agreement (DPA) with SVV as data controller
- EU/EEA-only data storage and processing -- mandatory
- Sub-processor register and management
- Privacy by design in the product architecture
- DPIA (Data Protection Impact Assessment) for AI features
- Right to deletion, portability, access requests
- Data breach notification procedures (72-hour window)

**Challenge for a new product**: We would need to document the entire data flow, all sub-processors (cloud provider, Rustici if used, email service, monitoring tools), and demonstrate GDPR compliance from day one. This is achievable but requires deliberate effort.

**Mitigation**: Use EU-region cloud services (Azure Norway East or AWS eu-north-1), minimize sub-processors, implement privacy controls from the architecture phase.

**Residual risk**: Low-Medium if done properly. GDPR compliance is well-understood and achievable for a greenfield product.

---

### 4.3 Arkivloven Compliance (Competency Certificates)

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | Medium |
| **Impact** | High (legal requirement for competency certificates) |
| **Effort** | 4-6 weeks |

**Requirement**: Competency certificates issued by the KKP portal are arkivpliktige (archival-mandatory). This means they must be:
- Stored in a format that preserves their evidential value
- Accessible for the legally required retention period
- Potentially integrated with Mime (SVV's archive system) via SOAP/REST APIs
- Formatted per Noark 5 standards if integrated with a public archive

**Assessment**: This is a niche Norwegian requirement that established LMS vendors also struggle with. Building Mime integration is 4-6 weeks of development if the Mime API is well-documented. The archival format requirements (PDF/A, metadata) are well-specified.

---

### 4.4 WCAG 2.1 AA Compliance

| Attribute | Assessment |
|-----------|------------|
| **Probability of difficulty** | High |
| **Impact** | High (legal requirement in Norway -- likestillings- og diskrimineringsloven) |
| **Effort** | 8-16 weeks (retrofit) or +30% effort (built-in) |

**Norwegian universal design law**: All ICT solutions directed at the public must comply with WCAG 2.1 AA. Tilsynet for universell utforming monitors compliance.

**Challenge for a new build:**
- WCAG compliance affects every UI component: navigation, forms, tables, modals, notifications, SCORM player chrome, PDF viewers, video players
- Requires screen reader testing (NVDA, JAWS, VoiceOver), keyboard navigation, color contrast validation, focus management
- The SCORM content itself must also be accessible, but that is the content author's responsibility -- the LMS player must not break accessibility
- A mature UI component library (e.g., Radix UI, Headless UI) helps, but custom components still need audit

**Mitigation**: Use an accessible-by-default component library. Build with semantic HTML. Test with axe-core in CI. Budget for a professional WCAG audit before launch.

**Residual risk**: Medium. Achievable if accessibility is a first-class concern from day one, but expensive to retrofit if bolted on later.

---

## 5. Financial Risks

### 5.1 Estimated Investment to Reach MVP

| Category | Low Estimate | High Estimate |
|----------|-------------|---------------|
| **Engineering team (6 months, 6-8 people)** | 4.5 MNOK | 7.0 MNOK |
| **UX/Design** | 0.5 MNOK | 1.0 MNOK |
| **Rustici Engine license** | 0.15 MNOK | 0.5 MNOK |
| **Cloud infrastructure (dev/staging/prod)** | 0.2 MNOK | 0.5 MNOK |
| **WCAG audit** | 0.1 MNOK | 0.3 MNOK |
| **Security audit / pen test** | 0.1 MNOK | 0.3 MNOK |
| **Legal (DPA, terms, compliance)** | 0.1 MNOK | 0.3 MNOK |
| **Total to MVP** | **5.7 MNOK** | **9.9 MNOK** |

**MVP definition** (what you get for this investment):
- Multi-tenant SaaS platform with basic course management
- SCORM 1.2/2004 content delivery (via Rustici)
- User management with OIDC/SAML SSO
- Basic enrollment, completion tracking, and reporting
- Norwegian language UI
- Simple admin dashboard
- Responsive, WCAG-compliant web application

**What MVP does NOT include**:
- AI-powered features
- Extended enterprise portal
- Learning paths with complex progression rules
- Competency management framework
- Social/collaborative learning features
- Advanced analytics and data export
- ServiceNow, PowerBI, or other integrations
- Mobile native app
- Comprehensive content authoring tools
- Gamification

---

### 5.2 Estimated Investment to Reach Production v1

| Category | Low Estimate | High Estimate |
|----------|-------------|---------------|
| **MVP investment (above)** | 5.7 MNOK | 9.9 MNOK |
| **Additional engineering (6 months, 8-10 people)** | 6.0 MNOK | 10.0 MNOK |
| **AI/ML development** | 1.0 MNOK | 2.5 MNOK |
| **Extended enterprise features** | 1.0 MNOK | 2.0 MNOK |
| **Integration framework** | 1.0 MNOK | 2.0 MNOK |
| **ISO 27001 certification** | 0.5 MNOK | 1.5 MNOK |
| **Total to Production v1** | **15.2 MNOK** | **27.9 MNOK** |

**Production v1 definition**: A product that could credibly compete in a public sector LMS tender -- with competency management, learning paths, external user portal, reporting/analytics, SCORM/xAPI support, WCAG compliance, and security certifications.

---

### 5.3 Time to First Revenue

| Milestone | Estimated Date (from today) |
|-----------|---------------------------|
| MVP feature-complete | +6-9 months (Sep-Dec 2026) |
| First pilot customer (free/discounted) | +9-15 months (Dec 2026 - Jun 2027) |
| Production v1 ready | +12-18 months (Mar-Sep 2027) |
| First paying customer | +15-24 months (Jun 2027 - Mar 2028) |
| First full-price contract | +18-30 months (Sep 2027 - Sep 2028) |

---

### 5.4 Runway Needed

| Scenario | Investment to Revenue | Monthly Burn | Runway Required |
|----------|----------------------|-------------|-----------------|
| **Lean (6-person team)** | 15 MNOK | 1.0 MNOK/month | 18-24 months |
| **Standard (10-person team)** | 22 MNOK | 1.5 MNOK/month | 18-24 months |
| **Ambitious (12-person team + sales)** | 30 MNOK | 2.2 MNOK/month | 18-24 months |

---

## 6. Build vs Buy Comparison

### 6.1 What We Gain by Building

| Advantage | Value | Probability of Realizing |
|-----------|-------|------------------------|
| **Full control over product roadmap** | Can prioritize Norwegian public sector needs exactly | High |
| **No vendor lock-in** | Own the code, control the destiny | High |
| **Norwegian-first design** | ID-porten, Arkivloven, WCAG, Norwegian UX patterns built in from day one | High |
| **Higher margins if successful** | SaaS margins (70-80%) vs. reseller margins (15-30%) | Medium -- only if we win customers |
| **IP asset creation** | Builds long-term company value | Medium |
| **Competitive differentiation** | "Built for Norwegian public sector" vs. "adapted for Norwegian public sector" | Medium-High |
| **Pricing flexibility** | Can undercut enterprise vendors on price | High |
| **Direct customer relationship** | No vendor intermediary, faster feature delivery | High |

### 6.2 What We Lose by Building

| Disadvantage | Impact | Severity |
|--------------|--------|----------|
| **Zero references** | Cannot pass pre-qualification for major tenders | Critical |
| **Feature gap (years)** | Cannot match Cornerstone's 25 years of feature development | Critical |
| **Time to market** | 12-18 months before a credible product exists | High |
| **SCORM battle-testing** | Years of real-world content compatibility testing vs. day one | High |
| **AI capabilities** | Cannot match Cornerstone Galaxy AI (40TB data, ML models trained on 30M learning hours/month) | High |
| **Support infrastructure** | No 24/7 support team, no knowledge base, no community | Medium |
| **Ecosystem** | No integration marketplace, no partner network | Medium |
| **Security certifications** | 9-18 months to ISO 27001; competitors already certified | High |
| **Revenue risk** | Investing 15-28 MNOK with no guaranteed customers | Critical |

### 6.3 Hybrid Approach: Building on Open Source

**Option: Build on Moodle/Totara as a base**

| Aspect | Assessment |
|--------|------------|
| **Starting point** | Moodle (GPL) provides course management, SCORM player, enrollment, grading, reporting, role-based access, Norwegian language pack |
| **What we add** | Modern SaaS wrapper (multi-tenant hosting, API layer, modern UI), Norwegian public sector integrations (ID-porten, ForgeRock SCIM), AI features, PowerBI export, Mime/Arkiv integration |
| **Advantages** | Faster time to market (skip 60-70% of core LMS development), SCORM already works, community-tested with millions of users, Norwegian language pack exists |
| **Disadvantages** | Moodle's PHP/MySQL architecture is dated, UX is not modern enterprise-grade, customization can create maintenance burden, Moodle Workplace (commercial) has licensing restrictions |
| **Time to market** | 6-12 months to a differentiated product (vs. 12-18 months from scratch) |
| **Estimated investment** | 8-15 MNOK to productize (vs. 15-28 MNOK from scratch) |

**Option: Build a modern frontend on Totara as backend**

| Aspect | Assessment |
|--------|------------|
| **Starting point** | Totara Learn (open source) provides enterprise LMS backend with competency management, certification workflows, multi-tenancy |
| **What we add** | Modern React/Next.js frontend (replacing Totara's Moodle-derived UI), Norwegian public sector integration layer, AI features |
| **Advantages** | Best competency management in the market (aligned with SVV needs), proven SCORM player, open-source licensing |
| **Disadvantages** | Totara has its own partner network and may restrict how we use their brand, dual-architecture complexity (modern frontend + PHP backend), community is smaller than Moodle |
| **Time to market** | 6-9 months to a differentiated product |
| **Estimated investment** | 6-12 MNOK to productize |

**Recommendation**: If we pursue a build strategy, the hybrid approach (modern SaaS layer on Moodle or Totara) is significantly more viable than building from scratch. It reduces time-to-market by 6-12 months and eliminates SCORM compatibility risk.

---

## 7. Risk Summary Matrix

### All Risks by Category

| ID | Risk | Probability | Impact | Risk Score | Mitigation | Residual Risk |
|----|------|-------------|--------|------------|-----------|---------------|
| **T-01** | SCORM 1.2/2004 compliance (own build) | High | Critical | **CRITICAL** | Use Rustici Engine | Medium (integration work) |
| **T-02** | SCORM content compatibility (real-world) | High | High | **HIGH** | Rustici Engine + extensive testing | Medium |
| **T-03** | Multi-tenant data isolation failure | Medium | Critical | **HIGH** | Row-level security, pen testing | Low-Medium |
| **T-04** | xAPI/cmi5 implementation | Medium | Medium | **MEDIUM** | Use open-source LRS, phase cmi5 | Low |
| **T-05** | Performance under seasonal load | Medium | Medium-High | **MEDIUM** | Auto-scaling, load testing | Low |
| **TL-01** | Cannot demo at SVV tender (May 2026) | Certain | Critical | **CRITICAL** | Accept: cannot meet this timeline | N/A |
| **TL-02** | SCORM delays cascade to all features | High | High | **HIGH** | Rustici Engine de-risks | Medium |
| **TL-03** | MVP takes 9+ months (not 6) | Medium | High | **HIGH** | Reduce MVP scope, use hybrid approach | Medium |
| **M-01** | Fail pre-qualification (no references) | Very High | Fatal | **CRITICAL** | Partner with established vendor | High |
| **M-02** | Cannot compete on functionality | Very High | Fatal | **CRITICAL** | Target smaller tenders first | High |
| **M-03** | 18+ months to first revenue | High | High | **HIGH** | Secure pilot customer early | Medium |
| **M-04** | Norwegian public sector sales cycle | High | High | **HIGH** | Build relationships pre-tender | Medium |
| **R-01** | ISO 27001 not achievable in time | High | High | **HIGH** | Start early, use consultant | Medium |
| **R-02** | GDPR compliance gaps | Medium | Critical | **HIGH** | Privacy by design, DPA framework | Low-Medium |
| **R-03** | Arkivloven non-compliance | Medium | High | **MEDIUM** | Engage archivist, build Mime integration | Low |
| **R-04** | WCAG 2.1 AA failure | Medium-High | High | **HIGH** | Accessible-first design, audit budget | Medium |
| **F-01** | Investment exceeds 25 MNOK before revenue | Medium | High | **HIGH** | Lean team, hybrid approach | Medium |
| **F-02** | Cannot reach break-even (insufficient customers) | Medium-High | Critical | **CRITICAL** | Validate market before committing | High |
| **F-03** | Cash flow crisis before first customer | High | Critical | **CRITICAL** | Secure 18-24 month runway upfront | Medium |

---

## 8. GO / NO-GO Recommendation

### For the SVV Tender (Case 25/223727): **NO-GO as a build-from-scratch product**

**Reasons:**
1. **Timeline impossibility**: The demo is in 7 weeks. We cannot build a credible product in 7 weeks.
2. **Pre-qualification failure**: With zero references, we will almost certainly fail the 65%-weighted experience criterion.
3. **Functional gap**: Against Cornerstone (25 years, 7,000 customers), our MVP would score near zero on the 50%-weighted functional requirements.
4. **Evaluator credibility**: SVV's IT architects (Ringstad, Kowalski) will immediately recognize a prototype vs. a production platform. This damages our reputation for future opportunities.

**Recommendation for SVV**: Pursue the reseller/integrator strategy as outlined in the existing delivery plan (Cornerstone Galaxy primary, Docebo alternative). This is the path with the highest probability of winning.

---

### For a Longer-Horizon Build Strategy: **CONDITIONAL GO**

A build strategy could be viable under these conditions:

| Condition | Requirement |
|-----------|-------------|
| **C-01: Independent funding** | Secure 15-25 MNOK in runway (18-24 months) that is NOT dependent on the SVV tender outcome |
| **C-02: Hybrid approach** | Build on Moodle or Totara as a base, not from scratch. This cuts time-to-market by 6-12 months and eliminates SCORM risk |
| **C-03: Pilot customer** | Identify a smaller Norwegian public sector organization willing to pilot (kommune, direktorat, or agency with 500-3,000 users) at a discounted rate |
| **C-04: Team** | Assemble a core team of 6-8 engineers with specific experience in: LMS domain, SCORM/xAPI, Norwegian IAM (ID-porten), multi-tenant SaaS, accessible frontend development |
| **C-05: Market validation** | Validate willingness-to-pay with 5+ potential customers before committing to full build |
| **C-06: Decouple from SVV** | Do NOT position the build as competing in the SVV tender. Win SVV as a reseller. Build the product for the broader market. |
| **C-07: First reference** | Target having a live reference customer within 12 months of starting development |

**If all conditions are met**: The Norwegian public sector LMS market is real (50-100 organizations, 100-300 MNOK total addressable market), underserved by Norwegian-first solutions, and the regulatory moat (ID-porten, Arkivloven, WCAG, GDPR) creates defensibility against global vendors. A well-executed product could capture meaningful market share in 3-5 years.

**If conditions C-01 through C-05 are NOT met**: Do not build. The risk of investing 15-25 MNOK with no guaranteed market is too high without validation.

---

## 9. Recommended Strategy: Two-Track Approach

```
TRACK A: SVV Tender (Immediate)          TRACK B: Product Build (18-month horizon)
─────────────────────────────────        ─────────────────────────────────────────
Resell Cornerstone Galaxy                Build Norwegian-first LMS SaaS
Win SVV contract (20-35 MNOK)            (on Moodle/Totara base)
Establish Norwegian public sector        Target smaller pilot customers
  references and relationships           Iterate and build references
Generate revenue and cash flow           Fund from Track A revenue + investment
Learn SVV's needs deeply as integrator   Apply learnings from Track A to product
                                         Enter major tenders in 2028+
```

**Why two tracks?**
1. **Track A** generates revenue, references, and deep domain knowledge immediately
2. **Track B** builds long-term IP value and higher-margin revenue
3. Track A learnings (what public sector customers actually need, which integrations are hardest, what Cornerstone fails at) directly inform Track B product decisions
4. Track A revenue can partially fund Track B development
5. If Track B succeeds, we can migrate Track A customers to our own platform over time

**Risk of two tracks**: Team spread thin. Mitigate by keeping Track B lean (3-4 engineers initially) and growing only after market validation.

---

## Appendix A: Key Assumptions

1. Team cost assumes average fully-loaded cost of 1.2-1.5 MNOK/year per engineer (Norwegian market rates)
2. SCORM complexity estimates based on published experiences from LMS vendors (Moodle, Canvas, Blackboard development histories)
3. Market size estimates based on DIFI/DFO registry of Norwegian government entities + SSB data on kommune/fylkeskommune count
4. Revenue estimates assume SaaS pricing competitive with Cornerstone/Docebo (50-70% of their pricing)
5. Timeline estimates assume an experienced, full-time team (not consultants splitting time across projects)

## Appendix B: Comparable Build Precedents

| Product | Build Time to MVP | Build Time to Competitive | Investment | Outcome |
|---------|-------------------|--------------------------|------------|---------|
| **Canvas LMS** (Instructure) | ~12 months | 3-4 years | $40M+ (VC) | Successful -- but education sector, not corporate |
| **Docebo** | ~18 months | 5+ years | $50M+ (VC) | Successful -- now public company |
| **LearnUpon** | ~12 months | 4+ years | $30M+ (VC) | Successful mid-market |
| **TalentLMS** | ~6 months | 3+ years | Self-funded (Epignosis) | Successful SMB market |
| **Learnifier** (Nordic) | ~12 months | 4+ years | ~20M SEK | Moderate success in Nordics |

**Pattern**: Successful LMS products consistently take 3-5 years and 30-50M+ NOK to reach a competitive enterprise offering. None competed in major enterprise tenders within their first 2 years.
