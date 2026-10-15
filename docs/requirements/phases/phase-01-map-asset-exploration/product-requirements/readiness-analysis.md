---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
stage: PRD readiness analysis
status: complete
authored: 2026-09-20
source: ../business-requirements/business-requirements.md (v1.0, baselined)
next: research-prd-context (phase P01)
---

# PRD Readiness Report: P01 - Map Asset Exploration (MAE)

## Phase boundary verification

| Check | Result |
| --- | --- |
| Phase ID, name, acronym, directory match the Intent Brief registry | **Pass** - `P01`, Map Asset Exploration, `MAE`, `docs/requirements/phases/phase-01-map-asset-exploration/` |
| All BRD items belong to P01 included scope | **Pass** - 27 of 27; none reaches into P02-P04 |
| Proposed product requirements can satisfy the registered outcome | **Pass** - the outcome decomposes into FR, NFR, DATA and UX without residue |
| Entry criteria clear | **Pass** - Intent Brief approved, requirements and design validated, licence recorded |
| Exit criteria covered | **Pass** - each restates as a measurable BRD requirement, all of which translate |
| Earlier-phase dependencies | **None** - P01 has no prerequisite phases |
| Any requirement silently moved to another phase | **No** |

No phase identity, boundary, split or merge was created here. The registry is
unchanged.

## BRD extraction summary

- **Total BRD requirements:** 27 (12 P0, 14 P1, 1 P2)
- **Translation to FR:** 14 | **NFR:** 13 | **DATA:** 5 | **UX:** 6
  *(totals exceed 27 because several BRD items translate to more than one
  category)*
- **Requires decomposition:** 6
- **Ambiguous, needs clarification:** 0
- **Not translated to product requirements:** 4 - these are project or process
  obligations rather than product behaviour

## Classification of every BRD requirement

| BRD ID | Pri | Target category | Decompose? | Note |
|:---|:---|:---|:---|:---|
| `BIZ-EXP.1` | P0 | **FR** + NFR | **Yes - major** | "Every corpus member opens" decomposes into archive access, SMF decode, SMT decode, definition evaluation and an error-free path for each. The single largest decomposition in the phase |
| `BIZ-EXP.2` | P0 | **NFR** (performance) + FR (camera) + UX | **Yes** | Traversal time is an NFR threshold; the camera and navigation components that achieve it are FR and UX |
| `BIZ-EXP.3` | P0 | **FR** + DATA | **Yes - major** | Eleven asset classes, each needing its own decode and presentation requirement |
| `BIZ-EXP.4` | P1 | **FR** + DATA | No | Directory input through the same access path |
| `BIZ-EFF.1` | P1 | **FR** + **UX** | **Yes** | Twenty checklist properties, each a distinct derivation or display |
| `BIZ-SAT.1` | P1 | *Not translated* | - | A project activity - recruit a mapper and record feedback. No product behaviour |
| `MARK-DIFF.1` | P0 | **NFR** (dependency) | No | Verified by running with no BAR installation present |
| `MARK-PARITY.1` | P2 | **NFR** (fidelity) + FR | **Yes** | Shader chain reproduction plus the comparison procedure |
| `OPER-WORKFLOW.1` | P0 | **FR** + **DATA** | No | One virtual file system over two backends |
| `OPER-WORKFLOW.2` | P1 | **FR** + **UX** | No | Option selection and re-evaluation |
| `OPER-WORKFLOW.3` | P1 | **NFR** (portability) | No | Windows and Linux |
| `OPER-WORKFLOW.4` | P1 | **UX** + FR | No | The checklist view; its content comes from `BIZ-EFF.1` |
| `OPER-MIGRATE.1` | P1 | **FR** + DATA | No | Legacy `.smd` TDF parsing |
| `OPER-SCALE.1` | P1 | **NFR** (robustness) | No | Absent optional assets, 10-436 file archives |
| `OPER-REPORT.1` | P1 | **FR** (tooling) | No | Repeatable corpus run producing per-map results |
| `OPER-REPORT.2` | P1 | *Partly translated* - **DATA** | No | Establishing the reference set is a project activity; storing and using it is a data requirement |
| `COMP-LICENSE.1` | P0 | *Not translated* | - | Repository and licence-file obligation |
| `COMP-LICENSE.2` | P0 | **NFR** (attribution in builds) | No | Attribution must appear in distributed builds |
| `COMP-CONTENT.1` | P0 | **NFR** (distribution) | No | No map content in the repository or builds |
| `GUARD-ACC.1` | P0 | **NFR** + **FR** + **UX** | **Yes** | The prohibition is an NFR; the unknown-value state needs a data representation and a visible interface treatment |
| `GUARD-REPUTE.1` | P1 | *Not translated* | - | Governs public claims, not product behaviour |
| `GUARD-REPUTE.2` | P1 | **UX** constraint | No | Pass/fail only where a numeric rule exists |
| `GUARD-SCOPE.1` | P0 | *Not translated* | - | Governs future requirement admission |
| `GUARD-SCOPE.2` | P0 | **NFR** (no write path) | No | Verifiable by absence: no map file opened for writing |
| `GUARD-SCOPE.3` | P1 | **NFR** (scope constraint) | No | No gadget execution |
| `GUARD-SCOPE.4` | P1 | *Not translated* | - | Scope boundary only |
| `GUARD-SCOPE.5` | P0 | **NFR** (dependency) | No | Same verification as `MARK-DIFF.1` |

