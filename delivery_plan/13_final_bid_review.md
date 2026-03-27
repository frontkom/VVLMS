# Final Bid Manager Review - Delivery Plan v2
## Statens vegvesen LMS Tender (Saksnr. 25/223727)

**Reviewer**: Bid Manager
**Date**: 2026-03-22
**Document reviewed**: DELIVERY_PLAN_v2.md
**Cross-referenced against**: 08_bid_management.md, 11_bid_manager_review_v1.md, original tender documents
**Previous verdict (v1)**: CONDITIONAL PASS (7/10)

---

## VERDICT: APPROVED WITH CONDITIONS

**Rating: 8.5/10** -- Significant improvement from v1. All 17 findings have been addressed, most substantively. The plan is now ready to guide qualification preparation and bid writing, subject to three conditions that must be met on the timelines specified below.

---

## 1. Finding Resolution Assessment

### 1.1 Blockers (F-01 through F-04) -- ALL ADDRESSED

| Finding | v1 Status | v2 Resolution | Assessment |
|---|---|---|---|
| F-01: Platform decision | Unresolved | Deadline set 24.03, Cornerstone recommended | **ADEQUATE** -- Decision is framed correctly with deadline. The plan cannot force the Steering Group decision but correctly escalates it as a P0 blocker. |
| F-02: No confirmed references | Unresolved | Sales Lead action by 25.03 | **ADEQUATE** -- Action assigned with deadline. Note: if Sales Lead fails to deliver by 25.03, the entire bid is in jeopardy. No contingency is possible for this -- references either exist or they don't. |
| F-03: Entity/partner structure | Unresolved | Steering Group decision by 24.03 | **ADEQUATE** -- Correctly identified as a Steering Group decision. The ESPD timeline (24.03) accounts for needing partner ESPDs if applicable. |
| F-04: No confirmed PM | Unresolved | HR action by 25.03, CV prep in parallel | **ADEQUATE** -- Timeline is tight but feasible. PM confirmation by 25.03 gives 13 days to prepare CV and Vedlegg 3 capacity documentation. |

**Verdict on blockers**: All four are properly escalated with owners and deadlines. The plan correctly recognizes these are decision dependencies, not writing tasks. The Bid Manager cannot resolve them alone -- they require Steering Group and Sales Lead action.

### 1.2 High-Priority Findings (F-05 through F-09) -- ALL ADDRESSED

| Finding | v2 Resolution | Assessment |
|---|---|---|
| F-05: .xlsb not analyzed | Post-qualification action item (Section 15.2) | **ADEQUATE** -- Deferring to post-qualification is acceptable since the functional requirements spreadsheet is not needed for the qualification submission. However, this must be the FIRST action on 10.04 (qualification result day). |
| F-06: Non-functional gaps | Cell-by-cell analysis planned (Section 15.2) | **ADEQUATE** -- Same deferral logic as F-05. |
| F-07: ForgeRock unproven | P0 verification with 3 specific actions (Section 5.1) | **GOOD** -- The plan now includes a fallback: custom SCIM adapter (40-60h added to L04) if no proven reference exists. This is prudent risk management. |
| F-08: Archiving without L24 | New Section 4.5 with compliance approach | **EXCELLENT** -- This was one of my most critical findings. The resolution is thorough: built-in PDF/A certificates, Noark-5 compatible export, batch export to SVV file system. L24 is repositioned as an efficiency upgrade, not a compliance dependency. This eliminates the A-krav risk. |
| F-09: Question deadline tomorrow | 3 questions drafted in Section 2 | **GOOD** -- The three proposed questions are well-targeted (CV timing, reference count, Kursoppslag API scope). All are material to our qualification and bid strategy. |

### 1.3 Medium-Priority Findings (F-10 through F-17) -- ALL ADDRESSED

