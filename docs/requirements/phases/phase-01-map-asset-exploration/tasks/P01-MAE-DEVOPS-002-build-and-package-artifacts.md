# P01-MAE-DEVOPS-002 - Build and package the release artifacts

## Description

The phase ships two artifacts per platform, and the separation is deliberate rather than incidental: the
application carries the extension and Terrain3D, while the conformance CLI is standalone and links no Godot at all.
That second artifact is what proves the product needs no game and no engine to read a map, and it is also what makes
the corpus run cheap enough to sit in continuous integration. This task builds both for Windows and Linux, attaches
the attribution the licences require, and verifies independence on a machine with nothing else installed.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `DEVOPS`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [NFR-PORT.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- [P01-MAE-QUALITY-001 - Build the corpus conformance suite](P01-MAE-QUALITY-001-build-corpus-conformance-suite.md)
- [P01-MAE-FRONTEND-007 - Build the inspection interface](P01-MAE-FRONTEND-007-build-inspection-interface.md)

### Dependents

- [P01-MAE-DOCUMENTATION-001 - Prepare the early-user evaluation](P01-MAE-DOCUMENTATION-001-prepare-early-user-evaluation.md)

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

- [Deployment and operations conventions](~/.claude/skills/convention-deployment/SKILL.md)
- [Developer tooling conventions](~/.claude/skills/convention-developer-tooling/SKILL.md)

## Traceability

- **Business Requirements:** `OPER-WORKFLOW.3`, `COMP-LICENSE.2`, `COMP-CONTENT.1`, `MARK-DIFF.1`
- **Product Requirements:** `NFR-PORT.1`, `NFR-LIC.1`, `NFR-LIC.2`, `NFR-DEP.1`
- **Technical Design:** section 6.1, ADR-13, FMEA F7
- **Phase Outcome / Exit Criteria:** Runs on both platforms with no game installed

## Scope

- Windows and Linux builds of both artifacts
- Dependency attribution in every build
- Clean-machine independence verification
- Version reporting

## Non-Goals

- Auto-update, which the product deliberately does not have
- Any distribution channel decision

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `.github/workflows/release.yml` | Create | Build matrix and packaging |
| `scripts/verify_clean_machine.sh` | Create | Independence check |
| `RELEASE.md` | Create | Per-release version and constant provenance record |

## Implementation Requirements

1. Build `mae-app` with `mae-gdext` and Terrain3D bundled, and `mae-conform` standalone, for Windows and Linux.
2. Include dependency attribution in every distributed build, per `NFR-LIC.1`.
3. Verify on a container with neither Godot nor Beyond All Reason installed that `mae-conform` runs a full corpus pass. This is the direct test of `NFR-DEP.1`.
4. Each release records its gdext, Terrain3D and Godot versions and the BAR commit its embedded constants were read from, so a user can tell whether a pathability result is current.
5. No map content is included in any artifact.
6. There is no auto-update mechanism; rollback is retaining the previous download, and nothing server-side can conflict with a downgrade.

## DevOps Details

Two artifacts per platform, four in total:

| Artifact | Contains | Proves |
|:---|:---|:---|
| `mae-app` | Godot export, `mae-gdext`, Terrain3D, attribution | The product |
| `mae-conform` | Rust binary, `mae-core` only | That the product needs no engine |

The constants provenance line in `RELEASE.md` is a mitigation for FMEA F7, the highest-rated failure in the design: a
user looking at a pathability result can check which BAR commit it was computed against.

## Acceptance Criteria

- [ ] **P01-MAE-DEVOPS-002-AC01:** Both artifacts build for Windows and Linux
- [ ] **P01-MAE-DEVOPS-002-AC02:** Dependency attribution is present in every distributed build
- [ ] **P01-MAE-DEVOPS-002-AC03:** `mae-conform` completes a full corpus pass on a machine with no Godot and no BAR
- [ ] **P01-MAE-DEVOPS-002-AC04:** Each release records its gdext, Terrain3D and Godot versions and the BAR constants commit
- [ ] **P01-MAE-DEVOPS-002-AC05:** No map content appears in any artifact

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-DEVOPS-002-AC01 | Trigger the release workflow | Four artifacts produced |
| P01-MAE-DEVOPS-002-AC02 | Inspect each artifact for attribution | Present in all four |
| P01-MAE-DEVOPS-002-AC03 | `scripts/verify_clean_machine.sh` | Full corpus pass on a bare container |
| P01-MAE-DEVOPS-002-AC04 | Read `RELEASE.md` | Four version records present |
| P01-MAE-DEVOPS-002-AC05 | Inspect artifact contents for map files | None found |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
