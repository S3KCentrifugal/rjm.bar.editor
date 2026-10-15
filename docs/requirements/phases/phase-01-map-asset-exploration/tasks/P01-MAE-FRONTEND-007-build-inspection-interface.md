# P01-MAE-FRONTEND-007 - Build the inspection interface

## Description

This is the surface no competing tool offers, and the strongest reason a reviewer would choose this
product over launching the game. It brings together the checklist view showing all twenty properties at once, the
asset inventory showing what the map contains and what it does not, the map-side Lua as readable source, and the
option picker. Its hardest requirement is restraint: a verdict appears for exactly five properties, and for the other
fifteen the product shows a value and keeps its opinion to itself. Three states - present, absent and unknown - are
distinguished by glyph as well as colour, which is both an accessibility requirement and the visible form of the
product's central promise.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-CHECK.1](../product-requirements/product-requirements.md#24-checklist-verification)

## Relationships

### Prerequisites

- [P01-MAE-API-001 - Implement the extension surface](P01-MAE-API-001-implement-extension-surface.md)
- [P01-MAE-BACKEND-010 - Implement the checklist evaluator](P01-MAE-BACKEND-010-implement-checklist-evaluator.md)
- [P01-MAE-FRONTEND-006 - Render grid overlays](P01-MAE-FRONTEND-006-render-grid-overlays.md)

### Dependents

- [P01-MAE-DEVOPS-002 - Build and package the release artifacts](P01-MAE-DEVOPS-002-build-and-package-artifacts.md)
- [P01-MAE-DOCUMENTATION-001 - Prepare the early-user evaluation](P01-MAE-DOCUMENTATION-001-prepare-early-user-evaluation.md)
- [P01-MAE-QUALITY-004 - Verify performance targets and system invariants](P01-MAE-QUALITY-004-verify-performance-and-invariants.md)

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

- **Business Requirements:** `BIZ-EFF.1`, `OPER-WORKFLOW.4`, `GUARD-REPUTE.2`, `GUARD-ACC.1`
- **Product Requirements:** `FR-CHECK.1`, `FR-CHECK.2`, `FR-CHECK.3`, `FR-SHOW.9`, `FR-SHOW.11`, `FR-READ.7`, `NFR-A11Y.1`
- **Technical Design:** section 5.2
- **Phase Outcome / Exit Criteria:** All twenty checklist properties verifiable without the game

## Scope

- Checklist view with all twenty properties and links to the map
- Asset inventory showing present, absent and unknown
- Map option picker
- Lua source viewer

## Non-Goals

- Computing any checklist property, which is P01-MAE-BACKEND-010
- Offering an overall verdict on a map

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/scenes/checklist_panel.tscn` | Create | Checklist view |
| `app/scenes/asset_inventory.tscn` | Create | Asset state panel |
| `app/scenes/option_picker.tscn` | Create | Map option selection |
| `app/scenes/lua_viewer.tscn` | Create | Read-only source viewer |
| `app/scripts/inspection_ui.gd` | Create | Panel behaviour |

## Implementation Requirements

1. Show all twenty checklist properties together, each with its value, and a pass or fail for exactly the five the checklist governs numerically.
2. For the other fifteen, show no verdict indicator of any kind. The extension omits the field entirely, and the interface must not invent one.
3. Offer no overall map verdict anywhere in the interface. Approval is a decision two people make, not a thing this tool produces.
4. Selecting a checklist property with a spatial referent enables its overlay or moves the camera to it.
5. Distinguish present, absent and unknown by glyph as well as colour, in both the inventory and the checklist.
6. Make every checklist action reachable by keyboard.
7. Present map-side Lua as read-only text. There is no execute affordance, because execution is prohibited.
8. The option picker lists declared options with defaults, and changing one triggers re-evaluation.

## Frontend Details

Panel layout, from the information architecture:

```
[Open…]  map name · source: archive | directory        [Options ▾]
┌──────────────┬──────────────────────────┬───────────┐
│ Asset        │                          │ Checklist │
│ inventory    │      3D viewport         │ 20 rows   │
│ ✓ present    │                          │ 5 judged  │
│ — absent     │                          │ 15 values │
│ ? unknown    │                          │           │
└──────────────┴──────────────────────────┴───────────┘
Overlays: [metal][type][slope][pathability][grass]   stage 3/3
```

The three glyphs carry the product's core guarantee into the interface, where `GUARD-ACC.1` becomes something a user
can see rather than something a document asserts.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-007-AC01:** All twenty properties appear in one view, with exactly five carrying a verdict
- [ ] **P01-MAE-FRONTEND-007-AC02:** No overall map verdict appears anywhere in the interface
- [ ] **P01-MAE-FRONTEND-007-AC03:** Selecting a spatial property enables its overlay or moves the camera
- [ ] **P01-MAE-FRONTEND-007-AC04:** Present, absent and unknown are distinguished by glyph as well as colour
- [ ] **P01-MAE-FRONTEND-007-AC05:** Every checklist action is reachable by keyboard
- [ ] **P01-MAE-FRONTEND-007-AC06:** Map Lua is presented read-only with no execute affordance
- [ ] **P01-MAE-FRONTEND-007-AC07:** The option picker lists declared options and triggers re-evaluation

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-007-AC01 | Open any corpus map and inspect the checklist view | Twenty rows; five verdicts; fifteen values |
| P01-MAE-FRONTEND-007-AC02 | `grep -riE 'approve|overall verdict|map is (good|valid)' app/` | No matches |
| P01-MAE-FRONTEND-007-AC03 | Select each linked property | Overlay enables or camera moves |
| P01-MAE-FRONTEND-007-AC04 | Screenshot in greyscale | Three states remain distinguishable |
| P01-MAE-FRONTEND-007-AC05 | Navigate the checklist with keyboard only | Every action reachable |
| P01-MAE-FRONTEND-007-AC06 | Inspect the Lua viewer | Read-only; no execute control |
| P01-MAE-FRONTEND-007-AC07 | Change an option | Derived values refresh |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
