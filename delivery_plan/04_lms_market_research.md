# LMS Market Research & Platform Evaluation

## For: Statens vegvesen (SVV) LMS Tender (Case 25/223727)
## Date: March 2026

---

## 1. Executive Summary

This document provides a comprehensive analysis of the enterprise LMS market to identify the best-fit platforms for Statens vegvesen's cloud-based learning management system procurement. The evaluation considers SVV's specific requirements: Norwegian language support, GDPR/EU data sovereignty, ForgeRock IAM integration (SCIM/SAML/OAuth2/OIDC), ID-porten/BankID for external users, SCORM/xAPI/cmi5 content standards, competency management, external user portals, PowerBI integration, AI capabilities, and a budget of 20-35 MNOK over up to 6 years.

**Top 3 Recommended Platforms:**
1. **Cornerstone Galaxy** -- Best overall enterprise fit, strongest AI, EU data centers, Norwegian language, extended enterprise portal
2. **Docebo** -- Best AI-powered learning experience, strong extended enterprise, EU hosting, excellent API ecosystem
3. **Totara TXP** -- Best value and flexibility through open-source model, strong competency management, Nordic-friendly architecture

---

## 2. Market Landscape Overview

### 2.1 Market Context

The enterprise LMS market in 2025-2026 is characterized by:

- **AI-first platforms**: Generative AI for course creation, adaptive learning agents, personalized recommendations
- **Talent Experience Platforms (TXP)**: Convergence of LMS, LXP, performance, and skills management
- **Extended enterprise**: Multi-tenant portals serving employees, partners, and external audiences
- **Skills-based architectures**: Competency frameworks replacing course-centric models
- **GDPR-driven data sovereignty**: EU hosting and processing requirements becoming table stakes

### 2.2 Scandinavian LMS Market

The HerbertNathan & Co. Scandinavian LMS Report (the only independent analysis of LMS vendors in the Nordic market) analyzed 17 vendors available in Scandinavia. Key findings:

- **Cornerstone OnDemand** was highly rated for wide functionality, learner experience focus, and advanced AI/ML technology
- **User experience and technology** are the primary differentiators among vendors in the Nordic market
- Nordic organizations prioritize data sovereignty, local language support, and integration with existing HR infrastructure

### 2.3 Nordic-Origin Providers

| Provider | HQ | Focus | Relevance |
|---|---|---|---|
| itslearning | Bergen, Norway | Education sector (K-12, HE) | Low -- education-focused, not enterprise corporate LMS |
| Learnifier | Stockholm, Sweden | Corporate learning | Medium -- Nordic focus, GDPR-friendly (Swedish hosting), but limited enterprise scale |
| Storyboard AS / Cursum | Norway/Denmark | SME corporate learning | Low -- SVV's current "Kilden" provider; contract expiring. Limited enterprise features |
| Datafisher | Helsinki/Stockholm | Compliance training | Low -- niche compliance focus |

---

## 3. Detailed Platform Evaluations

### 3.1 Cornerstone Galaxy (formerly Cornerstone OnDemand)

