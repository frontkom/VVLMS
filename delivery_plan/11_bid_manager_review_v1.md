# Bid Manager Review of Delivery Plan v1
## Statens vegvesen LMS Tender (Saksnr. 25/223727)

**Reviewer**: Bid Manager
**Date**: 2026-03-22
**Document reviewed**: DELIVERY_PLAN_v1.md
**Cross-referenced against**: 08_bid_management.md (Bid Management Plan), original tender documents
**Verdict**: CONDITIONAL PASS - Proceed with revisions before qualification submission

---

## Overall Assessment

The Delivery Plan v1 is a strong synthesis of specialist inputs that demonstrates genuine understanding of SVV's needs, the regulatory environment, and the competitive landscape. It correctly identifies the dominant win strategy (quality over price at 80/20 weighting) and provides actionable technical architecture.

However, the plan has **several gaps that could cost points or cause disqualification** if not addressed before bid writing begins. The most critical issues are: (1) incomplete mapping to tender document requirements, (2) a risky go-live timeline recommendation that could undermine our implementation capability score, (3) unresolved platform decision blocking all bid content, and (4) insufficient detail on how we meet specific A-krav.

**Rating: 7/10** - Good foundation but needs targeted revisions before it can guide actual bid writing.

---

## 1. Completeness Against Tender Requirements

### 1.1 FINDINGS - Gaps in Document Coverage

**CRITICAL GAP: Bilag 8 omitted from document checklist.** The delivery plan (Section 10.2) lists qualification documents, bid documents, and demo materials but does not mention Bilag 8 (Endringer av avtalen etter avtaleinngaelsen). While Bilag 8 is primarily a post-contract artefact, the SSA-L template requires Leverandoren to acknowledge the change management process. We should include it in our bid response even if left blank, to demonstrate awareness.

**GAP: Vedstaelsesfrist (bid validity period) not addressed.** The procurement rules state: "Leverandoren ma vedsta seg sitt tilbud til det tidspunktet som er angitt i kunngjoringen." The delivery plan does not discuss how long we must stand behind our bid. This needs to be checked in KGV and factored into our pricing validity.

**GAP: Avvik (deviations) strategy is weak.** The delivery plan mentions "minimal avvik" (Section 12.2) and the risk assessment flags SSA-L penalties (K-03), but there is no explicit deviation register or strategy for which contract terms we might propose modifications to (Bilag 7). The procurement rules are clear: "Avvik skal klart fremga av leverandorens foresporsler om kvalifikasjon og tilbudsbrev." Deviations must be listed explicitly in both the qualification letter AND the bid letter. We need a decision: zero deviations (safest) or a controlled list of minor deviations that improve our position without triggering avvisning (rejection).

**GAP: Loenns- og arbeidsvilkar (wages and working conditions) documentation.** Bilag 5 requires documentation of compliance with "gjeldende allmenngjorte tariffavtaler" (applicable collective agreements). The delivery plan does not address this. Required at contract signing but should be referenced in our qualification and bid submissions to demonstrate preparedness.

**GAP: Fakturering (invoicing) specifics missing.** Bilag 6 specifies EHF-format electronic invoicing with specific marking requirements (reference: KNNORD, contract number: 25/223727). The plan mentions pricing but not invoicing capability. Must confirm our finance systems support EHF.

### 1.2 FINDINGS - Coverage Strengths

The plan correctly covers:
- All major Bilag (1-7, 9) with content direction
- Vedlegg 1-7 template requirements
- ESPD requirement
- Integration architecture (6 systems with protocols)
- Key personnel requirements (CV, SPOC, PM mandatory attendance)
- DPA (databehandleravtale) requirement
- POC demonstration scenarios (all 5)
- Pricing structure categories (1A through 1I)

### 1.3 RECOMMENDATIONS

| # | Gap | Action Required | Priority | Deadline |
|---|---|---|---|---|
| 1.1 | Bilag 8 acknowledgment | Add to document checklist; prepare blank template with process description | Low | Before Bid 1 |
| 1.2 | Vedstaelsesfrist | Check KGV for bid validity period; ensure pricing models account for this | Medium | Before pricing |
| 1.3 | Avvik strategy | Make explicit Go/No-Go decision: zero deviations or controlled deviation list | High | Before Bid 1 |
| 1.4 | Loenns- og arbeidsvilkar | Prepare egenerklaring (self-declaration) for compliance | Medium | Before Bid 1 |
| 1.5 | EHF invoicing | Confirm finance system capability for EHF format | Low | Before contract signing |

