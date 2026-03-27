# Risk Assessor Review of Delivery Plan v1.0

**Reviewer**: Risk Assessor
**Date**: 2026-03-22
**Document reviewed**: DELIVERY_PLAN_v1.md
**Cross-referenced against**: 09_risk_assessment.md
**Verdict**: **CONDITIONAL APPROVAL -- requires revisions before submission**

---

## Overall Assessment

The delivery plan is a strong first draft that demonstrates good understanding of the tender, integrates inputs from all specialists, and correctly identifies the critical path through IAM integration. However, it has several weaknesses that SVV's evaluators -- particularly IT architects Ringstad and Kowalski, and procurement specialist Lidi -- will scrutinize. The plan is optimistic in places where credibility demands conservatism, and underspecified in areas where SVV's evaluation criteria demand precision.

**Rating: 3.5/5 -- Good foundation, needs hardening before submission.**

---

## 1. TIMELINE: THE 28-WEEK PLAN IS STILL AGGRESSIVE

### Finding
The plan proposes 28 weeks (Phases 1-6) with a go-live estimate of "Q1/Q2 2027" and correctly flags the 01.01.2027 target as CRITICAL risk. However, the 28-week timeline itself has structural problems:

### Issues

**1.1 Phase overlap is underspecified.** The plan says "phases overlap" but the week ranges show inconsistencies:
- Phase 2 (W3-W6) overlaps with Phase 3 (W5-W10) -- L02 needs analysis must produce the learning architecture *before* L03 configuration can start meaningfully. Starting configuration while needs analysis is still running creates rework risk.
- Phase 4 (W7-W14) starts integrations while core platform (Phase 3, W5-W10) may not be fully configured. ServiceNow integration (L05) requires understanding of the skills framework mapping, which depends on L02/L03 decisions.
- Phase 5 (W11-W16) starts migration while integrations (Phase 4, W7-W14) are still running. Migration testing requires working integrations (especially SCIM/IAM) to validate migrated users.

**1.2 The 4-week acceptance test (Phase 6, W15-W20) starts at Week 15, but SSA-L specifies 20 virkedager (business days) = 4 calendar weeks.** This means:
- Leveransemelding must be submitted at W15.
- Any A-feil or exceeded B-feil thresholds triggers a second round with 10 virkedager and *stricter* criteria (0 A-feil, 0 B-feil, 10 C-feil).
- The plan does not account for a potential second acceptance test round, which would add 2-3 weeks.
- With Phase 6 at W15-W20, the plan has zero buffer for a second round.

**1.3 The plan shows 20 weeks to W20, not 28.** The heading says 28 weeks but the phase table sums to 20 weeks of activity (W1-W20). The discrepancy is unexplained. If the intent is 28 calendar weeks with buffer, this should be explicitly stated and the buffer weeks identified.

### Recommendation
- **Add explicit buffer**: Define W21-W24 as contingency/second acceptance round. Show W25-W28 as hypercare and parallel operations.
- **Fix phase dependencies**: L02 must complete before L03 configuration decisions. Show this as a hard gate, not an overlap.
- **Add second acceptance test** to the plan as a contingency path with 2-3 week extension.
- **Clarify the 20-week vs 28-week discrepancy** -- this will confuse SVV evaluators.

**Risk assessment reference**: L-01 (CRITICAL), K-03 (HIGH)

---

## 2. INTEGRATION COMPLEXITY IS ACKNOWLEDGED BUT UNDERPLANNED

### Finding
Section 3.1 correctly identifies all 6 integrations and their protocols. However, the plan lacks the depth that SVV's IT architects will expect.

### Issues

**2.1 ADR-002 ("SVV owns Kursoppslag mapping layer; LMS exposes standard APIs") is problematic.** The Kursoppslag integration document explicitly states: "Leverandor bes besvare muligheter for at deres losning kan returnere felt som listes i integrasjonsbeskrivelsen, slik at Kursoppslag kan beholde sin funksjonalitet." SVV expects the *LMS vendor* to ensure field compatibility, not to push this to SVV. This ADR could be interpreted as the vendor avoiding responsibility.

**2.2 ADR-004 ("no middleware needed") is a bold claim.** With 4 mandatory integrations using different auth mechanisms (SCIM, OAuth2, SAML, REST + Maskinporten for machine-to-machine), the absence of middleware means the LMS itself must handle all protocol translation. This may be fine for major platforms but needs validation against the specific LMS chosen.

