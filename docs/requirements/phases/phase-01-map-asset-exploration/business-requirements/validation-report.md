---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
document: BRD Review Report
reviewed: 2026-09-20
subject: business-requirements.md v0.1, re-reviewed against v1.0
conditions_closed: 2026-09-20
next: analyze-prd-readiness (phase P01)
---

# BRD Review Report: P01 - Map Asset Exploration (MAE)

## Summary

- **Overall assessment:** **Pass** - conditions closed 2026-09-20, document baselined at v1.0
- **Issues found:** 9
- **Critical issues (must fix):** 4
- **Advisory issues (should fix):** 5

Two critical issues were structural defects in the document and were **fixed
during this review** (C1, C4). Two were approval conditions requiring
stakeholder input (C2, C3); **both were closed on 2026-09-20** and the document
was baselined at v1.0. See "Condition closure" below.

**Phase identity check.** Phase ID `P01`, name Map Asset Exploration, acronym
`MAE`, and directory `docs/requirements/phases/phase-01-map-asset-exploration/`
all match the Intent Brief registry. Scope stays within P01 throughout; no
requirement reaches into P02-P04. **Pass.**

## Checklist results

> These verdicts are the **original review of v0.1**, retained as the audit
> record of what was found. Sections 2 and 4 failed at that point; both were
> remediated - section 4 by the C1 fix during review, section 2 by the C2 and
> C3 closures on 2026-09-20. Against **v1.0 all five categories pass**. Counts
> below refer to v0.1's 22 requirements; v1.0 carries 27.

### 1. Completeness: **Pass**

All 8 sections present. Section 5e marked N/A with justification, which the
scaling guidance permits for a Major BRD where the exclusion is reasoned.

- 22 of 22 requirements use "shall" or "shall not" with a single obligation.
- 22 of 22 carry an ID, priority, evidence, owner and measurable outcome.
- Baseline and target present for all six headline metrics in Section 2.4.
- Glossary defines 14 terms including all four new section codes
  (`COMP-LICENSE`, `COMP-CONTENT`, `GUARD-ACC`, `GUARD-SCOPE`).

### 2. Scope & Boundaries: **Fail at v0.1 - Pass at v1.0** - see C2, C3, C4

- Non-goals explicit: six, each carrying a guardrail. **Pass.**
- All seven guardrails state a detection method and escalation path. **Pass.**
- Assumptions, dependencies, risks and open questions are in Section 7-8,
  separated from requirements. **Pass.**
- Budget and timeline stated with specifics - zero budget, explicitly untimed,
  with the reasoning recorded. **Pass.**
- **No P0 requirement may depend on an unresolved Open Question that changes
  scope, policy or success measurement: FAIL.** Three P0 requirements do. See
  C2 and C3.

### 3. Governance & AI: **N/A**

The initiative involves no AI, machine learning or automated decision-making.
The editor decodes and displays file data deterministically. Section 5e states
this with justification rather than omitting the section. Correctly handled.

### 4. Traceability & Consistency: **Fail at v0.1 - Pass at v1.0** - see C1

- All section codes documented in the Glossary. **Pass.**
- Relationships declared, including one explicit `conflicts-with` carrying a
  written resolution. **Pass.**
- **Circular dependency present: FAIL.** See C1.
- Forward traceability: all 22 requirements have a plausible PRD mapping. None
  reads as aspirational. **Pass.**
- Six decision records, each with alternatives rejected and rationale rather
  than a bare choice. **Pass.**
- Change Log present with the initial entry. **Pass.**

### 5. Format & Structure: **Pass**

Every entry type conforms to its prescribed format: requirements (8 fields),
guardrails (6 fields), stakeholders (5 fields), assumptions (4), dependencies
(5), risks (7), open questions (6), decision records (6). Spot-checked across
all eight entry types; no deviations found.

### 6. Evidence Spot-Check: **Pass**

Three citations verified at source, exceeding the 2-3 minimum.

| Citation | Claim | Verification | Result |
|:---|:---|:---|:---|
| `MR-REG-01` | Recoil is GPL v2-or-later, making a shader port a derivative work | Read `bar-RecoilEngine/LICENSE` directly: "under the terms of the GNU General Public License ... either version 2 of the License, or (at your option) any later version". Source headers corroborate: "GPL v2 or later" | **Supported** |
| `MR-CORP-06` | Normal map absent in 5 archives, specular in 9, splat in 4 | Re-derived from the corpus scan of all 254 archives conducted 2026-09-20; named archives match | **Supported** - original measurement, reproducible |
| `MR-CHK-16` | Wind 0-30, tidal 0-25 | Matches the published BAR Map Checklist as retrieved 2026-09-20 | **Supported** |

- **Competitor claims (Section 6):** gathered 2026-09-20, same day. Not stale.
- **Regulatory citations:** licensing only; verified above. No statutory regime
  applies, correctly stated.
- **Stale data:** none. All research is same-day. One single-sourced claim
  flagged as A4.

---

