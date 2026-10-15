# P01 - Map Asset Exploration Task Summary

> **Phase:** P01
> **Acronym:** MAE
> **Directory:** `docs/requirements/phases/phase-01-map-asset-exploration/`
> **Progress:** 0/32 (0%)

Generated from the flat task files in this directory. `Dependents` are derived
from authored prerequisites, so the two directions cannot disagree.

## Ready Now

Tasks that are `Todo`, whose prerequisites are all `Done`, and which have no
unresolved external blocker.

- [P01-MAE-FRONTEND-001 - Spike Terrain3D runtime data injection and Rust interop](P01-MAE-FRONTEND-001-spike-terrain3d-runtime-injection.md) - `FRONTEND` · `P0` · `M`
- [P01-MAE-FRONTEND-002 - Spike compressed texture upload path](P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md) - `FRONTEND` · `P0` · `S`
- [P01-MAE-DEVOPS-001 - Scaffold the workspace and record the licence](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md) - `DEVOPS` · `P0` · `M`

All three are phase-start boundary tasks with no prerequisites. The two spikes
resolve ADR-01, ADR-02, ADR-03 and ADR-04, which the design validation named as
conditions on the phase; the workspace task records the licence that the phase
entry criteria require.

## In Progress

- None

## Blocked

29 tasks are `Todo` with prerequisites not yet `Done`. They are listed in the
Execution Order table below with their blocking prerequisites.

## Progress by Phase

| Phase | Total | Todo | In Progress | Blocked | Done | Cancelled |
|:---|---:|---:|---:|---:|---:|---:|
| P01 - Map Asset Exploration | 32 | 32 | 0 | 0 | 0 | 0 |

*Blocked is reported as 0 because no task is blocked by an unresolved external
blocker; tasks awaiting prerequisites remain `Todo` by the standard's status set.*

## Progress by Type

| Type | Total | Todo | In Progress | Blocked | Done | Cancelled |
|:---|---:|---:|---:|---:|---:|---:|
| API | 2 | 2 | 0 | 0 | 0 | 0 |
| BACKEND | 10 | 10 | 0 | 0 | 0 | 0 |
| DATA | 3 | 3 | 0 | 0 | 0 | 0 |
| DEVOPS | 2 | 2 | 0 | 0 | 0 | 0 |
| DOCUMENTATION | 1 | 1 | 0 | 0 | 0 | 0 |
| FRONTEND | 9 | 9 | 0 | 0 | 0 | 0 |
| QUALITY | 4 | 4 | 0 | 0 | 0 | 0 |
| SECURITY | 1 | 1 | 0 | 0 | 0 | 0 |
| **Total** | **32** | **32** | **0** | **0** | **0** | **0** |

## Execution Order

Ordered by dependency layer first, then priority, parent deliverable, type
continuity, estimate and ID. Dependencies override every display preference.

