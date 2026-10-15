# P01-MAE-QUALITY-001 - Build the corpus conformance suite

## Description

The phase's headline commitment is that every map in the library opens, with no exclusion list and no
allowance, and this suite is what makes that claim checkable rather than aspirational. It runs the conformance CLI over
the whole corpus on both supported platforms and fails the build if a single map fails. Because the two platforms must
produce identical per-map results, it doubles as a cross-platform differential test - a category of bug that is
otherwise very easy to ship. The corpus is the user's own installed map library, which the repository never contains.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `QUALITY`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-CONF.1](../product-requirements/product-requirements.md#25-conformance-and-parity)

## Relationships

### Prerequisites

- [P01-MAE-API-002 - Implement the conformance CLI](P01-MAE-API-002-implement-conformance-cli.md)

### Dependents

- [P01-MAE-DEVOPS-002 - Build and package the release artifacts](P01-MAE-DEVOPS-002-build-and-package-artifacts.md)

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
- [Deployment and operations conventions](~/.claude/skills/convention-deployment/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EXP.1`, `OPER-REPORT.1`, `COMP-CONTENT.1`
- **Product Requirements:** `FR-CONF.1`, `NFR-PORT.1`, `NFR-LIC.2`
- **Technical Design:** section 6.3
- **Phase Outcome / Exit Criteria:** Produces the evidence that every corpus member opens

## Scope

- Conformance run wired into continuous integration on both platforms
- Per-map result comparison across platforms
- Failure reporting grouped by stage

## Non-Goals

- Performance measurement, which is P01-MAE-QUALITY-004
- Bundling map files, which `COMP-CONTENT.1` prohibits

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `.github/workflows/conformance.yml` | Create | CI workflow for both platforms |
| `scripts/compare_conformance.py` | Create | Cross-platform result differ |

## Implementation Requirements

1. Run the conformance CLI over the full corpus on Windows and on Linux.
2. Fail the build if any map fails on either platform. There is no threshold - `BIZ-EXP.1` admits no exclusions.
3. Compare per-map results between platforms and fail on any difference, which is how `NFR-PORT.1`'s identical-results requirement is enforced.
4. Group failures by decode stage and kind so a systemic failure is distinguishable from a single bad archive.
5. The corpus is supplied by the runner environment, never committed. No map file enters the repository.
6. Publish the report as a build artifact so a failure can be investigated without re-running.

## Quality Details

The corpus is 254 archives plus one unpacked directory in the local Beyond All
Reason installation. Because it cannot be committed, the workflow must locate it in the runner environment and fail
clearly if it is absent rather than silently passing an empty run - a zero-map success is the most dangerous possible
false green here.

## Acceptance Criteria

- [ ] **P01-MAE-QUALITY-001-AC01:** The conformance run executes over the full corpus on Windows and Linux in CI
- [ ] **P01-MAE-QUALITY-001-AC02:** Any single map failure fails the build, with no threshold
- [ ] **P01-MAE-QUALITY-001-AC03:** Per-map results are compared across platforms and any difference fails the build
- [ ] **P01-MAE-QUALITY-001-AC04:** An empty or missing corpus fails the run rather than passing vacuously
- [ ] **P01-MAE-QUALITY-001-AC05:** No map file is committed to the repository

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-QUALITY-001-AC01 | Trigger the workflow on both platforms | Both complete with a full report |
| P01-MAE-QUALITY-001-AC02 | Inject one corrupt archive | Build fails; the failing map and stage are named |
| P01-MAE-QUALITY-001-AC03 | Inject a platform-specific difference | Comparison step fails |
| P01-MAE-QUALITY-001-AC04 | Run with the corpus path unset | Run fails with a clear cause, not a pass |
| P01-MAE-QUALITY-001-AC05 | `git ls-files | grep -E '\\.sd7$'` | No matches |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
