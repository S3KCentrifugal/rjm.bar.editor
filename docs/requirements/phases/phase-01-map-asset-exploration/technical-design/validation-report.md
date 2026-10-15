---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
document: TDD Review Report
reviewed: 2026-09-20
subject: technical-design.md + information-architecture.md
weighted_score: 3.9
next: generate-tasks (phase P01)
---

# TDD Review Report: P01 - Map Asset Exploration (MAE)

## Summary

- **Overall assessment:** **Pass with Conditions**
- **Weighted architecture score:** **3.9 / 5.0** (was 3.75 before the fixes
  applied during this review)
- **Issues found:** 8
- **Critical (must fix):** 2 - both fixed during review
- **Advisory (should fix):** 6 - two fixed, four logged

**The score is capped by a real project property, not by document quality.**
Maintainability scores 3 because the bus factor is 1 - a sole volunteer
maintainer - which no amount of design work changes. That constraint is already
recorded as advisory A3 in the BRD validation and as `RISK-4`. The design itself
is unusually well specified for this stage.

**Phase consistency.** BRD, PRD, TDD and IA all carry `phase_id: P01`, the
acronym `MAE`, and the directory
`docs/requirements/phases/phase-01-map-asset-exploration/`. All four match the
Intent Brief registry. No cross-phase leakage found.

## 1. Traceability: **Pass**

**PRD → TDD (forward).** 54 of 54 PRD requirements covered.

- 28 FRs → TDD §2 technical behaviours T-1 to T-19 (several FRs collapse into
  one behaviour where they share a mechanism, each cited).
- 15 NFRs → TDD §2 targets N-1 to N-13, each with an architectural decision.
- 5 DATA → TDD §4, with `DATA-PROV.1` given a first-class type.
- 6 UX → IA §6 interaction model and §4 journey diagrams.
- **No uncovered PRD requirement found.**

**TDD → PRD (backward).** Every TDD section traces to at least one PRD
requirement. All 13 ADRs cite the requirement they optimise for. **No orphan
design sections.**

**IA → PRD.** All 10 data flow stages carry a PRD trace. All four sequence
diagrams map to a named persona journey - `UX-OPEN.1`/`UX-EXPLORE.1`,
`UX-CHECK.1`, `UX-VERIFY.1`, and the conformance run serving `FR-CONF.1`.

## 2. C4 Diagrams: **Pass**

**Level 1 (Context):** 6 elements, under the 10 limit. System boundary is a
single box. Runtime versus design-time dependencies are visually distinguished
by line style, which makes `NFR-DEP.1` legible at a glance - a good use of the
notation rather than decoration.

**Level 2 (Container):** every deployable unit shown; each labelled with its
technology; protocols on every arrow; abstraction level consistent - no tables
or classes leak in. Containers match the IA service catalogue exactly (6 units,
6 rows).

**Level 3:** two component diagrams, both justified - the decode pipeline is
genuinely non-obvious, and the thread and trust boundary diagram carries a
safety property that prose states less clearly. No invented abstraction levels.

## 3. Data Design: **Pass**

- Schemas present with types and cardinality; the `Provenanced<T>` type is
  specified in full because the phase's central guarantee rests on it.
- Every storage selection cites the scalability checklist. The decision *not*
  to use a database is argued rather than assumed, which is the right treatment
  given how reflexively one would otherwise be added.
- Caching: one layer, with key, invalidation and a stated rejection of the disk
  alternative (ADR-11).
- Lifecycle: retention per data type; map files marked explicitly as never
  written.
- **IA cross-check:** the IA store map (5 stores) matches the TDD storage
  selections (6 rows including conformance output). Persistence strategy covers
  every data type in the TDD schemas. **Consistent.**
- **CAP alignment:** not applicable, correctly - no distributed store, no
  partitions, no replicas.

## 4. Interface Contracts: **Pass with fixes**

No network API exists, and the TDD states this rather than leaving it implicit.
The three in-process contracts are specified with equivalent rigour.

- Extension surface: methods, parameter types, return shapes, validation rules
  and a consistent error envelope `{stage, kind, detail}`.
- Checklist entry: full JSON contract, including the deliberate choice that
  `verdict` is **absent from the payload** rather than null for unjudged
  properties - so no consumer can render a verdict that does not exist. This is
  a good example of a guardrail enforced at the schema level.
- CLI: arguments, exit codes, output schema, determinism guarantee.
- Thread-crossing contract: specified, which most designs leave as folklore.

