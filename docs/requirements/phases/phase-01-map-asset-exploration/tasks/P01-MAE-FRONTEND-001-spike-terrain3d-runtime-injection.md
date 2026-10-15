# P01-MAE-FRONTEND-001 - Spike Terrain3D runtime data injection and Rust interop

## Description

Two architecture decisions are blocked on unknowns that documentation cannot settle, and both sit under the
phase's terrain rendering. ADR-02 asks whether Terrain3D can accept terrain data programmatically at runtime: its
published documentation describes an editor importer expecting EXR or R16/RAW, and no runtime data API is documented.
ADR-01 asks how the Rust extension should reach Terrain3D at all, given that `gdext` has no type-safe API for classes
defined in other GDExtensions. This spike answers both by building the smallest possible prototype that loads a real
corpus heightmap into a Terrain3D surface and flies over it, then records what it cost. It exists first because
`DEP-2` is still Pending and because a negative result forces `DR-6` to be revisited through governance rather than
quietly worked around. The prototype is throwaway; the decision it produces is not.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [ADR-02](../technical-design/technical-design.md#8-alternatives-considered)

## Relationships

### Prerequisites

- None

### Dependents

- [P01-MAE-API-001 - Implement the extension surface](P01-MAE-API-001-implement-extension-surface.md)

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

- **Business Requirements:** `MARK-PARITY.1`, `BIZ-EXP.2`
- **Product Requirements:** `FR-SHOW.1`, `NFR-PERF.1`
- **Technical Design:** ADR-01, ADR-02, section 3.2
- **Phase Outcome / Exit Criteria:** Unblocks the exit criterion that every corpus member opens and is navigable

## Scope

- Prototype loading a decoded heightmap into Terrain3D at runtime, without using the editor importer
- Prototype driving Terrain3D from Rust directly, and via a thin GDScript layer, and comparing both
- Recording the outcome as a resolution to ADR-01 and ADR-02

## Non-Goals

- Production-quality terrain rendering, which is P01-MAE-FRONTEND-003
- Shader parity work, which is P01-MAE-FRONTEND-012
- Any decoding work beyond reading one heightmap by the simplest means available

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `spikes/terrain3d-injection/` | Create | Throwaway prototype project, retained for evidence |
| `docs/requirements/phases/phase-01-map-asset-exploration/technical-design/technical-design.md` | Modify | Record ADR-01 and ADR-02 outcomes as Accepted or Superseded |
| `spikes/terrain3d-injection/FINDINGS.md` | Create | What worked, what did not, and the measured cost of each option |

## Implementation Requirements

1. Read the Terrain3D source rather than relying on its documentation; the documented path is an editor tool and the question is whether a programmatic path exists at all.
2. Load a real corpus heightmap - use `mescaline_v2` and one 32x32 map so both extremes are exercised - not synthetic data.
3. Implement the same operation twice: once calling Terrain3D from Rust dynamically, once through a GDScript layer. Record lines of code, failure modes encountered, and whether errors surface at build or at run time.
4. If no runtime injection path exists, stop and raise a `propagate-change` request against `DR-6` rather than designing around it. That is a registry-level decision.
5. Record the Terrain3D release tag and Godot version used, per the versioning policy in technical design section 5.5.

## Frontend Details

The prototype must demonstrate a full round trip: heightmap bytes in, navigable
surface out. Success is not "Terrain3D compiles"; it is "a corpus map's real elevation is visible and can be flown
over". Capture a screenshot for the findings file.

Both interop options must be exercised against the same operation so the comparison is meaningful. The known risk is
that dynamic calls from Rust fail only at run time with unhelpful errors; if that is what happens, say so plainly,
because it is the evidence ADR-01 needs.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-001-AC01:** A real corpus heightmap is visible as a navigable Terrain3D surface, loaded at runtime without using the editor importer
- [ ] **P01-MAE-FRONTEND-001-AC02:** Both interop routes - direct Rust and GDScript glue - are implemented for the same operation and compared in writing
- [ ] **P01-MAE-FRONTEND-001-AC03:** ADR-01 and ADR-02 are updated from Proposed to Accepted, or a propagate-change request is raised against DR-6
- [ ] **P01-MAE-FRONTEND-001-AC04:** The Terrain3D release tag and Godot version used are recorded

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-001-AC01 | Run the prototype and observe the loaded map | The elevation of the chosen corpus map is recognisably rendered and can be flown over |
| P01-MAE-FRONTEND-001-AC02 | Read `spikes/terrain3d-injection/FINDINGS.md` | Both interop routes are described with measured cost and observed failure modes |
| P01-MAE-FRONTEND-001-AC03 | Inspect the technical design ADR section | ADR-01 and ADR-02 carry a status other than Proposed |
| P01-MAE-FRONTEND-001-AC04 | Inspect FINDINGS.md header | Terrain3D tag and Godot version recorded |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