| Finding | v2 Resolution | Assessment |
|---|---|---|
| F-10: Go-live timeline | MVP by 01.01.2027, full by Q1 2027 (Sections 3.2-3.3) | **EXCELLENT** -- This is the most important strategic change from v1. The MVP definition table (Section 3.3) is clear and credible. Core platform + IAM + ID-porten + priority content by 01.01.2027 is achievable. The framing as "phased value delivery" is exactly right. The SSA-L mapping note (single Leveringsdag at Full Go-Live, negotiated interim acceptance) shows contract awareness. |
| F-11: T&M hours strategy | Bid SVV estimates with scope definitions (Section 6.2) | **GOOD** -- Clear decision: bid 100h/100h/150h for evaluation, document scope assumptions, address realistic ranges in negotiations. This is the right approach for price optimization. |
| F-12: Avvik strategy | Zero deviations (Section 13.3) | **GOOD** -- Clean decision. "Leverandoren aksepterer SSA-L generell avtaletekst uten endringer" maximizes the 5% business principles score. The rationale (leverage better used elsewhere) is sound. |
| F-13: Demo fallback | Full contingency plan (Section 7.2) | **GOOD** -- Video backups, screenshot decks, golden paths, timekeeper, driver/narrator split, Q&A parking. Comprehensive. |
| F-14: Security evidence (RO-01 to RO-05) | Evidence preparation plan (Section 13.2) | **ADEQUATE** -- Lists the 6 evidence items needed with deadlines. Note: these depend on the platform vendor (pentest reports, ISO 27001, SOC 2). Add this to the platform vendor engagement agenda alongside ForgeRock verification (F-07). |
| F-15: Penalty strategy | Section 10.3 analysis + Section 16.3 negotiation approach | **GOOD** -- Quantified the daily penalty (3,000-4,500 NOK/day), assessed manageable exposure at 20 days delay. Mutual responsibility clause is a smart negotiation point. |
| F-16: KI-01 AI disclosure | Formal disclosure table (Section 8.1) | **GOOD** -- The three-column format (Required Disclosure / Our Response) directly maps to the A-krav. Technology, training data, and GDPR compliance are all addressed. |
| F-17: Pricing modeling | Both site license and per-user to be modeled (Section 10.1) | **ADEQUATE** -- Decision to model both is correct. Actual modeling work remains to be done during bid phase. |

---

## 2. New Content Assessment (v2 Additions)

### 2.1 Sections Added in v2

| New Section | Assessment |
|---|---|
| **Section 2: Critical Decisions** | Excellent. Placing the 4 blockers at the top of the document ensures they cannot be overlooked. |
| **Section 3.3: MVP Definition** | Excellent. Clear in/out table. Realistic scope for 16 weeks. |
| **Section 4.5: Archiving Without L24** | Excellent. Removes a potential disqualification risk. |
| **Section 7.2: Demo Contingency** | Good. Comprehensive fallback matrix. |
| **Section 8.1: KI-01 Disclosure** | Good. Structured as formal response. |
| **Section 10.2: Cash Flow Analysis** | Good. Quantifies pre-Leveringsdag exposure (3-4 MNOK). Milestone payment proposal (20/30/50) is a strong negotiation starting position. |
| **Section 10.3: Penalty Analysis** | Good. Quantified and manageable. |
| **Section 11: SLA Framework** | Good. Covers uptime, response times, exam blackout, Norwegian language support. Needs platform vendor validation of specific targets. |
| **Section 12: Change Management** | Good addition. Division-specific approach and cutover strategy with parallel operations addresses a real delivery risk. |
| **Section 13.2: Security Evidence Plan** | Adequate. Actionable list with deadlines. |
| **Section 13.3: Deviation Strategy** | Good. Clean zero-deviation decision. |
| **Section 16: Negotiation Strategy** | Good. SVV position analysis, leverage points, round-by-round approach, and BATNA all present. |
| **Section 17: Transition-Out Plan** | Good. Addresses EI-01/02/03 requirements. 90-day overlap and 60-day deletion timeline are reasonable. |
| **Appendix A: Finding Resolution** | Excellent. Complete traceability from 30 findings to v2 sections. |

### 2.2 Quality of New Content

