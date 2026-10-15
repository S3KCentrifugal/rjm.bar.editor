# P01-MAE-DATA-002 - Extract the shared decoder crate from rjm.bar.tools

## Description

`rjm.bar.tools` already contains a decoder that reproduces engine rules with source citations: archive
access, SMF grid parsing, sandboxed `mapinfo.lua` evaluation and terrain derivation, proven deterministic across 227
maps. This phase needs all of it, plus tile atlas, minimap, texture, model and provenance work that project does not
have. ADR-10 chose extraction into a shared crate over vendoring, because two decoders that must agree about how a map
decodes will eventually disagree, and because the tooling developer persona's whole workflow depends on the editor and
the analysis tool computing the same answer. This task performs that extraction and establishes the versioning
discipline that makes a cross-repository dependency safe.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `DATA`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `L`
- **Parent Deliverable / User Story:** [ADR-10](../technical-design/technical-design.md#8-alternatives-considered)

## Relationships

### Prerequisites

- [P01-MAE-DEVOPS-001 - Scaffold the workspace and record the licence](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md)
- [P01-MAE-DATA-001 - Implement the provenance value model](P01-MAE-DATA-001-implement-provenance-value-model.md)

### Dependents

- [P01-MAE-BACKEND-001 - Implement the virtual file system over archives and directories](P01-MAE-BACKEND-001-implement-virtual-file-system.md)
- [P01-MAE-SECURITY-001 - Harden the map definition sandbox](P01-MAE-SECURITY-001-harden-map-definition-sandbox.md)

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

- **Business Requirements:** `BIZ-EXP.1`, `BIZ-EXP.3`
- **Product Requirements:** `FR-READ.1`, `FR-READ.2`, `FR-READ.5`, `DATA-MAP.1`
- **Technical Design:** ADR-10, section 5.5, invariant 11
- **Phase Outcome / Exit Criteria:** Supplies the decode foundation every corpus map depends on

## Scope

- Extracting archive, SMF, Lua evaluation and terrain derivation into a shared crate
- Introducing provenance at the extracted crate's boundary without breaking the existing consumer
- Establishing semantic versioning and a compatibility rule across both repositories

## Non-Goals

- New decoders, which are separate tasks
- Changing `rjm.bar.tools` behaviour or its generated output

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/lib.rs` | Modify | Depend on the shared crate |
| `../rjm.bar.tools/` | Modify | Extract modules into a versioned crate; update its own use sites |
| `docs/decisions/shared-crate-compatibility.md` | Create | The cross-repository compatibility rule |

## Implementation Requirements

1. The extracted crate keeps `rjm.bar.tools`'s existing behaviour byte-for-byte. Its generated output must not change; that project's determinism guarantee is the regression test.
2. Provenance is introduced at the crate boundary, and must be optional or additive for the existing consumer so that extraction is not gated on changing that project's output.
3. Adopt semantic versioning. A breaking change to the decode API or to `Provenanced<T>` is a major bump and requires both consumers to be updated in the same change, per technical design section 5.5.
4. Record which engine commit each reproduced rule was read at, preserving the existing `file:line` citation discipline rather than weakening it.
5. Coordinate the change with `rjm.bar.tools` as a single reviewed change across both repositories; do not leave that project on a fork of the decoder.

## Data Details

Modules to extract, from the existing `rjm.bar.tools` layout: `archive` (`.sd7` access),
`smf` (grid parsing), `luaenv` (sandboxed Lua 5.1 with the VFS shim), `luajson`, `mapinfo` (typed view), `map`, and
`terrain`.

Not extracted, because they are specific to that tool's purpose: `analysis` (start-position classification for AI
configs) and `emit` (AngelScript and JSON rendering).

The compatibility rule is the deliverable that matters most here. Two repositories now depend on one answer to "how
does this map decode"; system invariant 11 requires their major versions to match, and the rule document is what makes
that checkable.

## Acceptance Criteria

- [ ] **P01-MAE-DATA-002-AC01:** A shared crate exists containing archive, SMF, Lua evaluation and terrain derivation
- [ ] **P01-MAE-DATA-002-AC02:** `rjm.bar.tools` consumes the shared crate and regenerates byte-identical output for its tracked map set
- [ ] **P01-MAE-DATA-002-AC03:** `mae-core` consumes the same crate
- [ ] **P01-MAE-DATA-002-AC04:** Semantic versioning and the cross-repository compatibility rule are documented
- [ ] **P01-MAE-DATA-002-AC05:** Every reproduced engine rule retains its file-and-line citation and gains the commit it was read at

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-DATA-002-AC01 | `cargo build --workspace` in both repositories | Both build against the shared crate |
| P01-MAE-DATA-002-AC02 | Regenerate `rjm.bar.tools` tracked output and diff | No change - determinism preserved |
| P01-MAE-DATA-002-AC03 | `cargo test -p mae-core` | Tests pass against the shared crate |
| P01-MAE-DATA-002-AC04 | Inspect `docs/decisions/shared-crate-compatibility.md` | Versioning scheme and major-version parity rule stated |
| P01-MAE-DATA-002-AC05 | `cargo test -p mae-core citations::completeness` | Every reproduced rule carries a file-and-line citation and an engine commit |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