**Vendor**: Cornerstone OnDemand, Inc. (US-based, acquired by Clearlake Capital)
**Product**: Cornerstone Galaxy platform with Learning SBX module

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | Full SCIM provisioning (documented Microsoft Entra integration), SAML 2.0 SSO, OAuth2/OIDC. Well-documented APIs for ForgeRock integration. | Strong |
| **Norwegian language** | Confirmed: Norwegian is one of 50+ supported languages. Language packs available (may require additional purchase and activation). | Strong |
| **GDPR / EU data centers** | EU data centers in France and Germany (Frankfurt). Dedicated DPO and privacy team. GDPR-Ready initiative with privacy consulting, data audits, and compliance action plans. Customers select hosting location at contract start. | Strong |
| **xAPI/SCORM/cmi5** | Full SCORM 1.2 and 2004 support. xAPI support. cmi5 support via integration (Rustici Engine compatible). | Strong |
| **API capabilities** | Comprehensive REST APIs (Edge APIs), OData reporting APIs, Data Exporter API (DEAPI), Reporting API (RAPI). Integration marketplace with 100+ connectors. | Strong |
| **AI features** | Galaxy AI (proprietary): 40+ TB labor market data, 30M learning hours/month. Adaptive Learning Agent, AI-Powered Course Assistant. Content recommendations, skills inference, agentic AI workflows (March 2026 release). Industry-leading AI investment. | Very Strong |
| **External user portal** | Extended Enterprise module: guest-user access, Portal Builder for custom portals, unlimited branded microsites, external audience segmentation, e-commerce for training monetization. | Very Strong |
| **Competency management** | Skills graph, competency frameworks, skills-based learning paths, gap analysis, workforce planning. | Strong |
| **PowerBI integration** | Official CSOD-Power-BI-Integration-Utility (open source on GitHub). OData URLs for direct Power BI Desktop connection. RTDW (Real-time Data Warehouse) access via DEAPI and RAPI. | Very Strong |
| **Mobile app** | Native iOS/Android apps. Responsive design. Offline learning support. | Strong |
| **Pricing model** | Custom enterprise pricing, subscription-based. Typically per-user with volume tiers. Language packs may be additional. Premium positioning. | Enterprise (high) |
| **Nordic/public sector references** | Highly rated in HerbertNathan Scandinavian LMS Report. Available in 180 countries. 7,000+ customers globally. Public sector vertical expertise (government page). Specific Nordic government references not publicly documented but likely given Scandinavian presence. | Moderate-Strong |

**Strengths for SVV**: Most mature enterprise platform. Best-in-class AI. Strong extended enterprise for KKP portal scenario. Official PowerBI tooling. EU data centers confirmed. 50+ languages including Norwegian.

**Weaknesses for SVV**: Complex platform -- may require significant configuration effort. Premium pricing. Language packs may require additional purchase. ForgeRock/ID-porten integration would need custom work (no out-of-box ID-porten support, but SAML/OIDC federation is standard).

---

### 3.2 Docebo

**Vendor**: Docebo Inc. (Canada/Italy-based, publicly traded on NASDAQ/TSX)
**Product**: Docebo Learning Suite (Learn LMS, Shape, Content, Learn Data)

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | SAML 2.0 SSO, SCIM provisioning (Enterprise plan), OKTA/ADFS integration, multiple authentication per tenant. | Strong |
| **Norwegian language** | 40+ languages out of the box, 50+ total. Norwegian likely supported (Docebo has Nordic presence), but not explicitly confirmed in public documentation. Needs vendor confirmation. | Moderate-Strong |
| **GDPR / EU data centers** | ISO 27001 certified, SOC 2 compliant. Hosted on AWS (EU regions available: Ireland, Frankfurt). GDPR-compliant with DPA. EU-US Privacy Shield certified. | Strong |
| **xAPI/SCORM/cmi5** | SCORM 1.2 and 2004 compliant. xAPI support. cmi5 via Rustici Engine integration. | Strong |
| **API capabilities** | Comprehensive REST APIs, webhooks, 400+ integrations. SSO providers, HRIS systems, CRM, content tools. Strong developer ecosystem. | Very Strong |
| **AI features** | AI-powered learning suite: automated enrollments, AI translations, content categorization, hyper-personalized learning paths. Creator Pack for generative AI course creation. Recognized as best-in-class for AI by analysts. | Very Strong |
| **External user portal** | Extended Enterprise module: multiple branded portals per tenant, custom URLs, separate authentication per portal, white-labeling, e-commerce gateways. Multi-tenant architecture. | Very Strong |
| **Competency management** | Skills management, development tracking, learning path alignment to roles/skills. Less mature than Cornerstone's skills graph but functional. | Moderate-Strong |
| **PowerBI integration** | Learn Data product: modeled data tables (users, enrollments, completions, certifications, skills) exported to Power BI, Tableau, Snowflake, BigQuery, or Redshift. API-based data extraction also available. | Strong |
| **Mobile app** | Native iOS/Android app. Responsive design. Offline access. White-label mobile app option. | Strong |
| **Pricing model** | Subscription based on yearly active users. Starting ~$25,000/year minimum. ~$7-10/user/month at scale. Two plans: Elevate and Enterprise. Premium positioning. | Enterprise (high) |
| **Nordic/public sector references** | Latvian School of Public Administration (60,000 civil servants, 160 entities). FedRAMP authorized (US government). UK Digital Marketplace listed. Growing public sector portfolio. No confirmed Nordic government references. | Moderate |

