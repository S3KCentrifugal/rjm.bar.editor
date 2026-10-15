# P01-MAE-DEVOPS-001 - Scaffold the workspace and record the licence

## Description

Everything else in this phase lands inside a workspace that does not exist yet. This task creates the four
owned units the technical design names - `mae-core`, `mae-gdext`, `mae-app` and `mae-conform` - with the dependency
boundary that makes several of the phase's guarantees structural rather than aspirational: `mae-core` must not be able
to name a Godot type, and that is enforced by its manifest, not by review. It also records the GPL v2-or-later licence
that `COMP-LICENSE.1` makes a phase entry criterion, and pins the external versions the project depends on. Getting the
dependency direction right here is what later makes the clean-machine test and the headless conformance run possible at
all.

## Metadata

- **Phase:** [P01 - Map Asset Exploration](../../../intent/intent-brief.md#p01---map-asset-exploration)
- **Type:** `DEVOPS`
- **Status:** `Todo`
- **Priority:** `P0`
- **Estimate:** `M`
- **Parent Deliverable / User Story:** [NFR-PORT.1](../product-requirements/product-requirements.md#3-non-functional-requirements)

## Relationships

### Prerequisites

- None

### Dependents

- [P01-MAE-DATA-001 - Implement the provenance value model](P01-MAE-DATA-001-implement-provenance-value-model.md)
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
- [Developer tooling conventions](~/.claude/skills/convention-developer-tooling/SKILL.md)
- [Deployment and operations conventions](~/.claude/skills/convention-deployment/SKILL.md)

## Traceability

- **Business Requirements:** `COMP-LICENSE.1`, `COMP-LICENSE.2`, `COMP-CONTENT.1`, `MARK-DIFF.1`
- **Product Requirements:** `NFR-LIC.1`, `NFR-LIC.2`, `NFR-DEP.1`, `NFR-PORT.1`
- **Technical Design:** section 3.2, section 6.1, ADR-13
- **Phase Outcome / Exit Criteria:** Satisfies the phase entry criterion that the licence decision is recorded

## Scope

- Cargo workspace with `mae-core`, `mae-gdext` and `mae-conform`
- Godot 4.4+ project for `mae-app` with the extension wired in
- Licence file, dependency attribution, and pinned external versions

## Non-Goals

- Any decoding, rendering or UI behaviour
- CI pipeline configuration, which is P01-MAE-DEVOPS-002

## Expected Artifacts

| Path | Action | Purpose |
|:-----|:-------|:--------|
| `Cargo.toml` | Create | Workspace manifest |
| `crates/mae-core/Cargo.toml` | Create | Core library; must not depend on Godot |
| `crates/mae-gdext/Cargo.toml` | Create | Extension crate; depends on gdext and mae-core |
| `crates/mae-conform/Cargo.toml` | Create | CLI binary; depends on mae-core only |
| `app/project.godot` | Create | Godot project |
| `LICENSE` | Create | GPL v2-or-later |
| `THIRD-PARTY.md` | Create | Attribution for MIT dependencies |
| `versions.toml` | Create | Pinned gdext, Terrain3D and minimum Godot versions with the date each was set |

## Implementation Requirements

1. `mae-core` must declare no dependency on `godot`, `gdext` or any Godot binding, directly or transitively. This is system invariant 1 and is the foundation of the thread-safety argument.
2. `mae-conform` must depend on `mae-core` only, so that it links and runs with no Godot present.
3. Record GPL v2-or-later in `LICENSE`, because porting Recoil's shaders makes this work a derivative and `COMP-LICENSE.1` makes the record a phase entry criterion.
4. Record attribution for Terrain3D and Godot, both MIT, in `THIRD-PARTY.md`.
5. Pin `gdext` to an exact release and Terrain3D to a release tag in `versions.toml`, each with the date it was set, per technical design section 5.5.
6. The repository must contain no map files at any point, per `COMP-CONTENT.1`.

## DevOps Details

The workspace layout is:

```
crates/mae-core/      Rust library - no Godot dependency, ever
crates/mae-gdext/     Rust cdylib - the only crate that knows about Godot
crates/mae-conform/   Rust binary - links mae-core, never Godot
app/                  Godot 4.4+ project, loads mae-gdext and Terrain3D
spikes/               Throwaway prototypes, retained for evidence
```

The dependency direction is the point of this task. If `mae-core` ever gains a Godot dependency, the clean-machine
test for `NFR-DEP.1` becomes impossible, `mae-conform` stops being runnable headlessly, and worker threads gain the
ability to touch Godot objects. A CI check for this is added in P01-MAE-QUALITY-005.

## Acceptance Criteria

- [ ] **P01-MAE-DEVOPS-001-AC01:** `cargo build` succeeds for all three crates
- [ ] **P01-MAE-DEVOPS-001-AC02:** `mae-core` has no direct or transitive dependency on any Godot binding
- [ ] **P01-MAE-DEVOPS-001-AC03:** `mae-conform` builds and runs on a machine with no Godot installed
- [ ] **P01-MAE-DEVOPS-001-AC04:** `LICENSE` records GPL v2-or-later and `THIRD-PARTY.md` attributes Terrain3D and Godot
- [ ] **P01-MAE-DEVOPS-001-AC05:** `versions.toml` pins gdext, Terrain3D and the minimum Godot version, each with a date

## Verification

| Acceptance Criterion | Command / Test | Expected Result |
|:---------------------|:---------------|:----------------|
| P01-MAE-DEVOPS-001-AC01 | `cargo build --workspace` | All crates compile |
| P01-MAE-DEVOPS-001-AC02 | `cargo tree -p mae-core | grep -i godot` | No matches |
| P01-MAE-DEVOPS-001-AC03 | `cargo run -p mae-conform -- --version` on a machine without Godot | Version printed, exit code 0 |
| P01-MAE-DEVOPS-001-AC04 | Inspect `LICENSE` and `THIRD-PARTY.md` | GPL v2-or-later present; Terrain3D and Godot attributed |
| P01-MAE-DEVOPS-001-AC05 | Inspect `versions.toml` | Three pins present, each dated |

## Completion Evidence

- **Pull Request:** pending
- **Commit:** pending
- **Tests:** pending
- **Deployment:** N/A
- **Migration:** N/A
