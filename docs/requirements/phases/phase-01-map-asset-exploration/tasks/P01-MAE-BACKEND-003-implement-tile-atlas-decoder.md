# P01-MAE-BACKEND-003 - Implement the tile atlas decoder

## Description

The `.smt` tile file holds the map's actual ground appearance and is the bulk of an archive - 25 MB of a
49 MB map. It is new work: the existing decoder in `rjm.bar.tools` deliberately skips it, because terrain analysis
never needed to see the ground. Without it the product shows correct geometry wearing no clothes, so it sits on the
critical path for anything visual. The file is a tile-indexed atlas: the SMF carries one index per terrain square into
a set of fixed-size compressed tiles, and the tile files are declared by count and name in the SMF rather than assumed.
How the decoded result reaches the GPU is settled by the texture upload spike.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-READ.3](../product-requirements/product-requirements.md#21-reading-and-decoding)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-002 - Implement the SMF decoder](P01-MAE-BACKEND-002-implement-smf-decoder.md)
- [P01-MAE-FRONTEND-002 - Spike compressed texture upload path](P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md)

### Dependents

- [P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md)
- [P01-MAE-FRONTEND-003 - Render the terrain surface](P01-MAE-FRONTEND-003-render-terrain-surface.md)

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
- **Product Requirements:** `FR-READ.3`, `FR-SHOW.1`, `NFR-PERF.3`
- **Technical Design:** section 3.3, ADR-03, FMEA F3
- **Phase Outcome / Exit Criteria:** Every asset class represented

## Scope

- Tile file header and tile set parsing
- Tile index grid resolution
- Producing GPU-uploadable tile data in the form the spike selected

## Non-Goals

- Uploading to Godot, which is P01-MAE-API-001 and the frontend tasks
- Any shading or material work

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/smt.rs` | Create | Tile file parser and index resolution |
| `crates/mae-core/tests/smt.rs` | Create | Corpus-wide tile resolution tests |

## Implementation Requirements

1. Read the tile file count and names from the SMF tile header; do not assume one file, even though all 254 corpus archives currently use exactly one. Files are numbered consecutively across the set, so the first file defines tiles 0..n-1 and the next continues from n.
2. The tile index grid is `(mapx/4) * (mapy/4)` 32-bit entries, following `texelPerSquare` of 8 and a tile size of 32.
3. Tile size in bytes is `SMALL_TILE_SIZE` as defined in `SMFFormat.h`; cite the file and commit.
4. Validate every tile index against the resolved tile count. An out-of-range index is `Unknown` for that square, not a crash and not a silently substituted tile.
5. Produce the output form chosen by ADR-03 in the texture upload spike - compressed blocks if accepted, RGBA otherwise.
6. Reading the tile file must be deferrable so that stage 1 of the loading sequence does not wait on 25 MB.

## Backend Details

Sum of declared tile counts across the file set must equal the number of tiles
resolved; that equality is the decoder's own consistency check and is asserted per map.

The deferral requirement matters for `NFR-PERF.3`: terrain must be navigable within 5 seconds, and the atlas is the
largest thing standing between opening a file and achieving that. Stage 1 needs the heightmap; the atlas belongs to
stage 2.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-003-AC01:** Every corpus archive's tile file decodes and every tile index resolves to a real tile
- [ ] **P01-MAE-BACKEND-003-AC02:** The declared tile-file count is read from the header rather than assumed
- [ ] **P01-MAE-BACKEND-003-AC03:** An out-of-range tile index produces Unknown for that square without crashing
- [ ] **P01-MAE-BACKEND-003-AC04:** The tile file can be read after the heightmap, not as a precondition of it
- [ ] **P01-MAE-BACKEND-003-AC05:** Output form matches the decision recorded in ADR-03

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-003-AC01 | `cargo test -p mae-core smt::corpus` | All 254 archives resolve; sum of declared counts equals tiles resolved |
| P01-MAE-BACKEND-003-AC02 | `cargo test -p mae-core smt::tile_file_count` | A synthetic multi-file fixture resolves by declared count, not by assuming one |
| P01-MAE-BACKEND-003-AC03 | `cargo test -p mae-core smt::index_bounds` | A synthetic out-of-range index yields Unknown, no panic |
| P01-MAE-BACKEND-003-AC04 | `cargo test -p mae-core smt::deferred` | Heightmap decode completes without reading the tile file |
| P01-MAE-BACKEND-003-AC05 | Inspect the decoder's output type against ADR-03 | Output form matches the decision recorded in the spike |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
