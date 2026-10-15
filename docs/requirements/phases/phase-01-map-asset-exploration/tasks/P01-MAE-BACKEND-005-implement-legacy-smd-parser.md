# P01-MAE-BACKEND-005 - Implement the legacy .smd definition parser

## Description

One map in the corpus - `mescaline_v2` - predates `mapinfo.lua` and carries a legacy TDF `.smd` definition
instead. The existing analysis tool skips it. This phase cannot, because the stakeholder decided that every corpus
member opens with no exclusion list, and the engine itself still supports the fallback. The work is small and
self-contained, and it exists entirely because of that decision: it is the concrete cost of "no exceptions", recorded
as such in `DR-4`.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `S`
- **Parent Deliverable / User Story:** [FR-READ.6](../product-requirements/product-requirements.md#21-reading-and-decoding)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-001 - Implement the virtual file system over archives and directories](P01-MAE-BACKEND-001-implement-virtual-file-system.md)
- [P01-MAE-DATA-001 - Implement the provenance value model](P01-MAE-DATA-001-implement-provenance-value-model.md)

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

- [Rust conventions](~/.claude/skills/convention-rust/SKILL.md)
- [Testing conventions](~/.claude/skills/convention-testing/SKILL.md)

## Traceability

- **Business Requirements:** `OPER-MIGRATE.1`, `BIZ-EXP.1`
- **Product Requirements:** `FR-READ.6`
- **Technical Design:** section 2.1, FMEA F6
- **Phase Outcome / Exit Criteria:** Required by the no-exceptions corpus criterion

## Scope

- TDF `.smd` parsing
- Mapping legacy fields onto the same definition model as `mapinfo.lua`

## Non-Goals

- Writing `.smd`
- Supporting legacy formats beyond what the corpus contains

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/smd.rs` | Create | TDF parser and field mapping |
| `crates/mae-core/tests/smd.rs` | Create | Tests against the one corpus map that needs it |

## Implementation Requirements

1. Use `.smd` only when `mapinfo.lua` is absent, as the engine does - `rts/Map/MapParser.cpp` falls back to `directory + filename + ".smd"`. Two corpus archives carry both; those must use `mapinfo.lua`.
2. Map legacy fields onto the same definition model the Lua path produces, so no downstream code branches on definition source.
3. Record the definition source on the decoded map so it can be displayed - a reviewer looking at a legacy map should be able to tell.
4. Fields with no legacy equivalent are `Absent`, not `Unknown`, and not defaulted.

## Backend Details

TDF is a simple bracketed key-value format, nested by section. Three corpus archives
carry a `.smd`: `mescaline_v2`, `titan_v3.1` and `titanduel_v3` - but only `mescaline_v2` lacks `mapinfo.lua` and
therefore actually depends on this parser. The other two are useful as cross-checks: where both definitions exist, the
Lua path is authoritative and the `.smd` can be parsed and compared during development.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-005-AC01:** `mescaline_v2.sd7` opens with its definition values resolved from `.smd`
- [ ] **P01-MAE-BACKEND-005-AC02:** `.smd` is used only when `mapinfo.lua` is absent
- [ ] **P01-MAE-BACKEND-005-AC03:** The definition source is recorded and displayable
- [ ] **P01-MAE-BACKEND-005-AC04:** Downstream code cannot distinguish definition source except by reading that field

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-005-AC01 | `cargo test -p mae-core smd::mescaline` | Opens successfully; definition values present |
| P01-MAE-BACKEND-005-AC02 | `cargo test -p mae-core smd::precedence` | The two archives carrying both definitions use `mapinfo.lua` |
| P01-MAE-BACKEND-005-AC03 | `cargo test -p mae-core smd::source_recorded` | Definition source is queryable |
| P01-MAE-BACKEND-005-AC04 | `cargo test -p mae-core smd::uniform_model` | A map decoded from `.smd` and one from `mapinfo.lua` expose the same definition type and field set |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
