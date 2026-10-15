# Phase Task Index

Cross-phase overview. Each phase summary remains authoritative for task-level
ordering within that phase.

Registry: [`../intent/intent-brief.md`](../intent/intent-brief.md#8-phase-registry)

## Phases

| Phase | Acronym | Outcome | Prerequisite Phases | Tasks | Status |
|:---|:---|:---|:---|---:|:---|
| [P01 - Map Asset Exploration](phase-01-map-asset-exploration/tasks/summary.md) | `MAE` | Any BAR map opens and is navigable, with every asset class represented | None | 32 | Tasks generated, none executed |
| P02 - Map Write Round Trip | `MWRT` | A re-saved map loads in Beyond All Reason with no loss of fidelity | [P01](phase-01-map-asset-exploration/) | 0 | Not started - requirements not drafted |
| P03 - Terrain Data Editing | `TDE` | Heightmap, typemap and metalmap editable and correct in game | P02 | 0 | Not started - requirements not drafted |
| P04 - Map Layer Editing | `MLE` | Texture layers, features and definition values editable and round-tripped | P03 | 0 | Not started - requirements not drafted |

## Entry and Exit Status

| Phase | Entry criteria | Exit criteria |
|:---|:---|:---|
| P01 | **Met** - Intent Brief approved; business, product and technical requirements complete and validated; licence decision recorded by [P01-MAE-DEVOPS-001](phase-01-map-asset-exploration/tasks/P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md) | Not met - 0 of 32 tasks done |
| P02 | Not met - awaiting P01 exit | Not met |
| P03 | Not met - awaiting P02 exit | Not met |
| P04 | Not met - awaiting P03 exit | Not met |

## Progress by Phase

| Phase | Total | Todo | In Progress | Blocked | Done | Cancelled |
|:---|---:|---:|---:|---:|---:|---:|
| P01 - Map Asset Exploration | 32 | 32 | 0 | 0 | 0 | 0 |
| P02, P03, P04 | 0 | 0 | 0 | 0 | 0 | 0 |
| **Project total** | **32** | **32** | **0** | **0** | **0** | **0** |

## Progress by Type

| Type | Total | Todo | Done |
|:---|---:|---:|---:|
| API | 2 | 2 | 0 |
| BACKEND | 10 | 10 | 0 |
| DATA | 3 | 3 | 0 |
| DEVOPS | 2 | 2 | 0 |
| DOCUMENTATION | 1 | 1 | 0 |
| FRONTEND | 9 | 9 | 0 |
| QUALITY | 4 | 4 | 0 |
| SECURITY | 1 | 1 | 0 |

## Ready Now, by Phase

### P01 - Map Asset Exploration

- [P01-MAE-FRONTEND-001 - Spike Terrain3D runtime data injection and Rust interop](phase-01-map-asset-exploration/tasks/P01-MAE-FRONTEND-001-spike-terrain3d-runtime-injection.md)
- [P01-MAE-FRONTEND-002 - Spike compressed texture upload path](phase-01-map-asset-exploration/tasks/P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md)
- [P01-MAE-DEVOPS-001 - Scaffold the workspace and record the licence](phase-01-map-asset-exploration/tasks/P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md)

### P02 to P04

- None. No requirements drafted; no tasks generated.

## In Progress

- None.

## Cross-Phase Dependency Graph

```mermaid
graph LR
    P01["P01 - Map Asset Exploration (MAE)<br/>32 tasks · reader"]
    P02["P02 - Map Write Round Trip (MWRT)<br/>0 tasks · writer"]
    P03["P03 - Terrain Data Editing (TDE)<br/>0 tasks"]
    P04["P04 - Map Layer Editing (MLE)<br/>0 tasks"]
    P01 -->|"prerequisite of"| P02
    P02 -->|"prerequisite of"| P03
    P03 -->|"prerequisite of"| P04
```

**Task-level cross-phase edges: none.** P01 has no prerequisite phases, and no
P01 task depends on a task outside P01. Later phases have no tasks yet, so no
edge into P01 exists to record in either direction.

## Critical Path

### Across phases

The project critical path currently equals the P01 path, because no later phase
has tasks. P01 gates every subsequent phase: the writer in P02 can only be built
on a reader proven correct across the whole corpus.

### Within P01

[P01-MAE-DEVOPS-001](phase-01-map-asset-exploration/tasks/P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md) → [P01-MAE-DATA-001](phase-01-map-asset-exploration/tasks/P01-MAE-DATA-001-implement-provenance-value-model.md) → [P01-MAE-DATA-002](phase-01-map-asset-exploration/tasks/P01-MAE-DATA-002-extract-shared-decoder-crate.md) → [P01-MAE-BACKEND-001](phase-01-map-asset-exploration/tasks/P01-MAE-BACKEND-001-implement-virtual-file-system.md) → [P01-MAE-BACKEND-002](phase-01-map-asset-exploration/tasks/P01-MAE-BACKEND-002-implement-smf-decoder.md) → [P01-MAE-BACKEND-003](phase-01-map-asset-exploration/tasks/P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md) → [P01-MAE-BACKEND-009](phase-01-map-asset-exploration/tasks/P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md) → [P01-MAE-API-001](phase-01-map-asset-exploration/tasks/P01-MAE-API-001-implement-extension-surface.md) → [P01-MAE-FRONTEND-003](phase-01-map-asset-exploration/tasks/P01-MAE-FRONTEND-003-render-terrain-surface.md) → [P01-MAE-FRONTEND-006](phase-01-map-asset-exploration/tasks/P01-MAE-FRONTEND-006-render-grid-overlays.md) → [P01-MAE-FRONTEND-007](phase-01-map-asset-exploration/tasks/P01-MAE-FRONTEND-007-build-inspection-interface.md) → [P01-MAE-DEVOPS-002](phase-01-map-asset-exploration/tasks/P01-MAE-DEVOPS-002-build-and-package-artifacts.md) → [P01-MAE-DOCUMENTATION-001](phase-01-map-asset-exploration/tasks/P01-MAE-DOCUMENTATION-001-prepare-early-user-evaluation.md)

**49 effort points across 13 tasks.**

## Blocked Phases and Issues

- **No graph or traceability issues.** P01 validates clean.
- P02, P03 and P04 are blocked by design: each awaits its predecessor's exit criteria.
- Two design conditions remain open and are the first two P01 tasks: ADR-02 (Terrain3D
  runtime data injection) and ADR-03 (compressed texture upload). `DEP-2` stays Pending
  until both resolve.

## Completed and Cancelled

- **Completed:** 0
- **Cancelled:** 0
- **Superseded:** 0