---

## 2. Alignment with Evaluation Criteria

### 2.1 Quality (80%) - Sub-criteria Analysis

**Functional Requirements (50% of quality = 40% of total score)**

The delivery plan identifies this as the dominant scoring factor (correct) but provides only high-level platform capability descriptions. The plan states "Map every requirement to specific platform capability with screenshots/examples" but does not establish HOW this mapping will be done.

**CONCERN**: The functional requirements spreadsheet (Bilag 1 vedlegg 1) is in .xlsb format. We do not know the exact number of requirements, how many are A-krav vs. evaluable (E-krav), or what the response format should be. The plan should have extracted and counted these requirements to establish scope of work for the bid phase.

**RECOMMENDATION**: Immediately convert and analyze the .xlsb file. Count total requirements, categorize A-krav vs E-krav, and estimate writing effort per category. This is essential for bid writing scheduling. Without this, the 4-week bid writing timeline in Section 10.1 is based on assumption, not data.

**Non-Functional Requirements (20% of quality = 16% of total score)**

Well covered in the technical architecture section (3.x) and compliance checklist (13.1). The risk assessment adds depth with T-01, T-02, C-01 risks. However:

**CONCERN**: The non-functional requirements spreadsheet (Bilag 1 vedlegg 2, .xlsx) has not been fully analyzed either. The compliance matrix in Section 13.1 lists requirement categories (IAM, PEO, POD, etc.) but does not enumerate individual requirements. Some A-krav may be buried in the spreadsheet that are not captured in our current compliance mapping.

**RECOMMENDATION**: Complete a cell-by-cell analysis of Bilag 1 vedlegg 2 to identify every A-krav. Create a definitive list with Y/N compliance status per requirement. This is a pre-requisite for the Go/No-Go decision.

**Usability (12.5% of quality = 10% of total score)**

Strong section. The UX strategy (Section 6) with 80-minute demo plan broken into two 40-minute sessions is well structured. The Nielsen's heuristics and ISO 9241-11 alignment demonstrates understanding of the evaluation framework. Key differentiators (Netflix-like recommendations, traffic light dashboard, 3-step onboarding) are compelling.

**CONCERN**: The demo timing in the delivery plan (Section 6.2) shows 40+40 minutes with a 5-minute break. My bid management plan (Section 10.1) estimated ~90 minutes total across 5 scenarios. We need to verify the exact time allocation from Bilag 1 vedlegg 7 and plan accordingly. Exceeding time limits will hurt our score.

**RECOMMENDATION**: Read Bilag 1 vedlegg 7 in detail to confirm exact demo format, time limits, and whether SVV prescribes the order of scenarios.

**Establishment & Implementation Capability (12.5% of quality = 10% of total score)**

The 6-phase, 28-week delivery plan (Section 2.2) is reasonable but has a significant strategic problem:

**CRITICAL CONCERN**: The plan recommends communicating a "realistic Q2 2027 target" instead of SVV's stated goal of 01.01.2027 go-live. This is a bid-losing recommendation. SVV has stated their target clearly. If we propose a timeline that misses their target by 3-6 months, competitors who commit to 01.01.2027 will score higher on implementation capability, even if our timeline is more realistic.

**RECOMMENDATION**: Restructure the implementation plan to show 01.01.2027 as achievable with defined scope. Use an MVP approach: core platform + IAM + user provisioning by 01.01.2027, with integrations (ServiceNow, PowerBI) and migration completing in Q1 2027. Present this as phased go-live, not as missing the deadline. The risk mitigation for L-01 is valid but should be managed through scope phasing, not by proposing a later deadline.

**Business Principles & Standard Terms (5% of quality = 4% of total score)**

Barely addressed. The plan mentions "Accept SSA-L with minimal avvik" but provides no analysis of which SSA-L terms might be problematic for our business model. We need legal to review the SSA-L general terms and flag any terms that require Bilag 7 modifications.

**RECOMMENDATION**: Legal review of SSA-L general terms with explicit recommendation: accept as-is or propose specific modifications. Document decision.

### 2.2 Price (20%)

The pricing strategy (Section 9) is sound: target 22-28 MNOK mid-range, site license, quality-over-price approach. The 4:1 quality-to-price ROI calculation is correct and useful for internal stakeholder communication.

