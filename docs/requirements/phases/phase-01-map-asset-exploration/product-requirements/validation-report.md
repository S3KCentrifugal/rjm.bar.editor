---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
document: PRD Review Report
reviewed: 2026-09-20
subject: product-requirements.md
next: analyze-tdd-readiness (phase P01)
---

# PRD Review Report: P01 - Map Asset Exploration (MAE)

## Summary

- **Overall assessment:** **Pass** - two critical issues found and fixed during
  review; five advisories logged, three fixed
- **Issues found:** 9
- **Critical issues (must fix before TDD):** 2 - both fixed
- **Advisory issues (should fix):** 7 - three fixed, four accepted or deferred

The most significant finding is **C1**: the PRD carried no security
requirement, despite the product executing Lua from untrusted third-party
files. The PRD research stage had concluded security was "structurally N/A"
because there is no server, no network and no user data. That reasoning was
wrong - it considered the product as a service and missed that **the input
itself is untrusted code**.

## 1. Traceability: **Pass**

**Forward (BRD → PRD):** 27 of 27 BRD items covered.

- Four marked *N/A by design*: `BIZ-SAT.1` (project activity),
  `COMP-LICENSE.1` (repository obligation), `GUARD-SCOPE.1` and
  `GUARD-SCOPE.4` (governance and scope boundaries). Each has a stated
  justification and remains binding in the BRD. **Accepted** - these are
  genuinely not product behaviour.
- No uncovered BRD item was found.

**Backward (PRD → BRD):** one orphan, disclosed by the author.

- `NFR-A11Y.1` has no BRD source. Retained as a quality target because the
  checklist view encodes pass and fail results, and colour-only encoding would
  fail the reviewing persona outright. **Accepted as justified, not
  gold-plating** - but see A4, since the same reasoning now also supports
  adding it to the BRD on the next change cycle.

**Matrix completeness:** forward matrix present with per-item coverage;
backward matrix present as a spot-check rather than exhaustively, which is
acceptable because every requirement carries a **Traces to** field inline; gap
report present and honest.

## 2. Acceptance Criteria Quality: **Pass with fixes**

**Subjective language:** none of the prohibited terms - "intuitive", "fast",
"easy", "user-friendly", "seamless", "responsive", "clean", "modern" - appears
as requirement language. One borderline case found and fixed (A1).

**Testability:** every FR carries at least one machine-format predicate. Human
criteria use Given/When/Then. Two criteria had thresholds with no basis (C2,
A2).

**NFR quantification:** all NFRs carry primary, secondary and guardrail layers.
Two lacked a benchmark source and were invented figures - the exact failure the
product's own `GUARD-ACC.1` prohibits in its output. Fixed under C2.

## 3. Completeness: **Fail at review - Pass after fixes**

**Structure:** all six core sections plus four agent-readiness sections present.

**NFR category coverage** - the checklist requires Performance, Security,
Reliability, Platform and Accessibility at minimum:

| Category | Status at review | After fix |
|:---|:---|:---|
| Performance | Present - `NFR-PERF.1` to `.4` | Present |
| Platform | Present - `NFR-PORT.1` | Present |
| Accessibility | Present - `NFR-A11Y.1` | Present |
| **Security** | **MISSING** | **`NFR-SEC.1` added** (C1) |
| **Reliability** | **MISSING** | **`NFR-REL.1` added** (C1) |

**Requirement metadata:** every requirement has ID, traces-to, statement and
criteria. Data requirements specify source, refresh, retention and privacy.
UX requirements include happy path, error states, edge cases and loading or
empty states.

**Persona coverage:** P-REV and P-AUT each have named journeys. **P-TOOL had no
journey of its own** (A3) - it appeared only inside `UX-EXPLORE.1`'s "All".
Fixed.

## 4. Agent-Readiness: **Pass with fixes**

**Semantic glossary:** 14 terms. Five domain terms were used in requirements
without definition - `tile atlas`, `overlay`, `movement class`, `geovent`,
`topolines` (A5). Fixed.

**Protection patterns:** eight locked decisions, each with a rationale and a
traceable source. No gaps.

**Phase structure:** phase ID, name, acronym, directory, outcome, entry and
exit criteria all match the Intent Brief registry. No forward dependency; P01
has no prerequisites. No cross-phase leakage - every deferred item names the
registered phase that owns it. Phase name describes an outcome and is not a
placeholder.

**Verification:** a 13-item checklist covers every exit criterion. Commands are
marked illustrative pending the technical design, which is correct at PRD
stage.

**Non-goals:** stated positively throughout, each naming the owning phase and a
reason.

## 5. Research Evidence: **Pass**

Spot-checked three references against current sources, all gathered 2026-09-20
and re-verified during this review:

- **`PR-TECH-01`** - gdext issue #372, no typed API for external GDExtension
  classes. Still open. The `ASSUM-3` risk stands.
- **`PR-TECH-03`** - Terrain3D `shader_override_enabled`. Still documented as
  fully replacing the generated shader. `MARK-PARITY.1`'s vehicle is intact.
- **`MR-CHK-16`** - wind 0-30, tidal 0-25. Matches the published checklist.

No competitive shift since research. No stale data - all references are
same-day. Data volume assumptions derive from direct measurement of 254
archives rather than estimation.

---

## Issues

### Critical

