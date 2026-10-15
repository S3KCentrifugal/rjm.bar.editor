# P01-MAE-FRONTEND-003 - Render the terrain surface

## Description

This is the moment the product stops being a decoder and becomes a viewer. The decoded heightmap becomes a
clipmap surface wearing the map's own ground texture, at a level of detail that holds across a 16,384-unit map. The
approach was settled by the spike: Terrain3D supplies the mesh and its ten levels of detail, reached through a thin
glue layer because `gdext` offers no typed access to another extension's classes. That glue is deliberately the only
place untyped calls live. Everything about the surface - its extent, its elevation, its texture - comes from the map
and nothing is invented.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-SHOW.1](../product-requirements/product-requirements.md#22-presentation)

## Relationships

### Prerequisites

- [P01-MAE-API-001 - Implement the extension surface](P01-MAE-API-001-implement-extension-surface.md)
- [P01-MAE-BACKEND-003 - Implement the tile atlas decoder](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md)

### Dependents

- [P01-MAE-FRONTEND-004 - Implement camera and navigation](P01-MAE-FRONTEND-004-implement-camera-and-navigation.md)
- [P01-MAE-FRONTEND-005 - Present sky, water, features and start positions](P01-MAE-FRONTEND-005-present-scene-assets.md)
- [P01-MAE-FRONTEND-006 - Render grid overlays](P01-MAE-FRONTEND-006-render-grid-overlays.md)

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

- **Business Requirements:** `BIZ-EXP.1`, `BIZ-EXP.3`
- **Product Requirements:** `FR-SHOW.1`, `NFR-PERF.3`
- **Technical Design:** section 3.2, ADR-01, ADR-02, FMEA F5
- **Phase Outcome / Exit Criteria:** Every corpus member opens and is navigable

## Scope

- Terrain3D surface built from the decoded heightmap at runtime
- Ground texture from the resolved tile atlas
- A glue layer confining untyped Terrain3D calls to one place

## Non-Goals

- Parity shading, which is P01-MAE-FRONTEND-009
- Overlays, which are P01-MAE-FRONTEND-006

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/scenes/terrain.tscn` | Create | Terrain scene |
| `app/scripts/terrain3d_glue.gd` | Create | The only place untyped Terrain3D calls live |
| `app/scripts/terrain_builder.gd` | Create | Heightmap to surface |

## Implementation Requirements

1. Build the surface from `get_height_data()` at runtime. Do not route through Terrain3D's editor importer.
2. World extent is `mapx * 8` by `mapy * 8` world units; a 32x32 map is 16,384 per side and must render at full extent.
3. Apply the resolved tile atlas as the ground texture using the upload path ADR-03 selected.
4. All Terrain3D calls live in `terrain3d_glue.gd`. No other script calls Terrain3D directly, so the untyped surface stays auditable.
5. Build the surface at stage S1, before textures are available, so terrain is navigable within the 5-second budget. The texture arrives at S2 and is applied in place.
6. If Terrain3D fails to load, fail with a named cause rather than presenting an empty viewport.

## Frontend Details

The staging requirement is what keeps `NFR-PERF.3` achievable: S1 carries the
heightmap only, and the 25 MB atlas cannot be on that path. The surface therefore appears untextured first and gains
its appearance moments later - a deliberate sequence, not a defect.

Terrain3D is documented to 65.5 km against a 16,384-unit requirement, so extent is not the constraint; the atlas
memory profile is, which is why the spike came first.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-003-AC01:** Every corpus map renders as a navigable surface at its true world extent
- [ ] **P01-MAE-FRONTEND-003-AC02:** The ground texture comes from the map's own tile atlas
- [ ] **P01-MAE-FRONTEND-003-AC03:** Terrain is navigable at stage S1, before textures are decoded
- [ ] **P01-MAE-FRONTEND-003-AC04:** Every Terrain3D call is inside the glue script and nowhere else
- [ ] **P01-MAE-FRONTEND-003-AC05:** A Terrain3D load failure reports a named cause rather than an empty viewport

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-003-AC01 | Open each reference-set map and observe | Terrain matches the map's known shape at full extent |
| P01-MAE-FRONTEND-003-AC02 | Compare the rendered ground against the map's own tile atlas | The surface carries the map's texture, not a placeholder |
| P01-MAE-FRONTEND-003-AC03 | Instrument stage timing on the largest corpus archive | Navigable surface before the atlas finishes decoding |
| P01-MAE-FRONTEND-003-AC04 | `grep -rl 'Terrain3D' app/scripts/ | grep -v terrain3d_glue.gd` | No matches |
| P01-MAE-FRONTEND-003-AC05 | Run with Terrain3D removed | Named failure reported, not a blank viewport |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
