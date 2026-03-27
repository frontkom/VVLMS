# Final Risk Review - Delivery Plan v2.0

**Reviewer**: Risk Assessor
**Date**: 2026-03-22
**Document reviewed**: DELIVERY_PLAN_v2.md
**Cross-referenced against**: 09_risk_assessment.md, 10_risk_review_v1.md

---

## VERDICT: APPROVED WITH CONDITIONS

The delivery plan v2.0 is substantially improved and addresses all 20 findings from the v1 review. The plan is now suitable for use as the foundation for the bid, subject to the conditions listed below.

**Rating: 4.3/5 -- Ready for bid preparation with 3 conditions to resolve in parallel.**

---

## Finding-by-Finding Verification

### P0 Findings (Blockers) -- All Resolved

| # | Finding | Resolution | Verified? |
|---|---------|-----------|-----------|
| 1 | Timeline 20w vs 28w discrepancy | Section 3.2: Clear 28-week phased plan with W1-W28 explicitly mapped. Phases 1-7 with hypercare. | **YES** -- Timeline is now internally consistent. Week ranges are explicit. |
| 2 | ForgeRock/SCIM unverified | Section 5.1: Flagged as P0 with 3 action steps and 40-60h adapter contingency. | **YES** -- Correctly handled as an open risk with contingency. Cannot be fully resolved in the plan itself. |
| 3 | MVP undefined | Section 3.3: Explicit In/Out table for MVP scope. SSA-L mapping addressed. | **YES** -- Clear, defensible MVP definition. Good framing as "phased value delivery." |

### P1 Findings (Must-Fix) -- All Resolved

| # | Finding | Resolution | Verified? |
|---|---------|-----------|-----------|
| 4 | Phase dependencies | Section 3.2: Hard gates between L01->L02->L03. | **YES** -- Sequential gates are explicit. |
| 5 | 2nd acceptance test | Section 6.1: 2-week buffer (W23-W24) for second round. | **YES** -- Adequate buffer. |
| 6 | Arkivloven without L24 | Section 4.5: Built-in PDF/A + Noark-5 export + batch export. | **YES** -- Well-structured. See minor note below. |
| 7 | ROS + DPIA missing | Section 6.1: Added to L01 deliverables. | **YES** |
| 8 | Cash flow unquantified | Section 10.2: ~3-4 MNOK exposure modeled with milestone payment proposal. | **YES** -- Good. See condition below. |
| 9 | No SLA targets | Section 11: 99.9% uptime, response times, compensation model. | **YES** -- Appropriate targets. |
| 10 | ADR-002 responsibility | Section 4.2: Vendor takes responsibility for field compatibility. | **YES** -- Correctly reframed. |

### P2 Findings (Should-Fix) -- All Resolved

| # | Finding | Resolution | Verified? |
|---|---------|-----------|-----------|
| 11 | Maskinporten | Section 4.3: Added for machine-to-machine auth. | **YES** |
| 12 | ServiceNow hours | Section 6.2: 100h bid / 150-200h realistic. | **YES** |
| 13 | Cutover plan | Section 12.2: 5-step cutover with parallel ops and rollback. | **YES** |
| 14 | Change management | Section 12: Phased strategy with division champions. | **YES** |
| 15 | SA as key personnel | Section 9.1: SA added with CV, 0.8->0.3 FTE through Phase 6. | **YES** |
| 16 | Pricing 25-30M | Section 10.1: Raised to 25-30 MNOK. | **YES** |
| 17 | Load testing | Referenced in L10 acceptance test. | **PARTIAL** -- See note below. |
| 18 | Transition-out plan | Section 17: Complete transition-out plan. | **YES** |
| 19 | Sopra Steria | Section 9.2: Explicit dependency with coordination note. | **YES** |
| 20 | External auth flow | Section 4.3: ID-porten -> ForgeRock -> LMS flow specified. | **YES** |

---

## Conditions for Approval

The following 3 conditions must be resolved **in parallel with bid preparation** (they do not block starting bid writing):

### Condition 1: ForgeRock/SCIM Verification Must Complete Before Bid 1 (06.05.2026)

The plan correctly flags this as P0 and proposes 3 verification actions. This is the single most important technical risk in the entire bid. The result of this verification determines:

- Whether L04 is a standard configuration or requires a custom SCIM adapter (40-60h additional)
- Whether we can claim "proven ForgeRock integration" as a differentiator
- Whether the fixed-price for L04 is adequate

**Required action**: Assign someone to contact Cornerstone's Nordic team by 24.03.2026. This cannot wait until after qualification. If no ForgeRock reference exists by 30.03.2026, the custom adapter contingency must be built into the L04 cost estimate and disclosed in the bid.

**Owner**: Solution Architect
**Deadline**: 30.03.2026 for initial answer; before Bid 1 for confirmation

### Condition 2: Milestone Payment Proposal Must Be Legally Vetted

Section 10.2 proposes a 20/30/50 milestone payment split. This deviates from the standard SSA-L payment model (full payment after acceptance). While the plan correctly positions this as a negotiation item for Round 1, we need to verify:

- Is SSA-L flexible enough to accommodate milestone payments? The generelle avtaletekst states payment terms, but Bilag 6 allows customization of faktureringstidspunkt.
- The SSA-L document says "Vederlag for leveranser fram til leveringsdag faktureres forst etter godkjent godkjenningsprove." This appears to explicitly prohibit pre-acceptance payment for fixed-price deliverables. Milestone payments may only be achievable through T&M invoicing.

