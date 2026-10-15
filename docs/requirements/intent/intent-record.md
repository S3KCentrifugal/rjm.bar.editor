---
stage: 0.1-0.7 (capture through intent brief) - complete
status: approved
confidence: capture (user-confirmed by proceeding) / decided (stage 0.3 answers) / research (landscape, stakeholders, failure patterns) / inference (stage 0.4 frameworks) / decided (stage 0.5 cycles)
captured: 2026-09-20T19:45:00Z
explored: 2026-09-20T20:10:00Z
assessed: 2026-09-20T20:35:00Z
refined: 2026-09-20T21:00:00Z
assessment_report: docs/requirements/intent/framework-assessment.md
verified_against:
  - RecoilEngine 92efda5e60 (2026-09-13)
  - Beyond All Reason 1d267c20d1 (2026-09-13)
  - rjm.bar.tools (working tree)
  - Local map library, 254 .sd7 archives plus 1 unpacked directory (%LOCALAPPDATA%\Programs\Beyond-All-Reason\data\maps)
  - BAR Map Checklist and Map Review Process guides (beyondallreason.info, retrieved 2026-09-20)
next: analyze-brd-readiness (phase P01)
intent_brief: docs/requirements/intent/intent-brief.md
---

# Intent Record

## Stage 0.1 - Capture

### Raw input (verbatim)

> The intent of this project is two fold. One to be able load and edit map
> layers and terrain data. For version 1 I just want to be able to loads and
> explore all of a maps assets. I require camera and navigation composes to
> quickly explore the map. It should be design in godot engine. Before doing
> any code lets write the full spec and task breaking starting with intent.

### Paraphrase (confirmed by proceeding to explore)

You want a Beyond All Reason map editor built in the Godot engine. Its ultimate
purpose is to load and edit a map's layers and terrain data. Version 1 stops
short of editing: it opens a map and lets you see and inspect **every** asset
the map contains, with camera and navigation components good enough to move
around a large map quickly. Specification comes first - the full spec and task
breakdown are to be written before any code is committed.

### Input type

**Seed** - a short directive that names an end state, a v1 boundary, one
explicit component requirement (camera and navigation) and a technology
(Godot). No personas, workflows, acceptance criteria or asset list were
supplied; everything downstream is to be derived.

## Stage 0.2 - Classify

### Domain tags

- Game development tooling / content authoring
- Real-time strategy (Beyond All Reason, Recoil engine)
- 3D geospatial and terrain visualisation

### Intent archetype

**Product** - a runnable desktop application built and shipped as an artifact,
not a study or a practice. Reinforced by the explicit engine choice and by a
versioned scope boundary ("for version 1"), which implies later versions.

### Motivation hypothesis (to be validated in explore-intent)

The user appears motivated by the absence of a modern, visual, *read-first* way
to see what a BAR map actually contains: the existing toolchain is a set of
one-way compilers and an in-game editor, so inspecting a map means either
launching the game or decompiling it into loose images. Having already built a
headless, exact decoder for BAR map archives (`rjm.bar.tools`), the user appears
to want the same fidelity presented visually and interactively - and to reach
editing from a foundation that already reads everything correctly, rather than
by extending a tool that was only ever built to write.

Signals supporting this reading:

- Version 1 is explicitly **load and explore**, not edit - reading is treated as
  a first-class deliverable, not a stepping stone to be rushed.
- The word chosen is *all* of a map's assets, not "the terrain".
- Camera and navigation are called out as a requirement by name, which is a
  statement about inspection speed, not about rendering.
- Prior investment in `rjm.bar.tools`, which reproduces engine rules
  (`ArchiveScanner.cpp`, `SMFReadMap.cpp`, `ReadMap.cpp`, `LuaParser.cpp`)
  rather than approximating them. That exactness exists to be built on.
- Godot, rather than a bespoke renderer, implies the value is in the editing and
  inspection experience, not in graphics engineering.

## Domain research context

### What "all of a map's assets" actually means

Enumerated from two real archives in the local library and cross-checked against
the engine's own resource table (`rts/Map/MapInfo.cpp`, `rts/Map/SMF/SMFFormat.h`).
This is the concrete surface v1 has to cover.

| Asset class | Files seen | Notes for the editor |
| --- | --- | --- |
| Compiled terrain | `maps/<name>.smf` | Header + heightmap (`u16[(mapx+1)*(mapy+1)]`), typemap, metalmap, minimap (DXT1, 9 mips), feature block, optional vegetation extra-header |
| Terrain tiles | `maps/<name>.smt` | The bulk of the archive (25 MB of a 49 MB map). Tile-indexed DXT1 atlas, `SMALL_TILE_SIZE` per tile. **Not decoded by `rjm.bar.tools`** - new work |
| Map definition | `mapinfo.lua`, `mapoptions.lua`, `mapconfig/mapinfo/*.lua`, `maphelper/mapinfo.lua` | A Lua *program*, not data: `VFS.Include` overlays, branches on map options, can rewrite the height range |
| Shading textures | `normalmap.dds`, `specular.dds`, `splat_distr.dds`, `*_dnts.tga`, `detailtex*.bmp`, `moltenmetal.dds`, `rock_holes.dds` | Engine keys: `detailTex`, `specularTex`, `splatDetailTex`, `splatDistrTex`, `splatDetailNormalTex1..4`, `detailNormalTex`, `grassShadingTex`, `skyReflectModTex`, `lightEmissionTex`, `parallaxHeightTex` |
| Override grids | keys `minimapTex`, `metalmapTex`, `typemapTex`, `grassmapTex` | `mapinfo.lua` may replace what the `.smf` supplies |
| Sky and water | `sky.dds` cubemap; water `texture` / `foamTexture` / `normalTexture` / caustics | Water is a full material block in `mapinfo.lua` |
| Map objects | `objects3d/*.s3o`, `unittextures/*.tga|dds`, `features/*.lua` | S3O is a Spring-specific model format Godot cannot import |
| Object placement | `mapconfig/featureplacer/set.lua`, `model.lua`, `config.lua` | Thousands of placements; the second editable layer after terrain |
| Map-side Lua | `LuaGaia/main.lua`, `LuaGaia/Gadgets/*.lua`, `LuaGaia/effects/*.png` | Runtime behaviour shipped inside the map |
| Start positions | `mapinfo.lua` `teams` table | Author's intended spots; lobby start boxes may differ |

Two archives inspected in full: `acidicquarry_5.17.sd7` (48 files, 12 folders,
49 MB) and `all_that_glitters_extended_v1.0.2.sd7`. 254 archives are installed
locally, so a real corpus for conformance testing already exists.

