# P01-MAE-FRONTEND-009 - Port the SMF shading chain

## Description

Visual parity with Recoil is what makes this a reference renderer rather than a viewer, and it is also the
one commitment the stakeholder designated as the first to be sacrificed if something has to give. That ranking is the
reason it sits last: the phase closes on maps opening, assets appearing and exploration being fast, and this work
improves fidelity without gating any of them. The approach is to port the engine's own shaders rather than to
reimplement their effect, starting from the shader Terrain3D generates when an override is enabled blank. Parity is
claimed only against a pinned engine commit with stored captures, never in the abstract.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P2`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [NFR-FID.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- [P01-MAE-FRONTEND-005 - Present sky, water, features and start positions](P01-MAE-FRONTEND-005-present-scene-assets.md)
- [P01-MAE-QUALITY-003 - Establish the reference set and parity baseline](P01-MAE-QUALITY-003-establish-reference-set-and-parity.md)

### Dependents

- None

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

- **Business Requirements:** `MARK-PARITY.1`, `GUARD-REPUTE.1`
- **Product Requirements:** `NFR-FID.1`, `FR-SHOW.2`
- **Technical Design:** ADR-08, section 5.4
- **Phase Outcome / Exit Criteria:** Target commitment, ranked last and designated the sacrifice

## Scope

- Porting the SMF vertex and fragment programs to Godot's shading language
- Feeding all ten engine texture slots correctly
- Comparison against the stored reference captures

## Non-Goals

- Claiming parity without stored captures at a recorded commit
- Blocking phase closure - this is P2 and sacrificial

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/shaders/smf_terrain.gdshader` | Create | Ported shading chain |
| `docs/decisions/shader-slot-mapping.md` | Create | Slot-by-slot correspondence and any gaps |

## Implementation Requirements

1. Start from the shader Terrain3D emits when `shader_override` is enabled blank, and modify it, rather than writing from nothing.
2. Port `SMFVertProg.glsl` and `SMFFragProg.glsl` from `cont/base/springcontent/shaders/GLSL/`, recording the engine commit ported from.
3. Feed all ten texture slots and the four override grids with correctly decoded data. Parity fails silently if a slot is fed the wrong texture.
4. Document any engine slot with no direct Godot equivalent, and what was done instead. This was an open gap in the architecture research and must not stay open once the work is done.
5. Compare against the stored reference captures at matching camera positions, and record the result.
6. Make no parity claim anywhere in the product or its documentation without a stored capture at a recorded commit.
7. This work is a derivative of GPL v2-or-later engine code, which the project licence already accounts for.

## Frontend Details

The parity test is human side-by-side comparison against captures taken from the
game at a pinned commit, over the reference map set - not an automated pixel diff, which was rejected as brittle
across drivers, and not the early user's subjective impression, which is not repeatable.

Because this task is P2 and explicitly sacrificial, it should be the last thing started and the first thing cut. If it
is dropped, the phase still closes: the three protected commitments do not depend on it.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-009-AC01:** The SMF vertex and fragment programs are ported and applied through the Terrain3D shader override
- [ ] **P01-MAE-FRONTEND-009-AC02:** All ten texture slots and four override grids are fed with correctly decoded data
- [ ] **P01-MAE-FRONTEND-009-AC03:** Any slot with no Godot equivalent is documented with what was done instead
- [ ] **P01-MAE-FRONTEND-009-AC04:** Side-by-side comparison against the reference captures is performed and recorded
- [ ] **P01-MAE-FRONTEND-009-AC05:** No parity claim exists anywhere without a stored capture at a recorded commit

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-009-AC01 | Inspect the terrain material at runtime | The ported shader is applied through the Terrain3D override |
| P01-MAE-FRONTEND-009-AC02 | Inspect shader uniforms at runtime | All ten slots and four overrides bound to the correct textures |
| P01-MAE-FRONTEND-009-AC03 | Read `docs/decisions/shader-slot-mapping.md` | Every engine slot accounted for, including any with no Godot equivalent |
| P01-MAE-FRONTEND-009-AC04 | Compare each reference-set map against its stored capture | Judged indistinguishable at matching camera positions, or the difference recorded |
| P01-MAE-FRONTEND-009-AC05 | `grep -riE 'parity' docs/ app/ --include=*.md --include=*.gd` | Every claim cites an engine commit and a stored capture |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