**CONCERN**: The T&M hour estimates diverge from SVV's budgeted estimates in several areas:
- L05 ServiceNow: SVV says 100h, we say 120-150h realistic
- L08 Migration: SVV says 150h, we say 200-250h realistic

Bidding significantly above SVV's estimates will increase our total price. Since price is evaluated on total cost over the entire agreement period, every hour above estimate adds directly to our evaluated price. We need to decide: bid SVV's estimates (risk absorbing overruns) or bid realistic estimates (higher evaluated price but honest).

**RECOMMENDATION**: Bid SVV's stated estimates (100h, 100h, 150h) as the T&M base for evaluation purposes. Document assumptions clearly and include risk contingency in our fixed-price components. During negotiations, address realistic hour ranges transparently. This approach optimizes our evaluated price while maintaining honesty through the narrative.

---

## 3. Competitive Positioning Strength

### 3.1 Strengths

- **Win themes are strong**: The five themes (unified platform, security architecture, 70/20/10, data-driven intelligence, responsible AI) align directly with SVV's stated objectives.
- **Ghost competitor analysis is useful**: Identifying Storyboard AS (incumbent), Sopra Steria (ServiceNow relationship), and potential Cornerstone/Docebo partners helps target our positioning.
- **Stakeholder-specific messaging** (Section 14) is excellent: tailored messages for Demarteau (HR), Nordin (project), Ringstad/Kowalski (IT), Lidi (procurement) shows political awareness.
- **Domain research** (Section 15) provides valuable competitive intelligence: KS Laering won by Task/Valamis, Sikt won by Canvas/Instructure, similar procurements mapped.

### 3.2 Weaknesses

**CRITICAL: Platform decision not made.** The plan recommends Cornerstone Galaxy as primary and Docebo as alternative, but this decision is listed as "Open Item #2" (Section 16). We cannot write a single line of the bid without this decision. Every document -- Bilag 2 (solution description), Bilag 3 (implementation plan), Bilag 4 (SLA), Bilag 6 (pricing), and the demo environment -- depends on which platform we are bidding.

