# P01-MAE-BACKEND-010 - Implement the checklist evaluator

## Description

The published BAR map checklist is twenty properties that two volunteer reviewers must verify before a map
enters the game, and no tool surfaces them together. This evaluator computes all twenty from the decoded map, and
judges exactly five - the ones the checklist governs with a stated numeric rule. The restraint is the design: for the
other fifteen the product reports a value and says nothing about whether it is good, because inventing a verdict the
checklist does not define would let a reviewer approve a map on the tool's opinion. Two of the twenty need constants
the archive does not carry, which is why the embedded constants task comes first.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-CHECK.1](../product-requirements/product-requirements.md#24-checklist-verification)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-007 - Implement terrain derivation](P01-MAE-BACKEND-007-implement-terrain-derivation.md)
- [P01-MAE-BACKEND-004 - Implement the minimap decoder](P01-MAE-BACKEND-004-implement-minimap-decoder.md)
- [P01-MAE-BACKEND-008 - Implement texture and model decoders](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md)

### Dependents

- [P01-MAE-API-001 - Implement the extension surface](P01-MAE-API-001-implement-extension-surface.md)
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

- [Rust conventions](~/.claude/skills/convention-rust/SKILL.md)
- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EFF.1`, `GUARD-REPUTE.2`
- **Product Requirements:** `FR-CHECK.1`, `FR-CHECK.2`, `FR-CHECK.4`
- **Technical Design:** section 5.2, invariant 4
- **Phase Outcome / Exit Criteria:** All twenty checklist properties verifiable without the game

## Scope

- All twenty checklist properties computed from the decoded map
- Pass or fail for exactly the five numerically-governed properties
- A spatial link for properties that have one

## Non-Goals

- The checklist interface, which is P01-MAE-FRONTEND-008
- Any overall verdict about a map - prohibited by `GUARD-REPUTE.2`

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/checklist.rs` | Create | Twenty property evaluations |
| `crates/mae-core/tests/checklist.rs` | Create | Cardinality, judgement and corpus tests |

## Implementation Requirements

1. Always produce exactly twenty entries, for every map. A property that cannot be computed is an entry in the Unknown or Absent state, never a shorter list.
2. Judge exactly five: map size at most 32x32, wind within 0 to 30, tidal within 0 to 25, splat resolution at least 1024 px, and normals resolution matching texture resolution.
3. For the other fifteen, emit the value with no verdict field at all. Not null, not `"unknown"` - the field is absent from the payload, so no consumer can render a verdict that does not exist.
4. Attach a spatial link where one exists: an overlay for metal, type, grass and pathability; a camera target for each start position and for a whole-map overview. Properties with no spatial referent, such as wind, carry no link and are exempt by definition.
5. Pathability and geovent buildability read the embedded constants and surface the BAR commit those constants came from.
6. Every property value carries provenance, so a reviewer can see which engine rule produced it.

## Backend Details

The twenty are the published properties `MR-CHK-01` through `MR-CHK-20`: size,
pathability, texture cleanliness, splat resolution, normals resolution, specular subtlety, metal distribution, type
map, grass map, minimap brightness, topolines, splat distribution, feature placement and reclaim, sun position,
lighting values, wind and tidal ranges, skybox and fog, water parameters, metal values, and start positions with the
teams table.

Exactly five of those carry a numeric rule. The asymmetry is deliberate and is system invariant 4: the count of judged
properties is always five, for every map, and a test asserts it.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-010-AC01:** Exactly twenty entries are produced for every corpus map
- [ ] **P01-MAE-BACKEND-010-AC02:** Exactly five entries are judged, always, and the other fifteen carry no verdict field
- [ ] **P01-MAE-BACKEND-010-AC03:** Pathability and geovent buildability surface the BAR commit of the constants they used
- [ ] **P01-MAE-BACKEND-010-AC04:** Spatial links resolve to an overlay or camera target for every property that has a referent
- [ ] **P01-MAE-BACKEND-010-AC05:** Every property value carries provenance

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-010-AC01 | `cargo test -p mae-core checklist::cardinality` | Twenty entries for every corpus map |
| P01-MAE-BACKEND-010-AC02 | `cargo test -p mae-core checklist::judged_count` | Exactly five judged, for every map |
| P01-MAE-BACKEND-010-AC03 | `cargo test -p mae-core checklist::no_verdict_field` | Unjudged entries omit the field entirely from serialised output |
| P01-MAE-BACKEND-010-AC04 | `cargo test -p mae-core checklist::constants_commit` | Commit surfaced for the two constant-dependent properties |
| P01-MAE-BACKEND-010-AC05 | `cargo test -p mae-core checklist::links` | Spatial links resolve for the properties that have referents |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