| Layer | Task | Parent Deliverable | Type | Priority | Estimate | Status | Prerequisites |
|---:|:---|:---|:---|:---|:---|:---|:---|
| 0 | [P01-MAE-FRONTEND-001](P01-MAE-FRONTEND-001-spike-terrain3d-runtime-injection.md) | `ADR-02` | `FRONTEND` | `P0` | `M` | `Todo` | None |
| 0 | [P01-MAE-FRONTEND-002](P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md) | `ADR-03` | `FRONTEND` | `P0` | `S` | `Todo` | None |
| 0 | [P01-MAE-DEVOPS-001](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md) | `NFR-PORT.1` | `DEVOPS` | `P0` | `M` | `Todo` | None |
| 1 | [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md) | `DATA-PROV.1` | `DATA` | `P0` | `M` | `Todo` | [P01-MAE-DEVOPS-001](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md) |
| 2 | [P01-MAE-DATA-002](P01-MAE-DATA-002-extract-shared-decoder-crate.md) | `ADR-10` | `DATA` | `P0` | `L` | `Todo` | [P01-MAE-DEVOPS-001](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md), [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md) |
| 2 | [P01-MAE-DATA-003](P01-MAE-DATA-003-embed-bar-constants-with-provenance.md) | `DATA-CONST.1` | `DATA` | `P1` | `S` | `Todo` | [P01-MAE-DEVOPS-001](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md), [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md) |
| 3 | [P01-MAE-BACKEND-001](P01-MAE-BACKEND-001-implement-virtual-file-system.md) | `FR-READ.1` | `BACKEND` | `P0` | `M` | `Todo` | [P01-MAE-DATA-002](P01-MAE-DATA-002-extract-shared-decoder-crate.md) |
| 4 | [P01-MAE-BACKEND-002](P01-MAE-BACKEND-002-implement-smf-decoder.md) | `FR-READ.2` | `BACKEND` | `P0` | `L` | `Todo` | [P01-MAE-BACKEND-001](P01-MAE-BACKEND-001-implement-virtual-file-system.md), [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md) |
| 4 | [P01-MAE-SECURITY-001](P01-MAE-SECURITY-001-harden-map-definition-sandbox.md) | `NFR-SEC.1` | `SECURITY` | `P0` | `M` | `Todo` | [P01-MAE-BACKEND-001](P01-MAE-BACKEND-001-implement-virtual-file-system.md), [P01-MAE-DATA-002](P01-MAE-DATA-002-extract-shared-decoder-crate.md) |
| 4 | [P01-MAE-BACKEND-005](P01-MAE-BACKEND-005-implement-legacy-smd-parser.md) | `FR-READ.6` | `BACKEND` | `P1` | `S` | `Todo` | [P01-MAE-BACKEND-001](P01-MAE-BACKEND-001-implement-virtual-file-system.md), [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md) |
| 4 | [P01-MAE-BACKEND-008](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md) | `FR-SHOW.2` | `BACKEND` | `P1` | `L` | `Todo` | [P01-MAE-BACKEND-001](P01-MAE-BACKEND-001-implement-virtual-file-system.md), [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md), [P01-MAE-FRONTEND-002](P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md) |
| 5 | [P01-MAE-BACKEND-003](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md) | `FR-READ.3` | `BACKEND` | `P0` | `L` | `Todo` | [P01-MAE-BACKEND-002](P01-MAE-BACKEND-002-implement-smf-decoder.md), [P01-MAE-FRONTEND-002](P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md) |
| 5 | [P01-MAE-BACKEND-004](P01-MAE-BACKEND-004-implement-minimap-decoder.md) | `FR-READ.4` | `BACKEND` | `P1` | `S` | `Todo` | [P01-MAE-BACKEND-002](P01-MAE-BACKEND-002-implement-smf-decoder.md) |
| 5 | [P01-MAE-BACKEND-006](P01-MAE-BACKEND-006-implement-option-evaluation.md) | `FR-READ.7` | `BACKEND` | `P1` | `M` | `Todo` | [P01-MAE-SECURITY-001](P01-MAE-SECURITY-001-harden-map-definition-sandbox.md), [P01-MAE-BACKEND-002](P01-MAE-BACKEND-002-implement-smf-decoder.md) |
| 5 | [P01-MAE-BACKEND-007](P01-MAE-BACKEND-007-implement-terrain-derivation.md) | `FR-SHOW.10` | `BACKEND` | `P1` | `M` | `Todo` | [P01-MAE-BACKEND-002](P01-MAE-BACKEND-002-implement-smf-decoder.md), [P01-MAE-DATA-003](P01-MAE-DATA-003-embed-bar-constants-with-provenance.md) |
| 6 | [P01-MAE-BACKEND-009](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md) | `NFR-REL.1` | `BACKEND` | `P0` | `M` | `Todo` | [P01-MAE-BACKEND-003](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md), [P01-MAE-BACKEND-006](P01-MAE-BACKEND-006-implement-option-evaluation.md), [P01-MAE-BACKEND-008](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md) |
| 6 | [P01-MAE-BACKEND-010](P01-MAE-BACKEND-010-implement-checklist-evaluator.md) | `FR-CHECK.1` | `BACKEND` | `P1` | `L` | `Todo` | [P01-MAE-BACKEND-007](P01-MAE-BACKEND-007-implement-terrain-derivation.md), [P01-MAE-BACKEND-004](P01-MAE-BACKEND-004-implement-minimap-decoder.md), [P01-MAE-BACKEND-008](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md) |
| 7 | [P01-MAE-API-001](P01-MAE-API-001-implement-extension-surface.md) | `FR-SHOW.1` | `API` | `P0` | `L` | `Todo` | [P01-MAE-BACKEND-009](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md), [P01-MAE-BACKEND-010](P01-MAE-BACKEND-010-implement-checklist-evaluator.md), [P01-MAE-FRONTEND-001](P01-MAE-FRONTEND-001-spike-terrain3d-runtime-injection.md) |
| 7 | [P01-MAE-API-002](P01-MAE-API-002-implement-conformance-cli.md) | `FR-CONF.1` | `API` | `P1` | `M` | `Todo` | [P01-MAE-BACKEND-009](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md) |
| 7 | [P01-MAE-QUALITY-002](P01-MAE-QUALITY-002-fuzz-decoders-and-sandbox.md) | `NFR-REL.1` | `QUALITY` | `P1` | `M` | `Todo` | [P01-MAE-BACKEND-009](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md), [P01-MAE-SECURITY-001](P01-MAE-SECURITY-001-harden-map-definition-sandbox.md) |
| 8 | [P01-MAE-QUALITY-001](P01-MAE-QUALITY-001-build-corpus-conformance-suite.md) | `FR-CONF.1` | `QUALITY` | `P0` | `M` | `Todo` | [P01-MAE-API-002](P01-MAE-API-002-implement-conformance-cli.md) |
| 8 | [P01-MAE-FRONTEND-003](P01-MAE-FRONTEND-003-render-terrain-surface.md) | `FR-SHOW.1` | `FRONTEND` | `P0` | `L` | `Todo` | [P01-MAE-API-001](P01-MAE-API-001-implement-extension-surface.md), [P01-MAE-BACKEND-003](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md) |
| 9 | [P01-MAE-FRONTEND-004](P01-MAE-FRONTEND-004-implement-camera-and-navigation.md) | `FR-NAV.1` | `FRONTEND` | `P0` | `M` | `Todo` | [P01-MAE-FRONTEND-003](P01-MAE-FRONTEND-003-render-terrain-surface.md) |
| 9 | [P01-MAE-FRONTEND-006](P01-MAE-FRONTEND-006-render-grid-overlays.md) | `FR-SHOW.10` | `FRONTEND` | `P1` | `M` | `Todo` | [P01-MAE-FRONTEND-003](P01-MAE-FRONTEND-003-render-terrain-surface.md), [P01-MAE-BACKEND-007](P01-MAE-BACKEND-007-implement-terrain-derivation.md) |
| 9 | [P01-MAE-FRONTEND-005](P01-MAE-FRONTEND-005-present-scene-assets.md) | `FR-SHOW.4` | `FRONTEND` | `P1` | `L` | `Todo` | [P01-MAE-FRONTEND-003](P01-MAE-FRONTEND-003-render-terrain-surface.md), [P01-MAE-BACKEND-008](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md) |
| 10 | [P01-MAE-FRONTEND-007](P01-MAE-FRONTEND-007-build-inspection-interface.md) | `FR-CHECK.1` | `FRONTEND` | `P1` | `L` | `Todo` | [P01-MAE-API-001](P01-MAE-API-001-implement-extension-surface.md), [P01-MAE-BACKEND-010](P01-MAE-BACKEND-010-implement-checklist-evaluator.md), [P01-MAE-FRONTEND-006](P01-MAE-FRONTEND-006-render-grid-overlays.md) |
| 10 | [P01-MAE-QUALITY-003](P01-MAE-QUALITY-003-establish-reference-set-and-parity.md) | `FR-CONF.2` | `QUALITY` | `P1` | `M` | `Todo` | [P01-MAE-FRONTEND-004](P01-MAE-FRONTEND-004-implement-camera-and-navigation.md) |
| 10 | [P01-MAE-FRONTEND-008](P01-MAE-FRONTEND-008-implement-open-and-reload-flows.md) | `UX-OPEN.1` | `FRONTEND` | `P1` | `M` | `Todo` | [P01-MAE-API-001](P01-MAE-API-001-implement-extension-surface.md), [P01-MAE-FRONTEND-004](P01-MAE-FRONTEND-004-implement-camera-and-navigation.md) |
| 11 | [P01-MAE-QUALITY-004](P01-MAE-QUALITY-004-verify-performance-and-invariants.md) | `NFR-PERF.1` | `QUALITY` | `P1` | `M` | `Todo` | [P01-MAE-QUALITY-003](P01-MAE-QUALITY-003-establish-reference-set-and-parity.md), [P01-MAE-FRONTEND-007](P01-MAE-FRONTEND-007-build-inspection-interface.md) |
| 11 | [P01-MAE-DEVOPS-002](P01-MAE-DEVOPS-002-build-and-package-artifacts.md) | `NFR-PORT.1` | `DEVOPS` | `P1` | `M` | `Todo` | [P01-MAE-QUALITY-001](P01-MAE-QUALITY-001-build-corpus-conformance-suite.md), [P01-MAE-FRONTEND-007](P01-MAE-FRONTEND-007-build-inspection-interface.md) |
| 11 | [P01-MAE-FRONTEND-009](P01-MAE-FRONTEND-009-port-smf-shading-chain.md) | `NFR-FID.1` | `FRONTEND` | `P2` | `L` | `Todo` | [P01-MAE-FRONTEND-005](P01-MAE-FRONTEND-005-present-scene-assets.md), [P01-MAE-QUALITY-003](P01-MAE-QUALITY-003-establish-reference-set-and-parity.md) |
| 12 | [P01-MAE-DOCUMENTATION-001](P01-MAE-DOCUMENTATION-001-prepare-early-user-evaluation.md) | `BIZ-SAT.1` | `DOCUMENTATION` | `P1` | `S` | `Todo` | [P01-MAE-DEVOPS-002](P01-MAE-DEVOPS-002-build-and-package-artifacts.md), [P01-MAE-FRONTEND-007](P01-MAE-FRONTEND-007-build-inspection-interface.md) |