**2.3 The Kursoppslag integration is listed as "Part of L03" (fixed price).** This underestimates it. The Kursoppslag API has 6 endpoints with specific field mappings (kursdeltakere, kursinfo, laeringsplandeltakere, laeringsplaninfo, plus v1 legacy endpoints). Building a mapping layer between the new LMS's data model and these exact field formats is non-trivial integration work, not "configuration." Pricing this under fixed-price L03 instead of as a dedicated integration deliverable creates commercial exposure.

**2.4 ServiceNow integration complexity is underestimated.** The plan estimates 100h (T&M) with a realistic range of 120-150h. However, the ServiceNow integration is bidirectional: LMS must receive jobbfamilier, jobbroller, and ferdigheter *from* ServiceNow, and send laeringslogg *to* ServiceNow (some via Kursoppslag, some direct). SVV has a 225 MNOK/15-year ServiceNow contract with Sopra Steria -- any integration with ServiceNow must coordinate with Sopra Steria's team, adding a third-party dependency not acknowledged in the plan.

**2.5 No mention of Maskinporten.** The integration document lists Maskinporten for machine-to-machine authentication. The plan's security architecture section mentions it nowhere. SVV's IAM team will expect this to be addressed.

### Recommendation
- **Rephrase ADR-002**: The vendor takes responsibility for exposing APIs that are compatible with Kursoppslag's field requirements. SVV maintains the Kursoppslag proxy.
- **Create a dedicated Kursoppslag integration deliverable** or explicitly scope it within L03 with an hour estimate.
- **Increase ServiceNow estimate to 150-200h** and explicitly note the Sopra Steria coordination dependency.
- **Add Maskinporten** to the security architecture section.
- **Add an integration dependency diagram** showing the sequence and dependencies between integrations.

**Risk assessment reference**: T-01 (CRITICAL), T-02 (HIGH), T-05 (MEDIUM)

---

## 3. MIGRATION RISK TREATMENT IS GOOD BUT INCOMPLETE

### Finding
The plan correctly identifies the 150h vs 250h gap and recommends the realistic 250h estimate. However, critical migration details are missing.

### Issues

**3.1 No rollback strategy.** What happens if migration fails validation after cutover? The plan mentions "Audit > Classify > Pilot > Full migration > Validation" but does not address: How long does SVV stay on Kilden if migration fails? Is there a parallel-running period? What is the cutover/rollback decision point?

**3.2 Historical data scope is undefined.** L08 requires "fullverdig migrering av innhold, metadata, tilgangsstrukturer og historikk." With 18,500 completions in 2025 alone, how many years of historical data must be migrated? The plan doesn't address this. There's a significant difference between migrating 1 year vs 5 years of completion history, and this directly impacts the hour estimate.

