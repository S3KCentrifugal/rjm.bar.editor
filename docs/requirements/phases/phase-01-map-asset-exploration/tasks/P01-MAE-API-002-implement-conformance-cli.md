# P01-MAE-API-002 - Implement the conformance CLI

## Description

The phase exits when every corpus member opens, and this is the tool that says whether they do. It links
`mae-core` directly and never loads Godot, which is what makes it both cheap enough to run in continuous integration
and capable of proving that the product needs no game installation. Its output is consumed by people and by automation,
so the exit codes and JSON shape are a contract rather than a convenience. Because `NFR-PORT.1` requires identical
per-map results on Windows and Linux, this run is a cross-platform differential test rather than a smoke test.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `API`
- **Status:** `Todo`
- **Priority:** `P1`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [FR-CONF.1](../product-requirements/product-requirements.md#25-conformance-and-parity)

## Relationships

### Prerequisites

- [P01-MAE-BACKEND-009 - Implement the error taxonomy and staged decode pipeline](P01-MAE-BACKEND-009-implement-error-taxonomy-and-staging.md)

### Dependents

- [P01-MAE-QUALITY-001 - Build the corpus conformance suite](P01-MAE-QUALITY-001-build-corpus-conformance-suite.md)

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

- **Business Requirements:** `OPER-REPORT.1`, `BIZ-EXP.1`, `MARK-DIFF.1`
- **Product Requirements:** `FR-CONF.1`, `NFR-DEP.1`, `NFR-PORT.1`
- **Technical Design:** section 5.3, section 6.3
- **Phase Outcome / Exit Criteria:** Produces the evidence for the corpus exit criterion

## Scope

- The CLI contract: arguments, exit codes, JSON output
- Parallel execution over a map library
- Determinism across runs

## Non-Goals

- Rendering or parity capture, which is P01-MAE-QUALITY-003
- Any Godot dependency whatsoever

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `crates/mae-conform/src/main.rs` | Create | CLI entry point |
| `crates/mae-conform/tests/cli.rs` | Create | Exit code and determinism tests |

## Implementation Requirements

1. Implement the contract in technical design section 5.3 exactly: `--library`, `--json`, `--jobs`, `--fail-fast`; exit 0 all passed, 1 one or more failed, 2 invalid invocation, 3 library unreadable.
2. Emit JSON with `library`, `total`, `passed`, `failed`, and a `results` array of `{map, ok, stage_reached, duration_ms}`.
3. On failure, record the stage and kind from the error taxonomy so results can be grouped by cause.
4. Re-running over unchanged inputs produces identical output apart from timings.
5. Link `mae-core` only. The binary must run on a machine with no Godot and no Beyond All Reason installed.
6. Process maps in parallel by default, bounded by `--jobs`.

## API Details

Exit code 1 on any failure is what makes `BIZ-EXP.1`'s no-exceptions rule enforceable in
CI: there is no threshold and no allowance, so a single failing map turns the build red.

Determinism matters because the report is a differential artifact - the Windows and Linux runs are compared to each
other, and non-deterministic ordering would produce false differences.

## Acceptance Criteria

- [ ] **P01-MAE-API-002-AC01:** The documented arguments, exit codes and JSON shape are implemented exactly
- [ ] **P01-MAE-API-002-AC02:** Exit code is 1 if any map fails, with no threshold or allowance
- [ ] **P01-MAE-API-002-AC03:** Failures record stage and kind from the error taxonomy
- [ ] **P01-MAE-API-002-AC04:** Re-running over unchanged inputs produces identical output apart from timings
- [ ] **P01-MAE-API-002-AC05:** The binary runs with no Godot and no BAR installed

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-API-002-AC01 | `mae-conform --library <corpus> --json out.json` | Exit 0 and a complete report when all maps pass |
| P01-MAE-API-002-AC02 | `mae-conform --library <corpus-with-one-corrupt>` | Exit 1; the failing map records its stage |
| P01-MAE-API-002-AC03 | Run twice and diff, ignoring durations | Identical output |
| P01-MAE-API-002-AC04 | Run on a container with no Godot and no BAR | Completes normally |
| P01-MAE-API-002-AC05 | `mae-conform --library /nonexistent` | Exit 3 |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
