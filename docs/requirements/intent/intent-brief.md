---
stage: 0.7 (intent brief)
status: approved
authored: 2026-09-20T21:20:00Z
source_record: docs/requirements/intent/intent-record.md (stages 0.1-0.6, converged)
assessment_report: docs/requirements/intent/framework-assessment.md
verified_against:
  - RecoilEngine 92efda5e60 (2026-09-13)
  - Beyond All Reason 1d267c20d1 (2026-09-13)
  - rjm.bar.tools (working tree)
  - Local map library, 254 .sd7 archives plus 1 unpacked directory (%LOCALAPPDATA%\Programs\Beyond-All-Reason\data\maps)
  - BAR Map Checklist and Map Review Process guides (beyondallreason.info, retrieved 2026-09-20)
research_currency: "All citations gathered 2026-09-20; none stale at time of writing"
next: analyze-prd-readiness (phase P01)
open_questions_resolved: 2026-09-20
---

# Intent Brief: BAR Map Editor (`rjm.bar.editor`)

## 1. Vision Statement

Beyond All Reason's map toolchain runs one way: authors compile images into a
binary map, and the only routine way to see the result is to launch the game.
This project is a Godot-based map editor that opens any BAR map - archived or
unpacked - and shows every asset it contains, faithfully enough to trust, with
camera and navigation built for moving across a large map quickly. Mappers get
a way to see their own work, and reviewers get a way to check it, without
starting the game; and the project earns the foundation it needs to edit map
layers and terrain later.

## 2. Origin & Motivation

- **Input Type:** Seed
- **Intent Archetype:** Product
- **Domain:** Game development tooling / content authoring; Real-time strategy
  (Beyond All Reason, Recoil engine); 3D geospatial and terrain visualisation
- **Motivation Profile:** Motivation is **integrated regulation shading into
  intrinsic** (PF-03) - the work is congruent with self-concept rather than
  instrumental, evidenced by five repositories in one domain each built well
  beyond immediate need. Autonomy and Competence are very high; Relatedness was
  the weakest need and is now addressed by recruiting an early user. The primary
  meaning pathway is **Creative** (PF-05): building a work that did not exist.
  Personality fits the task - high Openness with very high, orderliness-weighted
  Conscientiousness (PF-01) is close to optimal for long, exacting format work,
  with the caveat that the same orderliness resists declaring an unbounded
  quality target complete.

## 3. Target Audience

**Primary persona - The BAR mapper.** Makes maps for Beyond All Reason, or
reviews them. Works through World Machine or a paint tool, compiles with
Beherith's SMF compiler, places objects in SpringBoard, packs with 7-Zip. To
see the result they must launch the game. If they review, they must verify a
written checklist - pathability by movement class, metal distribution, normals
resolution, sun angle, wind and tidal ranges, feature reclaim values - as an
unpaid volunteer, and every map needs two of them.

What they need: to open a map they know and move around it immediately, and to
trust that what they see is what the game would show. Their first judgement will
be made flying over terrain they built themselves, and it will be instant.

**Secondary stakeholders**

- **The user**, as a direct beneficiary and sole maintainer.
- **AI and tooling developers**, verifying derived terrain analysis - slope,
  passability, metal spots, start-position classification - against the map it
  came from. Concretely `rjm.bar.tools`, `rjm.bar.ai` and `s3k-CircuitAI`.
- **BAR players**, who receive better-reviewed maps.
- **The BAR maps team**, and the `maps-metadata` and `map_blueprint`
  repositories the review process assumes.
- **Recoil engine maintainers**, whose shader chain becomes the parity target.

**Potential resistors**

- **SpringBoard and its maintainers** - functional overlap at the edges.
- **Authors invested in the existing workflow**, for whom another tool is
  another thing to learn.
- **Anyone who would treat a drifting non-engine renderer as authoritative.**
  A parity claim that silently diverges is worse than no claim, because
  reviewers would be approving maps against a fiction.

## 4. Value Proposition