**Gap found and fixed (C1):** *no versioning strategy* was stated for any
contract. This matters more than usual because ADR-10 shares `mae-core` with
`rjm.bar.tools` as a cross-repository crate - an unversioned shared dependency
between two repositories is a maintenance hazard. **Fixed:** a versioning
section was added covering the shared crate, the extension surface and the CLI.

**Authentication and authorization per endpoint:** not applicable - stated
rather than omitted.

## 5. Failure Modes: **Pass with fixes**

- FMEA present with 8 entries, RPN-scored.
- High-RPN modes have mitigations; F7 (constant staleness, RPN 168) receives
  extended treatment, correctly identified as the hardest to *detect* rather
  than to prevent.
- Two modes - F4 and F8 - are eliminated by construction rather than mitigated,
  which is stronger than a mitigation and is called out as such.
- Circuit breakers, retries and DLQs are marked not applicable **with a
  reason**: every dependency is a local file or in-process library, and
  retrying a malformed file yields the same malformed file. This is correct, not
  evasive.
- RPO and RTO stated as not applicable with justification.

**Gap found and fixed (C2):** the FMEA covered Terrain3D but **omitted the other
external dependencies** - `gdext` itself, the Godot runtime, and the filesystem.
`gdext` breaking changes are registered as `DEP-3` in the BRD but had no failure
mode. **Fixed:** F9, F10 and F11 added.

**IA cross-check:** resilience patterns in the IA service catalogue match TDD
§7. **Consistent.**

## 6. Alternatives Considered: **Pass**

- 13 ADRs in the research bundle; 4 reproduced in full in TDD §8 as required -
  storage, compute, primary protocol, and the security-equivalent decision.
- Each option carries pros, cons and evidence keys.
- Each decision cites the constraint it optimises for and states the trade-off
  accepted.
- **No straw men.** Checked specifically: the rejected options are all ones a
  competent engineer might choose. "Synchronous decode on the main thread" is
  rejected on a measured requirement, not ridiculed; "process isolation for
  Lua" is rejected as disproportionate to the threat model rather than as
  wrong; "direct dynamic calls from Rust" is a genuine contender that loses on
  a narrow margin.
- TDD §8 also records what was *not* decided - no auth architecture, and why -
  which prevents a later reader assuming an oversight.

## 7. Architecture Review Scores

| Dimension | Weight | Score | Key finding |
|:---|:---|:---:|:---|
| **Scalability** | 20% | **4** | The user-scale dimension does not exist (one user, one process). The scaling axis that does apply is map size: Terrain3D covers 65.5 km against a 16,384-unit requirement, roughly 4x headroom. Staged loading and a worker cap handle the real resource curve. Not 5 because the memory ceiling is deferred to measurement pending ADR-03 |
| **Security** | 20% | **4** | One boundary, correctly identified and well designed - whitelist plus resource ceilings, with capability and exhaustion treated as separate axes. Absence of auth, authz and encryption is structural and stated, not overlooked. Not 5 because a sandbox escape in the Lua implementation itself is uncontained, an accepted trade-off |
| **Maintainability** | 20% | **3** | Module boundaries are excellent and partly enforced by the type system; 10 machine-verifiable invariants; constraints in three tiers. **Capped by bus factor = 1.** A sole volunteer maintainer is a real property of this project, recorded as `RISK-4` and BRD advisory A3, and no design choice removes it. GPL-and-public (ADR-13, `COMP-LICENSE.1`) is the mitigation that exists |
| **Performance** | 15% | **4** | Budgets are quantified and, for traversal, externally anchored to the game itself rather than invented. Staged loading directly serves the 1-second interaction limit. Not 5 because `NFR-PERF.4` has no numeric ceiling until ADR-03's spike - honest, but incomplete |
| **Deployment** | 15% | **4** | Build matrix, two artifacts per platform with a stated reason, conformance run as a cross-platform differential gate, fuzzing and clean-machine checks. Raised from 3 by the C1 fix, which added the update and versioning story. Zero-downtime deployment and rollback are not applicable to a desktop download; user-side update is now addressed |
| **Documentation** | 10% | **5** | C4 at three levels, 13 ADRs with real trade-offs, three interface contracts, an IA with four sequence diagrams and an ER model, 10 invariants, extended TOC. Diagrams are Mermaid and live with the source |
| **Weighted total** | | **3.9** | Pass with conditions |

**Calculation:** (4 x 0.20) + (4 x 0.20) + (3 x 0.20) + (4 x 0.15) +
(4 x 0.15) + (5 x 0.10) = 0.80 + 0.80 + 0.60 + 0.60 + 0.60 + 0.50 = **3.90**.

**On not rounding up.** 3.9 sits just below the 4.0 pass threshold, and the
gap is entirely the maintainability dimension. Inflating that score to reach a
clean pass would misrepresent a real, documented project risk. The honest
reading is: the design is sound and ready, and the project's principal
structural weakness is that one person holds all of it.

