# P01-MAE-FRONTEND-008 - Implement open, reload and session state

## Description

The authoring persona's loop is compile, look, adjust, recompile, which means the cost that matters is not
opening a map once but reopening it repeatedly. This task covers the whole entry path: choosing an archive or a
directory, watching the map arrive in stages rather than behind a frozen window, noticing when the file has changed
underneath an open view, and remembering where the panels were. It is also where staged loading becomes visible to the
user - terrain first, appearance second, inspection third - which is the difference between a tool that feels
responsive and one that appears to hang for twenty seconds.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `FRONTEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [UX-OPEN.1](../product-requirements/product-requirements.md#5-user-workflows--ux-requirements)

## Relationships

### Prerequisites

- [P01-MAE-API-001 - Implement the extension surface](P01-MAE-API-001-implement-extension-surface.md)
- [P01-MAE-FRONTEND-004 - Implement camera and navigation](P01-MAE-FRONTEND-004-implement-camera-and-navigation.md)

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

- **Business Requirements:** `BIZ-EXP.4`, `GUARD-SCOPE.2`
- **Product Requirements:** `UX-OPEN.1`, `UX-RELOAD.1`, `DATA-SESS.1`, `NFR-PERF.3`, `NFR-VALID.2`
- **Technical Design:** section 3.3, ADR-11, DR-8
- **Phase Outcome / Exit Criteria:** Unpacked directories open as readily as archives

## Scope

- Open an archive or a directory
- Staged loading feedback
- Reload on external change
- Session state persistence

## Non-Goals

- Writing any map data
- Caching decoded maps to disk, rejected in ADR-11

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `app/scripts/open_flow.gd` | Create | Open and reload |
| `app/scripts/session_state.gd` | Create | Layout and last-map persistence |

## Implementation Requirements

1. Accept both a `.sd7` file and a directory through the same entry point, with no behavioural difference between them.
2. Show loading progress by stage. The interface must never block for more than one second at a time.
3. Detect that an open map's source has changed and offer reload. On reload, preserve camera position and overlay selections.
4. Always offer an explicit reload action, because a directory's contents can change without its own timestamp changing.
5. If the source is removed or is mid-write, refuse the reload with a reason and keep the previous view rather than discarding it.
6. Persist window layout, last-opened map and panel selections to a single local file. If the last map has moved, open empty rather than failing.
7. Write nothing except session state. No map file is opened for writing, ever.

## Frontend Details

Stage feedback maps to the publication stages: S1 makes terrain navigable within
five seconds, S2 applies appearance within twenty, S3 completes inspection.

The mid-write case is real rather than theoretical: an author recompiling a map is writing the file this product may
be reading, and the correct response is to decline and keep what is on screen, not to load half a map.

## Acceptance Criteria

- [ ] **P01-MAE-FRONTEND-008-AC01:** Archives and directories open through the same entry point with no behavioural difference
- [ ] **P01-MAE-FRONTEND-008-AC02:** Loading progress is shown by stage and the interface never blocks for more than a second
- [ ] **P01-MAE-FRONTEND-008-AC03:** An externally modified source is detected and reload is offered, preserving camera and overlay state
- [ ] **P01-MAE-FRONTEND-008-AC04:** An explicit reload action is always available
- [ ] **P01-MAE-FRONTEND-008-AC05:** A removed or mid-write source is refused with a reason, keeping the previous view
- [ ] **P01-MAE-FRONTEND-008-AC06:** Session state persists and a missing last map opens empty rather than failing

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-FRONTEND-008-AC01 | Open the same map as archive and as directory | Identical presentation |
| P01-MAE-FRONTEND-008-AC02 | Open the largest corpus archive and watch | Stage indicator advances; no block over one second |
| P01-MAE-FRONTEND-008-AC03 | Modify an open map's source externally | Reload offered; camera and overlays preserved |
| P01-MAE-FRONTEND-008-AC04 | Delete an open map's source and reload | Refused with a reason; previous view retained |
| P01-MAE-FRONTEND-008-AC05 | Restart with the last map moved | Opens empty, no error dialog |
| P01-MAE-FRONTEND-008-AC06 | `grep -rE 'FileAccess.WRITE' app/ | grep -v session_state` | Only session state writes |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
