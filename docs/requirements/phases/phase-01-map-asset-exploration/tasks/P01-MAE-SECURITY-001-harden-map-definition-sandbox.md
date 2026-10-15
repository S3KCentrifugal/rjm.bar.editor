# P01-MAE-SECURITY-001 - Harden the map definition sandbox

## Description

Map archives are files people download from strangers, and reading one correctly requires executing the Lua
inside it. That makes the sandbox the product's only security boundary, and it is a real one. The existing evaluator in
`rjm.bar.tools` handles capability - it exposes a restricted global set and a VFS shim - but capability denial and
resource exhaustion are separate axes, and only the first is currently addressed. A map with an infinite loop or an
unbounded allocation will hang or exhaust the process today. This task closes the second axis and makes the whole
boundary explicit and tested, rather than a property that happens to hold.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `SECURITY`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [NFR-SEC.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-001 - Implement the virtual file system over archives and directories](P01-MAE-BACKEND-001-implement-virtual-file-system.md)
- [P01-MAE-DATA-002 - Extract the shared decoder crate from rjm.bar.tools](P01-MAE-DATA-002-extract-shared-decoder-crate.md)

### Dependents

- [P01-MAE-BACKEND-006 - Implement map option selection and re-evaluation](P01-MAE-BACKEND-006-implement-option-evaluation.md)
- [P01-MAE-QUALITY-002 - Fuzz the decoders and the sandbox](P01-MAE-QUALITY-002-fuzz-decoders-and-sandbox.md)

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

- **Business Requirements:** `GUARD-SCOPE.3`, `BIZ-EXP.1`
- **Product Requirements:** `NFR-SEC.1`, `FR-READ.5`, `NFR-REL.1`
- **Technical Design:** section 3.4, section 8, ADR-05, FMEA F1, invariant 8
- **Phase Outcome / Exit Criteria:** Supports zero crashes across the corpus, including hostile input

## Scope

- Whitelist review of the exposed global set
- Instruction-count ceiling
- Memory ceiling
- A threat-model note recording what is and is not contained

## Non-Goals

- Executing map gadget Lua, which is prohibited outright by `NFR-VALID.3`
- Process-level isolation, rejected in the technical design as disproportionate to the threat model

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/sandbox.rs` | Create | Capability whitelist and resource ceilings |
| `crates/mae-core/tests/sandbox.rs` | Create | Hostile-input tests for all four capability classes |
| `docs/decisions/sandbox-threat-model.md` | Create | What is contained, what is not, and why |

## Implementation Requirements

1. Build the exposed global set as a **whitelist**, not by removing known-dangerous names. A blacklist grows a hole every time the language does.
2. Deny filesystem access other than through the VFS shim, which exposes archive contents only.
3. Deny networking: no networking library is present in the environment.
4. Deny process creation: `os.execute` and equivalents are absent from the whitelist.
5. Impose an instruction-count hook and a memory ceiling. This is the axis a global whitelist does not address, and without it FMEA F1 - a map that loops forever - is unmitigated.
6. Calibrate both ceilings against the corpus: measure the slowest and heaviest legitimate map, then set the limit with headroom. Do not guess a number.
7. On limit breach, terminate evaluation, mark dependent values Unknown with a reason, and leave the map open.
8. Record in the threat-model note that a sandbox escape in the Lua implementation itself is not contained, and why that trade-off was accepted.

## Security Details

Four capability classes must be denied, and the fourth is the one that is easy to
miss:

| Capability | Mechanism | Sufficient alone? |
|:---|:---|:---|
| Filesystem | Whitelist plus VFS shim | Yes |
| Network | No library exposed | Yes |
| Process creation | Absent from whitelist | Yes |
| Unbounded execution | Instruction hook plus memory ceiling | **No - a separate mechanism entirely** |

Test with deliberately hostile input, not only with real maps: an infinite loop, an unbounded table allocation, an
attempt to open a file outside the archive, an attempt to spawn a process.

## Acceptance Criteria

- [ ] **P01-MAE-SECURITY-001-AC01:** The exposed global set is a whitelist, and adding to it requires an explicit code change
- [ ] **P01-MAE-SECURITY-001-AC02:** Filesystem access outside the VFS shim, networking and process creation are all denied and tested
- [ ] **P01-MAE-SECURITY-001-AC03:** An instruction ceiling and a memory ceiling are enforced, calibrated from corpus measurement rather than guessed
- [ ] **P01-MAE-SECURITY-001-AC04:** A map that breaches a ceiling leaves dependent values Unknown and the map open
- [ ] **P01-MAE-SECURITY-001-AC05:** The threat model records what is not contained

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-SECURITY-001-AC01 | `cargo test -p mae-core sandbox::capabilities` | Filesystem, network and process attempts all fail |
| P01-MAE-SECURITY-001-AC02 | `cargo test -p mae-core sandbox::infinite_loop` | Evaluation terminates at the ceiling; values Unknown; map opens |
| P01-MAE-SECURITY-001-AC03 | `cargo test -p mae-core sandbox::memory_bomb` | Allocation is capped; process survives |
| P01-MAE-SECURITY-001-AC04 | `cargo test -p mae-core sandbox::corpus_headroom` | No legitimate corpus map breaches either ceiling |
| P01-MAE-SECURITY-001-AC05 | Inspect `docs/decisions/sandbox-threat-model.md` | Uncontained cases stated explicitly |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