**Strengths for SVV**: Excellent AI capabilities. Strong extended enterprise for KKP portal. Good API/webhook ecosystem. Learn Data product well-suited for PowerBI integration. Modern, intuitive UI.

**Weaknesses for SVV**: Norwegian language support needs vendor confirmation. No confirmed Nordic public sector references. Canadian company -- data processing may involve US/Canada sub-processors (needs DPA review). Competency management less mature than Cornerstone.

---

### 3.3 Totara TXP (Talent Experience Platform)

**Vendor**: Totara Learning (New Zealand-based, partner-delivered)
**Product**: Totara TXP (Learn + Engage + Perform)

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | SAML 2.0 SSO, OIDC, Microsoft/Google/Facebook SSO. SCIM support not confirmed natively -- may require custom plugin or middleware. | Moderate |
| **Norwegian language** | Based on Moodle architecture with 36+ language packs. Norwegian likely available via Moodle language pack inheritance, but Totara-specific strings (~1/3) may need separate translation. | Moderate |
| **GDPR / EU data centers** | Hosting depends on partner (self-hosted or partner-managed cloud). Full control over data location. Can be hosted in any EU/EEA data center. Open source = full data sovereignty. | Strong (flexible) |
| **xAPI/SCORM/cmi5** | SCORM compliant. xAPI support via plugins. cmi5 support limited/plugin-dependent. | Moderate-Strong |
| **API capabilities** | GraphQL API (Totara 13+). Web services API. Less extensive integration marketplace than Cornerstone/Docebo. Plugin architecture for custom integrations. | Moderate |
| **AI features** | Limited native AI compared to Cornerstone/Docebo. AI capabilities depend on partner implementations and plugins. No proprietary AI engine. | Weak |
| **External user portal** | Multi-tenancy support: separate branded environments per division/partner with central control. Custom themes, roles, and content per tenant. | Strong |
| **Competency management** | Very strong: competency frameworks, auto-assignment based on rules, learning plans, certification management, position-based hierarchies. Core strength of the platform. | Very Strong |
| **PowerBI integration** | No native PowerBI connector. Data extraction via database access (self-hosted) or API. Would require custom development. | Weak-Moderate |
| **Mobile app** | Responsive design. Mobile app available. Offline learning capabilities via plugins. | Moderate |
| **Pricing model** | Annual subscription per user via Totara partners. Open-source base = no lock-in. Typically lower cost than Cornerstone/Docebo. Partner implementation costs vary. | Cost-effective |
| **Nordic/public sector references** | Used by 1,500+ organizations worldwide. No specifically confirmed Nordic government references. Strong in UK/Australia public sector and education. | Moderate |

