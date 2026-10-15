# P01-MAE-QUALITY-003 - Establish the reference set and parity baseline

## Description

Two of the phase's commitments are defined against the game itself rather than against a number: traversal
must be no slower than in Beyond All Reason, and the rendering must be indistinguishable from it. Both need a baseline
captured once, from the game, at a recorded commit - camera positions, screenshots and traversal times over a set of
maps chosen to span the ways maps actually differ. This is the task that makes those two commitments measurable
instead of rhetorical, and it must exist before the camera can be judged fast enough or the shaders judged faithful.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `QUALITY`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-CONF.2](../product-requirements/product-requirements.md#25-conformance-and-parity)

## Relationships

### Prerequisites

- [P01-MAE-FRONTEND-004 - Implement camera and navigation](P01-MAE-FRONTEND-004-implement-camera-and-navigation.md)

### Dependents

- [P01-MAE-FRONTEND-009 - Port the SMF shading chain](P01-MAE-FRONTEND-009-port-smf-shading-chain.md)
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

- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `OPER-REPORT.2`, `MARK-PARITY.1`, `COMP-CONTENT.1`
- **Product Requirements:** `FR-CONF.2`, `DATA-REF.1`, `NFR-PERF.2`, `NFR-FID.1`
- **Technical Design:** section 5.4, DR-7, invariant 12
- **Phase Outcome / Exit Criteria:** Makes the traversal and parity commitments measurable

## Scope

- Selecting six to ten maps spanning the corpus composition variation
- Capturing in-game screenshots and traversal times at a pinned engine commit
- Tooling to re-capture from the editor and compare

## Non-Goals

- Achieving parity, which is P01-MAE-FRONTEND-009
- Tuning the camera, which is P01-MAE-FRONTEND-004

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `reference/manifest.json` | Create | Reference set with camera positions and engine commit |
| `reference/captures/` | Create | In-game screenshots |
| `crates/mae-reference/` | Create | Capture and compare tooling |

## Implementation Requirements

1. Select six to ten maps satisfying the documented coverage rule: one with a skybox and one without; one with features and one without; one missing a normal, specular or splat texture; `mescaline_v2`; an option-sensitive map whose options alter the height range; and the largest and smallest archives by file count.
2. Record camera positions in world units, and use byte-identical positions for the in-game and editor captures. A comparison at different positions is meaningless.
3. Record the engine commit every capture was taken at. Changing that commit invalidates every capture in the manifest at once.
4. Measure in-game corner-to-corner traversal time per map by a documented, repeatable procedure, and record the procedure alongside the numbers.
5. Record which coverage rule each selected map satisfies, so the set can be audited rather than trusted.
6. Store captures as images only. No map data enters the repository.

## Quality Details

The manifest schema is specified in technical design section 5.4. The
`selected_for` field is what makes the set auditable: a reviewer can check that the coverage rule is actually
satisfied rather than taking the selection on faith.

The traversal procedure needs as much care as the numbers. "Corner to corner under normal camera control" has to mean
the same thing in the game and in the editor, or the comparison measures the procedure rather than the product.

## Acceptance Criteria

- [ ] **P01-MAE-QUALITY-003-AC01:** Six to ten maps are selected and each records which coverage rule it satisfies
- [ ] **P01-MAE-QUALITY-003-AC02:** Every coverage rule in the documented list is satisfied by at least one selected map
- [ ] **P01-MAE-QUALITY-003-AC03:** Camera positions are recorded in world units and are identical between in-game and editor captures
- [ ] **P01-MAE-QUALITY-003-AC04:** The engine commit is recorded and invalidates all captures when changed
- [ ] **P01-MAE-QUALITY-003-AC05:** In-game traversal times are measured by a documented repeatable procedure
- [ ] **P01-MAE-QUALITY-003-AC06:** No map data is committed - captures are images only

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-QUALITY-003-AC01 | Validate `reference/manifest.json` against the documented schema | Valid; six to ten maps, each with `selected_for` |
| P01-MAE-QUALITY-003-AC02 | Audit `selected_for` against the coverage rule | Every rule in the documented list is covered by at least one map |
| P01-MAE-QUALITY-003-AC03 | Compare camera positions between in-game and editor manifests | Byte-identical |
| P01-MAE-QUALITY-003-AC04 | Change the engine commit and re-validate | All captures flagged invalid |
| P01-MAE-QUALITY-003-AC05 | Read the recorded traversal procedure and re-measure one map | The procedure is repeatable and reproduces the recorded time |
| P01-MAE-QUALITY-003-AC06 | `git ls-files reference/ | grep -E '\.(sd7|smf|smt)$'` | No matches - captures are images only |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
