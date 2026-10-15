# P01-MAE-BACKEND-008 - Implement texture and model decoders

## Description

Godot converts DDS files at import time and cannot load them at run time, and map textures arrive from
arbitrary archives at run time. That single fact makes texture decoding the product's own responsibility rather than
the engine's, and it is why this task exists at all. Ten shading slots plus four override grids must be decoded from
DDS, TGA and BMP into a form the extension can upload, and the S3O model format - fully documented in the engine
headers as a hierarchy of pieces with vertex and index tables at file offsets - must be parsed so that a map's rocks
and trees appear where the map puts them. Absent slots are the common case, not an error: only a third of corpus maps
ship a skybox.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [FR-SHOW.2](../product-requirements/product-requirements.md#22-presentation)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-001 - Implement the virtual file system over archives and directories](P01-MAE-BACKEND-001-implement-virtual-file-system.md)
- [P01-MAE-DATA-001 - Implement the provenance value model](P01-MAE-DATA-001-implement-provenance-value-model.md)
- [P01-MAE-FRONTEND-002 - Spike compressed texture upload path](P01-MAE-FRONTEND-002-spike-compressed-texture-upload.md)

### Dependents

- [P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md)
- [P01-MAE-BACKEND-010 - Implement the checklist evaluator](P01-MAE-BACKEND-010-implement-checklist-evaluator.md)
- [P01-MAE-FRONTEND-005 - Present sky, water, features and start positions](P01-MAE-FRONTEND-005-present-scene-assets.md)

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

- **Business Requirements:** `BIZ-EXP.3`, `OPER-SCALE.1`
- **Product Requirements:** `FR-SHOW.2`, `FR-SHOW.3`, `FR-SHOW.6`, `FR-READ.8`
- **Technical Design:** section 3.3, ADR-03, ADR-04
- **Phase Outcome / Exit Criteria:** Every asset class represented

## Scope

- DDS, TGA and BMP decoding for the ten engine shading slots and four override grids
- S3O model parsing with piece hierarchy, vertices and index tables
- Absent-slot handling as a normal condition

## Non-Goals

- Uploading to the GPU, which is P01-MAE-API-001
- Applying a material or shader, which is the frontend tasks

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/texture.rs` | Create | Image decoding for engine slots |
| `crates/mae-core/src/s3o.rs` | Create | S3O model parser |
| `crates/mae-core/tests/texture.rs` | Create | Slot coverage and absence tests |
| `crates/mae-core/tests/s3o.rs` | Create | Model parse tests |

## Implementation Requirements

1. Handle the ten shading slots the engine declares: `detailTex`, `specularTex`, `splatDetailTex`, `splatDistrTex`, `splatDetailNormalTex1` through `4`, `detailNormalTex`, `grassShadingTex`, `skyReflectModTex`, `lightEmissionTex`, `parallaxHeightTex`, plus the `minimapTex`, `metalmapTex`, `typemapTex` and `grassmapTex` overrides.
2. An absent slot is `Absent`, and the map renders. Five corpus maps lack a normal map, nine lack specular, four lack splat, and 166 lack a skybox - none of these is an error.
3. Parse S3O per `rts/Rendering/Models/s3o.h`: a piece carries name and child offsets, a vertex count and offset, a primitive type, and a vertex index table; pieces form a tree with per-piece offsets from the parent.
4. Handle all three S3O primitive types - triangles, triangle strips and quads - and the strip terminator `0xffffffff`.
5. Produce output in the form ADR-03 selected in the texture upload spike.
6. Bounds-check every offset read from a file header before use; these parsers are in scope for fuzzing.

## Backend Details

The engine's own resource table in `rts/Map/MapInfo.cpp` is the authority for which
slot names exist and how they default; reproduce that mapping and cite it.

S3O is more tractable than it looks. The header gives file offsets for a root piece; each piece gives offsets to its
children, its vertices and its index table. There is no compression and no external dependency. The main hazard is
trusting an offset without checking it against file length.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-008-AC01:** All ten shading slots and four override grids are decoded when present
- [ ] **P01-MAE-BACKEND-008-AC02:** An absent slot yields Absent and the map still renders, verified on the corpus maps that lack each slot
- [ ] **P01-MAE-BACKEND-008-AC03:** S3O models parse with piece hierarchy, vertices and index tables, across all three primitive types
- [ ] **P01-MAE-BACKEND-008-AC04:** Every offset read from a header is validated before use
- [ ] **P01-MAE-BACKEND-008-AC05:** Output form matches the decision recorded in ADR-03

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-008-AC01 | `cargo test -p mae-core texture::slots` | Every declared slot decodes for a map that carries it |
| P01-MAE-BACKEND-008-AC02 | `cargo test -p mae-core texture::absent` | The 5 normal-less, 9 specular-less and 4 splat-less corpus maps render |
| P01-MAE-BACKEND-008-AC03 | `cargo test -p mae-core s3o::corpus` | Every model in every corpus archive parses |
| P01-MAE-BACKEND-008-AC04 | `cargo test -p mae-core s3o::primitive_types` | Triangles, strips and quads all handled |
| P01-MAE-BACKEND-008-AC05 | `cargo test -p mae-core texture::bounds` | Truncated input yields typed errors, never a panic |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