**This decision must be made within 48 hours.** The qualification submission does not require a platform decision (it is about the bidder's capability, not the product), but bid preparation work for Bilag 2-6 must begin immediately after qualification to meet the 06.05.2026 deadline.

**CONCERN: Are we a platform vendor or an implementation partner?** The plan is ambiguous about our role. Section 4 recommends Cornerstone or Docebo, but it is unclear whether we ARE Cornerstone/Docebo or whether we are partnering with them. This has profound implications:

- If we ARE the platform vendor: references must be our own, capacity is inherent, sub-contractor relationships are different.
- If we are an IMPLEMENTATION PARTNER: we need Vedlegg 4 commitment declarations from the platform vendor, their ESPD, and their pricing embedded in ours. This adds significant qualification complexity with only 16 days remaining.

**RECOMMENDATION**: Clarify our bidding entity structure immediately. If partnering, the platform vendor's commitment declaration and ESPD are on the critical path for qualification.

### 3.3 Differentiation Gaps

The plan does not adequately differentiate us on the qualification criteria:
- **Experience (65% weight)**: References are listed as "Open Item #4" - still unconfirmed. We have zero confirmed references as of today. This is the single highest-risk item for qualification.
- **Capacity (35% weight)**: Key personnel listed as "Open Item #5" - project manager not confirmed.

**RECOMMENDATION**: Freeze all other planning work and prioritize reference selection and key personnel confirmation. Without these, qualification fails and nothing else matters.

---

## 4. Demonstration Readiness

### 4.1 Assessment

The demo strategy is one of the plan's strongest sections. The 80-minute structure, scenario-to-role mapping, and UX differentiators are well thought out.

### 4.2 Gaps

**GAP: No fallback plan for technical failures during demo.** Live demos fail. The plan should include:
- Offline video recordings of each scenario as backup
- Pre-loaded screenshots for each key screen
- Designated "driver" and "narrator" roles to manage flow
- Internet connectivity backup plan
- Identified "golden path" through each scenario to avoid edge-case failures

**GAP: Hands-on testing preparation underspecified.** SVV evaluators will test the platform themselves. The plan mentions "test accounts prepared for SVV evaluators" but does not address:
- What scenarios/tasks SVV will perform during hands-on testing
- Whether the demo environment can handle concurrent evaluator access
- Data isolation between evaluator sessions
- Self-service help/guidance available in the test environment

**GAP: Demo environment timeline is tight.** The plan schedules demo environment configuration from 13-17.04 (Week 1 of bid phase), with the demo on 11-12.05. That is only 4 weeks, running in parallel with bid writing. If the platform decision is delayed (Open Item #2), this timeline compresses further.

### 4.3 RECOMMENDATIONS

| # | Gap | Action Required | Priority |
|---|---|---|---|
| 4.1 | No fallback plan | Create demo failure contingency: video backups, screenshot deck, connectivity backup | High |
| 4.2 | Hands-on testing | Define evaluator self-service scenarios, data isolation, concurrent access plan | Medium |
| 4.3 | Timeline pressure | Begin demo environment setup as soon as platform decision is made; do not wait for qualification result | High |

---

## 5. Compliance Gaps - Absolute Requirements

Cross-referencing the delivery plan's compliance checklist (Section 13.1) against the bid management plan's compliance matrix (Section 4), I identify the following gaps:

### 5.1 A-krav Potentially Missing from Plan

| Req Area | Gap | Risk Level |
|---|---|---|
| **RO-01 to RO-05** (Security) | Plan mentions "data ownership" under security but does not explicitly address: ROS (risk assessment/risikovurdering), vulnerability disclosure program, penetration testing evidence, and minimal data collection principle. These are listed as A-krav in bid management plan but not addressed in delivery plan Section 13.1. | HIGH - could cause rejection |
| **TI-03** (No maintenance during exams) | Plan covers availability and uptime generally but does not specifically address the requirement that no maintenance may occur during exam periods. This is operationally significant given KKP's exam management function. | MEDIUM |
| **KI-01** (AI transparency) | Plan has a strong AI strategy (Section 7) but the A-krav specifically requires "Describe AI technology, training data, GDPR compliance." The plan's response must be structured as a formal disclosure, not just a strategy. SVV wants to know WHAT AI is used, not just HOW. | MEDIUM |
| **MIG-01** (Migration) | Plan addresses migration strategy well but does not explicitly confirm this as an A-krav that must be answered "Yes." If MIG-01 asks "Can you migrate data from existing systems?" the answer must be unequivocally "Ja" in the requirements spreadsheet. | LOW |
| **05.019** (Archiving compliance) | Plan flags this as "Requires Mime integration (option) or built-in." This is concerning -- if archiving compliance is an A-krav and we rely on an OPTION (L24) to meet it, we may be non-compliant. Must verify whether built-in certificate archiving meets the requirement without Mime. | HIGH |

### 5.2 Integration Compliance Concerns

**ForgeRock Integration**: The plan correctly identifies this as the critical path blocker. However, neither the delivery plan nor my bid management plan confirms whether our chosen platform (Cornerstone or Docebo) has a proven ForgeRock integration. This is listed as Open Item #9 and represents a genuine compliance risk. If neither platform has done ForgeRock before, we are claiming integration capability without evidence.

**Kursoppslag Proxy API**: The plan states "SVV owns Kursoppslag mapping layer; LMS exposes standard APIs" (ADR-002). This is correct strategically, but the A-krav may require us to maintain the existing API contract. We need to verify whether the requirement is "expose data through APIs" (easy) or "maintain the exact existing Kursoppslag API endpoints" (complex).

### 5.3 RECOMMENDATIONS

| # | Compliance Gap | Action Required | Priority |
|---|---|---|---|
| 5.1 | ROS/security A-krav | Document our ROS process, vulnerability disclosure policy, latest pentest report | High |
| 5.2 | Archiving without Mime | Verify if built-in certificate archiving meets arkivloven requirements | High |
| 5.3 | ForgeRock proof | Obtain platform vendor confirmation of ForgeRock compatibility; seek reference | Critical |
| 5.4 | KI-01 AI disclosure | Restructure AI strategy as formal disclosure document (technology, training data, GDPR basis) | Medium |
| 5.5 | Kursoppslag API contract | Clarify whether existing API contract must be preserved verbatim | Medium |

---

## 6. Qualification Submission Actionability (16 Days)

### 6.1 Assessment

The qualification submission plan (Delivery Plan Section 10.1 + Bid Management Plan Section 8) is logically structured but has critical dependencies that are not yet resolved.

### 6.2 Blockers

| Blocker | Impact | Resolution Path |
|---|---|---|
| **No confirmed references** (Open Item #4) | Cannot submit Vedlegg 2. References are 65% of qualification score. Without strong references, qualification fails regardless of other quality. | IMMEDIATE: Sales Lead must identify and confirm 3-5 references by 25.03.2026 (3 days from now). |
| **No confirmed key personnel** (Open Item #5) | Cannot demonstrate capacity (35% weight). PM CV may be requested at qualification stage. | IMMEDIATE: Confirm project manager by 25.03.2026. |
| **Partner/entity structure unclear** (Open Item #3) | If partnering with platform vendor, their ESPD and Vedlegg 4 are on critical path. These take time to obtain. | IMMEDIATE: Resolve within 48 hours. |
| **Platform not decided** (Open Item #2) | Not directly blocking qualification (which is about the bidder, not the product), but needed for reference selection strategy -- do we highlight Cornerstone-specific references or generic LMS references? | Decide before reference selection. |

### 6.3 Timeline Feasibility

The 16-day qualification timeline is feasible IF AND ONLY IF:
1. References are identified by 25.03 (3 days)
2. Partner structure is decided by 24.03 (2 days)
3. ESPD is completed by 24.03 (2 days)
4. All capacity documentation is compiled by 27.03 (5 days)
5. Internal review happens on 01.04 (10 days from now)
6. Final submission happens by 04.04 (13 days -- 3 days buffer)

**VERDICT**: Feasible but on the edge. No room for delays on reference confirmation.

### 6.4 Question Deadline (23.03.2026 - TOMORROW)

The delivery plan correctly flags that the question deadline for qualification is tomorrow. We should submit questions that:
1. Clarify any ambiguity in qualification documentation requirements
2. Confirm whether CV submissions are required at qualification stage (the procurement rules mention "etterspurt dokumentasjon" but are not explicit about CVs for qualification vs. bid stage)
3. Confirm the exact number of references required/recommended

**RECOMMENDATION**: Draft and submit 2-3 targeted clarification questions via KGV before end of day tomorrow.

---

## 7. Pricing Strategy Competitiveness

### 7.1 Assessment

The 22-28 MNOK target range within the 20-35 MNOK envelope is strategically sound. The quality-over-price logic is correct at 80/20 weighting.

### 7.2 Concerns

**CONCERN: Price floor risk.** If we bid at the lower end (22 MNOK) and a competitor bids 20 MNOK, the linear scoring means we only lose ~0.3 points on price while potentially gaining more on quality. However, if we bid 28 MNOK and a competitor bids 20 MNOK, we lose ~1.2 price points. The spread matters.

**CONCERN: Site license vs. per-user pricing.** The plan recommends site license (Section 9.1). SVV's prisskjema (Bilag 6 vedlegg 1) has tables for both site license (1F-1) and per-user pricing (1F). We must fill in BOTH tables. If our site license is more expensive than a competitor's per-user pricing at SVV's estimated user counts, we lose on price score. Need to model both options.

**CONCERN: Option pricing strategy unclear.** The plan states options L21-L24 are "not in evaluation," but the procurement rules indicate "ovrige opsjoner er ikke bindende for Oppdragsgiver, men skal besvares av Leverandoren som en del av tilbudet." Some options MUST be answered even if not evaluated. Need to clarify which are mandatory to answer.

### 7.3 RECOMMENDATIONS

| # | Pricing Issue | Action Required | Priority |
|---|---|---|---|
| 7.1 | Price range modeling | Model scenarios at 22M, 25M, 28M with competitor price assumptions | High |
| 7.2 | Site vs. per-user | Model BOTH pricing approaches; fill in both tables | High |
| 7.3 | Option pricing | Clarify which options must be priced; price all mandatory options | Medium |
| 7.4 | T&M hours strategy | Decision: bid SVV estimates or realistic estimates | High |

---

## 8. Negotiation Leverage Points

### 8.1 Identified Leverage

The delivery plan identifies several legitimate negotiation leverage points:
- **System consolidation value**: Kilden + KKP + Excel -> one platform is a strong TCO argument
- **AI capability**: If our platform genuinely offers superior AI, this differentiates us on the 50% functional requirements criterion
- **ForgeRock experience**: If we can prove ForgeRock integration, this is a genuine differentiator since few LMS vendors have done this
- **Extended enterprise portal**: KKP replacement capability for 15,000+ external users is a value add

### 8.2 Gaps in Negotiation Preparation

**GAP: No BATNA (Best Alternative to Negotiated Agreement) defined.** What is our walk-away position? At what price point or contract term does this deal become unprofitable?

**GAP: No analysis of SVV's negotiation position.** SVV is consolidating from multiple systems -- they NEED this procurement to succeed. Their current Kilden contract expires in 2027. This gives us some leverage, but the plan does not analyze SVV's alternatives or time pressure.

**GAP: Penalty clause strategy.** The SSA-L includes dagbot (daily penalties) at 0.15%/day of first 6 months' compensation. This can accumulate quickly. Our negotiation strategy should include proposals for:
- Capping total penalty exposure
- Clearly defining what constitutes "delay" vs. "SVV dependency delay"
- Mutual penalty framework (SVV delays should also have consequences)

### 8.3 RECOMMENDATIONS

| # | Negotiation Gap | Action Required | Priority |
|---|---|---|---|
| 8.1 | BATNA definition | Define walk-away criteria (minimum price, maximum penalty exposure) | Medium |
| 8.2 | SVV negotiation analysis | Analyze SVV's alternatives and time pressure; map concession trade space | Medium |
| 8.3 | Penalty clause strategy | Develop position paper on dagbot capping and mutual responsibility | High |

---

## 9. Document Completeness - Required Submissions

### 9.1 Cross-Reference: Bid Management Plan Checklist vs. Delivery Plan Coverage

| Document | In Bid Mgmt Plan | Addressed in Delivery Plan | Gap? |
|---|---|---|---|
| Q1: ESPD | Yes | Yes (Section 10.1) | No |
| Q2: Qualification letter (Vedlegg 1) | Yes | Yes | No |
| Q3: References (Vedlegg 2) | Yes | Yes, but unresolved (Open Item #4) | **YES - CRITICAL** |
| Q4: Capacity (Vedlegg 3) | Yes | Yes, but unresolved (Open Item #5) | **YES - CRITICAL** |
| Q5: Commitment declaration (Vedlegg 4) | Yes | Yes, if applicable | Depends on partner decision |
| Q6: Partner ESPD | Yes | Mentioned | Depends on partner decision |
| Q7: Tax certificate | Yes | Not explicitly mentioned in delivery plan | Minor gap |
| Q8: Company registration | Yes | Not explicitly mentioned in delivery plan | Minor gap |
| Q9: Financial statements | Yes | Not explicitly mentioned in delivery plan | Minor gap |
| B1-B22: All bid documents | Yes (detailed) | Summarized in Section 10.2 | Delivery plan refers to bid mgmt plan |
| D1-D5: Demo materials | Yes | Yes (Section 6) | No |

### 9.2 Missing from Delivery Plan (Present in Bid Mgmt Plan)

The delivery plan (Section 10.2) provides only a summary document checklist, referring implicitly to the bid management plan for the detailed B1-B22 list. This is acceptable since the two documents are meant to be used together, but the delivery plan should explicitly reference 08_bid_management.md as the authoritative document checklist.

### 9.3 RECOMMENDATION

Add explicit cross-reference in DELIVERY_PLAN_v1.md Section 10.2: "For complete document checklist with templates, formats, owners, and deadlines, see 08_bid_management.md Sections 3, 8, and 9."

---

## 10. Summary of Critical Findings

### Priority 1 - BLOCKERS (Must resolve within 48 hours)

| # | Finding | Risk if Unresolved |
|---|---|---|
| **F-01** | Platform decision not made (Cornerstone vs Docebo) | Cannot write any bid content; demo environment cannot be set up |
| **F-02** | No confirmed references | Qualification fails (65% weight); entire bid effort wasted |
| **F-03** | Partner/entity structure unclear | May need partner ESPD + Vedlegg 4; 16-day timeline cannot absorb delays |
| **F-04** | No confirmed project manager | Cannot demonstrate capacity (35% weight); PM must attend all SVV meetings |

### Priority 2 - HIGH (Must resolve before qualification submission, 07.04.2026)

| # | Finding | Risk if Unresolved |
|---|---|---|
| **F-05** | Functional requirements spreadsheet (.xlsb) not analyzed | Unknown number of A-krav; cannot scope bid writing work |
| **F-06** | Non-functional requirements spreadsheet not fully analyzed | Possible hidden A-krav not in compliance matrix |
| **F-07** | ForgeRock integration not proven for chosen platform | May claim capability we cannot deliver; A-krav failure risk |
| **F-08** | Archiving compliance (05.019) reliance on option L24 | If A-krav, we may be non-compliant without committing to Mime integration |
| **F-09** | Question deadline is TOMORROW (23.03.2026) | Must submit clarification questions to KGV before close of business |

### Priority 3 - MEDIUM (Must resolve before Bid 1, 06.05.2026)

| # | Finding | Risk if Unresolved |
|---|---|---|
| **F-10** | Go-live timeline recommends Q2 2027 instead of SVV's 01.01.2027 | Will score lower on implementation capability (12.5% of quality) |
| **F-11** | T&M hours strategy not decided (SVV estimates vs realistic) | Pricing uncertainty; evaluated price could be non-competitive |
| **F-12** | Avvik (deviations) strategy not defined | Risk of rejection if material deviations; lost points if unnecessary deviations |
| **F-13** | Demo fallback plan missing | Technical failure during demo loses 10% of total score |
| **F-14** | Security A-krav (RO-01 to RO-05) evidence not prepared | ROS, pentest reports, vulnerability disclosure needed for compliance |
| **F-15** | Penalty clause / dagbot negotiation strategy not developed | Could expose us to significant financial risk without capped liability |
| **F-16** | AI disclosure document not structured as formal A-krav response | KI-01 requires specific format, not general strategy |
| **F-17** | Site license vs per-user pricing not both modeled | Must fill both tables in prisskjema; competitive positioning depends on this |

---

## 11. Recommended Revisions for v2

### Structural Changes

1. **Add explicit cross-reference to 08_bid_management.md** for document checklists, writing assignments, and QA process. Avoid duplicating this content; the bid management plan is the authoritative source.

2. **Restructure implementation timeline (Section 2.2)** to show 01.01.2027 achievable through phased/MVP go-live. Present core platform + IAM as Phase 1 go-live, integrations as Phase 2 completion in Q1 2027.

3. **Add "Platform Decision" section** with clear recommendation, rationale, and decision deadline. This must be resolved before v2 is finalized.

4. **Add "Deviation Strategy" section** with explicit decision on Bilag 7 approach.

5. **Expand compliance checklist (Section 13)** from category-level to individual requirement-level for all A-krav, based on actual spreadsheet analysis.

### Content Additions

6. **Add security evidence preparation plan** covering ROS, vulnerability disclosure, penetration testing, minimal data collection documentation.

7. **Add demo contingency plan** with fallback procedures for technical failures.

8. **Add invoicing and administrative compliance section** covering EHF, loenns- og arbeidsvilkar, and vedstaelsesfrist.

9. **Add negotiation BATNA and SVV position analysis** to strengthen negotiation preparation.

10. **Add clarification questions list** for KGV submission by 23.03.2026.

---

## 12. Final Verdict

**CONDITIONAL PASS**. The delivery plan provides a strong strategic foundation and demonstrates deep understanding of the tender. The technical architecture, UX strategy, AI approach, and competitive positioning are all high quality.

However, four unresolved open items (F-01 through F-04) represent existential risks to the entire bid. These are not quality issues -- they are blockers. Without confirmed references, a chosen platform, a defined entity structure, and a named project manager, we cannot submit a qualification application, let alone write a competitive bid.

**Recommended actions in order of urgency:**

1. **TODAY**: Resolve entity/partner structure (F-03). This determines whether we need partner ESPDs.
2. **TODAY**: Begin reference identification (F-02). Sales Lead must have candidates by end of day.
3. **TOMORROW (23.03)**: Submit clarification questions to KGV (F-09). Deadline is absolute.
4. **By 24.03**: Decide platform (F-01). Cornerstone Galaxy recommended based on plan analysis.
5. **By 25.03**: Confirm project manager (F-04). CV preparation can begin in parallel.
6. **By 27.03**: Complete requirements spreadsheet analysis (F-05, F-06). Drives all bid writing effort estimates.
7. **By 01.04**: Internal review of qualification package (Gate 1).
8. **By 07.04**: Submit qualification.

If all four blockers are resolved by 25.03.2026, we have a credible path to qualification and a competitive bid. If any remain unresolved by that date, we should reassess the Go/No-Go decision.

---

*Review completed 2026-03-22. Next review: Post-revision of DELIVERY_PLAN_v2.md, targeting completion before Bid 1 submission.*