**Problem.** Seeing what a BAR map contains requires launching the game. The
entire recommended toolchain - World Machine, Photoshop or GIMP, Beherith's SMF
compiler, SpringBoard, PyMapConv, a pathability-checker script, 7-Zip -
authors, converts or packs; none of it shows a finished map. Reviewers must
verify a published checklist of properties with no tool that surfaces them, and
in-engine editing is structurally constrained: Spring saves only a delta
heightmap rather than a full map, and texture editing against live GL textures
is documented as unsolved. Derived terrain analysis has no visual verification
path at all.

**Solution direction.** A standalone editor that reads BAR maps directly and
shows them - terrain, textures, features, sky, water, placements and map-side
code - with movement fast enough to explore a 32x32 map comfortably. It reads
the file rather than the engine's memory, so it inherits none of the engine's
serialisation limits, and it reproduces engine rules rather than approximating
them, so what it shows can be trusted. Editing of layers and terrain follows,
on a foundation that already reads everything correctly.

**Impact.**

- A mapper sees their compiled map without launching the game - the one-way
  chain becomes a loop.
- A reviewer can inspect a submitted map directly, reducing the cost of a
  volunteer review performed twice per map.
- A tooling developer can confirm derived terrain facts against the source.
- Measurable at v1: **every one of the 254 installed map archives opens and is
  navigable.**

## 5. Framework Assessment Summary

**Ikigai alignment**

| Dimension | Score | Rationale |
| --- | --- | --- |
| Love | **Strong** | Unprompted, sustained investment across five repositories in this exact domain; a stated long horizon |
| Skill | **Strong, with a located weak spot** | Strong in binary format decoding, Rust, sandboxed Lua, systems architecture, engine-source reading; weak or unknown in Godot 4, `gdext`, GLSL porting, LOD |
| Need | **Strong - externally validated** | Two volunteer reviewers per map against a written checklist; a toolchain that only authors and packs; no external viewer; 254 archives installed |
| Viability | **Moderate, resolved as commons** | No revenue intent; resolved in refinement as GPL v2-or-later, public. Benchmark: Tiled, far more broadly useful, sustains ~200 sponsors at roughly $3,600/month funding two days a week |

Intersections: Love + Skill = **Passion**; Love + Need = **Mission**. Profile:
*Mission with a Passion core*.

**Value dimensions:** Physical 2/5, Emotional 3/5, Intellectual 5/5,
Spiritual 4/5.

**Composite profile:** Intellectual-dominant, Spiritual-strong, moderate
Emotional, peripheral Physical. Emotional was raised in refinement from
incidental to load-bearing when community adoption became the three-year
success condition.

**Maslow level.** For the user: Cognitive and Self-Actualization - mastery of a
difficult domain and authentic expression of a standard. For the audience:
Esteem and Cognitive - authors seeing and being able to judge their own work,
reviewers exercising competence without unnecessary friction.

**Motivation type (SDT):** Integrated regulation shading into intrinsic.
Sustainability signal is strong, which matters because no revenue path exists in
this niche - and the domain's own history supports it: PyMapConv and SpringBoard
are both still maintained inside an ecosystem running on community energy since
Spring's GPL release in 2005.

**Key strengths**

- Motivation is structurally sustainable without a business model (PF-03).
- Need is externally validated by the community's own process, not assumed
  (PF-04 Need).
- Three strong Ikigai legs - Love, Skill, Need (PF-04).
- The goal is flow-shaped: "all 254 maps open and are navigable" is binary,
  externally verifiable and immune to self-grading (PF-07 Clear Goals).
- The assignment is genuinely unique (PF-05): an exact decoder, a
  provenance-graded knowledge base, engine fluency and downstream consumers are
  held by one person. The work is not interchangeable.
- Signature strengths are load-bearing, not incidental (PF-08): Honesty and
  Judgment are exactly what a reference renderer's claim rests on.
- Eudaimonic profile strong on five of six dimensions (PF-06).

**Attention areas**