## Dependency Graph

```mermaid
graph TD
    FRONTEND_001["FRONTEND-001<br/>Spike Terrain3D runtime data injec"]
    FRONTEND_002["FRONTEND-002<br/>Spike compressed texture upload pa"]
    DEVOPS_001["DEVOPS-001<br/>Scaffold the workspace and record "]
    DATA_001["DATA-001<br/>Implement the provenance value mod"]
    DATA_002["DATA-002<br/>Extract the shared decoder crate f"]
    DATA_003["DATA-003<br/>Embed BAR constants with commit pr"]
    BACKEND_001["BACKEND-001<br/>Implement the virtual file system "]
    BACKEND_002["BACKEND-002<br/>Implement the SMF decoder"]
    SECURITY_001["SECURITY-001<br/>Harden the map definition sandbox"]
    BACKEND_005["BACKEND-005<br/>Implement the legacy .smd definiti"]
    BACKEND_008["BACKEND-008<br/>Implement texture and model decode"]
    BACKEND_003["BACKEND-003<br/>Implement the tile atlas decoder"]
    BACKEND_004["BACKEND-004<br/>Implement the minimap decoder"]
    BACKEND_006["BACKEND-006<br/>Implement map option selection and"]
    BACKEND_007["BACKEND-007<br/>Implement terrain derivation"]
    BACKEND_009["BACKEND-009<br/>Implement the error taxonomy and s"]
    BACKEND_010["BACKEND-010<br/>Implement the checklist evaluator"]
    API_001["API-001<br/>Implement the extension surface"]
    API_002["API-002<br/>Implement the conformance CLI"]
    QUALITY_002["QUALITY-002<br/>Fuzz the decoders and the sandbox"]
    QUALITY_001["QUALITY-001<br/>Build the corpus conformance suite"]
    FRONTEND_003["FRONTEND-003<br/>Render the terrain surface"]
    FRONTEND_004["FRONTEND-004<br/>Implement camera and navigation"]
    FRONTEND_006["FRONTEND-006<br/>Render grid overlays"]
    FRONTEND_005["FRONTEND-005<br/>Present sky, water, features and s"]
    FRONTEND_007["FRONTEND-007<br/>Build the inspection interface"]
    QUALITY_003["QUALITY-003<br/>Establish the reference set and pa"]
    FRONTEND_008["FRONTEND-008<br/>Implement open, reload and session"]
    QUALITY_004["QUALITY-004<br/>Verify performance targets and sys"]
    DEVOPS_002["DEVOPS-002<br/>Build and package the release arti"]
    FRONTEND_009["FRONTEND-009<br/>Port the SMF shading chain"]
    DOCUMENTATION_001["DOCUMENTATION-001<br/>Prepare the early-user evaluation"]
    DEVOPS_001 --> DATA_001
    DEVOPS_001 --> DATA_002
    DATA_001 --> DATA_002
    DEVOPS_001 --> DATA_003
    DATA_001 --> DATA_003
    DATA_002 --> BACKEND_001
    BACKEND_001 --> BACKEND_002
    DATA_001 --> BACKEND_002
    BACKEND_001 --> SECURITY_001
    DATA_002 --> SECURITY_001
    BACKEND_001 --> BACKEND_005
    DATA_001 --> BACKEND_005
    BACKEND_001 --> BACKEND_008
    DATA_001 --> BACKEND_008
    FRONTEND_002 --> BACKEND_008
    BACKEND_002 --> BACKEND_003
    FRONTEND_002 --> BACKEND_003
    BACKEND_002 --> BACKEND_004
    SECURITY_001 --> BACKEND_006
    BACKEND_002 --> BACKEND_006
    BACKEND_002 --> BACKEND_007
    DATA_003 --> BACKEND_007
    BACKEND_003 --> BACKEND_009
    BACKEND_006 --> BACKEND_009
    BACKEND_008 --> BACKEND_009
    BACKEND_007 --> BACKEND_010
    BACKEND_004 --> BACKEND_010
    BACKEND_008 --> BACKEND_010
    BACKEND_009 --> API_001
    BACKEND_010 --> API_001
    FRONTEND_001 --> API_001
    BACKEND_009 --> API_002
    BACKEND_009 --> QUALITY_002
    SECURITY_001 --> QUALITY_002
    API_002 --> QUALITY_001
    API_001 --> FRONTEND_003
    BACKEND_003 --> FRONTEND_003
    FRONTEND_003 --> FRONTEND_004
    FRONTEND_003 --> FRONTEND_006
    BACKEND_007 --> FRONTEND_006
    FRONTEND_003 --> FRONTEND_005
    BACKEND_008 --> FRONTEND_005
    API_001 --> FRONTEND_007
    BACKEND_010 --> FRONTEND_007
    FRONTEND_006 --> FRONTEND_007
    FRONTEND_004 --> QUALITY_003
    API_001 --> FRONTEND_008
    FRONTEND_004 --> FRONTEND_008
    QUALITY_003 --> QUALITY_004
    FRONTEND_007 --> QUALITY_004
    QUALITY_001 --> DEVOPS_002
    FRONTEND_007 --> DEVOPS_002
    FRONTEND_005 --> FRONTEND_009
    QUALITY_003 --> FRONTEND_009
    DEVOPS_002 --> DOCUMENTATION_001
    FRONTEND_007 --> DOCUMENTATION_001
```

