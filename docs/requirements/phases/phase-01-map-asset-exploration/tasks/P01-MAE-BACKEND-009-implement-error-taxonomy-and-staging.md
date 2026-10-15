# P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline

## Description

Two guarantees meet in this task. The first is that nothing crashes: the phase admits no exclusion list, so
a single panic on a single malformed archive blocks the whole phase, and the conformance run needs to say *which
stage* failed rather than merely that something did. The second is that the interface never freezes: a fifty-megabyte
archive cannot be decoded on the thread that draws frames, and the user must be flying over terrain within five
seconds of opening a file even though the tile atlas takes longer than that. The staged pipeline satisfies both -
decode runs on workers that cannot touch a Godot object, and publishes in three stages as each becomes available.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [NFR-REL.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-003 - Implement the tile atlas decoder](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md)
- [P01-MAE-BACKEND-006 - Implement map option selection and re-evaluation](P01-MAE-BACKEND-006-implement-option-evaluation.md)
- [P01-MAE-BACKEND-008 - Implement texture and model decoders](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md)

### Dependents

- [P01-MAE-API-001 - Implement the extension surface](P01-MAE-API-001-implement-extension-surface.md)
- [P01-MAE-API-002 - Implement the conformance CLI](P01-MAE-API-002-implement-conformance-cli.md)
- [P01-MAE-QUALITY-002 - Fuzz the decoders and the sandbox](P01-MAE-QUALITY-002-fuzz-decoders-and-sandbox.md)

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

- **Business Requirements:** `BIZ-EXP.1`, `BIZ-EXP.2`
- **Product Requirements:** `NFR-REL.1`, `NFR-PERF.3`, `FR-CONF.1`
- **Technical Design:** section 3.3, section 3.4, ADR-07, ADR-09, FMEA F2 and F4, invariant 7
- **Phase Outcome / Exit Criteria:** Zero crashes; terrain navigable within budget

## Scope

- Typed error enum per decode stage
- Panic-free contract in library paths
- Worker-thread execution with three-stage publication

## Non-Goals

- Marshalling into Godot types, which is P01-MAE-API-001
- Fuzzing, which is P01-MAE-QUALITY-002

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/error.rs` | Create | Stage-tagged error taxonomy |
| `crates/mae-core/src/pipeline.rs` | Create | Staged decode orchestration |
| `crates/mae-core/tests/pipeline.rs` | Create | Staging and degradation tests |

## Implementation Requirements

1. Define one error kind per decode stage: `vfs`, `smf`, `smt`, `minimap`, `definition`, `texture`, `model`. The conformance run groups by stage, which is why string errors were rejected in ADR-09.
2. Forbid `panic!`, `unwrap` and `expect` in `mae-core` library paths, enforced by lint rather than by review.
3. Publish in three stages: S1 header, heightmap and definition, at which point terrain is navigable; S2 tile atlas and shading textures; S3 models, derived overlays and checklist.
4. S1 is blocking - nothing is shown before it. S2 and S3 arrive asynchronously and update in place.
5. A failure in S2 or S3 degrades that stage only: the map stays open, and the failure is reported. A failure in S1 means the map did not open, which is a phase-blocking defect.
6. Worker threads operate only on `mae-core` types. They must not construct, read or hold any Godot type - which the crate's dependency graph already makes impossible.
7. Default worker count is available parallelism minus one, leaving a core for the main thread.

## Backend Details

Stage budgets from `NFR-PERF.3`: S1 within 5 seconds, S2 within 20 seconds, and no
interface block longer than 1 second at any point.

The degradation rule is what makes `NFR-REL.1`'s secondary metric achievable - "a deliberately corrupted archive
produces a named failure and leaves the application usable". A map whose textures fail is still a map you can fly
over and measure.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-009-AC01:** Every decode failure carries a stage tag from the controlled set
- [ ] **P01-MAE-BACKEND-009-AC02:** No `panic!`, `unwrap` or `expect` appears in `mae-core` library paths, enforced by lint
- [ ] **P01-MAE-BACKEND-009-AC03:** Terrain is navigable at S1 for every corpus map, before textures are decoded
- [ ] **P01-MAE-BACKEND-009-AC04:** A failure in S2 or S3 leaves the map open with the failure reported
- [ ] **P01-MAE-BACKEND-009-AC05:** Decode work runs on worker threads and no Godot type is reachable from them

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-009-AC01 | `cargo clippy -p mae-core -- -D clippy::unwrap_used -D clippy::expect_used -D clippy::panic` | No warnings |
| P01-MAE-BACKEND-009-AC02 | `cargo test -p mae-core pipeline::stages` | S1 completes before the tile file is read, for every corpus map |
| P01-MAE-BACKEND-009-AC03 | `cargo test -p mae-core pipeline::degradation` | An injected S2 failure leaves the map open |
| P01-MAE-BACKEND-009-AC04 | `cargo test -p mae-core error::stages` | Every error variant carries a stage from the controlled set |
| P01-MAE-BACKEND-009-AC05 | `cargo tree -p mae-core | grep -i godot` | No matches - workers cannot reach Godot types |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