## Condition closure - 2026-09-20

All ten open questions were resolved by the stakeholder, closing both critical
conditions and three of the five advisories.

| Issue | Status | Closure |
|:---|:---|:---|
| **C2** - `OQ-1` blocks P0 `BIZ-EXP.2` | **Closed** | `DR-7`: the target is now "equal to or faster than the same traversal performed in Beyond All Reason at the pinned engine commit", measured over a defined reference map set. Externally anchored, so no arbitrary threshold remains |
| **C3** - `OQ-3`, `OQ-6` block P0 `BIZ-EXP.3` | **Closed** | `DR-11`: a map-option picker with re-evaluation. `DR-9`: `LuaGaia` is inspectable source only. The meaning of "every asset class represented" is now fixed |
| **A1** - undefined reference map set | **Closed** | `DR-7` and new requirement `OPER-REPORT.2`: six to ten maps with a stated coverage rule spanning sky, features, missing texture slots, legacy `.smd`, option sensitivity and archive size |
| **A2** - `OQ-8` set unchosen | **Closed** | `DR-10`: all twenty properties, superseding the arbitrary count |
| **A4** - single-sourced trend claim | Open | Unchanged; affects context only, not any requirement |
| **A3** - single owner, no review independence | **Open, and now more material** | `DR-14` defers early-user contact until a runnable build exists, so no second reader will see this document before design completes. The author, owner and approver remain the same person throughout requirements and design |
| **A5** - persona unvalidated | **Open, and now more material** | `DR-10` and `DR-15` placed *more* weight on the unvalidated persona - all twenty checklist properties and a dedicated checklist view - while `DR-14` moved its validation later. `RISK-6` was raised to Medium-High probability and High impact in consequence |

### Re-review of v1.0 changes

Spot-checked the baseline edits for the defects this review exists to catch.

- **No new dependency cycles.** The five added requirements declare
  `depends-on`, `constrained-by` and `justified-by` edges only in the direction
  of existing requirements. `BIZ-EXP.2` now depends on `OPER-REPORT.2`, which
  depends on nothing. Graph remains acyclic.
- **P0 approval check passes.** No P0 requirement rests on an open question;
  none of the five additions is P0, so the phase-closure floor is unchanged.
- **New guardrail is well formed.** `GUARD-REPUTE.2` carries a prohibited
  outcome, detection method and escalation path, and directly constrains
  `OPER-WORKFLOW.4` - the correct relationship, since pass/fail reporting is
  what creates the risk of implying approval.
- **One honest reclassification.** `GUARD-SCOPE.3` and `GUARD-SCOPE.4` were
  demoted from project anti-goals to P01 scope boundaries because the
  stakeholder confirmed only one of three when offered all three (`DR-12`).
  Recording silence as assent would have breached the document's own
  `GUARD-ACC.1`. Correctly handled.
- **Counts corrected.** The requirement total was restated as 27 (12 P0, 14 P1,
  1 P2) after a recount; both the prose and the index agree.

## Issues

### Critical

**C1 - Circular dependency in the requirement graph. FIXED.**
`BIZ-EXP.1` declared `depends-on: [OPER-MIGRATE.1]`, while `OPER-MIGRATE.1`
declared `depends-on: [BIZ-EXP.1]`. This is a true cycle: neither can be
satisfied first. The intent was asymmetric - `BIZ-EXP.1` genuinely depends on
legacy `.smd` support to reach its no-exceptions target, whereas
`OPER-MIGRATE.1` is *justified by* `BIZ-EXP.1` rather than dependent on it.
**Action taken:** `OPER-MIGRATE.1`'s `depends-on` replaced with a
`justified-by` note; the new relationship type documented in the Glossary. The
graph is now acyclic.

**C2 - A P0 requirement's success measurement is unresolved. CONDITION.**
`BIZ-EXP.2` (P0) carries a target of "20 seconds or less" marked provisional
pending `OQ-1`, which is Open. The BRD's own P0 approval check argues this
affects only the threshold and not the measurement method. On review that
distinction does not hold: a traversal-time requirement with an unconfirmed
threshold and no defined traversal procedure cannot be verified, so the Open
Question does change success measurement. **Action required:** resolve `OQ-1`
before baseline, or demote `BIZ-EXP.2` to P1. Cannot be fixed by editing.

**C3 - Two Open Questions change the scope of a P0 requirement. CONDITION.**
`BIZ-EXP.3` (P0) requires every asset class to be represented, but `OQ-3` (are
map options evaluated at load?) and `OQ-6` (is `LuaGaia` source-only or must its
effect be shown?) both change what satisfying it means. `OQ-3` additionally
interacts with `GUARD-ACC.1`, since unevaluated options may force values to be
marked unknown. **Action required:** resolve `OQ-3` and `OQ-6` before baseline.
Cannot be fixed by editing.

