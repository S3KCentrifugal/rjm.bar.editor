# P01-MAE-DATA-001 - Implement the provenance value model

## Description

This is the single most pervasive structure in the phase and the one on which its central promise rests.
`GUARD-ACC.1` forbids the product from ever displaying an approximated value as though it were derived, and the
mechanism chosen for that is not a convention but a type: a value that can reach a user is wrapped together with where
it came from, so that constructing a displayable value without provenance is a compile error rather than a review
finding. The model carries three states, and the distinction between two of them matters as much as the guarantee
itself - a map with no skybox is complete and correct, while a malformed wind value is neither present nor
legitimately missing. Getting this wrong quietly makes every downstream value untrustworthy, which is why it is built
before anything that produces values.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `DATA`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [DATA-PROV.1](../product-requirements/product-requirements.md#4-data-requirements)

## Relationships

### Prerequisites

- [P01-MAE-DEVOPS-001 - Scaffold the workspace and record the licence](P01-MAE-DEVOPS-001-scaffold-workspace-and-licence.md)

### Dependents

- [P01-MAE-BACKEND-002 - Implement the SMF decoder](P01-MAE-BACKEND-002-implement-smf-decoder.md)
- [P01-MAE-BACKEND-005 - Implement the legacy .smd definition parser](P01-MAE-BACKEND-005-implement-legacy-smd-parser.md)
- [P01-MAE-BACKEND-008 - Implement texture and model decoders](P01-MAE-BACKEND-008-implement-texture-and-model-decoders.md)
- [P01-MAE-DATA-002 - Extract the shared decoder crate from rjm.bar.tools](P01-MAE-DATA-002-extract-shared-decoder-crate.md)
- [P01-MAE-DATA-003 - Embed BAR constants with commit provenance](P01-MAE-DATA-003-embed-bar-constants-with-provenance.md)

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

- **Business Requirements:** `GUARD-ACC.1`
- **Product Requirements:** `DATA-PROV.1`, `FR-SHOW.11`, `NFR-VALID.1`
- **Technical Design:** section 4.2, ADR-06, FMEA F8
- **Phase Outcome / Exit Criteria:** Underpins the exit criterion that no displayed value is an approximation

## Scope

- The `Provenanced<T>` type with its three states and rule citation
- Ergonomic construction and mapping helpers so the wrapper does not poison every call site
- Serialisation into the shape the extension surface and CLI contracts expect

## Non-Goals

- Any decoder that produces provenanced values
- Display treatment of the three states, which is P01-MAE-FRONTEND-007

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-core/src/provenance.rs` | Create | The value model |
| `crates/mae-core/tests/provenance.rs` | Create | Tests including the construction-is-impossible property |

## Implementation Requirements

1. Provide exactly three states: `Derived { value, rule }`, `Unknown { reason }` and `Absent`. Do not add a fourth, and do not let `Unknown` carry a value.
2. `RuleCitation` must carry a source location and the engine commit it was read at, both `&'static str`, so a citation cannot be fabricated at run time.
3. `UnknownReason` must be an enum, not a free string, so the reasons can be counted and tested.
4. Provide `map`, `and_then` and `as_ref` equivalents so callers can transform a derived value without unwrapping and re-wrapping, which is where provenance would otherwise be lost.
5. There must be no public API that yields a bare `T` for display purposes. Extraction for internal computation is acceptable; extraction for formatting is not.
6. Serialise to `{state, value?, rule?, reason?}` exactly as the extension surface contract in technical design section 5.1 requires.

## Data Details

```rust
pub struct Provenanced<T> { state: ValueState<T> }

pub enum ValueState<T> {
    Derived { value: T, rule: RuleCitation },
    Unknown { reason: UnknownReason },
    Absent,
}
```

The reason a wrapper was chosen over a parallel metadata map, recorded in ADR-06, is that a parallel map permits value
and provenance to drift apart - which is precisely the failure being guarded against. The cost is pervasive signature
impact, and the mitigation for that cost is the combinator set: if transforming a derived value is awkward, callers
will unwrap, and the guarantee erodes at exactly the points where it matters.

## Acceptance Criteria

- [ ] **P01-MAE-DATA-001-AC01:** `Provenanced<T>` exists with exactly three states and no way to construct a displayable bare value
- [ ] **P01-MAE-DATA-001-AC02:** A derived value always carries a rule citation with a source location and an engine commit
- [ ] **P01-MAE-DATA-001-AC03:** `Unknown` and `Absent` are distinct states and cannot be conflated by any public API
- [ ] **P01-MAE-DATA-001-AC04:** Combinators allow transforming a derived value without losing its citation
- [ ] **P01-MAE-DATA-001-AC05:** Serialisation matches the extension surface contract exactly

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-DATA-001-AC01 | `cargo test -p mae-core provenance` | All tests pass |
| P01-MAE-DATA-001-AC02 | `cargo test -p mae-core provenance::citation_required` | Constructing Derived without a citation fails to compile - verified by a compile-fail test |
| P01-MAE-DATA-001-AC03 | `cargo test -p mae-core provenance::unknown_is_not_absent` | The two states are distinguishable in every public accessor |
| P01-MAE-DATA-001-AC04 | `cargo test -p mae-core provenance::combinators` | A derived value is transformed through `map` and `and_then` with its citation intact |
| P01-MAE-DATA-001-AC05 | `cargo test -p mae-core provenance::serialisation` | Output matches the documented JSON shape |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