The new sections demonstrate genuine understanding of the tender requirements and are not superficial additions. Particularly strong:
- The MVP definition is specific enough to be contractually binding
- The cash flow analysis shows financial maturity
- The negotiation strategy reflects the procurement dynamics correctly
- The archiving compliance section resolves what was previously a potential show-stopper

---

## 3. Remaining Concerns

### 3.1 Condition 1: Blocker Resolution Verification (24-25.03.2026)

The plan is approved on the condition that the four blocker decisions (D-01 through D-04) are resolved by their stated deadlines. If any blocker remains unresolved by 26.03.2026, a formal re-evaluation of the Go/No-Go decision must occur.

**Verification checkpoints:**
- 24.03 EOD: Platform decision (D-01) and entity structure (D-03) confirmed?
- 25.03 EOD: References identified (D-02) and PM confirmed (D-04)?
- 26.03: If any unresolved, convene emergency Steering Group for Go/No-Go re-evaluation.

### 3.2 Condition 2: Requirements Spreadsheet Analysis (10-14.04.2026)

The deferral of .xlsb and .xlsx analysis to post-qualification is acceptable but creates risk. The bid writing schedule in 08_bid_management.md assumes we know the scope of functional and non-functional requirements. If the analysis reveals significantly more A-krav than anticipated, the 4-week bid writing timeline could be insufficient.

**Required action**: Begin analysis on 10.04 (qualification result day). Complete by 14.04 at the latest. If A-krav count exceeds current assumptions, immediately re-assess bid writing assignments and deadlines.

### 3.3 Condition 3: Platform Vendor Engagement Agenda (By 28.03.2026)

Several v2 items depend on the platform vendor (assumed Cornerstone Galaxy):
- ForgeRock SCIM integration proof (F-07 / Section 5.1)
- Security evidence: pentest reports, ISO 27001, SOC 2 (Section 13.2)
- SLA targets validation (Section 11.1)
- Sub-processor list (Section 13.1, PEO-15)
- Norwegian language confirmation
- Pricing/licensing terms for prisskjema

These are not independent requests -- they should be bundled into a single vendor engagement request with a deadline of no later than 15.04.2026 (before bid writing starts in earnest).

**Required action**: Compile a comprehensive vendor request document and send to Cornerstone Nordic by 28.03.2026.

---

## 4. Strengths Validated

The following elements of the plan are strong and should be preserved through the final bid:

1. **Win themes** (Section 1): Five themes are distinctive, relevant, and aligned with SVV's stated objectives. They should form the narrative backbone of Bilag 2.

2. **MVP/phased go-live** (Section 3.2-3.3): This reframing was the single most important change from v1 to v2. It transforms a potential weakness (missing the 01.01.2027 deadline) into a strength (early value delivery). The in/out table is clear and defensible.

3. **Archiving without L24** (Section 4.5): Eliminates a compliance risk while positioning the option as a value add. Smart.

4. **Zero deviations** (Section 13.3): Maximizes the 5% business principles score and eliminates avvisning risk. The right call for this procurement.

5. **T&M bidding strategy** (Section 6.2): Bidding SVV's estimates while documenting scope assumptions is the optimal approach for evaluated pricing. Transparent and defensible.

6. **Demo contingency** (Section 7.2): Professional risk management. Shows maturity that SVV evaluators will notice.

7. **Negotiation strategy** (Section 16): The SVV position analysis (they NEED this, Kilden expires 2027) correctly identifies our leverage. The 20/30/50 milestone payment proposal is a strong opening position.

8. **Explicit cross-reference to 08_bid_management.md** (Section 15.3): Avoids duplication and establishes the bid management plan as the authoritative document checklist. Good document architecture.

---

## 5. Minor Issues (Non-Blocking)

These do not affect the approval verdict but should be corrected in the final bid documents:

