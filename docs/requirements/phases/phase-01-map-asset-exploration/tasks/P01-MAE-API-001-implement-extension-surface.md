# P01-MAE-API-001 - Implement the extension surface

## Description

This is the only place in the system where Rust and Godot meet, and the whole thread-safety argument is
concentrated here. Worker threads produce plain data and can touch nothing of Godot's; this bridge receives that data
on the main thread and turns it into the images, meshes and dictionaries the application consumes. `gdext`'s own
threading support is experimental and warned as unsound, so crossing the boundary is done by channel and deferred
call, never by handing a Godot object to a worker. The surface is deliberately narrow, returns staged state rather
than blocking, and never throws - a decode failure is a value, not an exception.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `API`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-SHOW.1](../technical-design/technical-design.md#5-interface-design)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md)
- [P01-MAE-BACKEND-010 - Implement the checklist evaluator](P01-MAE-BACKEND-010-implement-checklist-evaluator.md)
- [P01-MAE-FRONTEND-001 - Spike Terrain3D runtime data injection and Rust interop](P01-MAE-FRONTEND-001-spike-terrain3d-runtime-injection.md)

### Dependents

- [P01-MAE-FRONTEND-003 - Render the terrain surface](P01-MAE-FRONTEND-003-render-terrain-surface.md)
- [P01-MAE-FRONTEND-007 - Build the inspection interface](P01-MAE-FRONTEND-007-build-inspection-interface.md)
- [P01-MAE-FRONTEND-008 - Implement open, reload and session state](P01-MAE-FRONTEND-008-implement-open-and-reload-flows.md)

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

- **Business Requirements:** `BIZ-EXP.1`, `BIZ-EXP.3`
- **Product Requirements:** `FR-SHOW.1`, `FR-READ.7`, `NFR-REL.1`, `NFR-PERF.3`
- **Technical Design:** section 5.1, section 5.6, ADR-07, FMEA F4 and F9, invariant 10
- **Phase Outcome / Exit Criteria:** The decode pipeline reaches the presentation layer safely

## Scope

- The `MaeMap` class and its accessors as specified in the technical design
- Thread-crossing by channel and deferred call
- Conversion from core types to Godot types, on the main thread only

## Non-Goals

- Scene construction or UI, which are frontend tasks
- Any decoding logic, which belongs to `mae-core`

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-gdext/src/lib.rs` | Create | Extension registration |
| `crates/mae-gdext/src/map.rs` | Create | The `MaeMap` surface |
| `crates/mae-gdext/src/bridge.rs` | Create | Channel and deferred-call marshalling |
| `app/addons/mae/mae.gdextension` | Create | Extension manifest |

## Implementation Requirements

1. Implement the surface exactly as specified in technical design section 5.1: `open`, `get_stage`, `get_height_data`, `get_grid`, `get_texture`, `get_atlas`, `get_features`, `get_definition`, `get_checklist`, `get_lua_sources`, `get_declared_options`, `reopen_with_options`.
2. `open` never throws and never aborts. Failure is an `MaeOpenResult` with `ok = false` and an error envelope of `{stage, kind, detail}`.
3. Reject unknown option keys rather than ignoring them, because silently dropping an option changes what the user sees without telling them.
4. Godot object construction happens on the main thread only. Worker completions arrive through a channel drained by a deferred call, per technical design section 5.6.
5. A worker result that arrives after its map has been closed is discarded without touching a Godot object.
6. Serialise provenance as `{state, value?, rule?, reason?}`, and omit the verdict field entirely for unjudged checklist entries.
7. Per-frame accessors used by the scene must be constant-time; the scene reads them every frame.

## API Details

The error envelope is the contract the conformance report and the interface both depend
on. `stage` is one of `vfs`, `smf`, `smt`, `minimap`, `definition`, `texture`, `model`; `kind` is a stable
machine-readable identifier; `detail` is human-facing.

`gdext` is pinned to an exact release, and upgrading it is a deliberate work item gated by the conformance run, per
FMEA F9 and the versioning policy.

## Acceptance Criteria

- [ ] **P01-MAE-API-001-AC01:** Every method in the documented surface exists with the documented signature
- [ ] **P01-MAE-API-001-AC02:** `open` returns a result for a corrupt archive and never throws or aborts
- [ ] **P01-MAE-API-001-AC03:** No Godot object is constructed off the main thread
- [ ] **P01-MAE-API-001-AC04:** A worker result arriving after close is discarded safely
- [ ] **P01-MAE-API-001-AC05:** Unknown option keys are rejected with a typed error
- [ ] **P01-MAE-API-001-AC06:** Unjudged checklist entries omit the verdict field in serialised output

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-API-001-AC01 | Godot test scene calling every documented method | All return the documented shapes |
| P01-MAE-API-001-AC02 | Open a deliberately corrupted archive from GDScript | `ok == false` with a populated error envelope; no crash or abort |
| P01-MAE-API-001-AC03 | `cargo test -p mae-gdext bridge::main_thread_only` | Godot construction off the main thread is unreachable |
| P01-MAE-API-001-AC04 | `cargo test -p mae-gdext bridge::late_result` | A result arriving after close is discarded without panic |
| P01-MAE-API-001-AC05 | Call `reopen_with_options` with an undeclared key | Rejected with a typed error rather than ignored |
| P01-MAE-API-001-AC06 | Inspect serialised output of `get_checklist()` | Unjudged entries contain no verdict field at all - not null, not a string |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
