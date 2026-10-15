# P01-MAE-FRONTEND-004 - Implement camera and navigation

## Description

Camera and navigation was the one component the stakeholder named by hand, and it carries a measurable
commitment: crossing a map corner to corner must take no longer than the same traversal does in the game itself. The
controls should be the ones a mapper already knows - ground-anchored panning, orbiting and zooming, with the point
under the cursor staying under the cursor - because a reviewer with a map to assess should not be learning a tool.
Movement speed scales with height above the terrain, which is what reconciles crossing 23,170 units quickly with
inspecting a fifty-unit feature closely.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-NAV.1](../product-requirements/product-requirements.md#23-camera-and-navigation)

## Relationships

### Prerequisites

- [P01-MAE-FRONTEND-003 - Render the terrain surface](P01-MAE-FRONTEND-003-render-terrain-surface.md)

### Dependents

- [P01-MAE-FRONTEND-008 - Implement open, reload and session state](P01-MAE-FRONTEND-008-implement-open-and-reload-flows.md)
- [P01-MAE-QUALITY-003 - Establish the reference set and parity baseline](P01-MAE-QUALITY-003-establish-reference-set-and-parity.md)

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

- **Business Requirements:** `BIZ-EXP.2`
- **Product Requirements:** `FR-NAV.1`, `FR-NAV.2`, `FR-NAV.3`, `NFR-PERF.2`
- **Technical Design:** section 2.1 T-14
- **Phase Outcome / Exit Criteria:** Corner-to-corner traversal at or faster than in-game

## Scope

- Ground-anchored pan, orbit and zoom
- Altitude-proportional movement speed
- Named camera targets including start positions and overview

## Non-Goals

- Measuring traversal against the game, which is P01-MAE-QUALITY-003
- Checklist linking, which is P01-MAE-FRONTEND-007

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/scripts/camera_controller.gd` | Create | Camera behaviour |
| `app/scenes/camera.tscn` | Create | Camera rig |

## Implementation Requirements

1. Keep the terrain point under the cursor under the cursor during pan, orbit and zoom, within a two-pixel drift tolerance.
2. Bind conventionally: left-drag pans, middle-drag orbits with horizontal controlling heading and vertical controlling pitch, wheel zooms toward the cursor, `WASD` moves and `QE` rotates.
3. Scale movement speed monotonically with height above terrain. At minimum altitude a fifty-unit feature must be approachable without overshoot; at altitude the map must be crossable within the traversal budget.
4. Provide direct positioning to each declared start position and to a whole-map overview, arriving within two seconds.
5. Constrain the camera to the map bounds rather than allowing it to be lost off the edge.
6. The camera must behave correctly on degenerate maps - entirely flat, or entirely water - where anchoring has no relief to work with.

## Frontend Details

The altitude-speed relationship is the crux, and practitioner reports are blunt
about the failure mode: unconstrained fly over large terrain feels like "slipping on ice". Speed that scales with
height above ground is the standard reconciliation and is recorded as inference `INF-P1`.

Because the traversal target is defined against the game rather than against a number, the camera cannot be tuned in
isolation - the reference measurement from `P01-MAE-QUALITY-003` is what says whether it is fast enough.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-004-AC01:** The point under the cursor stays under the cursor during pan, orbit and zoom, within two pixels
- [ ] **P01-MAE-FRONTEND-004-AC02:** The documented bindings are implemented
- [ ] **P01-MAE-FRONTEND-004-AC03:** Movement speed scales monotonically with height above terrain
- [ ] **P01-MAE-FRONTEND-004-AC04:** Every start position and a whole-map overview are reachable within two seconds
- [ ] **P01-MAE-FRONTEND-004-AC05:** The camera cannot leave the map bounds, and behaves correctly on flat and all-water maps

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-004-AC01 | Automated anchor-drift test during scripted pan, orbit and zoom | Drift never exceeds two pixels |
| P01-MAE-FRONTEND-004-AC02 | Manual check of each binding | All behave as documented |
| P01-MAE-FRONTEND-004-AC03 | `speed(h)` sampled across the altitude range | Monotonically increasing |
| P01-MAE-FRONTEND-004-AC04 | Trigger each named target | Camera arrives within two seconds |
| P01-MAE-FRONTEND-004-AC05 | Drive the camera beyond the map edge, and open a flat and an all-water map | Position constrained; anchoring still functions |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