### Analogous concepts

| Concept | What it is | Bearing on this intent |
| --- | --- | --- |
| **SpringBoard** | The in-engine Spring/Recoil map and scenario editor; Core module still updated March 2026 | The incumbent. Runs *inside* the engine, so it inherits engine limits: no full map serialisation (Spring saves only a delta heightmap), and texture editing against live GL textures is a known unsolved area. Closest thing to a competitor and the clearest statement of what an external editor is free from. |
| **MapConv / PyMapConv** | Cross-platform SMF compilers; PyMapConv also decompiles `.smf` to loose images | One-way batch converters with no viewer. Establishes that the format is fully round-trippable, which an editor depends on. |
| **SpringMapEdit** | Described by the community as the only complete map-making solution, and as "in no way visual", with arbitrary texture-size rules | The gap this project targets, stated by the existing tool's own users. |
| **Beherith's springrts_smf_compiler** | The compiler the official BAR mapping guide points authors at | Defines the current authoring pipeline: World Machine / Photoshop / GIMP produce images, a compiler bakes them, SpringBoard places objects, 7-Zip packs. Nothing in that chain *shows* you a finished map outside the game. |
| **`rjm.bar.tools` (`smrt-bar-map`)** | The user's own Rust decoder: `.sd7`, SMF grids, sandboxed `mapinfo.lua` evaluation, terrain derivation, 227 maps processed deterministically | Prior art inside the workspace, not a competitor. Already answers "how is a BAR map decoded correctly" for everything except tiles and minimap. |
| **Terrain3D / HTerrain / TerraBrush / M Terrain** | Godot 4 terrain systems (GDExtension C++ or GDScript) with heightmap import, texture painting, LOD, holes | Evidence that Godot carries editable large-scale terrain well, and a menu of build-versus-adopt options for the eventual editing phase. None of them reads a game-specific packed format. |
| **hendkai/bar-map-generator** | A BAR map generator project | Adjacent: generates rather than inspects, but shares the output format. |

### Market maturity

**Split, and the split is the opportunity.**

- *BAR map inspection and editing tooling:* **Nascent to stagnant.** The
  authoring chain is a decade-old set of compilers plus an in-engine editor;
  the community's own description of the complete solution is "in no way
  visual". No modern external viewer was found.
- *Godot as a tool platform:* **Growing.** Godot 4's GDExtension has produced
  several serious terrain systems (Terrain3D, M Terrain, TerraBrush in 2026),
  and using Godot to build tools rather than games is now ordinary.
- *Beyond All Reason as a platform:* **Growing.** Hooded Horse publishing deal
  signed June 2026 with a Steam release pending; player interest and map-making
  demand both rising.

### Key trends

1. **The authoring chain is one-way.** Images compile into `.smf`/`.smt`; the
   only routine way to see the result is to launch the game. An external reader
   inverts that.
2. **In-engine editing is structurally constrained.** SpringBoard's own
   documentation names full map serialisation and live-texture editing as open
   problems. An external editor owns the file and does not inherit either.
3. **Godot 4 has crossed the threshold for this class of tool.** GDExtension
   gives native-speed parsing and custom rendering while keeping Godot's editor,
   UI toolkit and camera plumbing.
4. **The hard part is already solved once, in Rust.** `rjm.bar.tools` reproduces
   engine rules with citations. Whether the editor consumes that crate, ports
   it, or reimplements in GDScript is a genuine and consequential open question.
5. **BAR is commercialising**, which enlarges the map-author audience and raises
   expectations of the tools they use.
6. **`mapinfo.lua` is executable, not declarative.** Any faithful reader must
   run Lua 5.1 in a sandbox with a VFS shim. This is a hard constraint on the
   architecture, already proven necessary by `rjm.bar.tools`.

## Stage 0.3 - Explore

Two rounds of guided questioning, each preceded by research. User answers are
recorded as **decided**; anything derived from them is marked as such.

### Historical context - why now

Two reasons given, both selected:

1. **Inspecting maps is painful.** Seeing what a map contains means launching
   the game or decompiling it to loose images. The authoring chain is one-way.
2. **Editing is the real goal.** V1 is the foundation, not the destination -
   layer and terrain editing is what the project is ultimately for.

Three enabling changes make now the moment rather than any earlier one:

- An exact decoder already exists in the workspace (`rjm.bar.tools`), covering
  archive access, SMF grids, sandboxed `mapinfo.lua` evaluation and terrain
  derivation across 227 maps deterministically.
- Godot 4's GDExtension has matured: `godot-rust/gdext` is described by its
  maintainers as usable for games, editor plugins and tools, with the majority
  of Godot APIs mapped, though it still ships occasional breaking changes.
- BAR is commercialising (Hooded Horse, June 2026), enlarging the map-author
  population and the value of better tooling.

### Stakeholders

**Primary beneficiaries** - all four selected; this is deliberately one tool for
four audiences.

| Stakeholder | What they need from it | Evidence |
| --- | --- | --- |
| **The user** | A personal instrument, answerable to no one else's workflow | Stated |
| **BAR map reviewers** | To verify the Map Checklist without launching the game. Two reviewers must approve every map (`@IceXuick` breaks ties); both are volunteers, and reviews are asked to be efficient and to point at documented standards | BAR Map Review Process guide |
| **Map authors** | To see their compiled `.smf`/`.smt` mid-build, closing the World Machine -> compiler -> game loop | BAR mapmaking guides; both `.sd7` and unpacked folders must open |
| **AI and tooling developers** | To confirm derived terrain analysis visually - slope, passability, metal spots, start-position classification - against the map it came from | `rjm.bar.tools` generates 227 map configs with no visual check |

**Secondary stakeholders**

- BAR players, who receive better-reviewed maps.
- The BAR maps team and the `maps-metadata` / `map_blueprint` repositories,
  which hold the metadata and starting files the review process assumes.
- `rjm.bar.ai` and `s3k-CircuitAI`, whose map reasoning is derived from the same
  grids this editor renders.
- Recoil engine maintainers, whose shader chain becomes the parity target.

**Potential resistors**

- SpringBoard and its maintainers: functional overlap at the edges, even though
  this tool is external and read-first in v1.
- Authors invested in the existing World Machine plus SpringBoard workflow, for
  whom another tool is another thing to learn.
- Anyone who would treat a non-engine renderer as authoritative. A parity claim
  that drifts is worse than no parity claim, because reviewers would be
  approving maps against a lie.

### Value vision

**Done for v1: open any of the 254 installed maps and fly around it.**

