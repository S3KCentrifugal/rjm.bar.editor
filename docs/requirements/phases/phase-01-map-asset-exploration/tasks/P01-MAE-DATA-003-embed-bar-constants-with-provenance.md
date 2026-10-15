# P01-MAE-DATA-003 - Embed BAR constants with commit provenance

## Description

Two of the twenty checklist properties - pathability by movement class, and whether a geothermal vent can
be built on - need data a map archive does not contain. The product cannot read it from an installed game, because
`NFR-DEP.1` requires it to work on a machine with no Beyond All Reason present. So the constants are compiled in, and
that creates the highest-rated failure mode in the whole design: a stale constant produces a wrong answer that looks
derived, carries a real citation, and passes every honesty check the product has. Detection, not prevention, is the
job here - the constants carry the commit they were read at, and that commit is shown to the user.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `DATA`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `S`
- **Parent Deliverable / User Story:** [DATA-CONST.1](../product-requirements/product-requirements.md#4-data-requirements)

## Relationships

### Prerequisites

- [P01-MAE-DEVOPS-001 - Scaffold the workspace and record the licence](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md)
- [P01-MAE-DATA-001 - Implement the provenance value model](P01-MAE-DATA-001-implement-provenance-value-model.md)

### Dependents

- [P01-MAE-BACKEND-007 - Implement terrain derivation](P01-MAE-BACKEND-007-implement-terrain-derivation.md)

### External Blockers

- None

### Supersession

- None

## References

### Specifications

- [Business requirements](../business-requirements/business-requirements.md)
- [Product requirements](../product-requirements/product-requirements.md)
- [Technical design](../technical-design/technical-design.md)
- [Information architecture](../technical-design/information-architecture.md)

### Technical Documentation, ADRs, and Contracts

- [Architecture research bundle and ADRs](../technical-design/research-bundle.md)
- [Design validation report](../technical-design/validation-report.md)

### Conventions to Apply

- [Rust conventions](~/.claude/skills/convention-rust/SKILL.md)
- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EFF.1`, `GUARD-ACC.1`, `MARK-DIFF.1`
- **Product Requirements:** `DATA-CONST.1`, `FR-CHECK.4`, `NFR-DEP.1`
- **Technical Design:** ADR-12, FMEA F7, section 4.1
- **Phase Outcome / Exit Criteria:** Pathability answerable without a game installation

## Scope

- Extraction script producing a checked-in constants file
- Commit provenance on every constant
- Making the commit displayable

## Non-Goals

- Reading BAR at run time, which is prohibited
- A build script that reads a sibling checkout, rejected in ADR-12

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/constants.rs` | Generate | Checked-in generated constants with provenance |
| `tools/extract_bar_constants.py` | Create | Deliberate extraction script |
| `crates/mae-core/tests/constants.rs` | Create | Provenance presence tests |

## Implementation Requirements

1. Extract movement definition constants - slope and depth limits per movement class - and geothermal parameters from `bar-Beyond-All-Reason`, reading only; that tree is strictly read-only.
2. Emit a checked-in generated file, not a build-script output. A build script would silently change values between builds, which is FMEA F7 happening quietly; a checked-in file makes every change a reviewable diff.
3. Every constant carries the BAR commit it was read from. A constant with no commit must not compile.
4. The commit must be exposed through the extension surface so the interface can display it wherever a dependent value is shown.
5. The file carries a do-not-edit-by-hand header and names the script that produces it.

## Data Details

FMEA F7 scores RPN 168, the highest in the system, and its detectability score is what
drives that: a stale movement constant yields a plausible pathability answer. The mitigations are deliberately all
about visibility rather than correctness - the commit is recorded, displayed, and revalidated at each milestone under
`ASSUM-7`.

`rjm.bar.tools` already reproduces the relevant thresholds - `DEPTH.MAX_SHALLOW` at 20 to wade and `DEPTH.MIN_SHALLOW`
at 8 to float, from BAR's `gamedata/movedefs.lua` - and is the reference for what to extract.

## Acceptance Criteria

- [ ] **P01-MAE-DATA-003-AC01:** Movement and geothermal constants are extracted into a checked-in generated file
- [ ] **P01-MAE-DATA-003-AC02:** Every constant carries the BAR commit it was read from, enforced at compile time
- [ ] **P01-MAE-DATA-003-AC03:** The commit is exposed through the extension surface for display
- [ ] **P01-MAE-DATA-003-AC04:** The generated file carries a do-not-edit header naming its producing script
- [ ] **P01-MAE-DATA-003-AC05:** No build-time or run-time read of a BAR checkout occurs

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-DATA-003-AC01 | `python tools/extract_bar_constants.py --check` | Regenerating produces no diff |
| P01-MAE-DATA-003-AC02 | `cargo test -p mae-core constants::provenance` | Every constant has a commit; a constant without one fails to compile |
| P01-MAE-DATA-003-AC03 | `cargo test -p mae-core constants::exposed` | Commit retrievable through the public API |
| P01-MAE-DATA-003-AC04 | `head -5 crates/mae-core/src/constants.rs` | Do-not-edit header present, naming the producing script |
| P01-MAE-DATA-003-AC05 | `cargo tree -p mae-core --build-deps` and a runtime file-access trace | No build or runtime read of a BAR checkout |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
