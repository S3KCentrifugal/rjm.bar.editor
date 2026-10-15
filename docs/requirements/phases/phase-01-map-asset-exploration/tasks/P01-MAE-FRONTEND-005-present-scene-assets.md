# P01-MAE-FRONTEND-005 - Present sky, water, features and start positions

## Description

"All of a map's assets" was the phrase the scope was written around, and this task delivers most of what
that means beyond the ground itself: the sky the map declares, water at the level it sets, its rocks and trees standing
where its placement data puts them, and markers at the start positions its teams table defines. The recurring theme is
absence. Two thirds of corpus maps ship no skybox and a quarter have no features at all, and in both cases the map is
complete and correct - so the presentation layer must treat missing assets as normal and say so, rather than rendering
something that looks broken.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-SHOW.4](../product-requirements/product-requirements.md#22-presentation)

## Relationships

### Prerequisites

- [P01-MAE-FRONTEND-003 - Render the terrain surface](P01-MAE-FRONTEND-003-render-terrain-surface.md)
- [P01-MAE-BACKEND-008 - Implement texture and model decoders](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md)

### Dependents

- [P01-MAE-FRONTEND-009 - Port the SMF shading chain](P01-MAE-FRONTEND-009-port-smf-shading-chain.md)

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

- **Business Requirements:** `BIZ-EXP.3`, `OPER-SCALE.1`
- **Product Requirements:** `FR-SHOW.4`, `FR-SHOW.5`, `FR-SHOW.6`, `FR-SHOW.8`, `FR-READ.8`
- **Technical Design:** section 3.2
- **Phase Outcome / Exit Criteria:** Every asset class represented

## Scope

- Sky and fog, with a defined default when the map supplies none
- Water at the map's level using its declared material values
- S3O feature instances placed with terrain-derived elevation
- Start position markers with team indices

## Non-Goals

- Feature property display, which belongs to the inspection UI
- Executing anything a map declares

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/scripts/scene_presenter.gd` | Create | Asset presentation |
| `app/scenes/sky.tscn` | Create | Sky with default |
| `app/scenes/water.tscn` | Create | Water plane and material |
| `app/scripts/feature_placer.gd` | Create | S3O instancing |

## Implementation Requirements

1. When a map supplies no skybox - true of 166 corpus maps - present a defined default and report the sky as absent, not as an error.
2. Place water at `y == 0` in map space per engine convention, using the water material values from the definition.
3. Place each feature instance at its declared position and rotation, deriving elevation from the terrain rather than using a fixed value.
4. Instance repeated models rather than duplicating meshes; a single map can carry thousands of placements.
5. Render start position markers with their team indices visible.
6. A feature whose model fails to load is reported as failed for that instance; the rest of the map still renders.

## Frontend Details

Corpus composition drives the design here: skybox present in 88 of 254 archives,
features present in 194. The absent cases are not edge cases; for sky they are the majority.

Feature elevation is worth care. The checklist specifically calls out that features must not use fixed Y values and
must adjust to water height, which means the product should derive elevation the same way and would misrepresent a map
if it did not.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-005-AC01:** A map with no skybox renders with the default and reports sky as absent
- [ ] **P01-MAE-FRONTEND-005-AC02:** Water renders at the map's level with its declared material values
- [ ] **P01-MAE-FRONTEND-005-AC03:** Feature instances appear at their declared positions with terrain-derived elevation
- [ ] **P01-MAE-FRONTEND-005-AC04:** A map with thousands of placements renders without duplicating meshes per instance
- [ ] **P01-MAE-FRONTEND-005-AC05:** Start position markers show team indices
- [ ] **P01-MAE-FRONTEND-005-AC06:** A failed model leaves the rest of the map rendering

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-005-AC01 | Open a skyless corpus map | Default sky; inventory reports absent |
| P01-MAE-FRONTEND-005-AC02 | Open a water map and compare the plane against the known water level | Correct level and material |
| P01-MAE-FRONTEND-005-AC03 | Open a feature-heavy map | Placements match count; elevation follows terrain |
| P01-MAE-FRONTEND-005-AC04 | Profile instance count on a feature-heavy map | Instanced, not duplicated per placement |
| P01-MAE-FRONTEND-005-AC05 | Open a map with multiple teams | Start position markers present with team indices visible |
| P01-MAE-FRONTEND-005-AC06 | Corrupt one model and reopen | That instance reports failure; the rest of the map still renders |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