## 8. Agent-Readiness: **Pass**

- Constraints use the three-tier system: 5 Always, 5 Ask First, 7 Never, each
  citing an ADR or requirement.
- 10 system invariants, each expressible as a test or lint - including two
  (`mae_core_manifest.dependencies ∌ godot`, `count(write_paths_to_map_files)
  == 0`) that are enforceable in CI rather than by review.
- Extended TOC present with per-section summaries.
- Section sizing is reasonable; no section requires the whole document as
  context.
- Code examples present (the provenance type); counter-examples present in the
  IA design principles as explicit violation cases - a stronger pattern than
  examples alone.

## 9. IA Validation: **Pass**

- Every service in the catalogue appears in the C4 Container diagram, and
  conversely. **6 for 6.**
- Communication topology matrix (8 pairs) matches the Container diagram arrows.
- Security boundaries explicitly drawn, including a dedicated trust-boundary
  diagram. No implicit trust assumption found - notably, gadget Lua is shown
  reaching a text viewer and *never* the sandbox.
- Cross-cutting concerns: 6 areas, each either designed or marked inapplicable
  with a reason. Telemetry's absence is argued rather than omitted, which
  prevents a later contributor "fixing" it.
- All diagrams Mermaid or ASCII; text-renderable.
- No implementation detail leaks into the IA - schemas and algorithms stay in
  the TDD. The boundary between the two documents is respected.

---

## Issues and remediation

| # | Issue | Severity | Action | Route to |
|:--|:---|:---|:---|:---|
| C1 | No versioning strategy for the shared crate, extension surface or CLI - acute because ADR-10 shares `mae-core` across two repositories | Critical | **Done** - §5.5 Versioning added to the TDD | This review |
| C2 | FMEA omitted `gdext`, the Godot runtime and the filesystem as external dependencies | Critical | **Done** - F9, F10, F11 added | This review |
| A1 | The shader port (`NFR-FID.1`) has an ADR but no design detail - no slot-by-slot mapping, no staging | Advisory | Logged. P2 and the designated sacrifice; detailing it now would invest in the first thing to be cut | `generate-tasks` |
| A2 | `mae-reference` appears in the catalogue and container diagram but has no contract - how camera positions are recorded, how comparison is performed | Advisory | **Done** - capture manifest contract added to TDD §5.3 | This review |
| A3 | Bus factor is 1 | Advisory | Not fixable by design. Mitigated by GPL-and-public so continuation is possible; already `RISK-4` | Accepted |
| A4 | Lua instruction and memory ceilings are stated as "calibrated against the corpus" with no procedure | Advisory | Logged - belongs in a task, not the design | `generate-tasks` |
| A5 | Two ADRs (02, 03) remain blocked on spikes; `DEP-2` is still Pending | Advisory | Logged - both are the first tasks generated | `generate-tasks` |
| A6 | No performance budget for the overlay renderer specifically, though five overlays draw over a 16k terrain | Advisory | Logged | `generate-tasks` |

## Recommendation

**Pass with Conditions - proceed to task generation.**

The two critical issues were document gaps rather than design faults and were
fixed during review. The four remaining advisories are all work items rather
than design defects, and each routes naturally into the task graph.

**Two conditions attach to the phase, not to the document:**

1. **ADR-02 and ADR-03 must be settled by spike before implementation commits
   to Terrain3D or a texture memory strategy.** `DEP-2` remains Pending. If
   Terrain3D cannot accept terrain data at runtime, `DR-6` must be revisited
   through `propagate-change` - that is a registry-level decision, not a coding
   choice.
2. **`NFR-PERF.4` has no numeric ceiling until ADR-03 is settled.** The
   falsifiable guardrail - never exhausts memory - binds in the meantime.

**Routing.** Proceed to `generate-tasks` for P01. The first tasks generated
should be the two spikes, because several downstream design commitments depend
on their outcome.

## Quality gates

| Gate | Status |
| --- | --- |
| All 9 validation categories evaluated | Met |
| Weighted architecture score calculated | Met - 3.9, with the calculation shown |
| Every issue logged with severity and remediation | Met - 8 issues, 2 critical |
| TDD and IA cross-checked for consistency | Met - service catalogue, topology matrix, store map and resilience patterns all reconciled |
| Agent-readiness confirmed | Met |
| Clear recommendation stated | Met - Pass with Conditions, two phase-level conditions named |
| Phase identity, scope, links and path consistent across inputs | Met - BRD, PRD, TDD, IA all P01/MAE |