| Area | Framework | Mitigation |
| --- | --- | --- |
| The fidelity bar sits in the weakest skill quadrant | PF-04 Skill, PF-07 Challenge-Skill | **Mitigated in refinement.** Parity ranked last in the priority stack and designated the first sacrifice; strict dependency order puts the decoder - the strongest quadrant - first |
| Feedback desert before first pixel | PF-07 Immediate Feedback | **Accepted knowingly.** Strict dependency order chosen with the cost understood; mitigation deferred to task ordering in the TDD |
| Very high orderliness against an unbounded target | PF-01, PF-06 | **Mitigated.** The priority stack gives P01 a floor that does not depend on parity landing |
| Relatedness, Justice virtues and Emotional value thin at the same point | PF-03, PF-08, Value Dimensions | **Mitigated.** An early user is to be recruited; community adoption is the three-year success condition |
| Viability undefined | PF-04 | **Resolved.** Community commons - GPL v2-or-later, public from the start |
| Reviewers may need a checklist panel, not a viewer | Value Dimensions Emotional, PF-03 | **Deferred** to the P01 product requirements |

## 6. Boundaries

### Priority stack

The ordering that governs every downstream trade-off. Established in refinement
cycle 2.

| Rank | Commitment | Status |
| --- | --- | --- |
| 1 | Every installed map opens and is navigable | Protected |
| 2 | Every asset class is represented | Protected |
| 3 | Exploration is fast - camera and navigation | Protected |
| 4 | Visual parity with Recoil | Target, and the first thing cut |

### Non-goals

1. **Not a general-purpose tool.** It is purpose-built for Recoil. Not a generic
   terrain or heightmap editor, not a Godot terrain plugin, not multi-engine.
   *Rationale: user's own words; generality is not a virtue here.*
2. **It does not guess.** Where an engine rule governs a value, that rule is
   reproduced with a citation. A value that cannot be derived exactly is
   reported as unknown rather than approximated. *Rationale: reviewers would
   otherwise approve maps against a plausible-looking fiction.*
3. **Not a scenario or gadget editor.** Terrain and map layers only; triggers,
   units and game logic remain SpringBoard's ground. *Retained as a **P01 scope
   boundary**, not a permanent project anti-goal - the stakeholder confirmed
   only non-goal 2 of the three derived ones (`DR-12`).*
4. **Not a map generator.** It reads, and later edits, maps that exist.
   *Likewise a P01 scope boundary rather than a permanent position (`DR-12`).*
5. **Not dependent on a running engine.** It reads files; it never launches
   Recoil and never requires the game to inspect a map.
6. **P01 does not edit.** Reading is the deliverable, not a stepping stone.

### Constraints

**Technical**

- `mapinfo.lua` is an executable Lua 5.1 program with `VFS.Include` overlays and
  map-option branching; a sandboxed interpreter with a VFS shim is mandatory.
  Already solved in `rjm.bar.tools` (`src/luaenv.rs`).
- The `.smt` tile atlas is the bulk of an archive - 25 MB of a 49 MB map -
  tile-indexed DXT1 at `SMALL_TILE_SIZE` per tile. **Not decoded today.**
- The minimap block is DXT1 with 9 mip levels at a fixed 699048 bytes. **Not
  decoded today.**
- **Godot does not load DDS at runtime**; it converts to CTEX at import. Map
  textures arrive from arbitrary archives at runtime, so the import pipeline is
  unusable and the extension must own texture decoding.
- Maps run to 32x32 - the checklist rejects larger - which is 16384 world units
  per side.
- Godot has native importers for none of SMF, SMT, S3O, DDS skybox cubemaps, or
  engine-slot detail textures.
- Parity requires feeding ten engine shader texture slots exactly, plus the
  `mapinfo.lua` grid overrides.
- `gdext` ships occasional breaking changes; a pinned version and a deliberate
  upgrade cadence are required.
- Terrain3D is a second GDExtension with its own data and material model,
  reached dynamically rather than typed from Rust.

**Licensing**

- Recoil is GPL v2-or-later; porting its SMF shader chain makes this editor a
  derivative work. **Accepted: GPL v2-or-later, public from the start.**
- Terrain3D is MIT, compatible with that choice.
- Map archives are third-party content; the editor displays them and must not
  redistribute them.

**Resources and skills**

- One developer, volunteer time.
- A 254-archive local corpus serving as both test suite and acceptance criterion.
- An existing citation-backed Rust decoder to build on.
- Skills to acquire: Godot 4 and `gdext`; GLSL-to-Godot shader porting; DXT1
  decoding and tile-atlas assembly; S3O parsing (tractable - fully documented in
  `rts/Rendering/Models/s3o.h`); Terrain3D integration.