**Ambiguity assessment: none.** Every BRD item has an unambiguous product
translation. This is a direct consequence of the ten open questions being
resolved before baseline - the items that would have been ambiguous
(`BIZ-EXP.2`'s threshold, `BIZ-EXP.3`'s treatment of options and `LuaGaia`,
`BIZ-EFF.1`'s property set) were each settled by a decision record.

## Product personas

### Persona 1: The reviewing mapper *(primary)*

- **Derived from:** BRD stakeholder "BAR map reviewer"
- **Product behaviours:** On-event, not daily. Opens the tool when a map is
  posted for review in the `#mapping` Discord thread - a burst of activity, then
  nothing for days. Two reviewers must sign off each map.
- **Key workflows:**
  1. Open a submitted map from a downloaded `.sd7`.
  2. Fly the terrain looking for texture, splat and topoline problems.
  3. Check the numeric properties - size, wind, tidal, normals resolution,
     splat resolution - against the checklist.
  4. Inspect metal distribution and start positions as overlays.
  5. Report findings back into the Discord thread.
- **Interaction context:** Desktop only. Time-pressured and doing unpaid work;
  the guidance explicitly asks reviewers to be efficient.
- **Technical proficiency:** Power user of map internals, but not of this tool.
  Expects to be productive within minutes, without documentation.
- **JTBD:** *"When a map is submitted for my review, I want to see everything
  the checklist asks about in one place, so that I can give a documented verdict
  without launching the game twice."*

### Persona 2: The authoring mapper *(primary)*

- **Derived from:** BRD stakeholder "BAR map author"
- **Product behaviours:** Iterative and frequent during a build - compile, look,
  adjust, recompile. Wants the look step to cost seconds.
- **Key workflows:**
  1. Open the map currently being built, often as an **unpacked directory**
     rather than an archive.
  2. Fly it to judge terrain shape, lighting and feature placement.
  3. Compare against a reference map they consider good.
  4. Check their own work against the checklist before submitting.
  5. Reopen after recompiling.
- **Interaction context:** Desktop, alongside World Machine, a paint tool and a
  text editor. Exploratory rather than time-pressured, but highly repetitive -
  so reopening must be cheap.
- **Technical proficiency:** Power user. Understands `mapinfo.lua` and will
  notice any value the tool gets wrong.
- **JTBD:** *"When I have just recompiled my map, I want to see it immediately
  and in the state a player would, so that I can judge my last change without
  starting a game."*

### Persona 3: The tooling developer *(secondary)*

- **Derived from:** BRD stakeholder "Tooling and AI developer"
- **Product behaviours:** Occasional, investigative. Arrives with a specific
  doubt about a derived value.
- **Key workflows:** Open a map whose analysis looks wrong; view the grid in
  question - slope, passability, metal - as an overlay; compare against what
  their own tool computed.
- **Interaction context:** Desktop, alongside a terminal and JSON output.
- **Technical proficiency:** Expert; will read the source if the display
  disagrees with them.
- **JTBD:** *"When my terrain analysis produces a result I doubt, I want to see
  the same grid rendered from the same file, so that I can tell whether the map
  or my code is wrong."*

**Persona provenance warning.** All three are reconstructed from published
process documents. `DR-14` defers contact with a real mapper until a runnable
build exists, so no persona here has been validated by a person. `RISK-6`
(Medium-High probability, High impact) governs. Every workflow above should be
treated as a hypothesis, and the PRD should avoid requirements that depend on
fine detail of these workflows being right.

## Fit-gap analysis

| BRD ID | Capability needed | Status | Notes |
|:---|:---|:---|:---|
| `BIZ-EXP.1` | Archive and directory reading, SMF decode, definition evaluation | **Partly existing** | `rjm.bar.tools` provides `.sd7` access, SMF grids, sandboxed `mapinfo.lua`, terrain derivation. Reusable via `DEP-1` |
| `BIZ-EXP.1` | `.smt` tile atlas decode | **New development** | Not decoded today; on the critical path for any rendering |
| `BIZ-EXP.1` | Minimap decode (DXT1, 9 mips, 699048 bytes) | **New development** | Not decoded today |
| `BIZ-EXP.2` | Camera and navigation components | **New development** | Established patterns exist to adopt - see research plan |
| `BIZ-EXP.2` | LOD terrain able to sustain traversal | **Configuration of an existing capability** | Terrain3D clipmap, 10 LOD levels, 64 m to 65.5 km (`DEP-2`) |
| `BIZ-EXP.3` | Terrain rendering | **Configuration + new** | Terrain3D mesh; material override is near-total |
| `BIZ-EXP.3` | Texture decode for all engine slots | **New development** | Godot cannot load DDS at runtime (`MR-TREND-05`), so decoding is owned by the extension |
| `BIZ-EXP.3` | S3O model loading | **New development** | Format fully documented in `rts/Rendering/Models/s3o.h` - tractable |
| `BIZ-EXP.3` | Sky and water presentation | **New development** | Sky in only 34.6% of archives, so absence is the common case |
| `BIZ-EXP.3` | Feature placement from `featureplacer` Lua | **Partly existing** | Lua sandbox exists; placement semantics are new |
| `BIZ-EXP.3` | Map-side Lua as source | **New development, trivial** | Text display only (`DR-9`) |
| `BIZ-EXP.4` | Directory backend | **New development** | Small, once the VFS abstraction exists |
| `BIZ-EFF.1` | 18 map-derived checklist properties | **Partly existing** | Several derive from grids `rjm.bar.tools` already computes |
| `BIZ-EFF.1` | Pathability by movement class; geovent buildability | **New development** | Requires embedded BAR constants with a cited commit (`ASSUM-7`) |
| `MARK-DIFF.1` | Zero engine or game dependency | **New development** | Verified by a clean-machine corpus run |
| `MARK-PARITY.1` | SMF shader chain port | **New development** | Terrain3D `shader_override_enabled` is the vehicle (`ASSUM-2`) |
| `OPER-WORKFLOW.1` | One VFS over archive and directory | **New development** | Extends the existing archive reader |
| `OPER-WORKFLOW.2` | Option picker with re-evaluation | **New development** | Requires re-running the Lua evaluation and refreshing derived state |
| `OPER-WORKFLOW.3` | Windows and Linux builds | **Configuration** | Godot and Rust both cross-compile |
| `OPER-WORKFLOW.4` | Checklist view | **New development** | The one substantial new UX surface in P01 |
| `OPER-MIGRATE.1` | TDF `.smd` parser | **New development** | Small, self-contained |
| `OPER-SCALE.1` | Absent-asset tolerance | **New development** | A cross-cutting behaviour, not a feature |
| `OPER-REPORT.1` | Headless corpus run | **Partly existing** | `rjm.bar.tools` already runs deterministically over the library |
| `OPER-REPORT.2` | Reference set storage and comparison | **New development** | Captures and timings recorded against an engine commit |
| `COMP-LICENSE.2` | Attribution in builds | **Configuration** | - |
| `COMP-CONTENT.1` | No map content shipped | **Configuration** | Repository hygiene |
| `GUARD-ACC.1` | Unknown-value representation and display | **New development** | Cross-cutting: every displayed value needs a derived-or-unknown state |
| `GUARD-REPUTE.2` | Pass/fail restricted to numeric rules | **New development** | A constraint on the checklist view |
| `GUARD-SCOPE.2` | No write path to map files | **Architectural constraint** | Verified by absence |
| `GUARD-SCOPE.3` | No gadget execution | **Architectural constraint** | Verified by absence |
| `GUARD-SCOPE.5` | No engine dependency | **Architectural constraint** | Same verification as `MARK-DIFF.1` |

### Decomposition needs

Six BRD items split into multiple PRD requirements:

1. **`BIZ-EXP.1`** - the read pipeline: VFS, SMF, SMT, minimap, definition
   evaluation, error handling. Largest decomposition in the phase.
2. **`BIZ-EXP.3`** - eleven asset classes, each with decode and presentation.
3. **`BIZ-EFF.1`** - twenty checklist properties, individually derived.
4. **`BIZ-EXP.2`** - the NFR threshold, the camera component, and the
   navigation interaction model.
5. **`MARK-PARITY.1`** - shader port plus the comparison procedure.
6. **`GUARD-ACC.1`** - the prohibition, the data representation, and the visible
   treatment.

## Quick competitive feature scan

Camera and navigation is the one area where established patterns should be
adopted rather than invented, and `BIZ-EXP.2` makes it a P0 concern.

- **Table stakes.** Ground-anchored gestures - the point under the cursor stays
  under the cursor while panning, orbiting and zooming. Drag-pan on left mouse,
  orbit on middle, zoom on wheel, WASD movement with QE rotation, and screen-edge
  panning. These recur across RTS camera implementations, including several for
  Godot specifically.
- **Two movement modes matter at scale.** Walk and fly are distinct, and fly is
  commonly reported as feeling like "slipping on ice" over large terrain -
  directly relevant to a 16384-unit map and to a traversal-time requirement.
- **Differentiation opportunity.** No comparable tool offers checklist-oriented
  inspection. The competitive gap is not in the camera, which should be
  conventional, but in `OPER-WORKFLOW.4`.

Sources: [RTS Camera 3D for Godot](https://store.godotengine.org/asset/alfredbaudisch/rts-camera-3d/),
[Look-From Camera Control for 3D Terrain Maps, CHI 2019](https://dl.acm.org/doi/10.1145/3290605.3300594),
[Blender walk versus fly navigation](https://pixelseekmedia.com/walk-mode-in-blender-wasd-navigation/).
Retrieved 2026-09-20.

## Research plan

| # | Research area | Priority | What to investigate | Serves |
|:--|:---|:---|:---|:---|
| 1 | **UX pattern research - camera and navigation** | **High** | Ground-anchored pan/orbit/zoom, walk versus fly at 16 km scale, speed scaling with altitude, bookmarks or jump-to-position, and what makes corner-to-corner traversal fast without overshoot | `BIZ-EXP.2`, `OPER-WORKFLOW.4` |
| 2 | **UX pattern research - inspection and overlays** | **High** | How comparable tools present alternating data layers (metal, type, slope, pathability) over terrain; toggling, opacity, legends; how unknown values are conventionally shown | `BIZ-EXP.3`, `GUARD-ACC.1`, `BIZ-EFF.1` |
| 3 | **NFR benchmarking** | **High** | Interactive frame-rate floors for terrain tools, acceptable cold-open times for a ~50 MB archive, memory ceilings for a 16384-unit terrain with a 25 MB tile atlas | `BIZ-EXP.2`, `OPER-SCALE.1` |
| 4 | **Technology constraints** | **High** | Terrain3D's data model and shader override in practice; feeding decoded DXT1 to Godot without the import pipeline; Rust-to-C++ GDExtension interop | `ASSUM-2`, `ASSUM-3`, `MARK-PARITY.1` |
| 5 | **Data architecture research** | **Medium** | Representation of a derived-or-unknown value; how the eleven asset classes and twenty checklist properties are modelled; caching a decoded map between reopens for the author's recompile loop | `GUARD-ACC.1`, `BIZ-EFF.1`, persona 2 |
| 6 | **Competitive feature analysis** | **Low** | Largely complete from the BRD Source Bundle. Remaining: what BARMapEdit implements | `MARK-DIFF.1` |

## Open items

**Ambiguous BRD items needing stakeholder clarification:** none. All ten open
questions were resolved before baseline.

**BRD open questions constraining PRD scope:** none outstanding.

**Constraints carried into the PRD from baselined decisions:**

- `DR-14` - the personas are unvalidated, and will remain so through the PRD and
  TDD. Avoid requirements that hinge on unverified workflow detail.
- `ASSUM-7` - embedded BAR constants must carry a cited commit, and the PRD must
  make their provenance visible rather than implicit.
- `ASSUM-2` and `ASSUM-3` - Terrain3D shader override and Rust-to-C++ interop
  are unvalidated; research area 4 should settle both before the technical
  design, as `DEP-2` requires.
- `GUARD-SCOPE.2` and `DR-8` - the PRD must state precisely which non-map writes
  are permitted, since a write path is otherwise prohibited.

**Items deliberately not translated to product requirements:** `BIZ-SAT.1`,
`COMP-LICENSE.1`, `GUARD-REPUTE.1`, `GUARD-SCOPE.1` and `GUARD-SCOPE.4` are
project or governance obligations. They remain binding and are tracked in the
BRD; they simply do not describe product behaviour.

## Quality gates

| Gate | Status |
| --- | --- |
| Every BRD requirement classified by PRD target category | Met - 27 of 27, with four explicitly not translated and justified |
| BRD personas enriched into product personas with behaviours and JTBD | Met - 3 personas, each with workflows, context, proficiency and JTBD |
| Fit-gap analysis complete | Met - 31 capability rows, each with a status |
| Decomposition needs identified | Met - 6 items, two of them major |
| Selected phase matches the intent brief registry exactly | Met |
| No new phase identity or boundary invented | Met |
| Research plan specifies priority areas from the gap analysis | Met - 6 areas, 4 High |
| Ambiguous items flagged | Met - none exist; the reason is recorded |
