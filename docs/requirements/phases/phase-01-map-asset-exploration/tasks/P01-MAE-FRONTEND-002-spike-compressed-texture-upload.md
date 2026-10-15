# P01-MAE-FRONTEND-002 - Spike compressed texture upload path

## Description

The tile atlas is the largest single asset in a map - roughly 25 MB of DXT1 in a 49 MB archive - and whether
it can be handed to Godot still compressed determines the memory profile of the whole application. Godot exposes
`Image.FORMAT_DXT1` and warns only that VRAM-compressed images cannot be *modified*, which this product never does; but
acceptance of pre-compressed block data by `create_from_data` is inferred from adjacent documentation rather than
stated. If compressed upload works, the atlas stays at its on-disk size in memory. If it does not, decoding to RGBA
multiplies it roughly six to eight times and streaming becomes necessary. `NFR-PERF.4` deliberately carries no numeric
ceiling until this is settled, because asserting one now would be the same substitution of a plausible value that the
product forbids in its own output.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `S`
- **Parent Deliverable / User Story:** [ADR-03](../technical-design/technical-design.md#8-alternatives-considered)

## Relationships

### Prerequisites

- None

### Dependents

- [P01-MAE-BACKEND-003 - Implement the tile atlas decoder](P01-MAE-BACKEND-003-implement-tile-atlas-decoder.md)
- [P01-MAE-BACKEND-008 - Implement texture and model decoders](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md)

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

- **Business Requirements:** `OPER-SCALE.1`
- **Product Requirements:** `FR-SHOW.2`, `NFR-PERF.4`
- **Technical Design:** ADR-03, ADR-04, section 4.1
- **Phase Outcome / Exit Criteria:** Unblocks the memory guarantee behind every corpus member opening

## Scope

- Prototype uploading real DXT1 tile data to Godot as a texture, compressed
- Measuring resident memory for compressed and RGBA-decoded paths on the same map
- Recording the outcome as a resolution to ADR-03 and ADR-04, and a measured ceiling for NFR-PERF.4

## Non-Goals

- Atlas assembly logic, which belongs to P01-MAE-BACKEND-003
- Any shading or material work

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `spikes/texture-upload/` | Create | Throwaway prototype, retained for evidence |
| `spikes/texture-upload/FINDINGS.md` | Create | Measured memory for both paths, and the recommended ceiling |
| `docs/requirements/phases/phase-01-map-asset-exploration/technical-design/technical-design.md` | Modify | Record ADR-03 and ADR-04 outcomes and the NFR-PERF.4 ceiling |

## Implementation Requirements

1. Use real tile data extracted from a corpus archive, not synthetic blocks. The largest corpus archive by file count and one median archive must both be measured.
2. Measure peak resident memory for each path with the same map open, on the same machine, and report both numbers.
3. If compressed upload is rejected by Godot, record the exact error, then measure the RGBA path so the fallback has a number attached.
4. Propose a concrete ceiling for `NFR-PERF.4` from the measurements, expressed as a multiple of the reference-set median rather than an absolute figure, so it survives hardware changes.

## Frontend Details

Only two questions matter: does it work, and what does it cost. Resist building
anything reusable here.

The measurement must be peak resident memory of the whole process with the map open, not the size of the texture
object, because the question `NFR-PERF.4` asks is whether the application fits in memory.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-002-AC01:** Real DXT1 tile data from a corpus archive is uploaded to Godot and rendered, or the attempt fails with the exact error recorded
- [ ] **P01-MAE-FRONTEND-002-AC02:** Peak resident memory is measured for both the compressed and RGBA paths on the largest and a median corpus archive
- [ ] **P01-MAE-FRONTEND-002-AC03:** ADR-03 and ADR-04 are updated from Proposed to Accepted
- [ ] **P01-MAE-FRONTEND-002-AC04:** A concrete ceiling for NFR-PERF.4 is proposed as a multiple of the reference-set median

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-002-AC01 | Run the prototype | The tile data renders, or the failure is recorded verbatim |
| P01-MAE-FRONTEND-002-AC02 | Read `spikes/texture-upload/FINDINGS.md` | Four memory measurements present: two paths by two archives |
| P01-MAE-FRONTEND-002-AC03 | Inspect the technical design ADR section | ADR-03 and ADR-04 carry a status other than Proposed |
| P01-MAE-FRONTEND-002-AC04 | Inspect `NFR-PERF.4` in the product requirements | A ceiling is proposed as a multiple of the reference-set median, not an absolute figure |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