| # | Issue | Recommendation |
|---|---|---|
| M-01 | Section 3.2 shows "W25-W28 hypercare" but Section 3.3 says "16 weeks from contract signing" for MVP and "24 weeks" for Full Go-Live. With Sept 2026 signing, 16 weeks = approximately Jan 2027 (correct). 24 weeks = approximately late Feb 2027. Verify calendar math against actual signing date. | Validate after contract signing date is confirmed. |
| M-02 | Section 10.1 target range revised to 25-30 MNOK. This is a narrower band than the original 20-35 MNOK envelope. Ensure commercial team understands the strategic rationale (credibility, quality signaling) and has modeled scenarios at both ends. | Commercial Lead to validate. |
| M-03 | Section 11.1 proposes 99.9% uptime. This is 8.77 hours/year downtime. Must confirm Cornerstone Galaxy's actual SLA aligns. Some SaaS vendors offer 99.5% standard with 99.9% premium. If 99.9% requires premium pricing, factor into prisskjema. | Validate with platform vendor. |
| M-04 | Section 16.4 BATNA states "walk away if total margin falls below 15%." This is internal-only information. Ensure this is never included in any externally-shared document. | Mark as CONFIDENTIAL in final version. |
| M-05 | Section 12.2 mentions "Kilden read-only archive for 6 months." This may have cost implications if Storyboard AS charges for continued hosting. Factor into pricing or flag as SVV's responsibility. | Clarify ownership of Kilden wind-down costs. |
| M-06 | The three KGV clarification questions (Section 2) are well-targeted but should be reviewed by Legal before submission tomorrow to ensure they don't inadvertently reveal our strategy or create binding expectations. | Legal review of questions by 23.03 morning. |

---

## 6. Comparison: v1 vs. v2

| Dimension | v1 Score | v2 Score | Change |
|---|---|---|---|
| Tender compliance | 6/10 | 8.5/10 | Major improvement: archiving, security, deviations, SLA |
| Evaluation criteria alignment | 7/10 | 9/10 | MVP restructuring dramatically improves implementation score |
| Competitive positioning | 7/10 | 8/10 | Stronger with zero deviations, phased delivery, penalty analysis |
| Pricing strategy | 6.5/10 | 8/10 | Cash flow analysis, T&M strategy, dual pricing model |
| Demo readiness | 7.5/10 | 8.5/10 | Contingency plan, hands-on testing prep, clear format |
| Compliance (A-krav) | 6/10 | 8/10 | Expanded compliance list, security evidence plan, archiving fix |
| Negotiation readiness | 5/10 | 8/10 | SVV analysis, BATNA, round strategy, penalty quantification |
| Actionability (16-day qual) | 6/10 | 8/10 | Blockers escalated with deadlines, question list ready |
| Document completeness | 7/10 | 9/10 | Cross-reference to bid mgmt plan, Bilag 8 noted, admin items listed |
| Risk management | 7/10 | 8.5/10 | ForgeRock fallback, parallel ops, rollback decision points |
| **Overall** | **7/10** | **8.5/10** | **Significant improvement** |

---

## 7. Final Approval Statement

**APPROVED WITH CONDITIONS**

The Delivery Plan v2 is approved as the working document for this bid. It is comprehensive, strategically sound, and adequately addresses all findings from the v1 review cycle.

**Three conditions for ongoing approval:**

1. **Blocker resolution by 25.03.2026**: All four decisions (D-01 through D-04) must be confirmed. If any remains unresolved by 26.03, convene emergency Go/No-Go review.

2. **Requirements analysis by 14.04.2026**: Full analysis of both requirements spreadsheets (.xlsb and .xlsx) must be completed within 4 business days of receiving the bid invitation. Results may trigger adjustments to bid writing schedule.

3. **Platform vendor engagement by 28.03.2026**: Comprehensive request to Cornerstone Galaxy covering ForgeRock proof, security evidence, SLA validation, sub-processor list, and pricing terms must be issued before end of March.

**Authorization**: This plan, together with the bid management plan (08_bid_management.md), forms the authoritative basis for qualification preparation and bid writing. All bid team members should work from these documents.

**Next milestones**:
- 23.03: Submit KGV clarification questions (after Legal review)
- 24.03: Platform and entity structure decisions
- 25.03: References and PM confirmed
- 01.04: Gate 1 qualification review
- 07.04: Qualification submission

---

*Final review completed 2026-03-22. This review supersedes the v1 review (11_bid_manager_review_v1.md).*