Breadth is the correctness test, not depth. A tool that renders one map
beautifully has proven nothing about `mapinfo.lua` option branching, missing
texture slots, unusual height ranges or malformed feature blocks; a tool that
opens all 254 has. `rjm.bar.tools` already found the shape of this: 154 of 227
maps read map options, some of which rewrite the height range, and one archive
(`mescaline_v2`) predates `mapinfo.lua` entirely.

Combined with the fidelity decision below, success looks like: pick any
installed map, it opens, it looks like the game, and you can get anywhere on it
quickly.

### Decisions taken in this stage

| Question | Decision | Consequence |
| --- | --- | --- |
| Asset scope | **Everything, rendered as faithfully as possible** | S3O feature models placed in world, sky cubemap, water, all texture slots, plus `LuaGaia` and featureplacer sources |
| Input forms | **Both `.sd7` archives and unpacked folders** | A VFS abstraction over two backends, not a 7-Zip reader with a folder special case |
| Decoder location | **Rust core via GDExtension (`godot-rust/gdext`)** | `rjm.bar.tools` becomes a shared library, extended with `.smt` tile and minimap decoding; one decoder serves both projects |
| Fidelity bar | **Visual parity with Recoil** | Port the engine's SMF shader chain rather than approximating with Godot's PBR |
| Definition of done | **All 254 installed maps open and are navigable** | Conformance against the real corpus is a first-class deliverable |

The fidelity decision is coherent with the anti-goal below: a tool purpose-built
for Recoil, aiming at parity, is a **reference renderer** for the format. That
is a stronger product than a generic viewer, and a significantly larger one.

### Constraints

**Technical**

- `mapinfo.lua` is an executable Lua 5.1 program with `VFS.Include` overlays and
  map-option branching, not a data file. Any faithful reader needs a sandboxed
  interpreter with a VFS shim. Already proven necessary and already solved in
  `rjm.bar.tools`.
- The `.smt` tile atlas is the bulk of an archive (25 MB of a 49 MB map), stored
  as a tile-indexed DXT1 set at `SMALL_TILE_SIZE` per tile. **Not decoded by
  `rjm.bar.tools` today** - new work, and on the critical path for any rendering.
- The minimap block is DXT1 with 9 mip levels at a fixed 699048 bytes; also not
  currently decoded.
- Maps run up to 32x32 (the checklist rejects anything larger), which is 16384
  world units per side. Terrain streaming and LOD are load-bearing, not polish.
- Godot has native importers for none of these formats: SMF, SMT, S3O, DDS
  cubemaps as skyboxes, or TGA/BMP detail textures in engine-specific slots.
- Visual parity means feeding the engine's shader inputs exactly: `detailTex`,
  `specularTex`, `splatDetailTex`, `splatDistrTex`, `splatDetailNormalTex1..4`,
  `detailNormalTex`, `grassShadingTex`, `skyReflectModTex`, `lightEmissionTex`,
  `parallaxHeightTex`, plus the `minimapTex` / `metalmapTex` / `typemapTex` /
  `grassmapTex` overrides that `mapinfo.lua` may substitute.
- `gdext` still introduces occasional breaking changes, with migration guides.
  A pinned version and a deliberate upgrade cadence are required.

**Skills gap**

- Godot 4 and the `gdext` build toolchain.
- Porting GLSL (`cont/base/springcontent/shaders/GLSL/SMFVertProg.glsl`,
  `SMFFragProg.glsl`, `SMFShadingTexture*`, `BumpWater*`) to Godot's shading
  language while preserving behaviour.
- DXT1 decoding and tile-atlas assembly.
- S3O parsing - tractable: the format is fully documented in
  `rts/Rendering/Models/s3o.h` as a hierarchy of pieces with vertex and index
  tables at file offsets.
- Large-terrain LOD in Godot.

**Licensing**

- Recoil is GPL v2 or later. Porting its SMF shader chain makes this editor a
  derivative work, so it is effectively **GPL v2-or-later** from the moment
  parity work begins. This is a decision to take consciously, not to discover.
- Map archives are third-party content under their authors' terms. The editor
  reads and displays them; it must not redistribute them.

**Resources**

- One developer, volunteer time.
- A 254-archive local corpus, which is both the test suite and the acceptance
  criterion.
- An existing, citation-backed Rust decoder to build on rather than start from.

### Anti-goals