**If milestone payments are not possible under SSA-L**: The cash flow exposure of 3-4 MNOK must be absorbed. This affects the Go/No-Go calculation and minimum acceptable margin. The commercial team must model this scenario.

**Required action**: Legal review of SSA-L payment flexibility.
**Owner**: Commercial Lead / Legal
**Deadline**: Before Round 1 negotiations (26.05.2026)

### Condition 3: Load Testing Must Be Explicit in L10 Plan

Finding #17 (load testing) is marked as "addressed in L10 acceptance test," but the L10 description in Section 6.1 does not explicitly mention performance or load testing. SVV's non-functional requirements (TI-01 to TI-05) specify response times and uptime. The godkjenningsprove should include:

- Load testing with realistic concurrent user volumes (500+ simultaneous for internal, scaling scenarios for KKP seasonal peaks)
- Response time validation against TI-01 targets
- No-degradation test during simulated exam periods

This does not need to change the plan structure -- it needs to be specified in the L10 acceptance criteria during L01 (test plan development). But we should mention it in the bid to demonstrate awareness.

**Required action**: Add explicit load/performance testing mention to the test plan description.
**Owner**: Test Lead
**Deadline**: Bid 1 (06.05.2026)

---

## Minor Observations (Non-Blocking)

These are small items noted during review that do not affect the approval verdict:

**1. Section 3.3 MVP scope -- "~50 migrated courses"**: The number 50 appears arbitrary. During bid writing, either derive this from a classification of the 297 courses (e.g., "all mandatory HMS/safety courses + top-20 by completion volume") or remove the specific number. SVV evaluators will ask "why 50?"

**2. Section 4.5 Archiving -- "digital signatures"**: PDF/A certificates with "digital signatures" is stated as a built-in capability. Verify that the chosen LMS platform actually supports digital signature embedding in PDF certificates. If not, this claim must be softened to "electronic signatures" or removed. Overpromising on a compliance feature is high-risk.

**3. Section 10.3 Penalty analysis -- "manageable"**: The analysis concludes penalties of 3,000-4,500 NOK/day are "manageable." This is true for short delays. But the plan should also consider the scenario of extended delay (e.g., 60+ business days if acceptance test fails twice). At 60 days x 4,500 NOK = 270,000 NOK -- still manageable but worth noting. More importantly, extended delay triggers reputational damage and potential contract termination, which is the real risk, not the financial penalty.

**4. Section 11.1 SLA -- 99.9% uptime**: This is a strong commitment (allows only ~43 minutes downtime/month). Verify that the chosen LMS platform vendor guarantees 99.9% in their own SLA. If the platform only guarantees 99.5%, we cannot commit to 99.9% without bearing the gap risk. This should be confirmed with the platform vendor.

**5. Section 12.2 Cutover -- "Kilden read-only for 6 months"**: This depends on SVV's agreement with Storyboard AS. If Kilden's contract ends before 6 months of parallel operation is complete, this is not viable. Confirm Kilden's exact contract end date and whether read-only extension is possible.

**6. Section 13.3 Zero deviations strategy**: Good strategy for scoring, but verify that the generelle avtaletekst does not contain terms that create unacceptable risk for us (e.g., unlimited liability, uncapped penalties). A "zero deviations" commitment made without reading the standard terms is reckless. Legal must review the SSA-L standard text before this decision is final.

**7. Section 16.4 BATNA -- "Walk away if margin below 15%"**: This is an internal decision that should not appear in the delivery plan if it risks being shared. Consider whether the delivery plan is an internal-only document or if parts will be adapted for SVV. Sensitive commercial thresholds should be in a separate document.

---

## Strengths of v2.0

The following elements are notably well-executed:

1. **MVP definition (Section 3.3)** is clear, defensible, and correctly maps to SSA-L with the "phased value delivery" framing. This will resonate with SVV's project leadership.

2. **Archiving without L24 (Section 4.5)** elegantly removes a compliance dependency from an optional deliverable. This demonstrates deep understanding of the requirements.

3. **Cash flow analysis (Section 10.2)** is transparent and actionable. The 20/30/50 milestone proposal shows commercial maturity.

4. **SLA framework (Section 11)** with specific targets, compensation model, and maintenance window policy is exactly what Bilag 4 requires. The exam blackout mechanism is a thoughtful detail.

5. **Change management section (Section 12)** addresses the organizational reality that SVV is not a monolithic organization. Division champions and phased rollout are best practice.

6. **Transition-out plan (Section 17)** proactively addresses contract-end scenarios, which will score well with procurement evaluators focused on long-term risk.

7. **Demo contingency plan (Section 7.2)** is professional and shows preparedness. The driver/narrator separation is smart.

8. **Zero deviations strategy (Section 13.3)** is commercially shrewd for a 5%-weight criterion. Maximum score for minimum effort.

9. **Appendix A finding-by-finding mapping** demonstrates rigor in the review process and will serve as an audit trail for internal quality assurance.

---

## Conclusion

Delivery Plan v2.0 is a mature, well-structured document that demonstrates credible understanding of SVV's requirements, realistic assessment of risks, and actionable plans for delivery. The three conditions above are resolvable in parallel with bid preparation and do not block proceeding.

**The plan is APPROVED for use as the foundation for bid writing, subject to the 3 conditions being resolved by the stated deadlines.**

The team should now focus on:
1. Resolving the 4 critical decisions in Section 2 (platform, references, entity structure, PM) by 24-25.03.2026
2. Submitting clarification questions by 23.03.2026
3. Beginning qualification document preparation immediately

---

*Final review completed 2026-03-22. No further review cycle required before qualification submission.*
