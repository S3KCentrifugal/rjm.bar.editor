# P01-MAE-DOCUMENTATION-001 - Prepare the early-user evaluation

## Description

Every reviewer- and author-facing requirement in this phase rests on a persona reconstructed from
published BAR process documents, because no mapper has been consulted. That is the largest evidential weakness in the
whole chain, recorded as `RISK-6` at raised probability and impact, and the stakeholder chose to close it only once a
runnable build exists. This task is that closure: recruit one mapper, have them fly a map they authored, and record
what they say. It is the last task in the phase by dependency, and deliberately so - but it is also the one most
likely to change what the next phase should do.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `DOCUMENTATION`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `S`
- **Parent Deliverable / User Story:** [BIZ-SAT.1](../business-requirements/business-requirements.md#4-business-objectives--outcomes)

## Relationships

### Prerequisites

- [P01-MAE-DEVOPS-002 - Build and package the release artifacts](P01-MAE-DEVOPS-002-build-and-package-artifacts.md)
- [P01-MAE-FRONTEND-007 - Build the inspection interface](P01-MAE-FRONTEND-007-build-inspection-interface.md)

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

- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-SAT.1`
- **Product Requirements:** Validates the personas underlying `BIZ-EFF.1` and the UX requirements
- **Technical Design:** Persona provenance note, section 1.3 of the product requirements
- **Phase Outcome / Exit Criteria:** The recruited early-user mapper has flown a map they authored and feedback is recorded

## Scope

- A build and a short guide a mapper can use unaided
- Recruiting one BAR mapper
- Recording their feedback against the persona assumptions

## Non-Goals

- Acting on the feedback within this phase
- Any public release or announcement

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `docs/early-user/guide.md` | Create | A one-page guide |
| `docs/early-user/feedback.md` | Create | Recorded session and findings |
| `docs/requirements/phases/phase-01-map-asset-exploration/business-requirements/business-requirements.md` | Modify | Record ASSUM-5 revalidation outcome |

## Implementation Requirements

1. Package a build that opens a map without setup steps a mapper would not tolerate. The reviewing persona expects productivity within minutes and without documentation, and that expectation is itself under test here.
2. Recruit one BAR mapper - an author or a reviewer - through the mapping community.
3. Have them open a map they authored themselves and fly it. This is the defined first task, chosen because they know that terrain better than anyone and errors will be obvious to them instantly.
4. Record their reaction against the specific persona assumptions: what they checked first, whether the checklist view matched how they actually review, whether anything shown was wrong, and whether it beat launching the game.
5. Record the outcome as a revalidation of `ASSUM-5`, stating plainly whether the persona held.
6. If the persona was materially wrong, raise a `propagate-change` request rather than quietly adjusting downstream documents.

## Documentation Details

The questions worth asking are the ones the specification could not answer
from documents: does the twenty-property checklist view match how a review is actually conducted; is the
`INF-P2` assumption about deep-linking from a property to the map correct; and does reopen cost actually dominate for
an author, as `INF-P3` assumes.

The honest framing is that this task exists to find out whether parts of this phase were built on a wrong model of its
users, and a negative finding is a successful outcome for it.

## Acceptance Criteria

- [ ] **P01-MAE-DOCUMENTATION-001-AC01:** A build and a one-page guide exist that a mapper can use without assistance
- [ ] **P01-MAE-DOCUMENTATION-001-AC02:** One BAR mapper who is not the project owner has flown a map they authored
- [ ] **P01-MAE-DOCUMENTATION-001-AC03:** Their feedback is recorded against the specific persona assumptions, including inferences INF-P2 and INF-P3
- [ ] **P01-MAE-DOCUMENTATION-001-AC04:** `ASSUM-5` is revalidated with a plain statement of whether the persona held
- [ ] **P01-MAE-DOCUMENTATION-001-AC05:** A material persona error raises a propagate-change request rather than a silent edit

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-DOCUMENTATION-001-AC01 | Inspect `docs/early-user/guide.md` | One page; no setup step a mapper would refuse |
| P01-MAE-DOCUMENTATION-001-AC02 | Inspect `docs/early-user/feedback.md` | Names the participant and the map they authored, with a dated session |
| P01-MAE-DOCUMENTATION-001-AC03 | Read the findings section | Each persona assumption addressed, including inferences INF-P2 and INF-P3 |
| P01-MAE-DOCUMENTATION-001-AC04 | Inspect the business requirements | `ASSUM-5` revalidation recorded with a plain statement of whether the persona held |
| P01-MAE-DOCUMENTATION-001-AC05 | Inspect the governance change record | A propagate-change request exists if the persona was materially wrong, or the finding states it was not |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
