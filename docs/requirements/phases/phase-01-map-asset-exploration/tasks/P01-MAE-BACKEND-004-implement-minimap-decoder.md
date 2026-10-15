# P01-MAE-BACKEND-004 - Implement the minimap decoder

## Description

The SMF carries a pre-rendered minimap as a fixed-size compressed block with a full mip chain. It is small
work, but it is not optional: the published map checklist asks reviewers to judge whether the minimap needs lightening,
and that question cannot be answered without showing them the minimap the map actually ships. Like the tile atlas, it
is new work that the existing analysis decoder skips.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `S`
- **Parent Deliverable / User Story:** [FR-READ.4](../product-requirements/product-requirements.md#21-reading-and-decoding)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-002 - Implement the SMF decoder](P01-MAE-BACKEND-002-implement-smf-decoder.md)

### Dependents

- [P01-MAE-BACKEND-010 - Implement the checklist evaluator](P01-MAE-BACKEND-010-implement-checklist-evaluator.md)

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
- **Product Requirements:** `FR-READ.4`, `FR-CHECK.1`
- **Technical Design:** section 3.3
- **Phase Outcome / Exit Criteria:** Every asset class represented; checklist property MR-CHK-10 answerable

## Scope

- Minimap block decode including its mip levels
- Provenance on the result

## Non-Goals

- Displaying the minimap, which belongs to the inspection UI
- Judging whether it needs lightening - the tool reports, the reviewer decides

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/minimap.rs` | Create | Minimap block decoder |
| `crates/mae-core/tests/minimap.rs` | Create | Size and mip-count tests |

## Implementation Requirements

1. The block is a fixed 699048 bytes carrying 9 mip levels, per `SMFFormat.h`; assert both rather than inferring them.
2. Cite the format source and the commit read.
3. A map whose minimap pointer is invalid yields Unknown for the minimap and still opens.

## Backend Details

`MINIMAP_SIZE` is 699048 and `MINIMAP_NUM_MIPMAP` is 9, both constants in
`rts/Map/SMF/SMFFormat.h`. The top level is 1024x1024 DXT1 with eight further levels below it.

This decoder exists to serve exactly one checklist property, and the product's job there is to show the image, not to
have an opinion about its brightness - `GUARD-REPUTE.2` restricts verdicts to the five numerically-governed
properties, and this is not one of them.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-004-AC01:** The minimap decodes for every corpus archive that carries a valid pointer
- [ ] **P01-MAE-BACKEND-004-AC02:** Block size and mip count are asserted against the documented constants
- [ ] **P01-MAE-BACKEND-004-AC03:** An invalid minimap pointer yields Unknown and the map still opens

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-004-AC01 | `cargo test -p mae-core minimap::corpus` | Decodes across the corpus |
| P01-MAE-BACKEND-004-AC02 | `cargo test -p mae-core minimap::constants` | 699048 bytes and 9 mip levels asserted |
| P01-MAE-BACKEND-004-AC03 | `cargo test -p mae-core minimap::invalid_pointer` | Unknown returned, map still opens |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