**3.3 Open Item #6 (Section 16) asks "Bid 150h (tender) or realistic 250h?"** This should not be an open question at this stage. The answer determines both our commercial risk and our credibility with SVV. My recommendation:
- **Bid 150h as the tender estimate** (matching SVV's budget expectation) but **include a scope definition** that specifies exactly what 150h covers (e.g., SCORM packages and active course metadata only).
- **Disclose migration complexity** in our solution description and flag that scope expansion (historical data, certificate archives, custom content formats) may require additional hours.
- This is more credible than bidding 250h against SVV's 150h estimate, which could appear as padding.

**3.4 Storyboard AS cooperation.** The plan mentions "contractual cooperation clause" as mitigation for L-04. But the cooperation obligation lies in *SVV's* contract with Storyboard, not in ours. We cannot require SVV to contractually bind Storyboard to cooperate. The plan should instead focus on what we can control: accepting data in whatever format SVV can extract, building migration tooling that doesn't require Storyboard's active participation.

### Recommendation
- **Add explicit rollback strategy** with decision point and parallel-running timeline.
- **Define historical data scope** as part of L01/L02 activities.
- **Resolve Open Item #6**: Bid 150h with scope definition, flag expansion risk.
- **Reframe Storyboard dependency**: Plan for minimal cooperation scenario as the base case.

**Risk assessment reference**: T-03 (HIGH), L-04 (HIGH)

---

## 4. MVP/INCREMENTAL APPROACH NEEDS DEFINITION

### Finding
The plan repeatedly references an "MVP approach" and "incremental delivery" but never defines what the MVP includes or excludes.

### Issues

**4.1 What is in MVP?** SVV evaluators will ask: If you propose Q1 2027 as MVP go-live and Q2 2027 as full scope, exactly what functionality is available at each stage? The plan must define:
- **MVP (Q1 2027)**: What deliverables? Which integrations? What user groups?
- **Full scope (Q2 2027)**: What is deferred?
- **Options timeline**: When can L21-L24 be exercised?

**4.2 SSA-L has a single "Leveringsdag."** The contract framework defines one delivery date, one acceptance test, one transition from implementation to operations. Proposing multiple "delivery days" must be explicitly negotiated with SVV -- it's not standard SSA-L. The plan should acknowledge this and propose how multiple milestones map to the SSA-L framework (e.g., incremental acceptance with a final Leveringsdag).

**4.3 From SVV's perspective, the MVP approach could look like risk-shifting.** SVV wants a working platform by Q1 2027. If we propose "MVP in Q1, full scope in Q2," they may interpret this as us hedging against our own delivery risk at their expense. The framing matters: position it as *phased value delivery* (they get usable functionality sooner) rather than *deferred completion* (we need more time).

### Recommendation
- **Define MVP scope explicitly**: I suggest MVP = L01 + L02 + L03 + L04 (SSO) + L07 (basic learning surfaces) + L09 (core training). This gives SVV a working platform with SSO and basic course delivery by Q1 2027.
- **Define Phase 2 scope**: L05 (ServiceNow), L06 (PowerBI), L08 (full migration) complete by Q2 2027.
- **Frame as "phased value delivery"**, not deferred risk.
- **Address how this maps to SSA-L's single Leveringsdag** -- propose a formal delivery of Phase 1 scope with a contractual amendment for Phase 2.

**Risk assessment reference**: L-01 (CRITICAL)

---

## 5. COMMERCIAL EXPOSURE AND CASH FLOW

### Finding
Section 9 acknowledges the cash flow challenge (no invoicing until acceptance test) but does not quantify the exposure or propose concrete mitigation.

### Issues

**5.1 Unquantified cash flow gap.** If contract signing is September 2026 and Leveringsdag is Q2 2027 (8 months), the vendor carries 8 months of implementation cost with no revenue (except T&M work on L05, L06, L08 which may be invoiced monthly). For a team of 4-5 FTEs, this is approximately 3-5 MNOK in unbilled labor plus LMS platform licensing costs. The plan should quantify this.

**5.2 The SSA-L penalty rate of 0.15%/day** (Section 15.1) is mentioned but not analyzed. 0.15% of what base? The first 6 months of vederlag. If annual SaaS + support is approximately 4 MNOK, then 6 months = 2 MNOK, and 0.15%/day = 3,000 NOK/day. Over 20 business days of delay, penalties could reach 60,000 NOK -- manageable but worth understanding. However, if the base includes implementation fees, the number could be significantly higher.

**5.3 The T&M invoicing schedule for L05/L06/L08 is not specified.** The plan says T&M work is invoiced "per agreed schedule" but doesn't propose a schedule. SSA-L Bilag 6 states T&M is invoiced monthly after Leveringsdag -- meaning T&M work *before* Leveringsdag follows the same "after acceptance" rule unless specifically negotiated otherwise. This needs clarification.

**5.4 The 22-28 MNOK pricing target** is characterized as "mid-to-lower." Given the integration complexity, migration scope, 6-year support commitment, and quality-heavy evaluation (80%), pricing below 25 MNOK may undermine credibility. SVV's envelope goes to 35 MNOK. A bid at 22 MNOK for this scope signals either cutting corners or unsustainable margins.

### Recommendation
- **Quantify cash flow exposure**: Model the unbilled labor cost from Sept 2026 to Leveringsdag.
- **Negotiate milestone payments**: In Round 1 negotiations, propose partial payment at interim milestones (L01 completion, L04 completion) rather than all-or-nothing at acceptance.
- **Clarify T&M invoicing timing** in the pricing structure.
- **Raise pricing target to 25-30 MNOK** given scope complexity and quality-dominant evaluation.

**Risk assessment reference**: K-03 (HIGH), K-01 (HIGH), K-02 (MEDIUM)

---

## 6. SLA RISKS ARE UNDERSTATED

### Finding
Section 11 (Risk Register) lists O-01 (6-year SLA sustainability) as HIGH but the plan's treatment of SLA in other sections is thin.

### Issues

**6.1 No SLA targets proposed.** The plan's compliance checklist references availability but never states what uptime the vendor will commit to. 99.5%? 99.9%? 99.95%? This directly affects penalty exposure and should be modeled.

**6.2 "No maintenance during exams" is acknowledged but not operationalized.** Who defines exam periods? How far in advance? What about emergency patches during exam periods? The plan should propose a maintenance window policy.

**6.3 Norwegian-language support sustainability.** The plan mentions first- and second-line support in Norwegian but does not address how this is staffed. If the vendor is not Norwegian-based, maintaining Norwegian L1/L2 support over 6 years requires a dedicated local team or partner. This cost must be in the pricing model.

**6.4 Bilag 4 requires the vendor to describe economic compensation for SLA breaches.** The plan does not propose a compensation model. This is a gap that will be evaluated.

### Recommendation
- **Propose specific SLA targets** with compensation model (e.g., service credits at 5% of monthly fee per 0.1% below target).
- **Define maintenance window policy** including exam blackout procedure.
- **Budget Norwegian L1/L2 support** explicitly.
- **Draft Bilag 4 response** with SLA compensation structure.

**Risk assessment reference**: O-01 (HIGH), K-03 (HIGH)

---

## 7. KEY PERSONNEL DEPENDENCIES

### Finding
Section 8 defines 7 roles but only the Project Manager is identified as SSA-L key personnel.

### Issues

**7.1 The SSA-L contract says "Prosjektleder er definert som nokkelpersonell" but also "det er onskelig at Leverandoren fremlegger CV for sentrale utviklere eller andre tekniske roller."** By submitting only the PM's CV, we signal minimal commitment. Submitting PM + Solution Architect CVs shows depth and confidence.

**7.2 The Solution Architect (0.8 FTE, Phases 1-4) is the most critical technical role but disappears after Phase 4 (Week 14).** What happens when integration issues surface during Phase 5-6 testing? Who handles architecture-level decisions during acceptance testing? The plan should keep the architect through Phase 6, even at reduced allocation.

**7.3 AI/ML Engineer at 0.3 FTE** for an AI-phased approach starting at Phase 3 is reasonable but may be insufficient if AI features are evaluated during the POC demonstration. The demo strategy (Section 6.2) includes "AI recommendations" -- who builds the demo AI features?

### Recommendation
- **Submit CVs for both PM and Solution Architect** as key personnel.
- **Extend Solution Architect allocation** through Phase 6 at 0.3 FTE.
- **Clarify AI demo responsibility**: Who configures AI features for the 80-minute POC?

**Risk assessment reference**: L-02 (MEDIUM), K-04 (MEDIUM)

---

## 8. COMPLIANCE GAPS

### Finding
Section 13 covers the major compliance areas but has several gaps.

### Issues

**8.1 Arkivloven treatment is insufficient.** The plan lists "Option L24 (Mime) or built-in certificate archiving" for arkivloven compliance. But competency certificates are *mandatory* archival documents (arkivpliktige). You cannot make compliance dependent on an option that SVV may choose not to exercise. The plan must describe how archiving requirements are met *without* L24, and position L24 as the enhanced automated solution.

**8.2 No DPIA (Data Protection Impact Assessment) is mentioned.** For a system processing competency data, completion records, and personal information for 5,300+ employees, a DPIA is likely required under GDPR Article 35. The plan should include DPIA as a deliverable within L01.

**8.3 ID-porten OIDC-only since January 2026 is mentioned but not architecturally addressed.** If the LMS platform supports SAML but not OIDC natively for external users, ForgeRock must broker the OIDC-to-SAML translation. This adds a dependency and potential failure point. The plan should confirm which auth flow applies for external users (ID-porten via ForgeRock vs direct OIDC).

**8.4 ROS-analyse (risk and vulnerability assessment) is a requirement (RO-01) but not included as a deliverable.** It should be part of L01 or L03.

### Recommendation
- **Define archiving without L24**: Built-in PDF/A certificate generation with manual/batch export to Mime-compatible format.
- **Add DPIA to L01 deliverables.**
- **Specify external user auth flow**: ID-porten -> ForgeRock (OIDC broker) -> LMS (SAML/OIDC). Confirm platform support.
- **Add ROS-analyse to L01 or L03.**

**Risk assessment reference**: C-01 (HIGH), C-02 (MEDIUM), C-04 (MEDIUM)

---

## 9. PLATFORM RECOMMENDATION RISK

### Finding
Cornerstone Galaxy is recommended with Docebo as alternative. Both are defensible choices. However:

### Issues

**9.1 Neither platform's ForgeRock SCIM integration has been verified.** Open Item #9 asks "Has anyone done ForgeRock/Cornerstone?" -- this is a critical unknown. If the answer is "no," we are proposing an unproven integration as the critical path (T-01 CRITICAL, T-02 HIGH). This must be resolved before the bid, not left as an open item.

**9.2 Docebo's Canadian parent (Docebo Inc., Toronto)** creates GDPR/data transfer risk. The plan mentions this but doesn't assess it against SVV's strict requirements. PEO-25 requires EU/EEA data storage. A Canadian parent with potential access to EU data may trigger SVV's personvernjurist to reject the platform. This should be flagged more prominently.

**9.3 The "Not Recommended" list dismisses Totara as "weak AI, no PowerBI connector."** Totara is open-source and highly customizable, which some Norwegian public sector organizations value for sovereignty. If a competitor bids Totara, we need a counter-argument beyond "weak AI." The ghosting analysis (Section 12.3) doesn't consider a Totara-based bid.

### Recommendation
- **Resolve ForgeRock/SCIM verification before bid submission** -- this is a P0 blocker.
- **If Docebo is the alternative, explicitly address the Canadian parent GDPR risk** with a mitigation plan (EU-only processing agreement, no data access from Canada).
- **Strengthen the Totara counter-argument**: Focus on SaaS vs self-hosted operational risk, enterprise support maturity, and TCO including platform hosting costs.

**Risk assessment reference**: T-02 (HIGH), T-04 (MEDIUM), C-01 (HIGH)

---

## 10. RISKS FROM ORIGINAL ASSESSMENT NOT ADEQUATELY ADDRESSED

Cross-referencing against 09_risk_assessment.md, the following risks are either missing or inadequately treated in the delivery plan:

| Risk ID | Risk | In Plan? | Assessment |
|---------|------|----------|------------|
| T-04 | Data sovereignty / EU/EEA transfers | Partially | Mentioned in security architecture but sub-processor management not detailed |
| T-06 | Performance at scale / seasonal peaks | Missing | No load testing or capacity planning mentioned |
| L-05 | Change management across 6 divisions | Partially | L02 workshops mentioned but no change management strategy |
| L-06 | Parallel operations / cutover strategy | Missing | No cutover plan or parallel-running strategy |
| K-04 | Key personnel lock-in / replacement costs | Partially | Mentioned in Section 8.2 but not costed in pricing |
| C-02 | Arkivloven / competency certificate archiving | Inadequate | Dependent on option L24 -- see Issue 8.1 |
| C-04 | InfoSec / pentesting / ROS-analyse | Missing | No ROS-analyse, no pentest plan |
| O-02 | Platform vendor dependency / pricing changes | Missing | No exit strategy or vendor lock-in mitigation |
| O-03 | Knowledge transfer at contract end | Missing | No transition-out plan |
| O-04 | Content production sustainability | Missing | No plan for ongoing content production tooling support |

### Recommendation
Add sections or sub-sections addressing:
- Load testing plan (pre-acceptance)
- Change management approach (beyond workshops)
- Cutover strategy (Kilden -> new LMS transition plan)
- Transition-out plan (data export, knowledge transfer at contract end)
- ROS-analyse deliverable
- Platform exit strategy

---

## Summary of Required Revisions

### Must-Fix (Before Bid Submission)

| # | Issue | Section | Priority |
|---|-------|---------|----------|
| 1 | Resolve 20-week vs 28-week timeline discrepancy | 2.2 | P0 |
| 2 | Verify ForgeRock/SCIM compatibility with chosen platform | 3.1, 16 | P0 |
| 3 | Define MVP scope explicitly (what is in/out for Q1 2027) | 2.2, new | P0 |
| 4 | Fix phase dependencies (L02 must gate L03) | 2.2 | P1 |
| 5 | Add second acceptance test contingency | 2.2 | P1 |
| 6 | Define archiving compliance without L24 option | 13.1 | P1 |
| 7 | Add ROS-analyse and DPIA to deliverables | 5.1 | P1 |
| 8 | Quantify cash flow exposure and propose milestone payments | 9.3 | P1 |
| 9 | Propose specific SLA targets and compensation model | New (Bilag 4) | P1 |
| 10 | Rephrase ADR-002 re Kursoppslag responsibility | 3.2 | P1 |

### Should-Fix (Strengthen the Bid)

| # | Issue | Section | Priority |
|---|-------|---------|----------|
| 11 | Add Maskinporten to security architecture | 3.3 | P2 |
| 12 | Increase ServiceNow hour estimate to 150-200h | 5.2 | P2 |
| 13 | Add cutover/transition plan | New | P2 |
| 14 | Add change management strategy | New | P2 |
| 15 | Submit Solution Architect CV as additional key personnel | 8.2 | P2 |
| 16 | Raise pricing target to 25-30 MNOK | 9.1 | P2 |
| 17 | Add load testing plan | New | P2 |
| 18 | Add transition-out/exit plan | New | P2 |
| 19 | Add Sopra Steria ServiceNow coordination to dependencies | 8.3 | P2 |
| 20 | Specify external user auth flow (ID-porten -> ForgeRock -> LMS) | 3.3 | P2 |

---

*This review is intended to strengthen the delivery plan before SVV evaluators see it. The foundation is solid -- the revisions above address gaps that experienced public sector procurement teams will probe.*
