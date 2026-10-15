# P01-MAE-BACKEND-002 - Implement the SMF decoder

## Description

The SMF file is the map's skeleton: the header that fixes its dimensions and height range, the elevation
grid, the terrain type and metal grids, the feature placement block, and an optional vegetation extension. Every
corpus map has exactly one, and every downstream capability - terrain rendering, overlays, most checklist properties -
reads from what this decoder produces. Its correctness is not a matter of taste: the height decode formula, the grid
strides and the extra-header walk are all engine behaviour that must be reproduced exactly and cited, not
approximated. Values that cannot be read are marked unknown rather than defaulted, and the map still opens.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-READ.2](../product-requirements/product-requirements.md#21-reading-and-decoding)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-001 - Implement the virtual file system over archives and directories](P01-MAE-BACKEND-001-implement-virtual-file-system.md)
- [P01-MAE-DATA-001 - Implement the provenance value model](P01-MAE-DATA-001-implement-provenance-value-model.md)

### Dependents

- [P01-MAE-BACKEND-003 - Implement the tile atlas decoder](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md)
- [P01-MAE-BACKEND-004 - Implement the minimap decoder](P01-MAE-BACKEND-004-implement-minimap-decoder.md)
- [P01-MAE-BACKEND-006 - Implement map option selection and re-evaluation](P01-MAE-BACKEND-006-implement-option-evaluation.md)
- [P01-MAE-BACKEND-007 - Implement terrain derivation](P01-MAE-BACKEND-007-implement-terrain-derivation.md)

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

- **Business Requirements:** `BIZ-EXP.1`, `BIZ-EXP.3`
- **Product Requirements:** `FR-READ.2`, `NFR-REL.1`, `NFR-VALID.1`
- **Technical Design:** section 3.3, section 4.3, FMEA F2
- **Phase Outcome / Exit Criteria:** Every corpus member opens

## Scope

- Header, heightmap, typemap, metalmap, feature block, vegetation extra-header
- Provenance on every decoded value
- Typed errors naming the `smf` stage

## Non-Goals

- Tile atlas decoding, which is P01-MAE-BACKEND-003
- Minimap decoding, which is P01-MAE-BACKEND-004

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/smf.rs` | Modify | Extend the shared crate's parser to full coverage |
| `crates/mae-core/tests/smf.rs` | Create | Corpus-wide structural tests |

## Implementation Requirements

1. Decode heights as `min + raw * (max - min) / 65536`, citing `rts/Map/SMFReadMap.cpp` and the commit read.
2. Heightmap length is `(mapx+1) * (mapy+1)`; typemap and metalmap are each `(mapx/2) * (mapy/2)`. Assert these rather than trusting them.
3. Walk the extra-header chain by declared size, not by assuming a count. A vegetation header may be absent, which is `Absent`, not `Unknown`.
4. Parse the feature block: type-name table then placement records. A map with no features is normal - 60 corpus archives have none.
5. Every bounds calculation must be checked. This decoder is the primary subject of FMEA F2 and of the fuzzing task, and a panic here fails `NFR-REL.1` outright.
6. Where `mapinfo.lua` overrides a stored grid, record both so the effective grid and its source can be reported, per `FR-SHOW.3`.

## Backend Details

Structural facts worth asserting from the corpus measurement: all 254 archives carry
exactly one SMF, so multi-SMF handling is not required - but the parser must not hardcode assumptions the *format*
does not guarantee where the cost of reading the real field is trivial.

The header layout is defined in `rts/Map/SMF/SMFFormat.h`; the pointers it carries - `heightmapPtr`, `typeMapPtr`,
`tilesPtr`, `minimapPtr`, `metalmapPtr`, `featurePtr` - are file offsets, and each must be validated against file
length before use.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-002-AC01:** Every corpus archive's SMF decodes without error
- [ ] **P01-MAE-BACKEND-002-AC02:** Heightmap, typemap and metalmap lengths match the header-derived expectations for every corpus map
- [ ] **P01-MAE-BACKEND-002-AC03:** The height decode formula matches the engine and carries a citation with a commit
- [ ] **P01-MAE-BACKEND-002-AC04:** A map with no vegetation header reports Absent, not Unknown; a malformed one reports Unknown
- [ ] **P01-MAE-BACKEND-002-AC05:** No panic occurs on any corpus archive or on truncated input

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-002-AC01 | `cargo test -p mae-core smf::corpus` | All 254 archives decode; zero errors |
| P01-MAE-BACKEND-002-AC02 | `cargo test -p mae-core smf::grid_dimensions` | Lengths match `(mapx+1)*(mapy+1)` and `(mapx/2)*(mapy/2)` for every map |
| P01-MAE-BACKEND-002-AC03 | `cargo test -p mae-core smf::height_formula` | Known values match engine-computed expectations |
| P01-MAE-BACKEND-002-AC04 | `cargo test -p mae-core smf::absent_vs_unknown` | The two states are produced correctly for the relevant corpus maps |
| P01-MAE-BACKEND-002-AC05 | `cargo test -p mae-core smf::truncated` | Truncated input yields a typed error, never a panic |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