## Critical Path

Longest dependency chain, weighted by estimate (`S`=1, `M`=3, `L`=5):

[P01-MAE-DEVOPS-001](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md) → [P01-MAE-DATA-001](P01-MAE-DATA-001-implement-provenance-value-model.md) → [P01-MAE-DATA-002](P01-MAE-DATA-002-extract-shared-decoder-crate.md) → [P01-MAE-BACKEND-001](P01-MAE-BACKEND-001-implement-virtual-file-system.md) → [P01-MAE-BACKEND-002](P01-MAE-BACKEND-002-implement-smf-decoder.md) → [P01-MAE-BACKEND-003](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md) → [P01-MAE-BACKEND-009](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md) → [P01-MAE-API-001](P01-MAE-API-001-implement-extension-surface.md) → [P01-MAE-FRONTEND-003](P01-MAE-FRONTEND-003-render-terrain-surface.md) → [P01-MAE-FRONTEND-006](P01-MAE-FRONTEND-006-render-grid-overlays.md) → [P01-MAE-FRONTEND-007](P01-MAE-FRONTEND-007-build-inspection-interface.md) → [P01-MAE-DEVOPS-002](P01-MAE-DEVOPS-002-build-and-package-artifacts.md) → [P01-MAE-DOCUMENTATION-001](P01-MAE-DOCUMENTATION-001-prepare-early-user-evaluation.md)

**Total estimated effort along the path: 49 points across 13 tasks.**

The path runs from the workspace through the shared decoder crate and the whole
decode pipeline into the extension bridge and the interface, which matches the
intent record's accepted decision to build in strict dependency order - decoder
complete and correct before anything renders.

## Traceability Gaps

**None.** Graph validation passes:

- 32 flat task files, each appearing exactly once; no subdirectories
- All IDs and filenames unique and consistent with their headings
- All prerequisites resolve to real tasks; no cycles
- Dependents are exactly the reverse of authored prerequisites
- No cross-phase edges exist - P01 has no prerequisite phases
- Every acceptance criterion has a verification command and expected result
- Dangling tasks are limited to documented phase-start and phase-end boundaries

### Coverage

Every P01 product requirement traces to at least one task, and every task traces
back to business requirements, product requirements and the technical design.
Requirements deliberately not translated into tasks are the governance
obligations recorded in the product requirements readiness analysis
(`COMP-LICENSE.1` is covered by [P01-MAE-DEVOPS-001](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md);
`GUARD-SCOPE.1` and `GUARD-SCOPE.4` govern requirement admission rather than product behaviour).

## Completed

- None. No task has been executed yet.

## Cancelled

- None.

