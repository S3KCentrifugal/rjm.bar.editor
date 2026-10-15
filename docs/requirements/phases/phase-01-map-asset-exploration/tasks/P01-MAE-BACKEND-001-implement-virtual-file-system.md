# P01-MAE-BACKEND-001 - Implement the virtual file system over archives and directories

## Description

Every other decoder reads through this layer, and it is the only component in the system that touches the
filesystem. A map reaches the product in one of two forms: a `.sd7` archive, which is how the 254 installed maps are
stored, or an unpacked directory, which is what a map author has on disk mid-build and which one member of the local
corpus already is. The product must present both identically, because an author judging their own work should not see
something different from a reviewer judging the packed submission. Lookup is case-insensitive to match engine
behaviour, and the interface is read-only by construction - there is no write method to misuse.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `BACKEND`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-READ.1](../product-requirements/product-requirements.md#21-reading-and-decoding)

## Relationships

### Prerequisites

- [P01-MAE-DATA-002 - Extract the shared decoder crate from rjm.bar.tools](P01-MAE-DATA-002-extract-shared-decoder-crate.md)

### Dependents

- [P01-MAE-BACKEND-002 - Implement the SMF decoder](P01-MAE-BACKEND-002-implement-smf-decoder.md)
- [P01-MAE-BACKEND-005 - Implement the legacy .smd definition parser](P01-MAE-BACKEND-005-implement-legacy-smd-parser.md)
- [P01-MAE-BACKEND-008 - Implement texture and model decoders](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md)
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

- **Business Requirements:** `OPER-WORKFLOW.1`, `BIZ-EXP.4`, `GUARD-SCOPE.2`
- **Product Requirements:** `FR-READ.1`, `NFR-VALID.2`
- **Technical Design:** section 3.3, invariants 2 and 9
- **Phase Outcome / Exit Criteria:** Both input forms open identically

## Scope

- Archive backend over `.sd7`
- Directory backend
- Case-insensitive path lookup shared by both
- Read-only interface with no write surface

## Non-Goals

- Parsing any file content
- Caching decoded results, which is P01-MAE-BACKEND-013

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/vfs.rs` | Create | The two backends behind one interface |
| `crates/mae-core/tests/vfs.rs` | Create | Equivalence and case-insensitivity tests |

## Implementation Requirements

1. Expose one interface with two backends. No caller may branch on which backend it has.
2. Path lookup is case-insensitive, matching the engine. Archives in the corpus contain mixed-case paths such as `maps/Mescaline_V2.smf` referenced from lowercase Lua.
3. The interface exposes read and enumerate only. There is no write, create, delete or open-for-append method anywhere on it - this is where `NFR-VALID.2` is enforced structurally.
4. Reading an entry must not require reading the whole archive. The tile file alone can be 25 MB, and staged loading depends on being able to defer it.
5. Return typed errors naming the `vfs` stage, per ADR-09.

## Backend Details

The equivalence requirement is the sharp edge of this task. For any map, opening the
archive form and the extracted directory form must produce the same result - this is system invariant 9, and it is
tested directly rather than assumed.

`supreme_isthmus_v1.8/` is present in the local corpus as an unpacked directory and serves as the natural test subject
alongside its packed peers.

## Acceptance Criteria

- [ ] **P01-MAE-BACKEND-001-AC01:** Both `.sd7` archives and directories open through one interface with no caller-visible difference
- [ ] **P01-MAE-BACKEND-001-AC02:** Path lookup is case-insensitive in both backends
- [ ] **P01-MAE-BACKEND-001-AC03:** The interface exposes no method that can write, create or delete a file
- [ ] **P01-MAE-BACKEND-001-AC04:** An individual entry can be read without reading the whole archive
- [ ] **P01-MAE-BACKEND-001-AC05:** Failures return typed errors naming the `vfs` stage

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-BACKEND-001-AC01 | `cargo test -p mae-core vfs::equivalence` | Archive and extracted-directory forms of the same map yield identical entry sets and contents |
| P01-MAE-BACKEND-001-AC02 | `cargo test -p mae-core vfs::case_insensitive` | Mixed-case lookups resolve in both backends |
| P01-MAE-BACKEND-001-AC03 | `grep -rE 'fn (write|create|remove|append)' crates/mae-core/src/vfs.rs` | No matches |
| P01-MAE-BACKEND-001-AC04 | `cargo test -p mae-core vfs::partial_read` | A single entry is read without decompressing the tile file |
| P01-MAE-BACKEND-001-AC05 | `cargo test -p mae-core vfs::errors` | A missing or unreadable entry yields a typed error tagged with the `vfs` stage |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
