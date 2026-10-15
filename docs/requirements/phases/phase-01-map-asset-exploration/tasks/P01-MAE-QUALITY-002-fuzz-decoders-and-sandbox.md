# P01-MAE-QUALITY-002 - Fuzz the decoders and the sandbox

## Description

The corpus proves the product handles 255 well-formed maps. It proves nothing about the 256th, or about a
deliberately hostile one - and since map archives come from strangers, hostile input is a realistic case rather than a
hypothetical. Every decoder in this phase reads offsets and lengths out of file headers and uses them to slice buffers,
which is the classic shape of a panic or a memory-safety failure. Fuzzing is what converts the panic-free contract from
a lint rule into evidence, and it targets the sandbox too, because resource exhaustion is the failure mode a capability
whitelist does not address.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `QUALITY`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [NFR-REL.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md)
- [P01-MAE-SECURITY-001 - Harden the map definition sandbox](P01-MAE-SECURITY-001-harden-map-definition-sandbox.md)

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
- [Rust conventions](~/.claude/skills/convention-rust/SKILL.md)

## Traceability

- **Business Requirements:** `BIZ-EXP.1`
- **Product Requirements:** `NFR-REL.1`, `NFR-SEC.1`
- **Technical Design:** ADR-09, FMEA F1 and F2, invariant 7
- **Phase Outcome / Exit Criteria:** Zero crashes, including on hostile input

## Scope

- Fuzz targets for SMF, SMT, minimap, TDF, texture and S3O decoding
- Fuzz target for the Lua sandbox including resource exhaustion
- Seed corpus derived from real archives

## Non-Goals

- Fixing individual defects found, which are their own work items
- Fuzzing the Godot layer, which is out of scope for this phase

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/fuzz/` | Create | Fuzz targets and seed corpus |
| `.github/workflows/fuzz.yml` | Create | Scheduled fuzzing run |

## Implementation Requirements

1. Provide a target per decoder: SMF, SMT, minimap, TDF, texture and S3O.
2. Provide a sandbox target that feeds generated Lua, including deliberately pathological programs - infinite loops, unbounded allocation, deep recursion.
3. Seed from real corpus archives, then mutate. Random bytes alone will not reach past the header checks.
4. Any panic is a defect. The contract is that a malformed file yields a typed error, never a crash.
5. Run on a schedule rather than per-commit, with any discovered crash filed and its input added to the regression corpus.
6. Add every crashing input found to the permanent regression set so a fixed defect stays fixed.

## Quality Details

FMEA F2 - a truncated or malformed file causing a panic - is rated RPN 108, and
its severity comes from `BIZ-EXP.1`: with no exclusion list permitted, one crash on one archive blocks the phase.

The sandbox target matters independently. `TR-SEC-01` established that capability denial and resource exhaustion are
separate axes; the capability half can be tested with fixed cases, but the exhaustion half benefits from
generation.

## Acceptance Criteria

- [ ] **P01-MAE-QUALITY-002-AC01:** A fuzz target exists for every decoder and for the sandbox
- [ ] **P01-MAE-QUALITY-002-AC02:** The seed corpus is derived from real archives, not synthetic bytes
- [ ] **P01-MAE-QUALITY-002-AC03:** No panic occurs in any target over a sustained run
- [ ] **P01-MAE-QUALITY-002-AC04:** Discovered crashing inputs are added to a permanent regression corpus
- [ ] **P01-MAE-QUALITY-002-AC05:** Fuzzing runs on a schedule in CI

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-QUALITY-002-AC01 | `cargo fuzz list` | Targets present for SMF, SMT, minimap, TDF, texture, S3O and sandbox |
| P01-MAE-QUALITY-002-AC02 | `cargo fuzz run smf -- -max_total_time=3600` | No panics |
| P01-MAE-QUALITY-002-AC03 | `cargo fuzz run sandbox -- -max_total_time=3600` | No hangs or memory exhaustion; ceilings hold |
| P01-MAE-QUALITY-002-AC04 | Inspect the regression corpus | Every historically crashing input is present |
| P01-MAE-QUALITY-002-AC05 | Inspect the workflow | Scheduled run configured |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