**C1 - No security requirement, despite executing untrusted code. FIXED.**
The product evaluates Lua from map archives that users download from the
internet and from other people. That is arbitrary third-party code, and the
sandbox around it is a genuine security boundary - the only one in the product.
The PRD research stage dismissed security as "structurally N/A" on the grounds
that there is no server, no network and no user data, which is true but
irrelevant to this threat. Reliability was likewise absent, despite a
no-exceptions corpus requirement that makes crash behaviour load-bearing.
**Action taken:** added `NFR-SEC.1` (sandbox containment, with explicit denial
of filesystem, network, process and unbounded-execution capability) and
`NFR-REL.1` (no crash on any corpus member, including deliberately malformed
input). Both trace to existing BRD sources.

**C2 - Invented numeric thresholds. FIXED.**
`NFR-PERF.4` specified 4 GB and 2 GB memory targets with no benchmark source;
the research bundle explicitly recorded memory as un-benchmarked for this
domain. A document whose product is forbidden from substituting plausible
values should not substitute plausible values in its own requirements.
**Action taken:** `NFR-PERF.4` restated as a measure-then-fix requirement - the
threshold is established from the reference set once the texture pipeline is
settled (`PR-TECH-05` gap), and the guardrail "shall not exhaust memory on any
corpus map" - which is falsifiable today - is retained as the binding
constraint.

### Advisory

**A1 - "visually distinct" in `FR-SHOW.11`. FIXED.** Borderline subjective.
Replaced with a testable predicate: the three states are distinguished by a
non-colour attribute as well as colour, consistent with `NFR-A11Y.1`.

**A2 - Arbitrary threshold in `FR-CHECK.3`. FIXED.** `linked_properties / 20 >=
0.5` had no basis. Replaced by naming the specific properties that must link,
which is verifiable and derived from what the overlays and camera can actually
address.

**A3 - P-TOOL had no journey. FIXED.** Added `UX-VERIFY.1`, the tooling
developer's compare-against-own-output workflow. Without it the persona
existed in the PRD but drove nothing.

**A4 - `NFR-A11Y.1` remains a PRD-only requirement.** Justified, but the
cleaner resolution is a BRD accessibility requirement on the next change cycle
via `propagate-change`. **Not fixed** - changing a baselined BRD to tidy a
justified orphan is not worth a change record now. Logged.

**A5 - Five undefined glossary terms. FIXED.** `tile atlas`, `overlay`,
`movement class`, `geovent` and `topolines` added.

**A6 - `FR-READ.6` priority inconsistency. NOT FIXED - logged deliberately.**
Legacy `.smd` support is P1, but `BIZ-EXP.1` is P0 and admits no exclusions, so
in practice it cannot be dropped. The PRD already notes this inline. Raising it
to P0 would be defensible; leaving it P1 with the note is also defensible,
since it affects exactly one corpus map. Flagged for the technical design to
treat as effectively mandatory.

**A7 - Verification commands are illustrative.** Correct at PRD stage; the
technical design must fix the actual interface. No action.

---

## Remediation actions

| # | Issue | Severity | Action | Route to |
|:--|:---|:---|:---|:---|
| 1 | C1 no security or reliability NFR | Critical | **Done** - `NFR-SEC.1`, `NFR-REL.1` added | This review |
| 2 | C2 invented memory thresholds | Critical | **Done** - restated as measure-then-fix with a falsifiable guardrail | This review |
| 3 | A1 subjective phrasing | Advisory | **Done** | This review |
| 4 | A2 arbitrary link threshold | Advisory | **Done** | This review |
| 5 | A3 P-TOOL without a journey | Advisory | **Done** - `UX-VERIFY.1` added | This review |
| 6 | A4 accessibility orphan | Advisory | Deferred - add to BRD at next change cycle | `propagate-change` |
| 7 | A5 undefined glossary terms | Advisory | **Done** | This review |
| 8 | A6 `FR-READ.6` priority | Advisory | Logged; treat as effectively mandatory | `draft-tdd` |
| 9 | A7 illustrative commands | Advisory | None needed | `draft-tdd` |

## Recommendation

**Pass for TDD handoff.** Traceability is complete in both directions, with the
single orphan disclosed and justified. Acceptance criteria are testable and free
of subjective language. Agent-readiness is strong: the protection patterns are
specific and sourced, and the phase boundary is clean.

The security gap (C1) is worth noting beyond its fix. It was missed because the
research stage reasoned about the product's *deployment shape* - no server, no
network, no accounts - rather than its *inputs*. A tool that executes Lua from
files strangers produced has a real attack surface, and the sandbox that
`FR-READ.5` already requires is now backed by an explicit security requirement
rather than being an implementation detail that happened to be mentioned.

**Routing.** Proceed to `analyze-tdd-readiness` for P01. The technical design
should treat three items as priorities: the `ASSUM-2` and `ASSUM-3` spikes
(Terrain3D runtime data injection and Rust-to-Terrain3D interop), the
`PR-TECH-05` gap (whether compressed tile data can be handed to Godot, which
determines `NFR-PERF.4`), and the sandbox threat model now required by
`NFR-SEC.1`.

## Quality gates

| Gate | Status |
| --- | --- |
| All 5 validation categories evaluated | Met |
| Every issue logged with severity and remediation | Met - 9 issues, 2 critical |
| Traceability verified in both directions | Met - 27 of 27 forward; 1 orphan, disclosed |
| Acceptance criteria quality confirmed | Met - after two fixes |
| Agent-readiness confirmed | Met - after glossary fix |
| Research spot-check complete | Met - 3 references re-verified |
| Clear recommendation stated | Met - Pass for TDD handoff |
