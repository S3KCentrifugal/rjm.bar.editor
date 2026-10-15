---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
stage: BRD context research
status: complete
gathered: 2026-09-20
source_plan: docs/requirements/phases/phase-01-map-asset-exploration/business-requirements/readiness-analysis.md
next: draft-brd (phase P01)
---

# Source Bundle: P01 - Map Asset Exploration (MAE)

All findings gathered **2026-09-20** unless stated. Facts are cited; inferences
are labelled and listed separately at the end. Two reference-key namespaces
extend the standard set because two required research areas have no commercial
equivalent: **`MR-CORP`** (corpus characterisation, original primary research)
and **`MR-CHK`** (requirements mining from the published Map Checklist). Both
are documented here rather than silently folded into another key.

## Competitor analysis

| # | Competitor | Type | Offering | Pricing | Scale | Strengths | Weaknesses | Source |
|:--|:---|:---|:---|:---|:---|:---|:---|:---|
| MR-COMP-01 | **SpringBoard** | Direct | In-engine map and scenario editor for Spring/Recoil | Free, community | Core module updated March 2026; installers and Discord support | The incumbent; scenario authoring; object placement; official guide recommends it | Runs inside the engine: no full map serialisation (Spring saves only a delta heightmap), live-texture editing documented as an unsolved area | [SpringBoard docs](https://springboard-core.readthedocs.io/), 2026-09-20 |
| MR-COMP-02 | **PyMapConv / springrts_smf_compiler** | Indirect | SMF compiler and decompiler | Free, community | Actively maintained; recommended by the official BAR guide | Round-trips the format; recommended for minimap lightening | One-way batch tooling; no viewer, no interactive inspection | [Beherith/springrts_smf_compiler](https://github.com/Beherith/springrts_smf_compiler), 2026-09-20 |
| MR-COMP-03 | **SpringMapEdit** | Direct | Community-described "only complete map making solution" | Free, community | Long-standing | Complete feature coverage | Described by its own community as "in no way visual", with arbitrary texture-size rules | [Spring RTS forum](https://springrts.com/phpbb/viewtopic.php?t=29865), 2026-09-20 |
| MR-COMP-04 | **tebeer/BARMapEdit** | Direct | A **Unity-based** map editor for Beyond All Reason | None stated | 22 commits, **0 stars, 0 forks, no releases** | Direct evidence that the external-game-engine editor approach is being reached for independently | Early-stage, unadopted, no documented format support or scope statement | [tebeer/BARMapEdit](https://github.com/tebeer/BARMapEdit), 2026-09-20 |
| MR-COMP-05 | **hendkai/bar-map-generator** | Indirect | Browser procedural generator; builds full BAR map assets and packages them | Free, community | Self-compiling packages, cross-platform build scripts | Generates heightmaps, metalmap, textures; OpenStreetMap import | **Delegates SMF compilation to PyMapConv** rather than writing the format; generates rather than inspects | [hendkai/bar-map-generator](https://github.com/hendkai/bar-map-generator), 2026-09-20 |
| MR-COMP-06 | **Terrain3D** | Not a competitor - adopted dependency | Godot 4 GPU clipmap terrain, C++ GDExtension | Free, MIT | 64 m to 65.5 km, 10 mesh LOD levels, 32 texture sets | `shader_override_enabled` fully replaces the generated terrain shader; MIT is GPL-compatible | Material model assumes 32 splatted texture sets, not a tile-indexed atlas; requires Godot 4.4+ | [TokisanGames/Terrain3D](https://github.com/TokisanGames/Terrain3D), [Terrain3DMaterial docs](https://terrain3d.readthedocs.io/en/stable/api/class_terrain3dmaterial.html), 2026-09-20 |
| MR-COMP-07 | **Tiled** | Indirect - sustainability benchmark | General-purpose level editor, 15+ years | Donation: ~200 sponsors, roughly $3,600/month | Funds two days a week of maintainer time | Longevity via a narrow, stable format promise | Serves many engines; a BAR-only tool draws from a far smaller pool | [Tiled on Open Collective](https://opencollective.com/tiled), 2026-09-20 |

**Finding.** No adopted external BAR map viewer exists. Of seven comparable
solutions, six author, convert, compile or generate; none provides interactive
inspection of a finished map outside the engine. The single direct attempt at
an external editor (MR-COMP-04) has zero adoption.

## Requirements mining - the BAR Map Checklist

The published checklist is the properties two volunteer reviewers must verify
before a map is accepted. It is the closest thing to a pre-written requirements
source for P01, and each row is a candidate capability.

| # | Checklist property | What an editor must show to make it verifiable | Source |
|:--|:---|:---|:---|
| MR-CHK-01 | Map must not exceed 32x32 in any dimension | World extent from the SMF header | [BAR Map Checklist](https://www.beyondallreason.info/guide/map-checklist), 2026-09-20 |
| MR-CHK-02 | Full pathability validation, with correct vehicle / bot / all-terrain distinctions | Slope and depth per cell, resolved by movement class | same |
| MR-CHK-03 | Texture map clean on paths and chokepoints; wild designs only at edges and cliffs | The ground texture as rendered, navigable | same |
| MR-CHK-04 | Splat textures minimum 1024 px, OpenGL-compatible | Source texture dimensions and formats, reported | same |
| MR-CHK-05 | Normals map at highest possible resolution, ideally 1:1 with the texture map | Normal map dimensions relative to texture dimensions | same |
| MR-CHK-06 | Specular map subtle; no overly bright or shiny surfaces | Specular contribution visible under map lighting | same |
| MR-CHK-07 | Metal map sane distribution with sharp shapes | Metal map as an overlay in world space | same |
| MR-CHK-08 | Type map optional but available | Type map as an overlay | same |
| MR-CHK-09 | Grass map required | Grass map presence and coverage | same |
| MR-CHK-10 | Minimap may require lightening | The minimap as stored | same |
| MR-CHK-11 | Topolines subtle, thin, naturally distorted; not on flat or steep terrain | Ground texture plus normal map at close range | same |
| MR-CHK-12 | Splat distribution balanced, organic, not repetitive | Splat distribution channel, inspectable | same |
| MR-CHK-13 | Features placed throughout; geovents buildable with T2 geos; correct normals and shaders; no fixed Y values; correct reclaim values | Feature placements in world, with per-feature metadata from `features/*.lua` | same |
| MR-CHK-14 | Sun on the northern map area; angle neither too low nor too high | Sun direction from `mapinfo.lua`, shown against the terrain | same |
| MR-CHK-15 | Map lighting darkened for unit visibility; feature lighting adjusted | Ambient, diffuse and specular values from `mapinfo.lua` | same |
| MR-CHK-16 | Wind between 0 and 30; tidal between 0 and 25 | Values from `mapinfo.lua`, with the valid range shown | same |
| MR-CHK-17 | Skybox and fog validated for density and thematic consistency | Sky cubemap and fog as rendered | same |
| MR-CHK-18 | Water parameters tested against reference maps | Water material values and rendering | same |
| MR-CHK-19 | Metal values sane and tested for balance | Metal map plus extractor radius and `maxMetal` | same |
| MR-CHK-20 | Fixed start positions configured; `teams{}` set for max start positions and team counts | Start positions in world, with team indices | same |

**Finding.** Of twenty checklist properties, the large majority are answerable
by displaying decoded map data faithfully. This is the strongest available
evidence that P01's read-only scope already serves the reviewer audience, and
it directly informs the open question about whether a checklist panel is
needed.

**Review process context.** Maps are posted in the `#mapping` Discord channel
in a thread, with the `@mapper` role pinged; **at least two reviewers** must
approve, or `@IceXuick` breaks a tie. Both reviewers and mapmakers are
volunteers, and the guidance asks reviewers to avoid moving goalposts and to
point at documented standards. Source:
[BAR Map Review Process](https://www.beyondallreason.info/guide/map-reviews-process), 2026-09-20.

## Corpus characterisation - original primary research

Method: every `.sd7` in the local library was listed with 7-Zip and its entries
classified. Conducted 2026-09-20 against
`%LOCALAPPDATA%\Programs\Beyond-All-Reason\data\maps`. This is direct
measurement of the population P01 must satisfy, not a citation.

| # | Finding | Value | Implication for P01 |
|:--|:---|:---|:---|
| MR-CORP-01 | Archives in the library | **254 `.sd7`**, plus **1 unpacked map directory** (`supreme_isthmus_v1.8/`) | Corrects the earlier figure of 510, which counted one `.md5.gz` per archive. The unpacked directory proves folder input is a real requirement, not hypothetical |
| MR-CORP-02 | SMF files per archive | **Exactly 1 in all 254** | No multi-SMF handling needed in practice |
| MR-CORP-03 | SMT files per archive | **Exactly 1 in all 254** | The format permits multiple tile files; the population does not exercise it. A scope reduction, provided the reader still parses the count field correctly |
| MR-CORP-04 | `mapinfo.lua` present | **253 of 254 (99.6%)** | The sole exception is `mescaline_v2.sd7` |
| MR-CORP-05 | Legacy `.smd` (TDF) definition present | **3 of 254**: `mescaline_v2`, `titan_v3.1`, `titanduel_v3` | Two of the three also carry `mapinfo.lua`; only `mescaline_v2` depends on `.smd` alone. The engine still supports the fallback (`rts/Map/MapParser.cpp:30`) |
| MR-CORP-06 | Optional texture slots are genuinely optional | Normal map absent in **5**; specular absent in **9**; splat absent in **4** | Named archives include `speedmetal_bar_v2`, `tma20x_1.8`, `prismatic_anomaly_v1.02`, `supreme_crossing_2.2.1`, `avalanche_3.4`, `tabula_remake_1.5`. Absent slots must render correctly, not fail |
| MR-CORP-07 | Sky present | **88 of 254 (34.6%)** | A skybox is the exception, not the rule. Default sky behaviour matters more than cubemap loading |
| MR-CORP-08 | Features directory present | **194 of 254 (76.4%)** - absent in 60 | Roughly a quarter of maps have no features; feature-less rendering is a normal case |
| MR-CORP-09 | `LuaGaia` present | **236 of 254 (92.9%)** | Map-side Lua is near-universal |
| MR-CORP-10 | `mapoptions.lua` present | **241 of 254 (94.9%)** | Corroborates the earlier `rjm.bar.tools` finding that most maps read options, some of which rewrite the height range |
| MR-CORP-11 | Archive file counts | min **10**, max **436**, mean **96** | A 40x spread; the reader must handle both trivial and very large archives |

## Trend analysis

| # | Trend | Direction | BRD implication | Timeline | Source |
|:--|:---|:---|:---|:---|:---|
| MR-TREND-01 | BAR commercialising - Hooded Horse publishing deal June 2026, Steam release pending | Opportunity | A larger, less expert author population raises the value of inspection tooling | Now to 1-2 years | [Notebookcheck on BAR engine update](https://www.notebookcheck.net/Open-source-RTS-Beyond-All-Reason-showcases-large-scale-engine-update-as-player-interest-climbs.1014152.0.html), 2026-09-20 |
| MR-TREND-02 | BAR map infrastructure has centralised: `maps-metadata` is the source of truth, exported from a Rowy instance, carrying playercount, startboxes and `maxPlayersPerStartbox`, with approved changes distributed automatically within minutes | Opportunity | A defined integration surface that did not exist at intent stage; a potential alignment target for editor output | Current | [beyond-all-reason/maps-metadata](https://github.com/beyond-all-reason/maps-metadata), 2026-09-20 |
| MR-TREND-03 | `beyond-all-reason/mapfeatures` holds features certified to work with Recoil | Opportunity | A canonical feature set the editor could validate placements against | Current | [beyond-all-reason/mapfeatures](https://github.com/beyond-all-reason/mapfeatures), 2026-09-20 |
| MR-TREND-04 | Godot 4 GDExtension matured; `gdext` usable for tools and editor plugins with most Godot APIs mapped, though breaking changes still occur | Opportunity with a maintenance cost | Supports the Rust-core decision; requires a pinned version and deliberate upgrade cadence | Current | [godot-rust book](https://godot-rust.github.io/book/), 2026-09-20 |
| MR-TREND-05 | Godot does **not** load DDS at runtime - DDS is converted to CTEX at import | Constraint | Map textures arrive from arbitrary archives at runtime, so the import pipeline is unusable and the extension must own texture decoding | Current | [Godot importing images docs](https://github.com/godotengine/godot-docs/blob/master/tutorials/assets_pipeline/importing_images.rst), [runtime DDS PR #69085](https://github.com/godotengine/godot/pull/69085), 2026-09-20 |

## Regulatory and licensing landscape

No statutory regime applies. The binding constraints are licensing.

| # | Instrument | Jurisdiction | Applies to | Key obligations | Category | Status | Source |
|:--|:---|:---|:---|:---|:---|:---|:---|
| MR-REG-01 | **GPL v2-or-later** | Universal (copyright) | Recoil engine code, including the SMF shader chain | Porting engine shaders makes this editor a derivative work; source must be offered under compatible terms | COMP-LIC | Current, accepted | `bar-RecoilEngine/LICENSE`; `cont/base/springcontent/shaders/GLSL/SMF*.glsl`, 2026-09-20 |
| MR-REG-02 | **MIT** (Terrain3D, Godot) | Universal | Adopted dependencies | Attribution; compatible with GPL v2-or-later distribution | COMP-LIC | Current | [TokisanGames/Terrain3D](https://github.com/TokisanGames/Terrain3D), 2026-09-20 |
| MR-REG-03 | **Third-party map content** | Universal | The 254 archives and any map opened | The editor displays map content; it must not redistribute it | COMP-CONTENT | Current | Map archives carry their authors' terms, 2026-09-20 |

## Customer / user research

### Persona: The BAR mapper (author or reviewer)

- **Context.** Volunteer. Authors maps with World Machine or a paint tool,
  compiles with Beherith's SMF compiler, places objects in SpringBoard, packs
  with 7-Zip. Submits through a `#mapping` Discord thread with the `@mapper`
  role pinged. Reviewers are experienced mapmakers from the BAR team; at least
  two must approve. Named mappers are publicly credited on the BAR site -
  Beherith, Nikuksis, BasiC, Takeo Kurita, RebelNode, Phalange, IceXuick.
- **Pain point.** To see a finished map they must launch the game. To review
  one they must verify twenty checklist properties with no tool that surfaces
  them; the only validation utility the official resources list is a SpringBoard
  pathability-checker script and a Python rotating-features converter.
- **Current alternatives.** Launch BAR; decompile with PyMapConv to loose
  images; inspect in SpringBoard inside the engine.
- **Evidence.** MR-USER-01: [BAR Map Review Process](https://www.beyondallreason.info/guide/map-reviews-process), 2026-09-20.
  MR-USER-02: [BAR Mapmaking Resources](https://www.beyondallreason.info/guide/mapmaking-resources), 2026-09-20.
  MR-USER-03: [BAR Map Checklist](https://www.beyondallreason.info/guide/map-checklist), 2026-09-20.
  MR-USER-04: [BAR maps directory, authors credited](https://www.beyondallreason.info/maps), 2026-09-20.

### Adoption and willingness to pay

- **Willingness to pay: none expected.** Every comparable tool in this domain
  is free and community maintained (MR-COMP-01 through MR-COMP-05). The only
  monetised comparable is Tiled, by donation, and it serves a far broader
  market (MR-COMP-07).
- This confirms the refinement decision to treat viability as commons rather
  than revenue, and means P01's objectives must be capability- and
  adoption-shaped.

## Audience sizing

The commercial TAM/SAM/SOM frame does not apply to a free community tool. The
equivalent question - how many people author or review BAR maps - is **not
publicly documented**.

| Metric | Estimate | Methodology | Source |
|:---|:---|:---|:---|
| MR-SIZE-01 | Live maps listed on the official BAR site | ~130+ across 13 pages of listings | Direct observation of the maps directory | [BAR maps](https://www.beyondallreason.info/maps), 2026-09-20 |
| MR-SIZE-02 | Maps installed in one player's local library | 254 archives, including multiple versions of the same map | Direct measurement (MR-CORP-01) | This analysis, 2026-09-20 |
| MR-SIZE-03 | Active map authors | **Unknown** | Publicly credited mappers are individually named on the BAR team and map pages, but no roster count or submission rate is published | Gap - see Data Gaps |
| MR-SIZE-04 | Active reviewers | **Unknown, but small** | The process names a `@mapper` role and a single named tie-breaker (`@IceXuick`), implying a small group | Inference - see Inferences |

## Data gaps

Recorded honestly rather than estimated.

1. **No primary user research exists or was found.** No surveys, interviews or
   structured feedback from BAR mappers about inspection workflows are publicly
   available. The persona is reconstructed entirely from published process
   documents. **Recommendation:** this is exactly what the intent's
   early-user recruitment is for. Ask one reviewer and one author directly what
   they check first and what would make an external tool worth opening. Until
   then, every persona-derived requirement should be treated as provisional.
2. **Active mapper and reviewer counts are unpublished**, as is the map
   submission rate. **Recommendation:** ask in the `#mapping` channel, or
   derive an approximation from `maps-metadata` commit history and
   `beyond-all-reason/Maps` releases. Not a blocker for P01, which is scoped by
   corpus rather than by audience size.
3. **BARMapEdit's actual capabilities are undocumented.** Its README does not
   state format support or scope. **Recommendation:** read the repository
   directly before making any claim of novelty in public material.
4. **No adoption-path evidence.** How a new tool actually reaches this
   community - announcement channel, packaging expectations, platform coverage -
   is not documented anywhere found. Relevant because the three-year success
   condition is adoption.
5. **The map popularity statistics site** surfaced in search
   (`dazazzell.github.io/barmaps`) returned HTTP 404 and could not be used.

## Inferences

Distinguished from the facts above.

| # | Inference | Basis | Confidence |
|:---|:---|:---|:---|
| INF-01 | The reviewer group is small - plausibly under a dozen | The process names one tie-breaking individual and a single Discord role; volunteers are asked to respect each other's time | Medium |
| INF-02 | P01's read-only scope already serves most reviewer needs | 20 of 20 checklist properties are answerable by faithfully displaying decoded map data (MR-CHK-01 to MR-CHK-20) | High - derived from the published checklist, but unvalidated by any reviewer |
| INF-03 | Multi-SMT support can be deprioritised without risk | Zero of 254 archives use more than one SMT (MR-CORP-03) | High for the current corpus; the format still permits it, so the count field must be parsed even if only one file is handled |
| INF-04 | Absent texture slots are a normal case, not corruption | 5, 9 and 4 archives lack normal, specular and splat respectively, and all are playable shipped maps (MR-CORP-06) | High |
| INF-05 | Adoption would be driven by individual mappers rather than an official endorsement | Every comparable tool is an individual's project that the guides later reference | Medium |

## Reference key index

| Key | Area | Title | Source | Date |
|:---|:---|:---|:---|:---|
| MR-COMP-01 | Competitor | SpringBoard in-engine editor | springboard-core.readthedocs.io | 2026-09-20 |
| MR-COMP-02 | Competitor | PyMapConv / springrts_smf_compiler | github.com/Beherith/springrts_smf_compiler | 2026-09-20 |
| MR-COMP-03 | Competitor | SpringMapEdit | springrts.com forum t=29865 | 2026-09-20 |
| MR-COMP-04 | Competitor | tebeer/BARMapEdit (Unity) | github.com/tebeer/BARMapEdit | 2026-09-20 |
| MR-COMP-05 | Competitor | hendkai/bar-map-generator | github.com/hendkai/bar-map-generator | 2026-09-20 |
| MR-COMP-06 | Dependency | Terrain3D | github.com/TokisanGames/Terrain3D | 2026-09-20 |
| MR-COMP-07 | Benchmark | Tiled funding model | opencollective.com/tiled | 2026-09-20 |
| MR-CHK-01..20 | Requirements mining | BAR Map Checklist properties | beyondallreason.info/guide/map-checklist | 2026-09-20 |
| MR-CORP-01..11 | Corpus | Local map library characterisation | Original measurement, 254 archives | 2026-09-20 |
| MR-TREND-01 | Trend | BAR commercialisation | notebookcheck.net | 2026-09-20 |
| MR-TREND-02 | Trend | maps-metadata as source of truth | github.com/beyond-all-reason/maps-metadata | 2026-09-20 |
| MR-TREND-03 | Trend | Certified map features | github.com/beyond-all-reason/mapfeatures | 2026-09-20 |
| MR-TREND-04 | Trend | gdext maturity | godot-rust.github.io/book | 2026-09-20 |
| MR-TREND-05 | Constraint | Godot has no runtime DDS loading | godot-docs importing_images.rst; godot PR #69085 | 2026-09-20 |
| MR-REG-01 | Licensing | Recoil GPL v2-or-later | bar-RecoilEngine/LICENSE | 2026-09-20 |
| MR-REG-02 | Licensing | Terrain3D and Godot MIT | github.com/TokisanGames/Terrain3D | 2026-09-20 |
| MR-REG-03 | Licensing | Third-party map content | Map archive terms | 2026-09-20 |
| MR-USER-01 | User | BAR Map Review Process | beyondallreason.info/guide/map-reviews-process | 2026-09-20 |
| MR-USER-02 | User | BAR Mapmaking Resources | beyondallreason.info/guide/mapmaking-resources | 2026-09-20 |
| MR-USER-03 | User | BAR Map Checklist | beyondallreason.info/guide/map-checklist | 2026-09-20 |
| MR-USER-04 | User | BAR maps directory, authors credited | beyondallreason.info/maps | 2026-09-20 |
| MR-SIZE-01..04 | Sizing | Audience sizing, partly unknown | Mixed; see table | 2026-09-20 |

## Quality gates

| Gate | Status |
| --- | --- |
| All research areas from the plan investigated | Met - all seven, in priority order |
| Every finding has a reference key and cited source | Met |
| Facts and inferences clearly distinguished | Met - five inferences listed separately with confidence |
| Data gaps documented honestly | Met - five gaps, each with a recommendation; no gap estimated over |
| Source Bundle compiled with reference key index | Met |
| Competitor analysis includes at least 3 competitors | Met - seven, four direct |
| Research integrity rules followed | Met - all findings dated; no fabricated citations; one search result (404) reported as unusable rather than worked around |
