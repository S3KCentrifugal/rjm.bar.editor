# P01-MAE-BACKEND-006 - Implement map option selection and re-evaluation

## Description

Nearly every map ships options - 241 of 254 carry `mapoptions.lua` - and some of them change the map
materially, including rewriting its own height range. A viewer that always evaluates with engine defaults shows one of
several true states and gives the user no way to know the others exist. The stakeholder chose an option picker over
defaults-with-a-warning, so the definition evaluation must be re-runnable with a chosen option set, and everything
derived from it must refresh. This is the one place in the decode pipeline that runs more than once for a single open
file.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-READ.7](../product-requirements/product-requirements.md#21-reading-and-decoding)

## Relationships

### Prerequisites

- [P01-MAE-SECURITY-001 - Harden the map definition sandbox](P01-MAE-SECURITY-001-harden-map-definition-sandbox.md)
- [P01-MAE-BACKEND-002 - Implement the SMF decoder](P01-MAE-BACKEND-002-implement-smf-decoder.md)

### Dependents

- [P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md)

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

- **Business Requirements:** `OPER-WORKFLOW.2`
- **Product Requirements:** `FR-READ.7`, `NFR-VALID.1`
- **Technical Design:** section 3.3, ADR-11, DR-11
- **Phase Outcome / Exit Criteria:** Option-dependent values are shown truthfully

## Scope

- Enumerating declared options and their defaults
- Re-evaluating the definition against a selected option set
- Invalidating and recomputing every value derived from the definition

## Non-Goals

- The option picker interface, which is P01-MAE-FRONTEND-009
- Re-reading the file - the bytes have not changed

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/options.rs` | Create | Option enumeration and re-evaluation |
| `crates/mae-core/tests/options.rs` | Create | Height-range change and invalidation tests |

## Implementation Requirements

1. Enumerate options as declared by the map, with their defaults. The first open uses the engine default of an empty option set.
2. Re-evaluation re-enters the pipeline at the definition stage. Stages already completed on unchanged bytes - archive indexing, SMF parsing - must not repeat.
3. Everything derived from the definition must be recomputed, including the height range, terrain geometry and every checklist property that reads a definition value. A stale derived value after an option change would be a wrong value wearing a valid citation.
4. Reject unknown option keys rather than ignoring them. Silently dropping an option would change what the user sees without telling them.
5. Record which options were in force when a value was derived, so the provenance remains meaningful.

## Backend Details

`rjm.bar.tools` found that 154 of 227 maps read options and that some rewrite the
height range itself; the corpus scan here found `mapoptions.lua` in 241 of 254 archives. A map whose options change its
height range is the required test case, because it exercises the full invalidation chain from definition through
geometry.

The invalidation boundary is the crux: too narrow and stale values survive; too wide and an option change costs a full
re-decode, which the file has not required.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-006-AC01:** Declared options and their defaults are enumerated for every map that has them
- [ ] **P01-MAE-BACKEND-006-AC02:** Changing an option re-evaluates the definition without re-reading the archive
- [ ] **P01-MAE-BACKEND-006-AC03:** Every value derived from the definition is recomputed, verified on a map whose options alter the height range
- [ ] **P01-MAE-BACKEND-006-AC04:** An unknown option key is rejected, not ignored
- [ ] **P01-MAE-BACKEND-006-AC05:** The option set in force is recorded alongside derived values

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-006-AC01 | `cargo test -p mae-core options::enumerate` | Options and defaults listed for a known map |
| P01-MAE-BACKEND-006-AC02 | `cargo test -p mae-core options::no_reread` | Archive is not re-indexed on option change |
| P01-MAE-BACKEND-006-AC03 | `cargo test -p mae-core options::height_range_change` | Terrain geometry and dependent values change when the option changes |
| P01-MAE-BACKEND-006-AC04 | `cargo test -p mae-core options::unknown_key` | Rejected with a typed error |
| P01-MAE-BACKEND-006-AC05 | `cargo test -p mae-core options::recorded_in_provenance` | The option set in force is retrievable alongside each derived value |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
