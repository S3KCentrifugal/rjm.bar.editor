# P01-MAE-FRONTEND-006 - Render grid overlays

## Description

Five decoded or derived grids - metal, terrain type, slope, passability and grass - are drawn over the
terrain in world space, each toggled independently. This is the surface the tooling developer persona actually comes
for: when their own analysis produces a slope value they doubt, they need to see the same grid rendered from the same
file and compare. It is also how a reviewer answers several checklist properties, most obviously whether metal
distribution is sane. Cell alignment must be exact, because an overlay that is half a cell out turns a correctness
tool into a source of false disagreement.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-SHOW.10](../product-requirements/product-requirements.md#22-presentation)

## Relationships

### Prerequisites

- [P01-MAE-FRONTEND-003 - Render the terrain surface](P01-MAE-FRONTEND-003-render-terrain-surface.md)
- [P01-MAE-BACKEND-007 - Implement terrain derivation](P01-MAE-BACKEND-007-implement-terrain-derivation.md)

### Dependents

- [P01-MAE-FRONTEND-007 - Build the inspection interface](P01-MAE-FRONTEND-007-build-inspection-interface.md)

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

- [UI/UX and adaptive design conventions](~/.claude/skills/convention-ui-ux/SKILL.md)
- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EXP.3`, `BIZ-EFF.1`
- **Product Requirements:** `FR-SHOW.10`, `FR-SHOW.3`, `UX-VERIFY.1`
- **Technical Design:** section 3.2
- **Phase Outcome / Exit Criteria:** Derived facts are visually verifiable

## Scope

- Five overlays drawn in world space, cell-aligned
- Independent toggling
- Per-cell value inspection with provenance

## Non-Goals

- Deriving the grids, which is P01-MAE-BACKEND-007
- Judging any grid - the tool reports, the reviewer decides

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/scripts/overlay_renderer.gd` | Create | Overlay drawing |
| `app/shaders/overlay.gdshader` | Create | Grid overlay shader |

## Implementation Requirements

1. Provide metal, type, slope, passability and grass overlays, each independently toggleable.
2. Alignment error must be zero cells. Grid strides differ - the metal and type grids are half-resolution relative to the heightmap - so the mapping must be explicit rather than assumed.
3. Show the passability overlay per movement class, since that is how the checklist asks the question.
4. Selecting a cell reveals its value and the engine rule that produced it, which is what makes the comparison workflow work.
5. An overlay whose grid has not decoded yet is shown as pending, not as empty.
6. Do not encode any overlay meaning by colour alone where the value is categorical.

## Frontend Details

Grid strides: heightmap is `(mapx+1) x (mapy+1)`; typemap and metalmap are
`(mapx/2) x (mapy/2)`. An overlay that ignores this is off by a factor of two and will look plausible while being
wrong - the worst kind of defect for a tool whose purpose is verification.

Per-cell provenance is what distinguishes this from a pretty picture. `PR-DATA-01` makes the point directly: the
developer needs not just the value but which rule produced it.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-006-AC01:** All five overlays render in world space with zero cell alignment error
- [ ] **P01-MAE-FRONTEND-006-AC02:** Overlays toggle independently
- [ ] **P01-MAE-FRONTEND-006-AC03:** Passability is shown per movement class
- [ ] **P01-MAE-FRONTEND-006-AC04:** Selecting a cell shows its value and rule citation
- [ ] **P01-MAE-FRONTEND-006-AC05:** An undecoded grid shows as pending rather than empty

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-006-AC01 | Alignment test against known grid values at map corners and centre | Zero cell offset for all five |
| P01-MAE-FRONTEND-006-AC02 | Toggle each overlay | Independent, no interference |
| P01-MAE-FRONTEND-006-AC03 | Switch movement class on the passability overlay | Result changes per class |
| P01-MAE-FRONTEND-006-AC04 | Select a cell | Value and citation displayed |
| P01-MAE-FRONTEND-006-AC05 | Open a map and inspect before S3 completes | Pending state shown |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
