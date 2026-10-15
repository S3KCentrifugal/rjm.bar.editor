# P01-MAE-BACKEND-007 - Implement terrain derivation

## Description

Slope, passability by movement class, metal spots and depth bands are not stored in a map; they are
computed from what is. They drive four of the five grid overlays, two checklist properties, and the whole reason the
tooling developer persona opens the product at all - to check a derived fact against the map it came from. Because
these values are computed rather than read, the discipline around them matters more, not less: each must reproduce a
documented engine rule and carry the citation, and passability additionally depends on constants the archive does not
contain.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-SHOW.10](../product-requirements/product-requirements.md#22-presentation)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-002 - Implement the SMF decoder](P01-MAE-BACKEND-002-implement-smf-decoder.md)
- [P01-MAE-DATA-003 - Embed BAR constants with commit provenance](P01-MAE-DATA-003-embed-bar-constants-with-provenance.md)

### Dependents

- [P01-MAE-BACKEND-010 - Implement the checklist evaluator](P01-MAE-BACKEND-010-implement-checklist-evaluator.md)
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

- [Rust conventions](~/.claude/skills/convention-rust/SKILL.md)
- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EFF.1`, `BIZ-EXP.3`
- **Product Requirements:** `FR-SHOW.10`, `FR-CHECK.4`, `NFR-VALID.1`
- **Technical Design:** section 3.3, ADR-10
- **Phase Outcome / Exit Criteria:** Derived facts are visually verifiable

## Scope

- Slope from surface normals
- Passability per movement class
- Metal spot identification
- Depth band classification

## Non-Goals

- Rendering the overlays, which is P01-MAE-FRONTEND-006
- Start-position classification, which belongs to `rjm.bar.tools` and not to this product

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/derive.rs` | Modify | Extend the shared crate's derivation |
| `crates/mae-core/tests/derive.rs` | Create | Cross-checks against rjm.bar.tools output |

## Implementation Requirements

1. Slope is `1 - normal.y`, blended from the steepest and mean face normal, citing `rts/Map/ReadMap.cpp:770` and the commit.
2. Passability compares slope against a movement class's `maxslope` converted by `1 - cos(theta)`, citing `MoveDefHandler.cpp`, and depth against the shallow and float thresholds from the embedded constants.
3. Cover the four movement classes the checklist distinguishes: bot, vehicle, hover and ship.
4. Metal spot identification must classify rather than blindly enumerate: maps whose entire surface is metal exist, and listing tens of thousands of extractor sites there is meaningless. Follow the `field` and `barren` classification the existing tool already established.
5. Every derived value carries its rule citation. A derived value with no citation cannot be constructed, by `Provenanced<T>`.
6. Cross-check results against `rjm.bar.tools` for the maps both cover; a disagreement between the two is a defect in one of them and must be resolved, not tolerated.

## Backend Details

The cross-check against `rjm.bar.tools` is the strongest available correctness
signal, and it is also the point of the shared crate: the two consumers must not drift. Where this product and that
one disagree about a slope value, one of them is wrong.

Passability is the first derived value that needs data the map does not contain, which is why `P01-MAE-DATA-003` is a
prerequisite. Without the embedded constants there is no `maxslope` to compare against, and guessing one would be
exactly the failure `GUARD-ACC.1` forbids.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-007-AC01:** Slope, passability for four movement classes, metal spots and depth bands are derived for every corpus map
- [ ] **P01-MAE-BACKEND-007-AC02:** Every derived value carries a rule citation with an engine commit
- [ ] **P01-MAE-BACKEND-007-AC03:** Metal is classified rather than enumerated on all-metal maps
- [ ] **P01-MAE-BACKEND-007-AC04:** Derived values agree with `rjm.bar.tools` for every map both cover

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-007-AC01 | `cargo test -p mae-core derive::corpus` | All four derivations complete for every corpus map |
| P01-MAE-BACKEND-007-AC02 | `cargo test -p mae-core derive::citations` | Every derived value carries a citation |
| P01-MAE-BACKEND-007-AC03 | `cargo test -p mae-core derive::metal_classification` | An all-metal map is classified, not enumerated |
| P01-MAE-BACKEND-007-AC04 | `cargo test -p mae-core derive::cross_check` | Values match `rjm.bar.tools` for the shared map set |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