1. **It is a purpose-built tool for Recoil.** (User's own words.) Not a
   general-purpose terrain or heightmap editor, not a Godot terrain plugin, not
   a multi-engine map tool. Recoil's formats and Recoil's rules are the whole
   target, and generality is not a virtue here.
2. *(Derived from the fidelity and decoder decisions, pending confirmation)*
   **It does not guess.** Where an engine rule governs a value, that rule is
   reproduced with a citation, as `rjm.bar.tools` already does. A value that
   cannot be derived exactly is reported as unknown rather than approximated -
   because reviewers would otherwise approve maps against a plausible-looking
   fiction.
3. *(Derived, pending confirmation)* **It does not become a scenario or gadget
   editor.** Terrain and map layers only; triggers, units and game logic are
   SpringBoard's ground.
4. *(Derived, pending confirmation)* **It does not generate maps.** It reads and
   eventually edits maps that exist.

### Exploration themes

- **Exactness is the identity.** Inherited from `rjm.bar.tools`, which
  reproduces engine rules with source citations rather than approximating them.
  Every decision in this stage points the same way: parity over convenience,
  the whole corpus over one showcase map, a shared Rust decoder over a
  convenient reimplementation.
- **Read before write, deliberately.** V1 cannot edit anything, and that is the
  point: editing a format you cannot yet display correctly is how the existing
  tools ended up one-way.
- **Breadth as proof.** 254 maps is a conformance suite disguised as a goal.
- **One tool, four audiences, one artifact.** No audience gets a separate mode;
  the same faithful view serves personal use, review, authoring and AI
  verification.
- **Parity as the product.** "Purpose-built for Recoil" plus "visual parity"
  makes this a reference renderer for the map format, not a viewer.

### Tensions to carry forward

These are recorded, not resolved. They are the substance of the phase registry
in `write-intent-brief`.

1. **Scope versus a solo maintainer.** Visual parity with Recoil's shader chain,
   full asset fidelity including S3O models, two input backends, four audiences
   and corpus-wide conformance is a large v1. The single most-cited failure mode for
   tools in this space is scope creep - one maintainer's stated cause of death
   was "infinitely growing scope, more features than I've ever wanted". The
   counter-example, Tiled, ran 15 years on a narrow, stable format promise.
   Phasing is the mitigation, and it belongs in the intent brief.
2. **Parity versus independence.** Tracking Recoil's shaders is a standing
   coupling to a moving upstream, in tension with a standalone tool that never
   needs the engine present. Parity has to be *versioned against* an engine
   commit - the discipline `rjm.bar.docs` already applies with
   `verified_against`.
3. **A read-only v1 at parity cost.** Parity-quality rendering is expensive for
   a version that cannot edit. Worth it only if rendering is genuinely the
   foundation editing stands on - which the user's "editing is the real goal"
   answer asserts.
4. **Reviewer trust.** If reviewers adopt this for checklist verification, its
   output becomes load-bearing for what enters the game. That raises the cost of
   any silent approximation from an inconvenience to a defect.

### Research findings

**Analogous audiences.** Reviewers and authors in this domain today work
through Discord threads with a `@mapper` role ping, a posted map file, and
at-least-two-reviewer approval against a written checklist. The tooling they are
pointed at is: World Machine templates, Photoshop or GIMP, Beherith's SMF
compiler, SpringBoard for painting and object placement, PyMapConv for
decompiling and minimap lightening, a SpringBoard pathability-checker script, a
Python rotating-features converter, and 7-Zip. Every one of those authors,
converts or packs. None of them *shows you a finished map*.

**Market gaps.**

- No external viewer for a compiled BAR map. Inspection means launching the game.
- The Map Checklist is a list of verifiable properties - pathability by movement
  class, metal distribution shape, normals resolution relative to texture, sun
  angle and lighting, wind 0-30, tidal 0-25, feature reclaim values, geovent
  buildability, splat distribution balance, minimap brightness - and there is no
  tool that surfaces them together. This is the closest thing to a
  pre-written requirements source the project has.
- In-engine editing is structurally constrained: Spring saves only a delta
  heightmap rather than a full map, and texture editing against live GL textures
  is described as an unsolved area. An external tool owns the file and inherits
  neither limit.
- Derived terrain analysis has no visual verification path.

**Failure patterns for comparable ventures.**

1. **Scope creep** - the most-cited cause, at both hobby and studio scale.
2. **Single-maintainer abandonment**, made terminal by licensing or
   documentation that prevents anyone else from continuing.
3. **Format drift** - tools pinned to a format snapshot break when the engine
   moves. Tiled's counter-example is a stable format promise held for 15 years.
4. **Authoring-first design** - building the writer before the reader is exactly
   how this domain arrived at a one-way chain.
5. **Inheriting the host's limits** by building inside the engine.

## Stage 0.4 - Assess

Full report: [`framework-assessment.md`](framework-assessment.md). Seven of
eight psychological frameworks applied plus the four value dimensions, grounded
in workspace artifacts and in research on comparable ventures. Summary only
here; the report carries the evidence.

**Value dimensions:** Physical 2/5, Emotional 3/5, Intellectual 5/5, Spiritual
4/5. Composite: Intellectual-dominant, Spiritual-strong, moderate Emotional,
peripheral Physical.

**Ikigai:** Love Strong, Skill Strong (with a located weak spot), Need Strong
and externally validated, Viability Moderate and probably the wrong question
for community infrastructure. Profile: *Mission with a Passion core*.

**Motivation (SDT):** Integrated shading into Intrinsic. Autonomy and
Competence very high; Relatedness the weakest leg.

**The three findings that should change what happens next:**

1. **The fidelity bar sits in the weakest skill quadrant.** Every demonstrated
   mastery is upstream of the renderer; everything parity needs is new. Argues
   for sequencing the decoder to done first and treating parity as a named,
   later, deliberately-learned phase.
2. **There is a feedback desert before first pixel**, and very high orderliness
   is aimed at an unbounded target. Argues for ordering the work so a crude
   greybox heightmap with a flying camera arrives early, and for writing the
   phase edge down rather than holding it as intention.
3. **Three frameworks independently converge on one gap** - thin SDT
   Relatedness, absent VIA Justice virtues, and Emotional at 3/5. Four
   audiences are named, two are communities, and nothing in the intent yet
   engages them. Either recruit an early user from the BAR mapping community,
   or scope the tool honestly as personal-first.

## Stage 0.5 - Refine

Two cycles, each preceded by targeted research. Minimum met; no further cycle
needed, since all seven convergence criteria resolved (stage 0.6).

### Cycle 1 - phasing, audience reality, licence, terrain mesh

**Insights presented.** PF-01 and PF-06: very high orderliness aimed at an
unbounded target has no natural stopping point. PF-07: a feedback desert sits
before first pixel. PF-03, PF-08 and Value Dimensions converge on one gap -
four audiences named, two communal, none engaged.

**Research conducted.**

- *Godot runtime texture loading.* Godot does **not** load DDS at runtime; it
  converts DDS to CTEX at import. Map textures arrive from arbitrary archives at
  runtime, so the import pipeline is unusable and the extension must own texture
  decoding. This strengthens the Rust-core decision rather than complicating it.
- *Large terrain.* Terrain3D is a GPU geometric-clipmap terrain supporting
  64 m to 65.5 km with up to 10 mesh LOD levels. A 16384-unit BAR map sits
  comfortably inside it, so large-terrain LOD is adopt-not-build.

**Decisions.**

| Question | Decision |
| --- | --- |
| Phase cut of v1 | **None. "V1 is a phase, it should be just one."** V1 is itself one phase of the project, delivered whole; the phase registry divides *the project* (explore, then edit), not v1 |
| Audience reality | **Recruit one BAR mapper as an early user now** |
| Licence and distribution | **GPL v2-or-later, public from the start** |
| Terrain mesh | **Adopt Terrain3D for mesh and LOD** |

### Cycle 2 - work order, early user, success, sacrifice

**Insights presented.** Terrain3D's `shader_override_enabled` replaces the
generated terrain shader entirely, so adopting it does not forfeit parity - the
principal risk in that choice. It is MIT, compatible with the GPL decision, and
is a C++ GDExtension for Godot 4.4+.

**Research conducted.**

- *Terrain3D shader override.* Custom shader code fully replaces the generated
  material; enabling the override with a blank shader emits the current
  generated shader as a starting point.
- *Terrain3D material model.* Built for up to 32 texture sets with splatting.
  BAR's ground is a tile-indexed DXT1 atlas, not a 32-texture splat set, so the
  override is expected to be near-total: Terrain3D supplies the clipmap mesh and
  LOD, not the look.
- *Integration wrinkle.* A Rust extension calling a second, C++ extension's
  classes is dynamic rather than typed. Flagged for the TDD.

**Decisions.**

| Question | Decision |
| --- | --- |
| Work order inside the phase | **Strict dependency order** - decoder complete and correct first, then rendering on top of it. The feedback desert is accepted knowingly |
| Early user's first task | **Fly around a map they made themselves** - they know that terrain better than anyone, so errors are obvious immediately |
| Success at three years | **"It's what BAR mappers reach for"** - adopted by the mapping community as a normal part of making and approving maps |
| First thing sacrificed under pressure | **Visual parity** |

### What cycle 2 changed

Two answers materially re-shape the intent, and both are refinements rather
than reversals.

**1. Community adoption is the destination, not a side effect.** Success at
three years is the community reaching for it. This raises SDT Relatedness and
the Emotional value dimension from incidental to load-bearing, and it makes
distribution, packaging and usability by strangers part of the intent rather
than a later nicety. It also retroactively justifies the GPL and
public-from-the-start decisions.

**2. Parity is the target *and* the designated sacrifice.** The fidelity bar
remains visual parity with Recoil. But asked what goes first if something must
give, the answer was parity - ahead of corpus breadth, full asset coverage and
exploration speed. That yields an explicit priority stack, which the earlier
stages lacked:

| Rank | Commitment | Status |
| --- | --- | --- |
| 1 | Every installed map opens and is navigable | Protected |
| 2 | Every asset class is represented | Protected |
| 3 | Exploration is fast - camera and navigation | Protected |
| 4 | Visual parity with Recoil | Target, and the first thing cut |

This resolves the assessment's central tension without abandoning the goal.
Parity keeps its place as what the tool is *for* - a purpose-built reference
renderer - while ceasing to be the thing that can prevent release. It also
aligns the risk with the skill profile: the ranked-last commitment is the one
sitting in the weakest skill quadrant.

### Primary persona

Derived from the cycle-1 and cycle-2 answers, and required by convergence
criterion 2.

**The BAR mapper.** Makes maps for Beyond All Reason, or reviews them. Works
through World Machine or a paint tool, compiles with Beherith's SMF compiler,
places objects in SpringBoard, packs with 7-Zip. To see the result they must
launch the game. If they review, they must verify a written checklist -
pathability, metal distribution, normals resolution, sun angle, wind and tidal
ranges, feature reclaim values - as an unpaid volunteer, twice over per map.

What they need: to open a map they know and move around it immediately, and to
trust that what they see is what the game would show. Their first judgement of
this tool will be made flying over terrain they built themselves, and it will
be instant.

### Accepted risks

Recorded as knowingly accepted, not unresolved.

- **The feedback desert is accepted.** Strict dependency order means the decoder
  reaches done before anything renders. This matches the strongest half of the
  skill profile (PF-04) and the "does not guess" ethic, at the cost of the
  PF-07 Immediate Feedback condition. Mitigation is a matter of task ordering in
  the TDD, not of intent.
- **V1 stays whole.** No sub-phases, decided twice. The scope-creep failure
  pattern is mitigated instead by the priority stack above: parity is
  sacrificial, so v1 has a floor that does not depend on the hardest work
  landing.
- **Terrain3D is a dependency.** MIT and GPL-compatible, but a second
  GDExtension with its own data and material model, reached dynamically from
  Rust.

## Stage 0.6 - Converge

| # | Criterion | Status | Basis |
| --- | --- | --- | --- |
| 1 | Problem clarity | **Met** | Seeing what a BAR map contains requires launching the game, because the authoring chain is one-way. Reviewers verify a written checklist with no tool that surfaces it. Nothing shows a finished map outside the engine |
| 2 | Audience defined | **Met** | The BAR mapper persona above, with a named first task and an early user to be recruited |
| 3 | Value articulated | **Met** | Open any installed map, move through it immediately, and trust that what you see is what the game would show - adopted by mappers as a normal part of making and approving maps |
| 4 | Motivation grounded | **Met** | PF-03 integrated-to-intrinsic regulation with very high Autonomy and Competence; PF-05 Creative pathway with a genuinely unique assignment |
| 5 | Feasibility acknowledged | **Met** | Rust core over `gdext`; Terrain3D for mesh and LOD; runtime texture decoding owned by the extension; `.smt` and minimap decoding as new work; GPL consequence accepted; skills gap named |
| 6 | Boundaries set | **Met** | Purpose-built for Recoil; does not guess; not a scenario or gadget editor; not a generator; v1 does not edit; priority stack names parity as the designated sacrifice |
| 7 | User confidence | **Met** | User directed the brief and business requirements to be written, on the record as it stands |

**Overall: pass on all seven.**

**Notes.** Convergence was reached in the minimum two cycles. No
non-convergence signals appeared: no conflated intents, no scope inflation
across cycles - scope in fact narrowed through the priority stack - and no
decision avoidance; every question drew a definite answer, one of them
overriding the option set offered.

## Structured record

```yaml
intent_record:
  # --- Stage 0.1: Capture ---
  raw_input: "The intent of this project is two fold. One to be able load and edit map layers and terrain data. For version 1 I just want to be able to loads and explore all of a maps assets. I require camera and navigation composes to quickly explore the map. It should be design in godot engine. Before doing any code lets write the full spec and task breaking starting with intent."
  paraphrase: "A Beyond All Reason map editor built in the Godot engine. Its ultimate purpose is to load and edit map layers and terrain data. Version 1 stops short of editing: it opens a map and lets the user see and inspect every asset the map contains, with camera and navigation components good enough to move around a large map quickly. The full specification and task breakdown are written before any code."
  input_type: "Seed"
  capture_timestamp: "2026-09-20T19:45:00Z"

  # --- Stage 0.2: Classify ---
  domain_tags:
    - "Game development tooling / content authoring"
    - "Real-time strategy (Beyond All Reason, Recoil engine)"
    - "3D geospatial and terrain visualisation"
  intent_archetype: "Product"
  motivation_hypothesis: "The user appears motivated by the absence of a modern, visual, read-first way to see what a BAR map contains - the existing chain is one-way compilers plus an in-engine editor - and, having already built an exact headless decoder in rjm.bar.tools, wants that fidelity presented interactively and wants to reach editing from a foundation that already reads everything correctly."

  # --- Domain Research Context ---
  analogous_concepts:
    - "SpringBoard - in-engine Spring/Recoil map and scenario editor; constrained by engine serialisation and live-texture limits"
    - "MapConv / PyMapConv - one-way SMF compilers and decompilers, no viewer"
    - "SpringMapEdit - community-described complete solution, explicitly non-visual"
    - "Beherith springrts_smf_compiler - the compiler the official BAR mapping guide recommends"
    - "rjm.bar.tools (smrt-bar-map) - the user's own exact Rust decoder for .sd7, SMF and mapinfo.lua"
    - "Terrain3D / HTerrain / TerraBrush / M Terrain - Godot 4 terrain systems; build-vs-adopt menu for the editing phase"
    - "hendkai/bar-map-generator - BAR map generation, adjacent output format"
  market_maturity: "Nascent-to-stagnant (BAR map tooling) inside two Growing platforms (Godot 4 as a tool platform; BAR, Hooded Horse deal June 2026)"
  key_trends:
    - "The BAR authoring chain is one-way; seeing a finished map normally means launching the game"
    - "In-engine editing is structurally constrained - no full map serialisation, live-texture editing unsolved"
    - "Godot 4 GDExtension has crossed the threshold for native-speed custom-format tooling"
    - "An exact decoder already exists in the workspace (rjm.bar.tools); consume, port or reimplement is an open question"
    - "BAR is commercialising, enlarging the map-author audience"
    - "mapinfo.lua is an executable Lua program, forcing a sandboxed Lua 5.1 + VFS shim into any faithful reader"

  # --- Stage 0.3: Explore ---
  historical_context: "Inspecting a BAR map currently means launching the game or decompiling it to loose images, because the authoring chain is one-way; and editing map layers and terrain is the real goal, for which loading and displaying correctly is the foundation. Now is the moment because an exact decoder already exists in the workspace (rjm.bar.tools), Godot 4's GDExtension has matured enough for native-speed custom-format tooling, and BAR's commercialisation is enlarging the map-author population."
  stakeholders:
    primary_beneficiaries:
      - "The user - a personal instrument answerable to no one else's workflow"
      - "BAR map reviewers - two volunteers per map who must verify the published Map Checklist, currently with no preview tool"
      - "Map authors - who need to see their compiled .smf/.smt mid-build without launching the game"
      - "AI and tooling developers - verifying derived terrain analysis visually against the map it came from"
    secondary_stakeholders:
      - "BAR players, who receive better-reviewed maps"
      - "The BAR maps team and the maps-metadata / map_blueprint repositories"
      - "rjm.bar.ai and s3k-CircuitAI, whose map reasoning uses the same grids"
      - "Recoil engine maintainers, whose shader chain becomes the parity target"
    potential_resistors:
      - "SpringBoard and its maintainers - functional overlap at the edges"
      - "Authors invested in the existing World Machine + SpringBoard workflow"
      - "Anyone who would treat a drifting non-engine renderer as authoritative"
  value_vision: "Pick any of the 254 installed maps, it opens, it looks like the game, and you can get anywhere on it quickly. Breadth is the correctness test: a tool that opens all 254 has proven what a tool rendering one map beautifully has not."
  constraints:
    resources:
      - "One developer, volunteer time"
      - "A 254-archive local corpus serving as both test suite and acceptance criterion"
      - "An existing citation-backed Rust decoder (rjm.bar.tools) to build on"
    skills_gap:
      - "Godot 4 and the godot-rust/gdext build toolchain"
      - "Porting Recoil's GLSL SMF shader chain to Godot's shading language"
      - "DXT1 decoding and .smt tile-atlas assembly"
      - "S3O parsing (tractable - fully documented in rts/Rendering/Models/s3o.h)"
      - "Large-terrain LOD and streaming in Godot"
    regulatory:
      - "Recoil is GPL v2 or later; porting its SMF shaders makes this editor a derivative work, effectively GPL v2-or-later"
      - "Map archives are third-party content; the editor displays but must not redistribute them"
    technical:
      - "mapinfo.lua is executable Lua 5.1 with VFS.Include overlays and map-option branching; needs a sandboxed interpreter"
      - ".smt tile atlas is the bulk of an archive (25 MB of 49 MB), tile-indexed DXT1; not decoded by rjm.bar.tools today"
      - "Minimap block is DXT1 with 9 mips at a fixed 699048 bytes; also not decoded today"
      - "Maps run to 32x32 (16384 world units per side); terrain streaming and LOD are load-bearing"
      - "Godot has native importers for none of SMF, SMT, S3O, DDS skybox cubemaps, or engine-slot TGA/BMP detail textures"
      - "Parity requires feeding ten engine shader texture slots exactly, plus the mapinfo.lua grid overrides"
      - "gdext still ships occasional breaking changes; a pinned version and deliberate upgrade cadence are required"
  anti_goals:
    - "A purpose-built tool for Recoil - not a general-purpose terrain editor, not a Godot terrain plugin, not multi-engine (user's own words)"
    - "It does not guess - engine rules are reproduced with citations; a value that cannot be derived exactly is reported as unknown (derived, pending confirmation)"
    - "Not a scenario or gadget editor - terrain and map layers only (derived, pending confirmation)"
    - "Not a map generator - it reads and eventually edits maps that exist (derived, pending confirmation)"
  exploration_themes:
    - "Exactness is the identity, inherited from rjm.bar.tools"
    - "Read before write, deliberately - editing a format you cannot display correctly is how the existing tools became one-way"
    - "Breadth as proof - 254 maps is a conformance suite disguised as a goal"
    - "One tool, four audiences, one artifact - no audience gets a separate mode"
    - "Parity as the product - purpose-built plus visual parity makes this a reference renderer for the format"
  tensions:
    - "Scope versus a solo maintainer - parity, full fidelity, two input backends, four audiences and corpus-wide conformance is a large v1, and scope creep is the top failure mode in this space"
    - "Parity versus independence - tracking Recoil's shaders couples the tool to a moving upstream; parity must be versioned against an engine commit"
    - "A read-only v1 at parity cost - justified only if rendering is genuinely the foundation editing stands on"
    - "Reviewer trust - if reviewers adopt it for checklist verification, silent approximation stops being an inconvenience and becomes a defect"
  research_findings:
    analogous_audiences:
      - "Reviewers and authors work through Discord threads with a @mapper role ping, a posted map file, and at-least-two-reviewer approval against a written checklist"
      - "Their recommended toolchain authors, converts or packs - World Machine, Photoshop/GIMP, Beherith's SMF compiler, SpringBoard, PyMapConv, a pathability-checker script, a rotating-features converter, 7-Zip - and none of it shows a finished map"
    market_gaps:
      - "No external viewer for a compiled BAR map; inspection means launching the game"
      - "The Map Checklist is a list of verifiable properties with no tool that surfaces them together - the closest thing to a pre-written requirements source this project has"
      - "In-engine editing is structurally constrained: delta-heightmap-only saves, live-texture editing unsolved"
      - "Derived terrain analysis has no visual verification path"
    failure_patterns:
      - "Scope creep - the most-cited cause, at both hobby and studio scale"
      - "Single-maintainer abandonment, made terminal by licensing or documentation that blocks continuation"
      - "Format drift - tools pinned to a format snapshot break when the engine moves; Tiled's counter-example held a stable format promise for 15 years"
      - "Authoring-first design - building the writer before the reader is how this domain arrived at a one-way chain"
      - "Inheriting the host's limits by building inside the engine"

  # --- Stage 0.4: Assess ---
  assessment_report: "docs/requirements/intent/framework-assessment.md"
  frameworks_applied: ["PF-01 Big Five", "PF-03 SDT", "PF-04 Ikigai", "PF-05 Logotherapy", "PF-06 Eudaimonia", "PF-07 Flow", "PF-08 VIA", "Value Dimensions"]
  value_dimensions:
    physical: 2
    emotional: 3
    intellectual: 5
    spiritual: 4
    composite_profile: "Intellectual-dominant, Spiritual-strong, moderate Emotional, peripheral Physical"
  ikigai:
    love: "Strong"
    skill: "Strong, with a located weak spot - Godot, gdext, GLSL porting, LOD all unevidenced"
    need: "Strong - externally validated by the BAR review process and checklist"
    viability: "Moderate - undefined rather than blocked; no revenue intent stated"
    intersection: "Mission with a Passion core - three strong legs"
  sdt_motivation: "Integrated regulation shading into Intrinsic; Autonomy and Competence very high, Relatedness weakest"
  logotherapy_pathway: "Creative - with a genuinely unique assignment: exact decoder, provenance-graded knowledge base, engine fluency and downstream consumers held by one person"
  assessment_strengths:
    - "Motivation is structurally sustainable in a niche with no revenue path (PF-03; PyMapConv and SpringBoard both still maintained)"
    - "Need is externally validated by the community's own review process and checklist (PF-04)"
    - "Three strong Ikigai legs - Love, Skill, Need (PF-04)"
    - "The goal is flow-shaped: binary, externally verifiable, immune to self-grading (PF-07)"
    - "The assignment is genuinely unique and not interchangeable (PF-05)"
    - "Personality fits the task - high Openness with very high orderliness (PF-01)"
    - "Signature strengths are load-bearing: Honesty and Judgment are what a reference renderer's claim rests on (PF-08)"
    - "Eudaimonic profile strong on five of six dimensions (PF-06)"
  assessment_gaps:
    - "The fidelity bar sits in the weakest skill quadrant - sequence the decoder to done first, phase parity later (PF-04 Skill, PF-07)"
    - "A feedback desert before first pixel - order the work so a greybox heightmap and flying camera arrive early (PF-07)"
    - "Very high orderliness against an unbounded target - write the phase edge down (PF-01, PF-06)"
    - "Relatedness, Justice virtues and Emotional value all thin at the same point - recruit an early community user or scope personal-first (PF-03, PF-08, Value Dimensions)"
    - "Viability undefined rather than weak - decide whether this is a personal tool, a community commons, or a funded project (PF-04)"
    - "Serving reviewers may need a checklist panel, not just a viewer - a different product surface (Value Dimensions Emotional, PF-03)"

  # --- Stage 0.5: Refine ---
  refinement_cycles:
    - cycle: 1
      insights_presented:
        - "PF-01/PF-06: very high orderliness aimed at an unbounded target has no natural stopping point"
        - "PF-07: a feedback desert sits before first pixel"
        - "PF-03/PF-08/Value Dimensions converge: four audiences named, two communal, none engaged"
      research_conducted:
        - "Godot does not load DDS at runtime - it converts to CTEX at import, so the extension must own texture decoding"
        - "Terrain3D is a GPU clipmap terrain supporting 64 m to 65.5 km with 10 mesh LODs; a 16384-unit BAR map fits comfortably"
      questions_asked:
        - "How should v1 be cut into phases? (gap-driven)"
        - "Personal-first or community-first? (contrast)"
        - "Licence and distribution? (gap-driven)"
        - "Adopt Terrain3D or own the terrain mesh? (reframing)"
      user_responses: "V1 is itself one phase and is not to be sub-divided. Recruit one BAR mapper as an early user now. GPL v2-or-later, public from the start. Adopt Terrain3D for mesh and LOD."
      record_changes:
        - "Phase registry will divide the project (explore, then edit), not v1"
        - "Licence decided: GPL v2-or-later, public"
        - "Terrain3D adopted as a dependency"
        - "Relatedness gap addressed by recruiting an early user"
      convergence_check: "partial"
    - cycle: 2
      insights_presented:
        - "Terrain3D shader_override_enabled fully replaces the generated terrain shader, so adopting it does not forfeit parity"
        - "Terrain3D is MIT and compatible with the GPL decision; C++ GDExtension for Godot 4.4+"
      research_conducted:
        - "Shader override emits the generated shader as a starting point when enabled blank"
        - "Terrain3D's material is built for 32 splatted texture sets; BAR's ground is a tile-indexed DXT1 atlas, so the override will be near-total"
        - "A Rust extension calling a second C++ extension's classes is dynamic rather than typed - flagged for the TDD"
      questions_asked:
        - "Does one-phase delivery also fix the internal work order? (constraint testing)"
        - "What should the recruited mapper do on first look? (gap-driven)"
        - "Three years out, which success sentence is it? (future projection)"
        - "If one commitment must be sacrificed, which? (constraint testing)"
      user_responses: "Build in strict dependency order - decoder first, then rendering. The early user flies a map they made themselves. Success at three years is that BAR mappers reach for it. Visual parity is the first thing sacrificed."
      record_changes:
        - "Community adoption promoted from side effect to destination; distribution and usability become part of the intent"
        - "Priority stack established: open-and-navigable > all asset classes > exploration speed > visual parity"
        - "Parity reclassified as target and designated sacrifice, resolving the central tension without abandoning the goal"
        - "Primary persona written: the BAR mapper"
        - "Feedback desert and whole-v1 scope recorded as knowingly accepted risks"
      convergence_check: "pass"
  priority_stack:
    - "1. Every installed map opens and is navigable (protected)"
    - "2. Every asset class is represented (protected)"
    - "3. Exploration is fast - camera and navigation (protected)"
    - "4. Visual parity with Recoil (target, and the first thing cut)"
  primary_persona: "The BAR mapper - makes or reviews maps for Beyond All Reason; currently must launch the game to see a finished map, and verifies a written checklist as an unpaid volunteer. Needs to open a map they know, move through it immediately, and trust that what they see is what the game would show."
  accepted_risks:
    - "The feedback desert is accepted: strict dependency order means the decoder reaches done before anything renders"
    - "V1 stays whole, decided twice; scope creep is mitigated by the priority stack rather than by sub-phasing"
    - "Terrain3D is a second GDExtension dependency with its own data and material model, reached dynamically from Rust"

  # --- Stage 0.6: Converge ---
  convergence_assessment:
    problem_clarity: true
    audience_defined: true
    value_articulated: true
    motivation_grounded: true
    feasibility_acknowledged: true
    boundaries_set: true
    user_confidence: true
    overall: "pass (all seven)"
    notes: "Converged in the minimum two cycles. No non-convergence signals: no conflated intents, no scope inflation - scope narrowed through the priority stack - and no decision avoidance."
```

## Quality gates

### Stage 0.1-0.2

| Gate | Status |
| --- | --- |
| User confirms the paraphrase captures their meaning | Met - confirmed by proceeding to explore |
| Input type classified | Met - Seed |
| Domain tags and archetype assigned | Met - 3 tags, Product |
| Domain research conducted, 3 or more analogous concepts | Met - 7 concepts, maturity split, 6 trends |
| Motivation hypothesis exists | Met - recorded above, unvalidated by design |

### Stage 0.3

| Gate | Status |
| --- | --- |
| At least two rounds of guided questioning complete | Met - 2 rounds, 8 questions, each preceded by research |
| Primary and secondary stakeholders identified | Met - 4 primary, 4 secondary, 3 resistor groups |
| Anti-goals explicitly stated (at least 2) | Met - 1 stated by the user, 3 derived and flagged as pending confirmation |
| Constraints documented | Met - technical, skills, licensing, resources |
| Domain research informed the questioning | Met - Map Review Process, Map Checklist, mapmaking resources, gdext maturity, s3o.h, SMF shader paths |
| Failure patterns investigated | Met - 5 patterns, with a counter-example |
| User confirms the enriched record captures their thinking | Met - confirmed by directing the assessment, brief and business requirements to proceed on this record |

### Stage 0.4

| Gate | Status |
| --- | --- |
| At least 4 of 8 frameworks applied | Met - 7 of 8, PF-02 exclusion justified |
| Ikigai and Value Dimensions included | Met |
| All four value dimensions scored | Met - 2 / 3 / 5 / 4 |
| Strengths and gaps identified with framework citations | Met - 8 strengths, 6 gaps |
| Research evidence supports key assessments | Met - 10 sources |
| Comparable ventures investigated | Met - Tiled funding; PyMapConv and SpringBoard longevity |

### Stage 0.5-0.6

| Gate | Status |
| --- | --- |
| Minimum 2 refinement cycles completed | Met - 2 cycles, 8 questions, 5 strategy types used |
| All 7 convergence criteria met | Met on 6; user confidence **pending** |
| Research conducted during refinement | Met - Godot runtime DDS handling, Terrain3D clipmap limits, shader override, material model, licence |
| Critical assessment gaps addressed or deferred | Met - all six addressed; see below |
| User confirms the refined intent captures their vision | Met - confirmed 2026-09-20; the Intent Brief built on it was approved and P01 business requirements baselined |
| Contradictions between goals and framework insights resolved | Met - the parity-versus-scope tension resolved by the priority stack |

### Disposition of the six assessment gaps

| Gap | Disposition |
| --- | --- |
| Fidelity bar in the weakest skill quadrant | **Addressed** - parity ranked last and designated the sacrifice; strict dependency order puts the decoder, the strongest quadrant, first |
| Feedback desert before first pixel | **Accepted knowingly** - strict dependency order chosen with the cost understood; mitigation deferred to task ordering in the TDD |
| Orderliness against an unbounded target | **Addressed** - the priority stack gives v1 a floor that does not depend on parity landing |
| Relatedness, Justice virtues and Emotional value thin | **Addressed** - an early user is to be recruited, and community adoption is now the three-year success condition |
| Viability undefined | **Addressed** - resolved as community commons: GPL v2-or-later, public from the start |
| Reviewers may need a checklist panel, not a viewer | **Deferred** - the persona is defined by flying a map they made; checklist surfacing remains an open question for the PRD |

## Constraints established during capture

1. **Specification before code.** No Godot project, dependency or script until
   the task breakdown authorises it.
2. **Godot is the engine.** Stated, not derived. Version and language
   (GDScript / C# / GDExtension) remain open.
3. **Version 1 is read-only.** Load and explore all assets. Editing of layers
   and terrain is the second intent and a later version.
4. **Camera and navigation are in v1 scope by name**, as components, and are
   judged on how quickly a map can be explored.
5. **"All of a map's assets"** is the coverage bar, not "the terrain".
6. **Read-only reference trees.** BAR and Recoil checkouts are inspected, never
   written. Game facts come from `../rjm.bar.docs/knowledge/`.

## Open questions

### Resolved in stage 0.3

| Question | Answer |
| --- | --- |
| Who is v1 for? | All four: the user, map reviewers, map authors, AI/tooling developers |
| Does "all assets" include what Godot cannot natively render? | Yes - everything, rendered as faithfully as possible |
| Where does decoding live? | Rust core via GDExtension (`godot-rust/gdext`), reusing and extending `rjm.bar.tools` |
| Can that carry the Lua 5.1 sandbox? | Yes - `rjm.bar.tools` already contains it (`src/luaenv.rs`) |
| Engine parity or data-honest rendering? | Visual parity with Recoil |
| `.sd7` only, or unpacked folders too? | Both |
| What must it not become? | A general-purpose tool - it is purpose-built for Recoil |

### Carried forward to assess-intent and write-intent-brief

- **How is v1 phased?** Parity, full fidelity, two input backends, four
  audiences and corpus-wide conformance in a single release is the shape that
  history says fails. The phase registry has to cut this, and the cut is a
  vision question, not a scheduling one.
- What does "explore quickly" mean measurably - time to first render of a 32x32
  map, sustained frame rate while flying, time to locate a named feature?
- What does parity mean *operationally*? Pinned to an engine commit and
  `verified_against` like `rjm.bar.docs`, or tracked continuously? Is there a
  test that can fail when the engine's shaders move?
- Which map options are evaluated at load? 154 of 227 maps read them and some
  rewrite the height range. Does v1 expose an option picker or assume engine
  defaults, as `rjm.bar.tools` does?
- Is GPL v2-or-later accepted as the licence, given that shader parity makes the
  editor a derivative of Recoil?
- Does v1 write anything at all - exports, screenshots, session state - or is it
  strictly read-only on disk?
- Are the three derived anti-goals (no guessing, no scenario editing, no
  generation) confirmed?
- Is `LuaGaia` gadget code shown as source only, or must its runtime effect on
  the map be represented?
- Does serving reviewers imply anything beyond rendering - a checklist panel
  that surfaces the verifiable properties together - or is that a later phase?
