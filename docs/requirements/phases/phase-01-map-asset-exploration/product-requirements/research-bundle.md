---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
stage: PRD context research
status: complete
gathered: 2026-09-20
source_plan: readiness-analysis.md
next: draft-prd (phase P01)
---

# PRD Research Bundle: P01 - Map Asset Exploration (MAE)

Feature-level research - *how to build*, not *what market to enter*. Market
positioning is in the BRD Source Bundle and is not repeated here. All findings
gathered 2026-09-20; observations and inferences are distinguished, and access
limitations are stated.

## Key findings summary

1. **Camera should be conventional, not invented.** Ground-anchored
   pan/orbit/zoom with WASD and QE is a settled pattern with existing Godot
   implementations. The differentiation is in `OPER-WORKFLOW.4`, not here.
2. **Fly mode at 16 km scale is a known usability trap.** Practitioners
   describe unconstrained fly as "slipping on ice"; altitude-proportional speed
   is the standard mitigation and is directly relevant to `BIZ-EXP.2`.
3. **Rust-to-Terrain3D is the weakest technical link.** `gdext` has **no
   type-safe API for classes defined in other GDExtensions** (open issue #372).
   Access would be dynamic and unchecked. A GDScript glue layer is the
   lower-risk alternative and should be weighed in the TDD.
4. **Terrain3D's import path is an editor tool, not a runtime API.** Its
   documented heightmap flow expects EXR or R16/RAW through an importer scene.
   Feeding a decoded BAR heightmap at runtime is not a documented path.
   `ASSUM-2` is less safe than it looked.
5. **Whether Godot accepts pre-compressed DXT1 blocks at runtime is
   unresolved** and could not be settled from documentation. The fallback -
   decoding to RGBA in Rust - is certain but costs memory. This needs a spike,
   not a requirement.

## Competitive feature analysis

Feature coverage across comparable tools. Per the readiness plan this area was
Low priority, since the BRD Source Bundle already covers positioning; what
follows is the feature-level view only.

### Feature: external inspection of a compiled map

**BRD source:** `BIZ-EXP.1`, `BIZ-EXP.3`, `MARK-DIFF.1`

| Competitor | Implementation | Strengths | Weaknesses | Source |
|:---|:---|:---|:---|:---|
| SpringBoard | In-engine editing and inspection | Full engine fidelity by construction; object placement; scenario tools | Requires the engine; cannot serialise a full map; live-texture editing unsolved | `MR-COMP-01` |
| PyMapConv | Decompiles `.smf` to loose image files | Exact; scriptable; recommended by the official guide | No view; the user assembles a mental model from separate images | `MR-COMP-02` |
| BARMapEdit | Unity-based external editor | Same architectural bet as this project | **Access limitation:** repository documents neither format support nor scope; capability could not be assessed without reading the source. 0 stars, 0 forks, no releases | `MR-COMP-04` |
| bar-map-generator | Generates assets, delegates compilation to PyMapConv | Browser-based; full pipeline | Generates rather than inspects | `MR-COMP-05` |

**Differentiation opportunity.** No tool presents the twenty checklist
properties together. Every competitor either shows the map without the
properties (SpringBoard, in-engine) or the properties without the map
(PyMapConv, loose images).

**Recommended approach.** Conventional 3D inspection plus a checklist view that
no competitor offers. `PR-FEAT-01`.

## UX pattern research

### Workflow: navigating a 16,384-unit map

**Personas:** reviewing mapper, authoring mapper, tooling developer
**BRD source:** `BIZ-EXP.2`

**Established patterns** (`PR-UX-01`):

- **Ground-anchored gestures.** The point under the cursor stays under the
  cursor while panning, orbiting and zooming. Recurs across RTS camera
  implementations, including Godot-specific ones.
- **Conventional bindings.** Drag-pan on left mouse; orbit on middle mouse, with
  horizontal drag changing heading and vertical drag changing pitch; wheel to
  zoom, zoom-to-cursor; WASD movement; QE rotation; screen-edge panning.
- **Walk versus fly as distinct modes.** Practitioner reports describe
  unconstrained fly over large terrain as feeling like "slipping on ice", with
  walk preferred for controlled inspection (`PR-UX-02`).
- **Look-from camera control** (CHI 2019) lets a viewer transition quickly
  between top-down, oblique and ground-level views of terrain - the exact
  transition a reviewer makes when moving from "is the metal distribution sane"
  to "are the topolines subtle" (`PR-UX-03`).

**Inference (not observed for this domain):** altitude-proportional movement
speed is the standard reconciliation of "cross 23,170 units quickly" with
"inspect a 50-unit feature". Confidence: high, by analogy with mapping and DCC
tools; no BAR-specific source found.

**Error and edge-case patterns** (`PR-UX-04`):

- **Empty and absent states are the common case here**, not an exception:
  65.4% of archives have no skybox, 23.6% no features, and some lack normal,
  specular or splat textures (`MR-CORP-06` to `MR-CORP-08`). The interface must
  treat absence as normal and say so, rather than rendering a broken-looking
  scene.
- **Progressive loading.** A ~50 MB archive with a 25 MB tile atlas cannot open
  instantly; staged presentation - terrain first, textures as decoded - is the
  conventional handling.
- **Unknown-value display.** No domain-specific convention was found for
  marking a value as underivable. This is a genuine design gap that
  `GUARD-ACC.1` forces the project to solve; see Data Gaps.

**Accessibility** (`PR-UX-05`). No `COMP-*` requirement in the BRD mandates a
conformance level, and no statutory regime applies to a volunteer desktop tool.
WCAG 2.2 AA is the usual reference point but is written for web content; the
applicable subset for a desktop 3D tool is keyboard operability, non-colour-only
encoding of information, and sufficient contrast in the checklist view. Treated
as a quality target, not a compliance obligation. **Inference, flagged as such.**

### Workflow: verifying a map against the checklist

**Personas:** reviewing mapper (primary), authoring mapper
**BRD source:** `BIZ-EFF.1`, `OPER-WORKFLOW.4`, `GUARD-REPUTE.2`

No comparable tool exists, so no pattern could be observed in this domain
(**access limitation: none exists to observe**). Transferable patterns from
adjacent tooling (`PR-UX-06`):

- A **checklist or inspector panel** listing every property with its current
  value, where the list is fixed and the values change per subject - the
  pattern used by linters and validation panels generally.
- **Three states, not two**: pass, fail, and *reported value with no
  automatic judgement*. `GUARD-REPUTE.2` requires the third state for the
  fifteen properties the checklist does not govern numerically.
- **Deep-linking from the panel to the thing**: selecting a property moves the
  camera or toggles the relevant overlay. This is the interaction that would
  make the panel worth using rather than a table someone reads once.

**Inference:** the deep-link behaviour is the highest-value part of the panel
for the reviewing persona. Unvalidated - no reviewer has been consulted
(`RISK-6`).

### Workflow: reopening after a recompile

**Personas:** authoring mapper
**BRD source:** `BIZ-EXP.4`, persona 2

The authoring loop is compile, look, adjust, recompile. The look step must be
cheap **repeatedly**, which makes reopen cost, not first-open cost, the metric
that matters for this persona. Standard handling is to watch the source for
change and offer reload, or to cache decoded data keyed by file modification
time. `PR-UX-07`. **Inference from the stated workflow; unvalidated.**

## NFR benchmarking

### Performance

**BRD source:** `BIZ-EXP.2`, `OPER-SCALE.1`

| Metric | Industry benchmark | Recommended target | Rationale | Key |
|:---|:---|:---|:---|:---|
| Interactive frame rate | 60 fps is the desktop-tool norm; 30 fps the floor at which camera control degrades noticeably | **60 fps sustained**, 30 fps hard floor, while traversing any corpus map | Below 30, corner-to-corner traversal becomes imprecise and `BIZ-EXP.2` is unachievable in practice | `PR-NFR-01` |
| Corner-to-corner traversal | **Set by `DR-7`: equal to or faster than Beyond All Reason** at the pinned engine commit | Per reference-set map | Externally anchored; no invented figure | `PR-NFR-02` |
| Time to first navigable frame | No domain benchmark found | Staged: terrain navigable before textures complete | A ~50 MB archive with a 25 MB atlas cannot open instantly; the alternative to staging is a long blocking wait | `PR-NFR-03` |
| Memory ceiling | Not benchmarked for this domain | Must hold a 2048x2048 heightmap, a 25 MB tile atlas and up to ten texture slots | If DXT1 must be decoded to RGBA (see Data Gaps), the atlas expands roughly 6-8x, which is the dominant memory risk | `PR-NFR-04` |

### Availability, security, scalability

**Not applicable, and the reason is structural.** P01 is a local desktop
application with no server, no network dependency, no authentication and no user
data. `MARK-DIFF.1` and `GUARD-SCOPE.5` make independence a requirement rather
than an omission. The BRD's only `COMP-*` items are licensing and content
redistribution, neither of which has a technical threshold. Recording this
explicitly so the PRD does not invent NFRs the phase does not need.

### Portability

**BRD source:** `OPER-WORKFLOW.3`

Windows and Linux. Terrain3D requires **Godot 4.4+**; both Godot and Rust
cross-compile to both targets. `PR-NFR-05`.

## Data architecture research

**BRD source:** `GUARD-ACC.1`, `BIZ-EFF.1`, `BIZ-EXP.3`

| # | Finding | Implication |
|:--|:---|:---|
| `PR-DATA-01` | **Every displayed value needs three states**: derived (with the engine rule it came from), unknown (underivable, per `DR-5`), or absent (the map does not contain it). Absence and unknown are different: a map with no skybox is complete, whereas an unparseable value is not | The data model carries provenance alongside every value. This is the single most pervasive structural consequence in P01 |
| `PR-DATA-02` | Data sources are the archive and nothing else, except the embedded BAR constants for two checklist properties (`ASSUM-7`), which must carry a cited commit | No network, no external repository (`DR-16`) |
| `PR-DATA-03` | Freshness: a map file is static once opened. The only freshness concern is the authoring persona's recompile loop, which changes the file underneath an open view | Watch-and-reload, or reopen, rather than any caching-invalidation scheme |
| `PR-DATA-04` | Data volumes: archives 10-436 files, mean 96; up to ~50 MB with a 25 MB `.smt`; heightmap `u16[(mapx+1)*(mapy+1)]`, which for a 32x32 map is 2049x2049 | Sizes are bounded and known; no growth projection is required |
| `PR-DATA-05` | Privacy: **no personal data of any kind**. Map files are third-party creative content governed by `COMP-CONTENT.1`, not by privacy law | No retention, consent or pseudonymisation requirements apply |

## Technology constraints

| # | Constraint | Assessment | Source |
|:--|:---|:---|:---|
| `PR-TECH-01` | **`gdext` has no type-safe API for classes defined in other GDExtensions.** Open issue #372 requests exactly this. Access to Terrain3D from Rust would be dynamic and unverified at compile time | **Material risk to `ASSUM-3`.** A GDScript glue layer between the Rust decoder and Terrain3D is the lower-risk alternative and should be evaluated in the TDD | [gdext issue #372](https://github.com/godot-rust/gdext/issues/372), 2026-09-20 |
| `PR-TECH-02` | **Terrain3D's documented heightmap path is an editor importer** (`addons/terrain_3d/tools/importer.tscn`), expecting EXR or R16/RAW. Runtime injection of decoded data is not a documented flow | **Weakens `ASSUM-2`.** The spike must confirm that terrain data can be supplied programmatically at runtime, not only through the editor | [Terrain3D import/export docs](https://terrain3d.readthedocs.io/en/stable/docs/import_export.html), 2026-09-20 |
| `PR-TECH-03` | Terrain3D `shader_override_enabled` fully replaces the generated shader, emitting the generated one as a starting point when enabled blank | Supports `MARK-PARITY.1`; the material model is 32 splatted texture sets, so the override will be near-total | [Terrain3DMaterial docs](https://terrain3d.readthedocs.io/en/stable/api/class_terrain3dmaterial.html), 2026-09-20 |
| `PR-TECH-04` | **Godot does not load DDS at runtime**; DDS becomes CTEX at import | Confirms the extension must own texture decoding. Already accepted in the BRD | `MR-TREND-05` |
| `PR-TECH-05` | Whether `Image` accepts **pre-compressed DXT1 block data** at runtime could not be confirmed from documentation | See Data Gaps. Fallback is RGBA decode in Rust, which is certain but multiplies atlas memory | Godot Image class docs, 2026-09-20 |
| `PR-TECH-06` | `gdext` ships occasional breaking changes with migration guides | Pin a version; treat upgrades as deliberate work (`DEP-3`) | `MR-TREND-04` |
| `PR-TECH-07` | Vendor lock-in: Terrain3D is MIT and replaceable by a custom clipmap at known cost; Godot itself is the deeper commitment | Acceptable. `DR-6` weighed and accepted the dependency | `MR-COMP-06` |

## Data gaps

1. **Runtime compressed-texture support in Godot.** Whether `Image` can be
   constructed from pre-compressed DXT1 blocks, or whether tiles must be
   decoded to RGBA first, was not resolvable from documentation.
   **Recommendation:** a spike, before the technical design fixes the texture
   pipeline. This determines `PR-NFR-04`'s memory profile.
2. **Terrain3D runtime data injection.** Documentation covers the editor
   importer, not programmatic supply. **Recommendation:** fold into the
   `ASSUM-2` spike; read the Terrain3D source rather than relying on docs.
3. **Rust-to-Terrain3D interop in practice.** Issue #372 establishes the
   absence of a typed API but not how painful the dynamic path is.
   **Recommendation:** include a GDScript-glue alternative in the `ASSUM-3`
   spike.
4. **No unknown-value display convention exists** in this or an adjacent
   domain. **Recommendation:** treat as an original design problem in the PRD;
   do not cite a pattern that was not found.
5. **No validated user research.** Every UX inference here rests on published
   process documents, per `DR-14` and `RISK-6`. **Recommendation:** revisit the
   checklist-panel interaction design once the early user is recruited.
6. **BARMapEdit capability unassessed** - documentation absent; would require
   reading its source.

## Inferences

| # | Inference | Basis | Confidence |
|:---|:---|:---|:---|
| INF-P1 | Altitude-proportional camera speed is necessary to satisfy `BIZ-EXP.2` without ruining close inspection | Standard in mapping and DCC tools; the 23,170-unit diagonal versus feature-scale detail | High |
| INF-P2 | Deep-linking from checklist property to camera position or overlay is the panel's highest-value behaviour | Derived from the reviewing persona's workflow | Medium - persona unvalidated |
| INF-P3 | Reopen cost matters more than first-open cost for the authoring persona | Derived from the stated compile-look-adjust loop | Medium - persona unvalidated |
| INF-P4 | A GDScript glue layer will prove less painful than dynamic Rust-to-Terrain3D calls | `PR-TECH-01`; no typed API exists, and GDScript's dynamic dispatch is idiomatic rather than a workaround | Medium |
| INF-P5 | Staged loading is required rather than optional | `PR-NFR-03`; archive sizes make a single blocking decode user-hostile | High |

## Reference key index

| Key | Area | Subject | Source | Date |
|:---|:---|:---|:---|:---|
| PR-FEAT-01 | Feature | External map inspection across comparable tools | BRD Source Bundle `MR-COMP-01` to `-05` | 2026-09-20 |
| PR-UX-01 | UX | Ground-anchored RTS camera conventions | [RTS Camera 3D for Godot](https://store.godotengine.org/asset/alfredbaudisch/rts-camera-3d/) | 2026-09-20 |
| PR-UX-02 | UX | Walk versus fly over large terrain | [Blender walk-mode navigation](https://pixelseekmedia.com/walk-mode-in-blender-wasd-navigation/) | 2026-09-20 |
| PR-UX-03 | UX | Look-from camera control for terrain | [CHI 2019](https://dl.acm.org/doi/10.1145/3290605.3300594) | 2026-09-20 |
| PR-UX-04 | UX | Absent-asset and staged-loading handling | `MR-CORP-06` to `-08` | 2026-09-20 |
| PR-UX-05 | UX | Accessibility target for a desktop 3D tool | WCAG 2.2 AA, applied as a quality target | 2026-09-20 |
| PR-UX-06 | UX | Checklist and inspector panel patterns | Transferable; no domain instance exists | 2026-09-20 |
| PR-UX-07 | UX | Reopen-after-recompile handling | Derived from persona 2 | 2026-09-20 |
| PR-NFR-01 | NFR | Interactive frame-rate floor | Desktop-tool norm | 2026-09-20 |
| PR-NFR-02 | NFR | Traversal target | `DR-7` | 2026-09-20 |
| PR-NFR-03 | NFR | Staged time-to-first-frame | Derived from archive sizes | 2026-09-20 |
| PR-NFR-04 | NFR | Memory profile | Derived from `MR-CORP` measurements | 2026-09-20 |
| PR-NFR-05 | NFR | Platform targets | Terrain3D requires Godot 4.4+ | 2026-09-20 |
| PR-DATA-01..05 | Data | Value provenance, sources, freshness, volumes, privacy | This analysis | 2026-09-20 |
| PR-TECH-01 | Tech | No typed cross-GDExtension API in gdext | [gdext #372](https://github.com/godot-rust/gdext/issues/372) | 2026-09-20 |
| PR-TECH-02 | Tech | Terrain3D import is an editor path | [Terrain3D docs](https://terrain3d.readthedocs.io/en/stable/docs/import_export.html) | 2026-09-20 |
| PR-TECH-03 | Tech | Shader override replaces the terrain shader | [Terrain3DMaterial docs](https://terrain3d.readthedocs.io/en/stable/api/class_terrain3dmaterial.html) | 2026-09-20 |
| PR-TECH-04 | Tech | No runtime DDS loading in Godot | `MR-TREND-05` | 2026-09-20 |
| PR-TECH-05 | Tech | Runtime compressed-texture support unconfirmed | Godot Image docs | 2026-09-20 |
| PR-TECH-06 | Tech | gdext breaking changes | `MR-TREND-04` | 2026-09-20 |
| PR-TECH-07 | Tech | Dependency lock-in assessment | `MR-COMP-06` | 2026-09-20 |

## Quality gates

| Gate | Status |
| --- | --- |
| All research areas from the plan investigated | Met - all six, in priority order |
| Every finding has a reference key and cited source | Met |
| Competitive feature analysis covers at least 3 competitors per feature area | Met - four, with one access limitation stated |
| NFR benchmarks include quantified thresholds with sources | Met - frame rate, traversal, staging, memory, platform. Availability and security recorded as structurally N/A rather than invented |
| UX patterns reference established standards, not just competitor observation | Met - CHI 2019 paper, Godot camera implementations, practitioner reports; one area has no existing instance and says so |
| Data gaps documented honestly | Met - six, each with a recommendation |
| Facts and inferences clearly distinguished | Met - five inferences with confidence ratings |