**Strengths for SVV**: Best competency management (aligns well with SVV's competency framework needs and ServiceNow integration). Open-source = full data sovereignty and no vendor lock-in. Cost-effective. Flexible multi-tenancy. Strong compliance/certification workflows.

**Weaknesses for SVV**: Weak AI capabilities. No native PowerBI integration. SCIM support uncertain. Requires strong implementation partner. Norwegian language may need work. Less polished UX than Cornerstone/Docebo.

---

### 3.4 SAP SuccessFactors Learning

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | Full SAP Identity Authentication/Provisioning services. SAML 2.0. | Strong |
| **Norwegian language** | Supported (SAP has extensive Nordic presence) | Strong |
| **GDPR / EU data centers** | SAP EU data centers (Germany, Netherlands). Strong GDPR posture. | Very Strong |
| **xAPI/SCORM/cmi5** | SCORM support. xAPI limited. | Moderate |
| **API capabilities** | SAP Integration Suite. OData APIs. Complex but powerful. | Moderate-Strong |
| **AI features** | SAP Joule AI assistant. AI-based recommendations. | Moderate |
| **External user portal** | Limited external user capabilities compared to dedicated LMS. | Weak |
| **Competency management** | Strong -- part of SuccessFactors HCM suite. | Strong |
| **PowerBI integration** | SAP Analytics Cloud primary. PowerBI possible via OData. | Moderate |
| **Pricing** | Suite licensing (expensive). Best when part of SAP HCM. | Very High |

**Verdict**: Best fit only if SVV already uses SAP HCM. Overkill for standalone LMS. External user portal capabilities insufficient for KKP scenario. Not recommended as standalone.

---

### 3.5 Microsoft Viva Learning

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | Azure AD / Entra ID native. | Strong |
| **Norwegian language** | Full Microsoft localization. | Strong |
| **GDPR / EU data centers** | EU Data Boundary. | Very Strong |
| **Overall LMS capability** | Content aggregator, NOT a full LMS. No enrollment management, no compliance tracking, no competency management, no external users. Microsoft GM stated: "We are not going to be in a position to have the same level of specificity that an LMS can offer." | Very Weak |

**Verdict**: Not suitable as standalone LMS. Could complement an LMS for Microsoft 365 content delivery but cannot meet SVV's requirements for course management, compliance, external portals, or competency tracking. **Not recommended.**

---

### 3.6 Absorb LMS

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | SAML SSO. SCIM support unclear. | Moderate |
| **Norwegian language** | Not confirmed in supported languages. | Weak |
| **GDPR / EU data centers** | GDPR compliant. SOC 2. EU data center not confirmed. | Moderate |
| **xAPI/SCORM/cmi5** | SCORM and xAPI support. | Strong |
| **API capabilities** | RESTful APIs, webhooks (event-driven). Well-documented. | Strong |
| **AI features** | Absorb Intelligent Assist. AI recommendations. | Moderate |
| **External user portal** | Extended enterprise capabilities exist but less mature. | Moderate |
| **Competency management** | Competency tracking via webhooks. Basic. | Moderate |
| **PowerBI integration** | Via API data extraction. No native connector. | Moderate |
| **Pricing** | Per-user subscription. Mid-range. | Moderate |

**Verdict**: Solid mid-market LMS but lacks confirmed Norwegian language support and EU data center options. Not ideal for Norwegian public sector with external user requirements at SVV's scale. **Not in top 3.**

---

### 3.7 Moodle Workplace

| Criterion | Assessment | Rating |
|---|---|---|
| **SCIM/SAML/OAuth2** | SAML via plugin. SCIM not natively supported. | Moderate |
| **Norwegian language** | Full Norwegian language pack (Moodle heritage). | Strong |
| **GDPR / EU data centers** | Depends on hosting partner. Full control if self-hosted. | Strong (flexible) |
| **xAPI/SCORM/cmi5** | SCORM, xAPI via plugins. | Moderate-Strong |
| **API capabilities** | Web services API. Less sophisticated than commercial LMS. | Moderate |
| **AI features** | Limited native AI. Plugin-dependent. | Weak |
| **External user portal** | Multi-tenancy with separate branded environments. | Strong |
| **Competency management** | Automatic assignment and competency granting. | Moderate-Strong |
| **PowerBI integration** | Database access or custom API. No native connector. | Weak |
| **Pricing** | Open source (partner costs for Workplace edition). | Cost-effective |

**Verdict**: Similar profile to Totara but less mature for corporate learning. Moodle heritage = strong Norwegian language support and flexibility. However, enterprise UX and AI capabilities lag behind commercial alternatives. Could be considered as a cost-effective alternative if budget is constrained. **Honorable mention but not top 3.**

---

### 3.8 Other Platforms (Brief Assessment)

| Platform | Key Strength | Key Weakness for SVV | Verdict |
|---|---|---|---|
| **360Learning** | Collaborative learning, GDPR (ISO 27001, SOC 2), SAML/SCIM (Enterprise plan) | Limited competency management, no confirmed Norwegian support, collaborative focus may not fit SVV's compliance-heavy needs | Not recommended |
| **iSpring Learn** | Norwegian (Bokmal) confirmed, 28 languages, integrated authoring | SME-focused, limited enterprise scale, basic competency management, no extended enterprise portal at scale | Not recommended |
| **Kallidus** | Norwegian language confirmed, UK-focused enterprise LMS | Limited Nordic market presence, no confirmed EU data center outside UK, primarily UK public sector | Not recommended |
| **TalentLMS** | Easy setup, affordable | SME-focused, limited enterprise features, insufficient for SVV's complexity | Not recommended |
| **Litmos** | SCORM/xAPI, compliance focus | Ownership changes (acquired multiple times), limited competency management, unclear Nordic support | Not recommended |
| **Learn Amp** | LMS+LXP hybrid, skills management, UK enterprise customers | No confirmed Norwegian support, small vendor, limited public sector references | Not recommended |
| **Learnifier** | Swedish-hosted (GDPR-friendly), Nordic focus, SCORM support | Limited enterprise scale for 5,000+ users + 15,000 external, basic competency management | Could supplement but not primary LMS |

---

## 4. Comparison Matrix

### 4.1 Requirements Coverage Matrix

| Requirement | Weight | Cornerstone Galaxy | Docebo | Totara TXP | SAP SF Learning | Absorb |
|---|---|---|---|---|---|---|
| **IAM (SCIM/SAML/OAuth2/OIDC)** | Critical | 5 | 5 | 3 | 5 | 3 |
| **ForgeRock/ID-porten compatibility** | Critical | 4 | 4 | 3 | 4 | 3 |
| **Norwegian language (Bokmal)** | Critical | 5 | 4* | 3* | 5 | 2 |
| **GDPR / EU data residency** | Critical | 5 | 4 | 5 | 5 | 3 |
| **SCORM/xAPI support** | Critical | 5 | 5 | 4 | 4 | 5 |
| **cmi5 support** | Desired | 4 | 4 | 3 | 2 | 3 |
| **REST API / webhooks** | Critical | 5 | 5 | 3 | 4 | 4 |
| **AI content recommendations** | High | 5 | 5 | 2 | 3 | 3 |
| **AI content generation** | High | 5 | 5 | 1 | 2 | 2 |
| **External user portal (KKP)** | Critical | 5 | 5 | 4 | 2 | 3 |
| **Competency management** | Critical | 4 | 3 | 5 | 4 | 3 |
| **PowerBI integration** | High | 5 | 4 | 2 | 3 | 3 |
| **Reporting/analytics** | High | 5 | 5 | 3 | 4 | 4 |
| **Mobile app / responsive** | High | 5 | 5 | 3 | 4 | 4 |
| **Multi-tenant / branded portals** | High | 5 | 5 | 4 | 2 | 3 |
| **Classroom/ILT management** | High | 5 | 4 | 4 | 4 | 4 |
| **Compliance/certification tracking** | Critical | 5 | 4 | 5 | 4 | 4 |
| **Social/collaborative learning** | Medium | 4 | 4 | 3 | 2 | 3 |
| **WCAG accessibility** | Critical | 4 | 4 | 4 | 4 | 4 |
| **Scalability (20,000+ users)** | Critical | 5 | 5 | 4 | 5 | 4 |
| **Vendor stability/maturity** | High | 5 | 4 | 3 | 5 | 3 |

**Scoring**: 5 = Fully meets/exceeds, 4 = Mostly meets, 3 = Partially meets, 2 = Limited, 1 = Not supported
*Asterisk = needs vendor confirmation

### 4.2 Weighted Score Summary

| Platform | Weighted Score (approx.) | Rank |
|---|---|---|
| **Cornerstone Galaxy** | 4.7 / 5.0 | 1 |
| **Docebo** | 4.4 / 5.0 | 2 |
| **Totara TXP** | 3.4 / 5.0 | 3 |
| **SAP SuccessFactors** | 3.6 / 5.0 | 4 (but not standalone) |
| **Absorb LMS** | 3.3 / 5.0 | 5 |

---

## 5. Pricing Estimation for SVV

### 5.1 Estimated Annual Costs

Based on publicly available pricing data and typical enterprise negotiations for ~5,300 internal + ~1,100 consultant + ~15,000 external users:

| Platform | Est. Annual License | Est. Implementation (Year 1) | Est. 6-Year Total | Notes |
|---|---|---|---|---|
| **Cornerstone Galaxy** | 2.5-4.0 MNOK | 3-5 MNOK | 20-30 MNOK | Volume discounts likely for 20k+ users. Language packs may be additional. Extended Enterprise module pricing separate. |
| **Docebo** | 2.0-3.5 MNOK | 2-4 MNOK | 16-26 MNOK | Based on yearly active users. External users may have different pricing tier. Learn Data add-on for PowerBI. |
| **Totara TXP** | 1.0-2.0 MNOK | 3-5 MNOK | 12-18 MNOK | Lower license cost but higher implementation effort. Partner costs for hosting, customization, and Norwegian localization. |

*Note: All estimates are approximate and subject to vendor negotiation. SVV's budget of 20-35 MNOK over 6 years is within range for all three recommended platforms.*

### 5.2 Pricing Model Fit

SVV requests both site license and per-user pricing options (Bilag 1F). Key considerations:

- **Cornerstone**: Typically per-user subscription. May offer tiered pricing for internal vs. external users.
- **Docebo**: Yearly active users model. External users who access infrequently may cost less per user.
- **Totara**: Per-user annual subscription through partners. Open source = no per-user license ceiling for self-hosted components.

---

## 6. Integration Fit Assessment

### 6.1 ForgeRock AM/IDM Integration

All three recommended platforms support SAML 2.0 and OAuth2/OIDC, which are the standard protocols for ForgeRock federation. Key considerations:

- **SCIM provisioning**: Cornerstone and Docebo have confirmed SCIM support. Totara may need custom development or middleware.
- **ID-porten**: ID-porten uses OpenID Connect (primarily) and SAML. Integration would be via ForgeRock as identity broker. All platforms can receive federated authentication.
- **BankID**: Handled at the ForgeRock/ID-porten layer, transparent to the LMS.
- **MFA**: Delegated to ForgeRock. LMS platforms need to support SP-initiated SSO correctly.

### 6.2 ServiceNow Integration

- **Cornerstone**: Pre-built ServiceNow connector available in integration marketplace.
- **Docebo**: REST API + webhooks enable ServiceNow integration (custom build required).
- **Totara**: Web services API for custom ServiceNow integration. Requires more development effort.

### 6.3 PowerBI Integration

- **Cornerstone**: Best-in-class. Official open-source utility, OData-based reporting APIs, Real-time Data Warehouse access.
- **Docebo**: Strong. Learn Data product provides modeled tables to PowerBI/Snowflake/BigQuery. Alternative: API-based extraction.
- **Totara**: Weakest. Requires custom data pipeline from database or API to PowerBI.

### 6.4 Kursoppslag Proxy API

All platforms have REST APIs that can be wrapped by SVV's Kursoppslag proxy. Key: the LMS API must expose endpoints for course participants, course info, and learning plan data. Cornerstone and Docebo have the most mature REST API documentation.

---

## 7. Risk Analysis per Platform

### 7.1 Cornerstone Galaxy

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Complexity of implementation | High | Medium | Experienced Nordic implementation partner. Phased rollout. |
| Premium pricing exceeds budget | Medium | High | Negotiate volume pricing. Explore EU-specific pricing. |
| Language pack costs | Low | Low | Confirm Norwegian included in base or negotiate inclusion. |
| PE ownership (Clearlake Capital) may affect long-term strategy | Low | Medium | 6-year contract with SLA protections. |

### 7.2 Docebo

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Norwegian language confirmation | Medium | High | Confirm before bid. If not available, localization tool allows custom translations. |
| Canadian parent company -- GDPR sub-processor risk | Medium | Medium | Ensure DPA specifies EU-only data processing. Confirm AWS EU-only hosting. |
| Limited Nordic public sector references | Medium | Low | Latvian public administration reference. Growing European public sector portfolio. |
| Learn Data as separate add-on cost | Medium | Low | Include in pricing negotiation. |

### 7.3 Totara TXP

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Lack of native AI capabilities | High | High | Supplement with third-party AI tools. Accept limitation vs. competitors. |
| SCIM provisioning gap | High | Medium | Custom development or middleware solution. Budget additional integration hours. |
| Implementation partner dependency | High | Medium | Select experienced Nordic Totara partner (e.g., Catalyst EU). |
| Norwegian translation completeness | Medium | Medium | Leverage Moodle community Norwegian translations. Budget for Totara-specific string translation. |
| PowerBI integration effort | High | Medium | Budget for custom data pipeline development (150h+). |

---

## 8. Recommendation

### 8.1 Primary Recommendation: Cornerstone Galaxy

Cornerstone Galaxy is the strongest overall fit for SVV's requirements:

- **Comprehensive feature coverage**: Highest scores across nearly all evaluation criteria
- **Enterprise maturity**: 7,000+ customers, 100M+ users, available in 180 countries
- **AI leadership**: Most advanced AI capabilities in the market (Galaxy AI, Adaptive Learning Agent)
- **Extended Enterprise**: Best-in-class external portal for KKP competency certificate scenario
- **PowerBI**: Official integration utility and OData reporting APIs
- **EU data sovereignty**: Confirmed data centers in France and Germany
- **Norwegian language**: Confirmed support (50+ languages)
- **Scandinavian validation**: Highly rated in HerbertNathan & Co. Nordic LMS Report

**Risk**: Higher cost and implementation complexity. Requires experienced implementation partner.

### 8.2 Strong Alternative: Docebo

Docebo is an excellent alternative if:

- SVV prioritizes AI-powered learning experiences and modern UX
- The budget favors a platform with strong extended enterprise at potentially lower total cost
- Norwegian language support is confirmed during vendor engagement

**Key advantage over Cornerstone**: Potentially more intuitive/modern user interface (relevant for the 12.5% UX weighting in award criteria). Learn Data product is well-structured for analytics.

### 8.3 Value Alternative: Totara TXP

Totara TXP should be considered if:

- SVV wants to minimize vendor lock-in (open source)
- Data sovereignty is the absolute top priority (self-hosted option)
- Competency management is weighted more heavily than AI capabilities
- Budget constraints require a more cost-effective solution

**Key advantage**: Best competency management framework, aligning well with SVV's ServiceNow competency register and the emphasis on structured compliance/certification tracking.

### 8.4 Recommended Bid Strategy

For the bid response, we recommend:

1. **Lead with Cornerstone Galaxy** as primary platform recommendation due to broadest requirement coverage and lowest delivery risk
2. **Prepare Docebo as alternative** in case vendor engagement reveals better pricing or UX advantage
3. **Reference Totara TXP** as evidence of market knowledge and to demonstrate consideration of open-source, cost-effective alternatives
4. **Regardless of platform**: The bid should emphasize our integration expertise (ForgeRock, ServiceNow, PowerBI), migration methodology (Kilden data), and Norwegian public sector understanding

---

## 9. Key Vendor Contact Points for Due Diligence

Before finalizing the bid, the following should be confirmed with vendors:

| Item | Vendor | Priority |
|---|---|---|
| Norwegian Bokmal language pack -- included or additional cost? | Cornerstone | High |
| Norwegian language support -- confirm Bokmal is in the 40+ language list | Docebo | Critical |
| EU-only data processing (no US/Canada sub-processor access) | Docebo | Critical |
| Extended Enterprise pricing for 15,000 external users | Cornerstone, Docebo | High |
| ForgeRock AM/IDM integration reference cases | All three | High |
| ID-porten/BankID federation experience | All three | High |
| SCIM provisioning native support | Totara | High |
| Nordic public sector references (available for SVV to contact) | All three | Medium |
| cmi5 native support status | All three | Medium |
| PowerBI integration licensing/add-on costs | Docebo (Learn Data) | Medium |

---

## 10. Sources and References

- HerbertNathan & Co., "Learning Management Systems in Scandinavia" (Nordic LMS Report)
- Cornerstone OnDemand Trust Center and product documentation (cornerstoneondemand.com)
- Docebo Help & Support, product pages, and compliance documentation (docebo.com)
- Totara Learning product documentation and community (totara.com, totara.help)
- Gartner Peer Insights: Corporate Learning Technologies comparisons
- G2 and Capterra platform reviews (2025-2026)
- Research.com platform reviews and feature analyses
- People Managing People: Enterprise LMS rankings (2026)
- Talent eLearning: Enterprise LMS Trends 2026