### Open questions

**All resolved 2026-09-20**, during P01 business requirements. Full rationale in
that phase's decision records `DR-7` to `DR-16`.

| # | Question | Resolution |
|:--|:---|:---|
| 1 | Does serving reviewers need a checklist panel, or is faithful rendering enough? | **A dedicated checklist view**, showing all twenty properties together with pass or fail only where the checklist states a numeric rule (`DR-10`) |
| 2 | What does "explore quickly" mean measurably? | **Corner-to-corner traversal, equal to or faster than the same traversal in Beyond All Reason** at a pinned engine commit (`DR-7`) |
| 3 | What does parity mean operationally? | **Side-by-side screenshot comparison** against captures from a pinned engine commit, over a fixed reference map set of six to ten maps spanning the corpus variation (`DR-2`, `DR-7`) |
| 4 | Which map options are evaluated at load? | **An option picker with re-evaluation** - not engine defaults (`DR-11`) |
| 5 | Is `LuaGaia` source-only, or must its effect be shown? | **Inspectable source only** (`DR-9`) |
| 6 | Does P01 write anything at all? | **Corpus run reports, parity captures and session state.** Never map data (`DR-8`) |
| 7 | Are the three derived anti-goals confirmed? | **Only "it does not guess" is confirmed as a project anti-goal.** "Not a scenario editor" and "not a map generator" are retained as P01 scope boundaries, not permanent project positions (`DR-12`) |
| 8 | Who is the early-user mapper, and via which channel? | **Approached once a runnable build exists**, not during design. Channel deferred to that point (`DR-14`) |

Two decisions taken alongside these: **Windows and Linux** are both supported
(`DR-13`), and P01 **does not read** `maps-metadata` or `mapfeatures` (`DR-16`).

## 7. Recommended Next Steps

**Focus first.** P01's business requirements should be built around the
externally validated need, not the technology. The single most valuable input
already exists and is public: the **BAR Map Checklist**, which enumerates the
properties a map must satisfy and which two volunteers verify per map. Treat it
as a primary source during `research-brd-context`, because it is the closest
thing to a pre-written requirements document this project has.

**Research needed before BRD drafting.**

- The full Map Checklist and Map Review Process text, captured verbatim and
  mapped property-by-property to what an editor would have to display for each.
- The `maps-metadata` repository's schemas, to learn what the BAR infrastructure
  already records about maps and where an editor's output could align.
- Adoption evidence: how tools actually reach this community, given that the
  three-year success condition is mappers reaching for it.
- Corpus characterisation from `rjm.bar.tools`: how many of the 254 archives
  carry unusual shapes - missing texture slots, option-rewritten height ranges,
  pre-`mapinfo.lua` archives such as `mescaline_v2` - since P01's exit criterion
  is that all of them open.

**Stakeholders to consult.** The early-user mapper, once identified, before the
product requirements are fixed. Open question 8 should be closed early, since
the persona's needs are currently inferred from published process documents
rather than from a person.

**Framework-identified risks needing early attention.**

- The feedback desert is a real accepted cost (PF-07). It is an intent-level
  acceptance but a task-level problem; flag it for the technical design so the
  task graph does not compound it.
- Parity's operational definition (open question 3) should be settled during
  business requirements, not deferred. Without a falsifiable definition, the
  ranked-last commitment has no completion test - and an unbounded target is
  precisely the PF-01 risk identified.

## 8. Phase Registry

There is exactly one project-level intent. The registry below divides that
intent into delivery phases. Phase IDs, acronyms and directories are
**immutable after baseline**.

