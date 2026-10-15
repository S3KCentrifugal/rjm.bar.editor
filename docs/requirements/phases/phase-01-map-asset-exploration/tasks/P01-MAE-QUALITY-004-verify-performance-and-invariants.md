# P01-MAE-QUALITY-004 - Verify performance targets and system invariants

## Description

Several of this phase's guarantees are structural rather than behavioural - the core cannot depend on
Godot, no write path to a map file exists, exactly five checklist properties are ever judged - and structural
guarantees decay silently unless something checks them. This task turns the twelve system invariants into automated
checks and measures the four performance targets against the reference set. It also settles the one number the design
deliberately left open: `NFR-PERF.4` carries no memory ceiling until it is measured, and this is where the
measurement happens.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `QUALITY`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [NFR-PERF.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- [P01-MAE-QUALITY-003 - Establish the reference set and parity baseline](P01-MAE-QUALITY-003-establish-reference-set-and-parity.md)
- [P01-MAE-FRONTEND-007 - Build the inspection interface](P01-MAE-FRONTEND-007-build-inspection-interface.md)

### Dependents

- None

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

- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)
- [Deployment and operations conventions](~/.claude/skills/convention-deployment/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EXP.2`, `OPER-SCALE.1`
- **Product Requirements:** `NFR-PERF.1`, `NFR-PERF.2`, `NFR-PERF.3`, `NFR-PERF.4`, `NFR-VALID.2`, `NFR-DEP.1`
- **Technical Design:** section 10, section 2.2
- **Phase Outcome / Exit Criteria:** Traversal at or faster than in-game; structural guarantees enforced

## Scope

- Automated checks for all twelve system invariants
- Frame rate, traversal, staged load and memory measurement against the reference set
- Setting the NFR-PERF.4 ceiling from measurement

## Non-Goals

- Optimising anything, which is separate work if a target is missed
- Parity comparison, which is P01-MAE-FRONTEND-009

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `tests/invariants/` | Create | One check per system invariant |
| `tests/performance/` | Create | Performance harness |
| `.github/workflows/invariants.yml` | Create | CI enforcement |

## Implementation Requirements

1. Implement a check for each of the twelve invariants in technical design section 10, including the dependency check on `mae-core`, the absence of map write paths, the five-judged-properties rule, and the cross-repository major version parity.
2. Measure sustained frame rate during scripted traversal of every reference-set map; 60 fps target, 45 fps at the 1% low, never below 30.
3. Measure corner-to-corner traversal time per reference-set map using the same procedure as the in-game baseline, and compare.
4. Measure time to navigable terrain and time to full texture resolution; 5 and 20 seconds respectively, with no interface block over one second.
5. Measure peak resident memory per reference-set map and set the `NFR-PERF.4` ceiling from those measurements, expressed as a multiple of the median rather than an absolute figure.
6. Fail the build on any invariant violation. Report performance regressions against the recorded baseline rather than failing on absolute thresholds, which vary by hardware.

## Quality Details

The twelve invariants are in technical design section 10. Two are already
enforceable today and should be first: `mae_core_manifest.dependencies ∌ godot` and
`count(write_paths_to_map_files) == 0`. Both are greppable, both protect guarantees that are otherwise easy to erode
by a well-meaning change.

Performance is measured against the reference set rather than the whole corpus, because the reference set is where the
in-game baseline exists to compare against.

## Acceptance Criteria

- [ ] **P01-MAE-QUALITY-004-AC01:** All twelve system invariants have automated checks and any violation fails the build
- [ ] **P01-MAE-QUALITY-004-AC02:** Sustained frame rate, 1% low and minimum are measured for every reference-set map
- [ ] **P01-MAE-QUALITY-004-AC03:** Corner-to-corner traversal is measured and compared against the in-game baseline per map
- [ ] **P01-MAE-QUALITY-004-AC04:** Time to navigable terrain and to full textures are measured against their budgets
- [ ] **P01-MAE-QUALITY-004-AC05:** A memory ceiling for NFR-PERF.4 is set from measurement and recorded in the product requirements

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-QUALITY-004-AC01 | `cargo test --test invariants` | Twelve checks, all passing |
| P01-MAE-QUALITY-004-AC02 | Introduce a Godot dependency into `mae-core` | Invariant check fails the build |
| P01-MAE-QUALITY-004-AC03 | Run the performance harness over the reference set | Frame rate, traversal, load and memory all recorded |
| P01-MAE-QUALITY-004-AC04 | Compare traversal against the in-game baseline | Editor time at or below in-game time for every reference map |
| P01-MAE-QUALITY-004-AC05 | Inspect the product requirements | `NFR-PERF.4` carries a measured ceiling |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