**C4 - Undeclared tension between a guardrail and two requirements. FIXED.**
`GUARD-SCOPE.2` prohibits P01 from writing, while `OPER-REPORT.1` (corpus run
artifact) and `MARK-PARITY.1` (reference captures) both imply writing files.
The guardrail carried a clarifying note, but the tension was not declared as a
relationship, so a reader following only the requirement graph would miss it.
**Action taken:** `GUARD-SCOPE.2` narrowed to name map files explicitly, and
`conflicts-with` relationships declared on both affected requirements pointing
at the resolution and at `OQ-2`.

### Advisory

**A1 - `MARK-PARITY.1` cites an undefined artifact.** Its success metric depends
on "a defined reference map set" that no requirement creates or specifies.
*Action:* define the reference set in the product requirements, or add a
supporting requirement. Low urgency - `MARK-PARITY.1` is P2.

**A2 - `BIZ-EFF.1` commits to a count before choosing its members.** The target
is "at least 16 of 20" checklist properties while `OQ-8` leaves the specific set
open. Defensible for a P1 requirement, but the number is currently arbitrary.
*Action:* close `OQ-8` during product requirements.

**A3 - All 22 requirements share one owner.** Correct for a single-maintainer
volunteer project, and the format is satisfied by a role. Noted because it
removes review independence: the author, owner and approver are the same person,
so the Source Bundle and this report carry more of the assurance burden than
they would in a team setting.

**A4 - One single-sourced market claim.** `MR-TREND-01` (Hooded Horse
publishing deal, June 2026) rests on a single secondary article. It informs no
requirement directly, only context. *Action:* corroborate before any external
publication that repeats it.

**A5 - The persona remains unvalidated, and a numeric target rests on it.**
Disclosed honestly as Data Gap 1, `ASSUM-5` and `RISK-6`, with `BIZ-SAT.1` as
the remedy - this is good practice, not a defect. Flagged because `BIZ-EFF.1`'s
"16 of 20" target derives from inference `INF-02`, which no reviewer has
confirmed. *Action:* treat `BIZ-EFF.1` as provisional until `DEP-4` resolves.

---

## Remediation actions

| # | Issue | Severity | Action required | Responsible |
|:--|:---|:---|:---|:---|
| 1 | C1 circular dependency | Critical | **Done** - `OPER-MIGRATE.1` relationship changed to `justified-by`; Glossary updated | This review |
| 2 | C2 `OQ-1` blocks P0 `BIZ-EXP.2` | Critical | Resolve the traversal threshold and procedure, or demote to P1 | Stakeholder elicitation |
| 3 | C3 `OQ-3`, `OQ-6` block P0 `BIZ-EXP.3` | Critical | Resolve map-option handling and `LuaGaia` representation | Stakeholder elicitation |
| 4 | C4 undeclared guardrail tension | Critical | **Done** - guardrail narrowed to map files; `conflicts-with` declared on both requirements | This review |
| 5 | A1 undefined reference map set | Advisory | Define during product requirements | `draft-prd` |
| 6 | A2 `OQ-8` set unchosen | Advisory | Choose the 16 properties | `draft-prd` |
| 7 | A3 single owner, no review independence | Advisory | Accept, or have the early user review the BRD | Stakeholder |
| 8 | A4 single-sourced trend claim | Advisory | Corroborate before external use | `research-brd-context` |
| 9 | A5 persona unvalidated | Advisory | Treat `BIZ-EFF.1` as provisional until `DEP-4` resolves | Stakeholder elicitation |

## Recommendation

**Pass.** *(Originally Pass with Conditions; conditions closed 2026-09-20.)*
The document is structurally sound, comprehensively evidenced and internally
consistent. Its evidence
base is stronger than typical at this stage, because the corpus characterisation
is original measurement rather than citation, and because the twenty checklist
properties give business objectives an external anchor that did not have to be
invented.

**Baselined at v1.0 on 2026-09-20.** All ten open questions resolved; no P0
requirement rests on an open question.

**Two advisories remain open and became more material at baseline**, and both
concern the same thing - that nobody outside the project has seen any of this.
`DR-14` defers the early user until a runnable build exists, while `DR-10` and
`DR-15` committed more of the phase to serving a persona reconstructed entirely
from published documents. `RISK-6` now carries Medium-High probability and High
impact. This is a recorded, accepted risk rather than an unresolved question,
but it is the thing most likely to be wrong in this document.

**Routing.** Proceed to `analyze-prd-readiness` for P01. No return to
`research-brd-context` is warranted - the remaining evidence advisory (A4)
affects context only, not any requirement.

## Quality gates

| Gate | Status |
| --- | --- |
| All 5 checklist categories evaluated | Met - plus Governance marked N/A with justification |
| Every issue logged with severity and remediation | Met - 9 issues, 4 critical, 5 advisory |
| Evidence spot-check covered 2-3 citations | Met - 3 verified at source, one against the primary file |
| Clear recommendation stated | Met - Pass with Conditions, with baseline blocked |
| Remediation routing specified | Met - per issue |
| Phase ID, name, acronym, scope and path match registry | Met |