| Phase ID | Meaningful Name | Acronym | Directory | Outcome | Prerequisite Phases |
|:---|:---|:---|:---|:---|:---|
| P01 | Map Asset Exploration | MAE | `docs/requirements/phases/phase-01-map-asset-exploration/` | Any installed BAR map opens and is navigable, with every asset class represented | None |
| P02 | Map Write Round Trip | MWRT | `docs/requirements/phases/phase-02-map-write-round-trip/` | A map opened and re-saved by the editor loads in Beyond All Reason with no loss of fidelity | [P01](#p01---map-asset-exploration) |
| P03 | Terrain Data Editing | TDE | `docs/requirements/phases/phase-03-terrain-data-editing/` | Heightmap, typemap and metalmap can be modified in the editor and the result plays correctly in game | [P02](#p02---map-write-round-trip) |
| P04 | Map Layer Editing | MLE | `docs/requirements/phases/phase-04-map-layer-editing/` | Texture layers, feature placement and map definition values can be modified and round-tripped | [P03](#p03---terrain-data-editing) |

> **Provenance note.** P01 is specified from a fully converged intent (stages
> 0.1-0.6). P02-P04 derive from the second stated intent - "to be able load and
> edit map layers and terrain data" - and have not each been through their own
> refinement. Their outcomes and criteria are firm enough to baseline; their
> internal scope will be elaborated when each phase's business requirements are
> drafted.

### P01 - Map Asset Exploration

- **Phase ID:** `P01` (immutable after baseline)
- **Name:** `Map Asset Exploration`
- **Acronym:** `MAE`
- **Directory:** `docs/requirements/phases/phase-01-map-asset-exploration/`
- **Outcome:** Any BAR map in the local library opens in the editor and can be
  explored in three dimensions, with every asset class the archive contains
  represented, and with what is shown traceable to an engine rule rather than an
  approximation.
- **Included scope:**
  - Reading both `.sd7` archives and unpacked map directories through one
    virtual file system abstraction.
  - Decoding the compiled map: SMF header, heightmap, typemap, metalmap,
    minimap, feature block and vegetation extra-header; and the `.smt` tile
    atlas.
  - Evaluating `mapinfo.lua`, `mapoptions.lua` and their overlays in a sandboxed
    Lua 5.1 environment with a VFS shim.
  - Parsing the legacy `.smd` (TDF) map definition, which the engine still
    supports as a fallback (`rts/Map/MapParser.cpp:30`) and which at least one
    installed archive (`mescaline_v2`) relies on. Added to scope by the
    no-exceptions corpus decision.
  - Loading and presenting every asset class: shading and override textures,
    sky, water, S3O feature models with their textures, feature placements,
    start positions, and map-side Lua as inspectable source.
  - Three-dimensional terrain presentation with level of detail sufficient for a
    32x32 map.
  - Camera and navigation components sized for fast exploration of a large map.
  - Visual parity with Recoil's SMF shading chain, as the ranked-last commitment.
  - Conformance verification across the installed map corpus.
- **Entry criteria:**
  - This Intent Brief is approved.
  - P01 business requirements, product requirements and technical design are
    complete and validated.
  - The GPL v2-or-later licence decision is recorded in the repository.
- **Exit criteria:**
  - **Every archive in the local map library opens without error. No
    exceptions**, and no exclusion list. Stakeholder decision, 2026-09-20;
    consequence recorded in scope above.
  - Each asset class enumerated in the included scope is represented for maps
    that contain it.
  - Corner-to-corner traversal time under normal camera control meets the
    threshold set in the product requirements - the agreed measure of
    "explore quickly".
  - No displayed value is an approximation: a value that cannot be derived
    exactly is **marked unknown in the interface**, never substituted with a
    plausible one, and the map still opens.
  - The recruited early-user mapper has flown a map they authored and their
    feedback is recorded.
- **Prerequisite phases:** None

### P02 - Map Write Round Trip

- **Phase ID:** `P02` (immutable after baseline)
- **Name:** `Map Write Round Trip`
- **Acronym:** `MWRT`
- **Directory:** `docs/requirements/phases/phase-02-map-write-round-trip/`
- **Outcome:** A map opened by the editor and written back out is loadable by
  Beyond All Reason and equivalent to the original, establishing the write path
  that all editing depends on.
- **Included scope:** Writing SMF and SMT; writing or preserving `mapinfo.lua`
  and the remaining archive contents; packing an archive and producing an
  unpacked directory; and round-trip verification across the corpus.
- **Entry criteria:** P01 exit criteria met; P02 requirements and design
  complete and validated.
- **Exit criteria:** For a defined corpus sample, open-write-reopen produces an
  equivalent map by the editor's own reader; a written map loads in Beyond All
  Reason; and any intentional divergence from the original bytes is documented
  and justified.
- **Prerequisite phases:** [P01 - Map Asset Exploration](#p01---map-asset-exploration)

### P03 - Terrain Data Editing

- **Phase ID:** `P03` (immutable after baseline)
- **Name:** `Terrain Data Editing`
- **Acronym:** `TDE`
- **Directory:** `docs/requirements/phases/phase-03-terrain-data-editing/`
- **Outcome:** The terrain data grids can be modified in the editor, and a map
  so modified behaves correctly in game.
- **Included scope:** Editing the heightmap, typemap and metalmap; undo and
  redo; and presenting the gameplay consequences of an edit - slope,
  passability by movement class, metal yield - using the same engine rules the
  reader already reproduces.
- **Entry criteria:** P02 exit criteria met; P03 requirements and design
  complete and validated.
- **Exit criteria:** Each terrain grid can be edited and saved through the P02
  write path; an edited map loads and plays in Beyond All Reason; and edits are
  reversible within a session.
- **Prerequisite phases:** [P02 - Map Write Round Trip](#p02---map-write-round-trip)

### P04 - Map Layer Editing

- **Phase ID:** `P04` (immutable after baseline)
- **Name:** `Map Layer Editing`
- **Acronym:** `MLE`
- **Directory:** `docs/requirements/phases/phase-04-map-layer-editing/`
- **Outcome:** The non-terrain layers of a map - texture layers, feature
  placement and map definition values - can be modified and round-tripped,
  completing the second stated intent.
- **Included scope:** Editing texture layers and splat distribution; placing,
  moving and removing features; editing `mapinfo.lua` values through a
  structured surface; and validating an edited map against the properties the
  BAR Map Checklist enumerates.
- **Entry criteria:** P03 exit criteria met; P04 requirements and design
  complete and validated.
- **Exit criteria:** Each layer in scope can be edited and saved; an edited map
  loads and plays in Beyond All Reason; and the editor reports which checklist
  properties it can and cannot verify.
- **Prerequisite phases:** [P03 - Terrain Data Editing](#p03---terrain-data-editing)

### Registry validation

| Check | Result |
| --- | --- |
| Exactly one project-level intent | Pass |
| Phase IDs sequential and unique | Pass - P01, P02, P03, P04 |
| Acronyms unique, 2-6 uppercase alphanumeric | Pass - MAE, MWRT, TDE, MLE |
| Directory numeric portion matches Phase ID | Pass |
| No forbidden folder names (`brd`, `prd`, `tdd`, `api`, `qa`, `ui`) | Pass |
| No placeholder names (`MVP`, `Core`, `Initial`, `Phase 1`) | Pass |
| Every phase has outcome, scope, entry, exit, prerequisites, path | Pass |
| No earlier phase depends on a later phase | Pass - strictly linear P01 -> P02 -> P03 -> P04 |
| Each phase is an independently verifiable increment | Pass - each exit criterion is observable without reference to a later phase |

## Appendix

- **Intent Record** (stages 0.1-0.6, full traceability from raw input):
  [`intent-record.md`](intent-record.md)
- **Framework Assessment Report** (all framework outputs and scores):
  [`framework-assessment.md`](framework-assessment.md)

## Final validation

| Check | Status |
| --- | --- |
| Vision Statement is concrete | Met - names the problem, the audience, and a measurable v1 bar |
| Every claim traceable to a stage of the Intent Record | Met |
| Framework citations accurate against the assessment | Met - Ikigai and value dimension scores match |
| Research evidence cited for market claims | Met - checklist, review process, Tiled funding, toolchain, gdext, Terrain3D |
| Open Questions genuinely unresolved | Met - eight, none answered elsewhere in the record |
| Recommended Next Steps specific and actionable | Met - named sources, named risks, one stakeholder action |
| Exactly one project-level intent | Met |
| Phase registry complete before phase BRD work | Met |
| Phase identity, uniqueness and completeness checks | Met - see Registry validation |
| User has reviewed and approved | Met - approved 2026-09-20; all eight open questions resolved during P01 business requirements |
