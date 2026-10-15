---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
document: Product Requirements Document
status: Validated - Pass
authored: 2026-09-20
validated: 2026-09-20
implements: ../business-requirements/business-requirements.md (v1.0, baselined)
next: validate-prd (phase P01)
---

# Product Requirements Document: P01 - Map Asset Exploration (MAE)

## 1. Product Overview & Scope

### 1.1 Product context

This document implements the baselined business requirements for **P01 - Map
Asset Exploration**. It defines *what* the product does. Why it exists is in
[`business-requirements.md`](../business-requirements/business-requirements.md);
how it is built will be in the technical design.

Business objectives addressed:

- **`BIZ-EXP.1`** - every map in the reference corpus opens, without exception.
- **`BIZ-EXP.2`** - a map can be traversed corner to corner at least as fast as
  in Beyond All Reason.
- **`BIZ-EXP.3`** - every asset class a map contains is represented.
- **`BIZ-EXP.4`** - unpacked directories open as readily as archives.
- **`BIZ-EFF.1`** - all twenty BAR Map Checklist properties are verifiable
  without launching the game.
- **`MARK-DIFF.1`**, **`MARK-PARITY.1`** - standalone operation; visual parity
  with Recoil as the ranked-last commitment.

### 1.2 Phase scope

**Phase:** `P01` - Map Asset Exploration (`MAE`)
**Directory:** `docs/requirements/phases/phase-01-map-asset-exploration/`
**Prerequisite phases:** None. P01 is the first registry entry.

**In scope.** Reading BAR maps from archives and directories; decoding the
compiled map, tile atlas and map definition; evaluating map options;
representing all eleven asset classes; three-dimensional presentation with
camera and navigation; the checklist view; conformance verification across the
corpus; and visual parity with Recoil's shading chain.

**Explicitly deferred, stated positively:**

- **Writing map data belongs to P02 - Map Write Round Trip (`MWRT`)**, because
  a faithful writer can only be built on a reader proven correct across the
  whole corpus.
- **Terrain grid editing belongs to P03 - Terrain Data Editing (`TDE`)**,
  because editing requires the write path P02 establishes.
- **Layer and feature editing belongs to P04 - Map Layer Editing (`MLE`)**, for
  the same reason.
- **Reading `maps-metadata` and `mapfeatures` is out of scope for P01**
  (`DR-16`), because no P01 outcome needs them and both would add an external
  repository dependency to a tool whose claim is that it reads map files alone.
- **Executing map-side Lua is out of scope for P01** (`DR-9`), because gadget
  behaviour is runtime state rather than map data.

### 1.3 Product personas

> **P-REV - The reviewing mapper**
> **JTBD:** When a map is submitted for my review, I want to see everything the
> checklist asks about in one place, so that I can give a documented verdict
> without launching the game twice.
> **Key workflows:** open a submitted archive; fly the terrain looking for
> texture, splat and topoline problems; check numeric properties against the
> checklist; inspect metal and start positions as overlays; report back.
> **Context:** desktop; on-event, in bursts; time-pressured unpaid volunteer
> work; expects productivity within minutes without documentation.
> **Proficiency:** power user of map internals, novice with this tool.

> **P-AUT - The authoring mapper**
> **JTBD:** When I have just recompiled my map, I want to see it immediately and
> in the state a player would, so that I can judge my last change without
> starting a game.
> **Key workflows:** open the map under construction, often as an **unpacked
> directory**; fly it to judge shape, lighting and features; compare against a
> reference map; self-check against the checklist; reopen after recompiling.
> **Context:** desktop, alongside World Machine, a paint tool and a text editor;
> exploratory but highly repetitive - reopen cost dominates.
> **Proficiency:** power user; will notice any wrong value.

> **P-TOOL - The tooling developer**
> **JTBD:** When my terrain analysis produces a result I doubt, I want to see
> the same grid rendered from the same file, so that I can tell whether the map
> or my code is wrong.
> **Key workflows:** open a suspect map; view the grid in question as an
> overlay; compare against their own computed output.
> **Context:** desktop, alongside a terminal; occasional and investigative.
> **Proficiency:** expert; will read the source if the display disagrees.

**Persona provenance.** All three are reconstructed from published BAR process
documents. `DR-14` defers contact with a real mapper until a runnable build
exists, so none has been validated by a person (`RISK-6`: Medium-High
probability, High impact). Requirements below avoid depending on fine workflow
detail.

---

## 2. Functional Requirements

### 2.1 Reading and decoding

> **ID:** `FR-READ.1`
> **Traces to:** `OPER-WORKFLOW.1`, `BIZ-EXP.4`
> **Persona:** P-AUT, P-REV
> **Job Story:** When my map exists as a folder mid-build, I want to open it the
> same way as a packed map, so that I do not have to pack it to look at it.
> **Requirement:** The system shall read map contents through a single virtual
> file system that supports both `.sd7` archives and unpacked directories.
> **Acceptance Criteria (Human):**
> - Given a `.sd7` archive, When it is opened, Then all contained entries are readable by path.
> - Given the same map extracted to a directory, When it is opened, Then the presentation is identical to the archive form.
> **Acceptance Criteria (Machine):**
> - `vfs.backends == {archive, directory}`
> - `presentation(archive_form) == presentation(directory_form)` for every reference-set map
> - `entry_lookup` is case-insensitive, matching engine behaviour
> **Research Evidence:** `MR-CORP-01` - one unpacked directory is present in the live corpus.

> **ID:** `FR-READ.2`
> **Traces to:** `BIZ-EXP.1`, `BIZ-EXP.3`
> **Persona:** All
> **Job Story:** When I open a map, I want its terrain data read exactly as the engine reads it, so that what I see is not an approximation.
> **Requirement:** The system shall decode the SMF file: header, heightmap, typemap, metalmap, minimap, feature block, and the vegetation extra-header when present.
> **Acceptance Criteria (Human):**
> - Given any corpus map, When decoded, Then heightmap dimensions equal `(mapx+1) x (mapy+1)`.
> - Given a map with a vegetation extra-header, When decoded, Then the grass grid is available; when absent, Then the asset is reported absent rather than unknown.
> **Acceptance Criteria (Machine):**
> - `height_grid.len() == (header.mapx + 1) * (header.mapy + 1)`
> - `height_at(x,z) == header.min_height + raw * (header.max_height - header.min_height) / 65536`
> - `type_map.len() == metal_map.len() == (mapx/2) * (mapy/2)`
> - `decode_errors == 0` across the corpus
> **Research Evidence:** `MR-CORP-02` - exactly one SMF per archive in all 254.

> **ID:** `FR-READ.3`
> **Traces to:** `BIZ-EXP.1`, `BIZ-EXP.3`
> **Persona:** All
> **Job Story:** When I look at a map, I want its actual ground texture, so that I am judging the map and not a placeholder.
> **Requirement:** The system shall decode the `.smt` tile atlas and resolve the per-square tile index into a usable ground texture.
> **Acceptance Criteria (Human):**
> - Given a map, When the atlas is decoded, Then the tile count read from the file matches the number of tiles resolved.
> - Given a map declaring more than one tile file, When decoded, Then the declared count is honoured rather than assumed to be one.
> **Acceptance Criteria (Machine):**
> - `tile_index_grid.len() == (mapx/4) * (mapy/4)`
> - `sum(tile_counts_declared) == tiles_resolved`
> - `tile_file_count` is parsed from the header, never hardcoded to 1
> **Research Evidence:** `MR-CORP-03` - all 254 archives use exactly one SMT, but the format permits more, so the count field is parsed.

> **ID:** `FR-READ.4`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EFF.1`
> **Persona:** P-REV
> **Job Story:** When the checklist asks whether the minimap needs lightening, I want to see the stored minimap, so that I can judge it.
> **Requirement:** The system shall decode the stored minimap, including its mipmap levels.
> **Acceptance Criteria (Machine):**
> - `minimap_block_size == 699048`
> - `minimap_mip_levels == 9`

> **ID:** `FR-READ.5`
> **Traces to:** `BIZ-EXP.1`, `BIZ-EXP.3`
> **Persona:** All
> **Job Story:** When a map computes its own settings in Lua, I want those settings evaluated, not guessed, so that the values shown are the real ones.
> **Requirement:** The system shall evaluate `mapinfo.lua`, `mapoptions.lua` and their `VFS.Include` overlays in a sandboxed Lua 5.1 environment backed by the virtual file system.
> **Acceptance Criteria (Human):**
> - Given a map whose `mapinfo.lua` includes overlay files, When evaluated, Then the merged result is used.
> - Given map Lua attempting filesystem or network access, When evaluated, Then the attempt fails and the map still opens.
> **Acceptance Criteria (Machine):**
> - `lua_sandbox.filesystem_access == false` and `lua_sandbox.network_access == false`
> - `mapinfo_keys` are lowercased on lookup, matching `LuaParser.cpp`
> - `evaluation_errors == 0` across the corpus
> **Research Evidence:** `rjm.bar.tools/src/luaenv.rs` already implements this.

> **ID:** `FR-READ.6`
> **Traces to:** `OPER-MIGRATE.1`
> **Persona:** All
> **Job Story:** When a map predates `mapinfo.lua`, I want it to open anyway, so that no map is excluded.
> **Requirement:** The system shall parse the legacy `.smd` (TDF) map definition when `mapinfo.lua` is absent, as the engine does.
> **Acceptance Criteria (Human):**
> - Given `mescaline_v2.sd7`, When opened, Then it opens successfully with definition values resolved from its `.smd`.
> **Acceptance Criteria (Machine):**
> - `open("mescaline_v2.sd7").status == OK`
> - `definition_source("mescaline_v2") == "smd"`
> **Research Evidence:** `MR-CORP-04`, `MR-CORP-05`; engine fallback at `rts/Map/MapParser.cpp:30`.

> **ID:** `FR-READ.7`
> **Traces to:** `OPER-WORKFLOW.2`
> **Persona:** P-REV, P-AUT
> **Job Story:** When a map changes with its options, I want to choose the options, so that I can see each state the map actually has.
> **Requirement:** The system shall let the user select map options and shall re-evaluate the map definition and all derived values against the selection.
> **Acceptance Criteria (Human):**
> - Given a map declaring options, When the view is opened, Then the available options and their defaults are listed.
> - Given an option that alters the height range, When it is changed, Then the terrain and every derived value refresh to match.
> **Acceptance Criteria (Machine):**
> - `options_listed == options_declared_by_map`
> - `on_option_change: height_range, terrain_mesh, derived_values all recomputed`
> - `default_option_set == {}` (engine default) on first open
> **Research Evidence:** `MR-CORP-10` - 94.9% of archives ship `mapoptions.lua`.

> **ID:** `FR-READ.8`
> **Traces to:** `OPER-SCALE.1`, `BIZ-EXP.3`
> **Persona:** All
> **Job Story:** When a map simply has no skybox, I want it to open normally, so that I am not misled into thinking it is broken.
> **Requirement:** The system shall treat an absent optional asset as a normal condition, distinct from an error and distinct from an unknown value.
> **Acceptance Criteria (Human):**
> - Given a map with no skybox, features, normal map, specular map or splat texture, When opened, Then it renders and each absent asset is reported as absent.
> **Acceptance Criteria (Machine):**
> - `asset_state ∈ {present, absent, unknown}` for every asset class
> - `open(m).status == OK` for all 60 feature-less, 166 sky-less, 5 normal-less, 9 specular-less, 4 splat-less corpus maps
> **Research Evidence:** `MR-CORP-06`, `MR-CORP-07`, `MR-CORP-08`; `PR-UX-04`.

### 2.2 Presentation

> **ID:** `FR-SHOW.1`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EXP.2`
> **Persona:** All
> **Job Story:** When I open a map, I want to see its terrain in three dimensions, so that I can judge shape and scale.
> **Requirement:** The system shall present the decoded heightmap as a three-dimensional surface textured with the resolved tile atlas, with level of detail sufficient for a 32x32 map.
> **Acceptance Criteria (Machine):**
> - `world_extent == (mapx * 8, mapy * 8)` world units
> - `max_supported_extent >= 16384` per side
> - `surface_renders == true` for every corpus map

> **ID:** `FR-SHOW.2`
> **Traces to:** `BIZ-EXP.3`, `MARK-PARITY.1`
> **Persona:** P-REV, P-AUT
> **Requirement:** The system shall load and apply every shading texture slot the map declares.
> **Acceptance Criteria (Machine):**
> - Slots handled: `detailTex`, `specularTex`, `splatDetailTex`, `splatDistrTex`, `splatDetailNormalTex1..4`, `detailNormalTex`, `grassShadingTex`, `skyReflectModTex`, `lightEmissionTex`, `parallaxHeightTex`
> - `texture_decode_owner == "extension"` (Godot cannot load DDS at runtime)
> **Research Evidence:** `PR-TECH-04`.

> **ID:** `FR-SHOW.3`
> **Traces to:** `BIZ-EXP.3`
> **Persona:** P-TOOL, P-REV
> **Requirement:** The system shall apply the `minimapTex`, `metalmapTex`, `typemapTex` and `grassmapTex` overrides from the map definition in place of the values stored in the SMF when they are declared.
> **Acceptance Criteria (Machine):**
> - `effective_grid(name) == override if declared else smf_value`
> - `grid_source` is recorded and displayable for each grid

> **ID:** `FR-SHOW.4`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall present the map's sky and fog as declared, and shall present a defined default when no skybox is supplied.
> **Acceptance Criteria (Machine):**
> - `sky_state ∈ {map_supplied, default}`; never an error
> - Applies to the 166 corpus maps with no skybox
> **Research Evidence:** `MR-CORP-07` - only 34.6% supply a skybox.

> **ID:** `FR-SHOW.5`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall present water at the map's water level using the water material values from the map definition.
> **Acceptance Criteria (Machine):**
> - Water plane at `y == 0` in map space, per engine convention
> - Water material values sourced from the definition, displayable as values

> **ID:** `FR-SHOW.6`
> **Traces to:** `BIZ-EXP.3`
> **Persona:** P-REV, P-AUT
> **Requirement:** The system shall load S3O models and their textures, and shall place each feature instance at its declared position and rotation.
> **Acceptance Criteria (Machine):**
> - S3O piece hierarchy, vertices and index tables parsed per `rts/Rendering/Models/s3o.h`
> - `placed_features == declared_placements` for every map with features
> - Feature Y position derived from terrain height, not a fixed value

> **ID:** `FR-SHOW.7`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall expose each feature's declared properties, including reclaim values, from the map's `features/*.lua` definitions.
> **Acceptance Criteria (Machine):**
> - `feature_properties` include metal and energy reclaim where declared
> - Missing property → `unknown`, never a substituted default

> **ID:** `FR-SHOW.8`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall present the map's start positions in world space with their team indices.
> **Acceptance Criteria (Machine):**
> - `start_positions == definition.teams[*].startpos`
> - Team index displayed per position

> **ID:** `FR-SHOW.9`
> **Traces to:** `BIZ-EXP.3`, `GUARD-SCOPE.3`
> **Persona:** P-REV, P-TOOL
> **Requirement:** The system shall present map-side Lua - `LuaGaia`, `featureplacer` and map configuration scripts - as readable source text, and shall not execute it.
> **Acceptance Criteria (Machine):**
> - `lua_files_listed == lua_files_present`
> - `gadget_execution_paths == 0`
> **Research Evidence:** `DR-9`; `MR-CORP-09` - `LuaGaia` present in 92.9% of archives.

> **ID:** `FR-SHOW.10`
> **Traces to:** `BIZ-EXP.3`, `BIZ-EFF.1`
> **Persona:** P-TOOL, P-REV
> **Job Story:** When I doubt a derived terrain fact, I want to see that grid over the terrain, so that I can compare it against my own output.
> **Requirement:** The system shall present the metal, type, slope, passability and grass grids as overlays on the terrain surface, individually toggleable.
> **Acceptance Criteria (Human):**
> - Given an open map, When an overlay is toggled, Then it is drawn in world space aligned to the terrain.
> **Acceptance Criteria (Machine):**
> - Overlays available: `metal`, `type`, `slope`, `passability`, `grass`
> - `slope == 1 - normal.y`, per `ReadMap.cpp`
> - Overlay alignment error `== 0` cells

> **ID:** `FR-SHOW.11`
> **Traces to:** `GUARD-ACC.1`
> **Persona:** All
> **Job Story:** When the tool cannot work something out, I want it to say so, so that I never act on a number it invented.
> **Requirement:** The system shall display any value it cannot derive exactly as **unknown**, and shall never substitute an estimated, default or approximated value in its place.
> **Acceptance Criteria (Human):**
> - Given a value that cannot be derived, When it is displayed, Then it reads as unknown and is distinguished from both a derived value and an absent asset by a text or shape attribute, not by colour alone.
> - Given such a value, When the map is opened, Then the map still opens.
> **Acceptance Criteria (Machine):**
> - `displayed_values.all(v => v.state ∈ {derived, unknown, absent})`
> - Each of the three states carries a distinct non-colour attribute (label or icon)
> - `v.state == derived` implies `v.rule_citation != null`
> - `substituted_defaults == 0`
> **Research Evidence:** `DR-5`; no existing display convention was found (`PR-UX` Data Gap 4), so this is an original design.

### 2.3 Camera and navigation

> **ID:** `FR-NAV.1`
> **Traces to:** `BIZ-EXP.2`
> **Persona:** All
> **Job Story:** When I explore a map, I want camera controls I already know, so that I am not learning the tool instead of reading the map.
> **Requirement:** The system shall provide ground-anchored camera control in which the terrain point under the cursor remains under the cursor during pan, orbit and zoom.
> **Acceptance Criteria (Human):**
> - Given the cursor over a terrain point, When panning, orbiting or zooming, Then that point stays under the cursor.
> **Acceptance Criteria (Machine):**
> - `anchor_drift_pixels <= 2` during pan, orbit and zoom
> - Bindings: left-drag pan; middle-drag orbit (horizontal → heading, vertical → pitch); wheel zoom-to-cursor; `WASD` move; `QE` rotate
> **Research Evidence:** `PR-UX-01`.

> **ID:** `FR-NAV.2`
> **Traces to:** `BIZ-EXP.2`
> **Persona:** All
> **Requirement:** The system shall scale camera movement speed with height above terrain, so that both corner-to-corner traversal and close inspection are practical without a manual speed control.
> **Acceptance Criteria (Machine):**
> - `move_speed = f(height_above_terrain)`, monotonically increasing
> - Corner-to-corner traversal satisfies `NFR-PERF.2`
> - At minimum altitude, a 50-unit feature can be approached without overshoot
> **Research Evidence:** `INF-P1` (inference, high confidence); `PR-UX-02`.

> **ID:** `FR-NAV.3`
> **Traces to:** `BIZ-EXP.2`, `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall provide direct camera positioning to named locations on the map, including each start position and a whole-map overview.
> **Acceptance Criteria (Machine):**
> - Targets: `overview`, `start_position[i]` for all declared positions
> - `time_to_reposition <= 2` seconds
> **Research Evidence:** `PR-UX-03`.

### 2.4 Checklist verification

> **ID:** `FR-CHECK.1`
> **Traces to:** `BIZ-EFF.1`, `OPER-WORKFLOW.4`
> **Persona:** P-REV (primary), P-AUT
> **Job Story:** When I review a map, I want every checklist property in one view, so that I do not have to remember which of twenty things I have already looked at.
> **Requirement:** The system shall present all twenty BAR Map Checklist properties in a single view, each showing the open map's value.
> **Acceptance Criteria (Human):**
> - Given an open map, When the checklist view is shown, Then all twenty properties appear with a value, an absent marker or an unknown marker.
> **Acceptance Criteria (Machine):**
> - `checklist_properties.count == 20`
> - `properties_with_state.count == 20` for every corpus map
> - Property set equals `MR-CHK-01` through `MR-CHK-20`

> **ID:** `FR-CHECK.2`
> **Traces to:** `BIZ-EFF.1`, `GUARD-REPUTE.2`
> **Persona:** P-REV
> **Requirement:** The system shall show a pass or fail result only for checklist properties the checklist governs by a stated numeric rule, and shall report the value without judgement for all others.
> **Acceptance Criteria (Human):**
> - Given the map size, wind, tidal, splat resolution or normals resolution property, When displayed, Then a pass or fail is shown against the published rule.
> - Given any other property, When displayed, Then a value is shown and no verdict is implied.
> **Acceptance Criteria (Machine):**
> - `judged_properties == {size <= 32, wind ∈ [0,30], tidal ∈ [0,25], splat_px >= 1024, normals_res == texture_res}`
> - `judged_properties.count == 5`; `reported_only.count == 15`
> - No interface element states or implies map approval
> **Research Evidence:** `GUARD-REPUTE.2`; `PR-UX-06`.

> **ID:** `FR-CHECK.3`
> **Traces to:** `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall let the user move from a checklist property to the thing it describes, by positioning the camera or enabling the relevant overlay.
> **Acceptance Criteria (Human):**
> - Given the metal distribution property, When selected, Then the metal overlay is enabled.
> - Given a start-position property, When selected, Then the camera moves to that position.
> **Acceptance Criteria (Machine):**
> - Properties that must link to an overlay: metal distribution, type map, grass map, pathability
> - Properties that must link to a camera position: each start position, whole-map overview
> - `unlinked_properties` are those with no spatial referent (wind, tidal, licence-like values) and are exempt by definition
> **Research Evidence:** `INF-P2` (inference, medium confidence, persona unvalidated). The linked set is derived from which properties have a spatial referent the camera or an overlay can address, not from a target count.

> **ID:** `FR-CHECK.4`
> **Traces to:** `BIZ-EFF.1`
> **Persona:** P-REV
> **Requirement:** The system shall derive pathability by movement class and geothermal buildability using constants embedded in the application, each carrying the Beyond All Reason commit it was read from.
> **Acceptance Criteria (Human):**
> - Given a map, When pathability is shown, Then the movement classes and the source commit of the constants are both visible.
> **Acceptance Criteria (Machine):**
> - `embedded_constants.bar_commit != null`
> - Movement classes covered: bot, vehicle, hover, ship
> - `requires_bar_installation == false`
> **Research Evidence:** `ASSUM-7`; `DR-10`.

### 2.5 Conformance and parity

> **ID:** `FR-CONF.1`
> **Traces to:** `OPER-REPORT.1`, `BIZ-EXP.1`
> **Persona:** Project owner
> **Requirement:** The system shall provide a repeatable run that opens every member of a given map library and records a pass or fail result per map.
> **Acceptance Criteria (Machine):**
> - Run is non-interactive and returns a non-zero exit code if any map fails
> - Output lists every map with `{pass|fail}` and, on failure, the failing stage
> - Re-running over unchanged inputs produces identical output

> **ID:** `FR-CONF.2`
> **Traces to:** `OPER-REPORT.2`, `MARK-PARITY.1`, `BIZ-EXP.2`
> **Persona:** Project owner
> **Requirement:** The system shall capture screenshots at recorded camera positions and measure corner-to-corner traversal time for each reference-set map, so that both can be compared against the in-game reference.
> **Acceptance Criteria (Machine):**
> - `reference_set.size ∈ [6,10]`
> - Reference set includes: a map with a skybox and one without; one with features and one without; one missing a normal, specular or splat texture; `mescaline_v2`; an option-sensitive map altering the height range; the largest and smallest archives by file count
> - Captures and timings stored against an engine commit recorded in `verified_against`

---

## 3. Non-Functional Requirements

> **ID:** `NFR-PERF.1`
> **Traces to:** `BIZ-EXP.2`
> **Requirement:** The system shall sustain an interactive frame rate while the camera is moving over any corpus map.
> **Primary Metric:** 60 fps sustained during traversal on the reference set.
> **Secondary Metric:** 1% low frame rate >= 45 fps.
> **Guardrail:** Frame rate shall never fall below **30 fps**, at which point camera control degrades and `NFR-PERF.2` becomes unachievable.
> **Benchmark Source:** `PR-NFR-01`.

> **ID:** `NFR-PERF.2`
> **Traces to:** `BIZ-EXP.2`, `DR-7`
> **Requirement:** The system shall complete a corner-to-corner traversal in no more time than the same traversal performed in Beyond All Reason at the pinned engine commit.
> **Primary Metric:** `editor_traversal_seconds <= bar_traversal_seconds` for every reference-set map.
> **Secondary Metric:** Median across the reference set at or below 90% of the in-game time.
> **Guardrail:** No reference-set map shall exceed the in-game time by any margin.
> **Benchmark Source:** `PR-NFR-02`; externally anchored, no invented figure.

> **ID:** `NFR-PERF.3`
> **Traces to:** `BIZ-EXP.2`, `OPER-SCALE.1`
> **Requirement:** The system shall present navigable terrain before all textures have finished decoding, so that opening a large archive does not block interaction.
> **Primary Metric:** Terrain navigable within 5 seconds of opening any corpus map.
> **Secondary Metric:** All textures resolved within 20 seconds.
> **Guardrail:** The interface shall never be unresponsive for more than 1 second at a time during loading.
> **Benchmark Source:** `PR-NFR-03`; `INF-P5`.

> **ID:** `NFR-PERF.4`
> **Traces to:** `OPER-SCALE.1`
> **Requirement:** The system shall open every corpus archive within a memory footprint measured and fixed against the reference set.
> **Primary Metric:** Peak resident memory recorded for every reference-set map, and a ceiling set from those measurements once the texture pipeline is settled.
> **Secondary Metric:** Peak memory for the largest corpus archive no more than twice the reference-set median.
> **Guardrail:** The system shall not exhaust memory on any corpus map. This is falsifiable today and is the binding constraint until the ceiling is measured.
> **Benchmark Source:** `PR-NFR-04`. **No numeric ceiling is asserted here.** The research bundle records memory as un-benchmarked for this domain, and the dominant variable - whether the tile atlas can remain compressed in memory - is an open technical question (`PR-TECH-05`). Asserting a figure now would be the same substitution of a plausible value that `GUARD-ACC.1` forbids in the product's own output.

> **ID:** `NFR-PORT.1`
> **Traces to:** `OPER-WORKFLOW.3`
> **Requirement:** The system shall run on Windows and Linux.
> **Primary Metric:** A full conformance run passes on both platforms.
> **Secondary Metric:** Identical per-map results on both.
> **Guardrail:** No platform-specific map result divergence.
> **Benchmark Source:** `PR-NFR-05`; Terrain3D requires Godot 4.4+.

> **ID:** `NFR-DEP.1`
> **Traces to:** `MARK-DIFF.1`, `GUARD-SCOPE.5`
> **Requirement:** The system shall open a map on a machine with no Beyond All Reason installation and no running engine.
> **Primary Metric:** Conformance run passes on a clean machine with map files only.
> **Secondary Metric:** No code path queries a game installation location.
> **Guardrail:** Zero runtime dependency on engine or game binaries.

> **ID:** `NFR-VALID.1`
> **Traces to:** `GUARD-ACC.1`
> **Requirement:** The system shall not display an approximated value as derived.
> **Primary Metric:** Every displayed value carries a state of derived, unknown or absent.
> **Secondary Metric:** Every derived value carries a citation to the engine rule producing it.
> **Guardrail:** Zero substituted defaults anywhere in the displayed value set.

> **ID:** `NFR-VALID.2`
> **Traces to:** `GUARD-SCOPE.2`, `DR-8`
> **Requirement:** The system shall not open any map file for writing.
> **Primary Metric:** No write path to an archive, `.smf`, `.smt`, `.smd`, `mapinfo.lua` or any archive member exists in the codebase.
> **Secondary Metric:** Permitted writes are limited to conformance reports, parity captures and session state.
> **Guardrail:** Map files are byte-identical before and after any session.

> **ID:** `NFR-VALID.3`
> **Traces to:** `GUARD-SCOPE.3`
> **Requirement:** The system shall not execute map-side gadget Lua.
> **Primary Metric:** Zero gadget execution paths.
> **Guardrail:** The Lua sandbox is used only for map definition evaluation (`FR-READ.5`), never for `LuaGaia` content.

> **ID:** `NFR-SEC.1`
> **Traces to:** `FR-READ.5`, `BIZ-EXP.1`, `GUARD-SCOPE.3`
> **Requirement:** The system shall execute map-supplied Lua only inside a sandbox that denies filesystem access, network access, process creation and unbounded execution, because map archives are untrusted third-party files obtained from other people and from the internet.
> **Primary Metric:** Sandbox denies all four capability classes; verified by a test map exercising each.
> **Secondary Metric:** Map Lua execution is bounded by an instruction or time limit, so that a malicious or defective map cannot hang the application.
> **Guardrail:** No map-supplied Lua shall ever reach the host filesystem, the network, or a process boundary. A sandbox escape is a P0 defect.
> **Threat model note:** This is the product's only untrusted input and its only security boundary. `FR-SHOW.9` and `NFR-VALID.3` further ensure gadget Lua is never executed at all - only map definition Lua runs, and only in this sandbox.

> **ID:** `NFR-REL.1`
> **Traces to:** `BIZ-EXP.1`, `OPER-SCALE.1`, `DR-4`
> **Requirement:** The system shall not crash, hang or corrupt its own state on any input, including malformed, truncated or hostile map files.
> **Primary Metric:** Zero crashes across a full conformance run of the corpus.
> **Secondary Metric:** A deliberately corrupted archive produces a named failure and leaves the application usable.
> **Guardrail:** No input shall cause process termination. Because `BIZ-EXP.1` admits no exclusions, a crash on any corpus member blocks phase closure.

> **ID:** `NFR-LIC.1`
> **Traces to:** `COMP-LICENSE.2`
> **Requirement:** The system shall carry attribution for its MIT-licensed dependencies in distributed builds.
> **Primary Metric:** Attribution present in every distributed build.
> **Guardrail:** No build ships without dependency attribution.

> **ID:** `NFR-LIC.2`
> **Traces to:** `COMP-CONTENT.1`
> **Requirement:** The system shall not bundle third-party map content.
> **Primary Metric:** Zero map files in the repository or in any distributed build.
> **Guardrail:** Reference-set captures are images, never map data.

> **ID:** `NFR-A11Y.1`
> **Traces to:** Quality target, not a compliance obligation
> **Requirement:** The system shall keep the checklist view operable by keyboard and shall not encode any checklist result by colour alone.
> **Primary Metric:** All checklist actions reachable by keyboard.
> **Secondary Metric:** Pass and fail distinguished by shape or text as well as colour.
> **Guardrail:** No information conveyed by colour alone.
> **Benchmark Source:** `PR-UX-05`. WCAG 2.2 AA applied as a quality target; no statutory regime applies and no `COMP-*` requirement mandates a level.

> **ID:** `NFR-FID.1`
> **Traces to:** `MARK-PARITY.1`
> **Requirement:** The system shall reproduce Recoil's SMF shading chain such that side-by-side captures at matching camera positions are visually indistinguishable to a human reviewer.
> **Primary Metric:** Indistinguishable on every reference-set map at the pinned engine commit.
> **Secondary Metric:** All ten shader texture slots fed with correctly decoded data.
> **Guardrail:** No parity claim is made for any map or engine commit without a stored capture (`GUARD-REPUTE.1`).
> **Priority note:** P2 - the designated sacrifice. Failure here does not block phase closure.

---

## 4. Data Requirements

> **ID:** `DATA-PROV.1`
> **Traces to:** `FR-SHOW.11`, `NFR-VALID.1`, `GUARD-ACC.1`
> **Data Element:** Value provenance - every displayed value carries a state of `derived`, `unknown` or `absent`, and a derived value additionally carries the engine rule citation that produced it.
> **Source of Truth:** The decoder.
> **Refresh Frequency:** On map open and on option change.
> **Retention Period:** Session only.
> **Volume Estimate:** One provenance record per displayed value; hundreds per map.
> **Privacy Classification:** Public - no personal data.
> **Note:** The most pervasive structural requirement in P01 (`PR-DATA-01`).

> **ID:** `DATA-MAP.1`
> **Traces to:** `FR-READ.2` to `FR-READ.8`
> **Data Element:** The decoded map - grids, definition values, texture references, feature placements, start positions, Lua sources.
> **Source of Truth:** The map archive or directory. No other source.
> **Refresh Frequency:** On open; on option change; on reopen after external modification.
> **Retention Period:** Session only. Nothing written back.
> **Volume Estimate:** Archives 10-436 files, mean 96; up to ~50 MB, of which ~25 MB is the tile atlas; heightmap up to 2049x2049 `u16`.
> **Privacy Classification:** Public; third-party creative content under `COMP-CONTENT.1`.

> **ID:** `DATA-CONST.1`
> **Traces to:** `FR-CHECK.4`, `ASSUM-7`
> **Data Element:** Embedded Beyond All Reason constants - movement definitions and geothermal parameters - needed for two checklist properties.
> **Source of Truth:** `bar-Beyond-All-Reason` at a recorded commit, read once and embedded.
> **Refresh Frequency:** On deliberate revalidation, not at runtime.
> **Retention Period:** Life of the build.
> **Volume Estimate:** Small, tens of values.
> **Privacy Classification:** Public.
> **Note:** Each constant carries its source commit, and that commit is displayable. Staleness here would produce a wrong value that looks derived - the failure mode `GUARD-ACC.1` exists to prevent.

> **ID:** `DATA-REF.1`
> **Traces to:** `FR-CONF.2`, `OPER-REPORT.2`
> **Data Element:** The reference set - map identities, camera positions, in-game captures, in-game traversal times, and the engine commit they were taken at.
> **Source of Truth:** Recorded once from Beyond All Reason.
> **Refresh Frequency:** On engine commit change.
> **Retention Period:** Life of the project; versioned.
> **Volume Estimate:** 6-10 maps, a handful of captures each.
> **Privacy Classification:** Public. Images only - never map data (`NFR-LIC.2`).

> **ID:** `DATA-SESS.1`
> **Traces to:** `DR-8`
> **Data Element:** Session state - window layout, last-opened map, overlay and option selections.
> **Source of Truth:** The application.
> **Refresh Frequency:** On change.
> **Retention Period:** Until cleared by the user.
> **Volume Estimate:** Kilobytes.
> **Privacy Classification:** Internal. Contains local file paths; no personal data.

---

## 5. User Workflows & UX Requirements

> **ID:** `UX-OPEN.1`
> **Traces to:** `FR-READ.1`, `FR-READ.8`, `NFR-PERF.3`
> **Persona:** P-REV, P-AUT
> **Workflow:** Opening a map
> **Happy Path:** Choose an archive or a directory → terrain appears and becomes navigable → textures resolve progressively → the asset inventory shows what the map contains.
> **Error States:**
> - Unreadable archive → the failure is named with the stage that failed; the application stays usable.
> - Malformed definition → values that cannot be derived show as unknown; **the map still opens** (`DR-5`).
> **Edge Cases:**
> - No skybox, no features, missing texture slots → shown as absent, not as errors (166, 60 and up to 9 corpus maps respectively).
> - Legacy `.smd` map → opens via `FR-READ.6`.
> - Map declaring options → engine defaults applied on first open; options offered.
> **Loading and Empty States:** Terrain navigable within 5 seconds; texture resolution visibly in progress; no modal block longer than 1 second.
> **UX Pattern Reference:** `PR-UX-04`, `PR-NFR-03`.

> **ID:** `UX-EXPLORE.1`
> **Traces to:** `FR-NAV.1`, `FR-NAV.2`, `FR-NAV.3`, `FR-SHOW.10`
> **Persona:** All
> **Workflow:** Exploring a map
> **Happy Path:** Start at overview → drag, orbit and zoom toward a region of interest → speed falls with altitude → toggle overlays to inspect metal, slope or passability → jump to a start position.
> **Error States:**
> - Camera leaves the map bounds → position is constrained rather than lost.
> **Edge Cases:**
> - Very small map → overview framing still correct.
> - Entirely flat or entirely water map → camera anchoring still functions.
> **Loading and Empty States:** Overlays unavailable until their grid is decoded; shown as pending rather than empty.
> **UX Pattern Reference:** `PR-UX-01`, `PR-UX-02`, `PR-UX-03`.

> **ID:** `UX-CHECK.1`
> **Traces to:** `FR-CHECK.1`, `FR-CHECK.2`, `FR-CHECK.3`, `NFR-A11Y.1`
> **Persona:** P-REV
> **Workflow:** Verifying a map against the checklist
> **Happy Path:** Open the checklist view → all twenty properties listed with values → five show a pass or fail against published numeric rules → select a property to jump to it on the map → form a verdict.
> **Error States:**
> - A property cannot be derived → shown as unknown with the reason; never guessed.
> **Edge Cases:**
> - Property refers to an absent asset → shown as absent, distinct from unknown.
> - Map fails a numeric rule → fail shown for that property alone; **no overall verdict is offered** (`GUARD-REPUTE.2`).
> **Loading and Empty States:** Properties populate as their data decodes.
> **UX Pattern Reference:** `PR-UX-06`. No comparable tool exists; this is original design.

> **ID:** `UX-RELOAD.1`
> **Traces to:** `FR-READ.1`, `BIZ-EXP.4`
> **Persona:** P-AUT
> **Workflow:** Reopening after a recompile
> **Happy Path:** Recompile externally → the open map is detected as changed → reload on request → camera position and overlay selections are preserved.
> **Error States:**
> - Source removed or mid-write → reload is refused with the reason; the previous view remains.
> **Edge Cases:**
> - Directory contents change without the directory timestamp changing → an explicit reload action is always available.
> **UX Pattern Reference:** `PR-UX-07`; `INF-P3` (inference, persona unvalidated).

> **ID:** `UX-UNKNOWN.1`
> **Traces to:** `FR-SHOW.11`, `NFR-VALID.1`
> **Persona:** All
> **Workflow:** Encountering a value the tool cannot derive
> **Happy Path:** The value reads as unknown, is visually distinct from a derived value and from an absent asset, and states why it could not be derived.
> **Error States:** Not applicable - this *is* the handled state.
> **Edge Cases:**
> - Many unknowns at once → the map still opens and remains navigable.
> **UX Pattern Reference:** None found in this or an adjacent domain (Data Gap 4). Original design, and a candidate for early-user feedback once `DR-14` allows it.

> **ID:** `UX-VERIFY.1`
> **Traces to:** `FR-SHOW.10`, `FR-SHOW.3`, `BIZ-EXP.3`
> **Persona:** P-TOOL
> **Workflow:** Checking a derived terrain fact against the map
> **Happy Path:** Open the map whose analysis is in doubt → enable the overlay for the grid in question - slope, passability, metal or type → read the value at a specific cell → compare against the value their own tool produced.
> **Error States:**
> - The grid cannot be derived → shown as unknown with the reason, so the developer learns the tool could not compute it rather than that their own value is wrong.
> **Edge Cases:**
> - An override grid replaces the SMF value → the overlay shows the effective grid and states which source it came from (`FR-SHOW.3`).
> - Option-sensitive map → the overlay reflects the currently selected options (`FR-READ.7`).
> **Loading and Empty States:** Overlay shown as pending until its grid is decoded.
> **UX Pattern Reference:** `PR-DATA-01`. Provenance display is what makes this workflow work: the developer needs to know not just the value but which rule produced it.

---

## 6. Feature Prioritization

MoSCoW within P01 only.

### Must have (P0) - failure means the phase fails

`FR-READ.1`, `FR-READ.2`, `FR-READ.3`, `FR-READ.5`, `FR-READ.8`,
`FR-SHOW.1`, `FR-SHOW.11`, `FR-NAV.1`, `FR-NAV.2`,
`NFR-PERF.1`, `NFR-PERF.2`, `NFR-DEP.1`, `NFR-SEC.1`, `NFR-REL.1`,
`NFR-VALID.1`, `NFR-VALID.2`, `NFR-VALID.3`, `NFR-LIC.1`, `NFR-LIC.2`,
`DATA-PROV.1`, `DATA-MAP.1`, `UX-OPEN.1`, `UX-EXPLORE.1`, `UX-UNKNOWN.1`

These deliver the registered outcome: every map opens, every asset class is
represented, exploration is fast, and nothing is guessed.

### Should have (P1) - phase viable but degraded

`FR-READ.4`, `FR-READ.6`, `FR-READ.7`, `FR-SHOW.2` to `FR-SHOW.10`,
`FR-NAV.3`, `FR-CHECK.1` to `FR-CHECK.4`, `FR-CONF.1`, `FR-CONF.2`,
`NFR-PERF.3`, `NFR-PERF.4`, `NFR-PORT.1`, `NFR-A11Y.1`,
`DATA-CONST.1`, `DATA-REF.1`, `DATA-SESS.1`, `UX-CHECK.1`, `UX-RELOAD.1`,
`UX-VERIFY.1`

**Note on `FR-READ.6`:** legacy `.smd` support is P1 by requirement priority but
is a hard prerequisite of `BIZ-EXP.1`'s no-exceptions target, because
`mescaline_v2` depends on it. It cannot in practice be dropped.

### Could have (P2) - deferrable without blocking release

`NFR-FID.1` (visual parity) - the designated sacrifice per the Intent Brief
priority stack.

### Won't have this phase - stated positively with rationale

- **Map writing belongs to P02 (`MWRT`)**, which establishes the write path all
  editing depends on.
- **Terrain grid editing belongs to P03 (`TDE`)**, which requires P02's writer.
- **Layer and feature editing belongs to P04 (`MLE`)**, for the same reason.
- **Reading `maps-metadata` and `mapfeatures` is deferred indefinitely**
  (`DR-16`) - no P01 outcome needs them.
- **Gadget execution is out of scope** (`DR-9`) - runtime behaviour is not map
  data.

---

## 7. Agent-Readiness

### 7.1 Semantic glossary

| Term | Definition | Example |
|:---|:---|:---|
| **Corpus** | The reference population: 254 `.sd7` archives plus 1 unpacked directory in the local BAR installation, measured 2026-09-20 | `acidicquarry_5.17.sd7` |
| **Corpus member** | One archive or one unpacked map directory | `supreme_isthmus_v1.8/` |
| **Asset class** | One of eleven categories: compiled terrain, tile atlas, map definition, shading textures, override grids, sky, water, map objects, object placement, map-side Lua, start positions | Sky |
| **Derived value** | A value produced by reproducing a documented engine rule, carrying a citation to that rule | Slope, from `1 - normal.y` per `ReadMap.cpp` |
| **Unknown value** | A value that cannot be derived exactly from available data. Displayed as unknown; never substituted | A malformed wind value |
| **Absent asset** | An asset class the map legitimately does not contain. Distinct from unknown | No skybox - true of 166 corpus maps |
| **Map definition** | The evaluated result of `mapinfo.lua` and overlays, or of `.smd` where `mapinfo.lua` is absent | Wind range, sun direction |
| **Checklist property** | One of the twenty properties published in the BAR Map Checklist | Wind between 0 and 30 |
| **Judged property** | One of the five checklist properties governed by a stated numeric rule, for which a pass or fail may be shown | Map size at most 32x32 |
| **Reported property** | One of the fifteen checklist properties shown as a value with no verdict | Splat distribution balance |
| **Reference set** | 6-10 corpus maps chosen to span composition variation, with recorded in-game captures and traversal times | Includes `mescaline_v2` |
| **Corner-to-corner traversal** | Moving the camera from one map corner to the diagonally opposite corner under normal camera control, by a defined repeatable procedure | ~23,170 units on a 32x32 map |
| **Ground-anchored** | Camera behaviour in which the terrain point under the cursor stays under the cursor during pan, orbit and zoom | - |
| **Visual parity** | Human-judged indistinguishability of side-by-side captures at matching camera positions against a pinned engine commit | - |
| **World unit** | The engine's distance unit. A 32x32 map is 16,384 world units per side | - |
| **Tile atlas** | The `.smt` file: a set of fixed-size compressed texture tiles that the SMF indexes per terrain square to form the ground texture | 25 MB of a 49 MB archive |
| **Overlay** | A decoded grid drawn over the terrain surface in world space, aligned cell-for-cell, toggleable independently | The metal overlay |
| **Movement class** | One of the engine's unit movement categories, each with its own slope and depth limits | bot, vehicle, hover, ship |
| **Geovent** | A geothermal vent feature on a map, on which geothermal energy structures can be built | Checked by `MR-CHK-13` |
| **Topolines** | Contour-like elevation lines authored into a map's ground texture and normal map to convey height | Checked by `MR-CHK-11` |
| **Sandbox** | The restricted Lua 5.1 environment in which map definition Lua is evaluated, denying filesystem, network, process and unbounded execution | `NFR-SEC.1` |

### 7.2 Protection patterns - DO NOT MODIFY

Locked decisions. Downstream design and implementation must preserve these
exactly.

- **No map file is ever opened for writing** (`NFR-VALID.2`, `GUARD-SCOPE.2`,
  `DR-8`). Permitted writes are conformance reports, parity captures and
  session state. Nothing else.
- **No value is ever approximated** (`NFR-VALID.1`, `GUARD-ACC.1`, `DR-5`).
  Underivable values are marked unknown and the map still opens. Substituting a
  plausible default is prohibited, not merely discouraged.
- **Every corpus member must open. No exclusion list** (`BIZ-EXP.1`, `DR-4`).
  This is why `FR-READ.6` exists.
- **No dependency on a Beyond All Reason installation or a running engine at
  runtime** (`NFR-DEP.1`, `GUARD-SCOPE.5`). Embedded constants carry a cited
  commit instead (`DATA-CONST.1`).
- **Map-side Lua is never executed** (`NFR-VALID.3`, `DR-9`). The Lua sandbox
  serves map definition evaluation only.
- **Pass or fail is shown only for the five numerically-governed checklist
  properties** (`FR-CHECK.2`, `GUARD-REPUTE.2`). The tool never implies
  approval.
- **Visual parity is the designated sacrifice** (`NFR-FID.1`, P2). If a
  trade-off is required, parity yields - never the three P0 exploration
  commitments.
- **Parity is claimed only against a pinned engine commit with stored captures**
  (`GUARD-REPUTE.1`).
- **Map-supplied Lua runs only inside the sandbox, which denies filesystem,
  network, process creation and unbounded execution** (`NFR-SEC.1`). Map
  archives are untrusted third-party input; this is the product's only security
  boundary and must not be widened for convenience.

### 7.3 Phase dependencies

```
Phase:       P01 - Map Asset Exploration (MAE)
Directory:   docs/requirements/phases/phase-01-map-asset-exploration/
Entry:       Intent Brief approved; P01 business, product and technical
             requirements complete and validated; GPL v2-or-later licence
             recorded in the repository.
Exit:        Every corpus member opens; every asset class represented;
             traversal at or faster than in-game; no approximated values;
             early-user feedback recorded.
Earlier-phase dependencies: None. P01 is the first registry entry.
Downstream:  P02 - Map Write Round Trip (MWRT) depends on P01.
```

### 7.4 Phase 01 verification

- [ ] Every corpus member opens without error - 255 of 255, no exclusions
- [ ] All eleven asset classes represented for maps that contain them
- [ ] Absent optional assets reported as absent, not as errors
- [ ] `mescaline_v2.sd7` opens via its legacy `.smd` definition
- [ ] Map options selectable, with derived values refreshed on change
- [ ] Corner-to-corner traversal at or faster than in-game on every reference-set map
- [ ] Frame rate never below 30 fps during traversal
- [ ] All twenty checklist properties present; exactly five judged
- [ ] Every displayed value is derived, unknown or absent - zero substituted defaults
- [ ] No write path to any map file exists
- [ ] Conformance run passes on Windows and on Linux
- [ ] Conformance run passes on a machine with no BAR installation
- [ ] Sandbox denies filesystem, network, process creation and unbounded execution
- [ ] Zero crashes across a full conformance run, including deliberately corrupted input
- [ ] Early-user mapper has flown a map they authored; feedback recorded

Commands:

```
maeditor conform --library <path>       # FR-CONF.1; non-zero exit on any failure
maeditor conform --library <path> --clean-machine
maeditor reference --capture --engine-commit <sha>   # FR-CONF.2
maeditor reference --compare
```

*Command names are illustrative; the technical design fixes the interface.*

---

## 8. Traceability Matrix

### 8.1 Forward traceability (BRD → PRD)

| BRD ID | Requirement | PRD ID(s) | Coverage |
|:---|:---|:---|:---|
| `BIZ-EXP.1` | Every corpus member opens | `FR-READ.1`-`FR-READ.8`, `FR-CONF.1` | Full |
| `BIZ-EXP.2` | Traversal within threshold | `FR-NAV.1`, `FR-NAV.2`, `FR-NAV.3`, `NFR-PERF.1`, `NFR-PERF.2`, `NFR-PERF.3` | Full |
| `BIZ-EXP.3` | Every asset class represented | `FR-SHOW.1`-`FR-SHOW.11` | Full |
| `BIZ-EXP.4` | Unpacked directories | `FR-READ.1`, `UX-RELOAD.1` | Full |
| `BIZ-EFF.1` | Twenty checklist properties | `FR-CHECK.1`-`FR-CHECK.4`, `UX-CHECK.1` | Full |
| `BIZ-SAT.1` | Early-user validation | *Project activity* - verification checklist item | N/A by design |
| `MARK-DIFF.1` | Standalone operation | `NFR-DEP.1` | Full |
| `MARK-PARITY.1` | Visual parity | `NFR-FID.1`, `FR-CONF.2` | Full |
| `OPER-WORKFLOW.1` | One access path | `FR-READ.1` | Full |
| `OPER-WORKFLOW.2` | Option picker | `FR-READ.7` | Full |
| `OPER-WORKFLOW.3` | Windows and Linux | `NFR-PORT.1` | Full |
| `OPER-WORKFLOW.4` | Checklist view | `FR-CHECK.1`, `UX-CHECK.1` | Full |
| `OPER-MIGRATE.1` | Legacy `.smd` | `FR-READ.6` | Full |
| `OPER-SCALE.1` | Corpus variation | `FR-READ.8`, `NFR-PERF.4`, `NFR-REL.1` | Full |
| `OPER-REPORT.1` | Conformance run | `FR-CONF.1` | Full |
| `OPER-REPORT.2` | Reference set | `FR-CONF.2`, `DATA-REF.1` | Full |
| `COMP-LICENSE.1` | GPL, public | *Repository obligation* | N/A by design |
| `COMP-LICENSE.2` | MIT attribution | `NFR-LIC.1` | Full |
| `COMP-CONTENT.1` | No map redistribution | `NFR-LIC.2` | Full |
| `GUARD-ACC.1` | No guessing | `FR-SHOW.11`, `NFR-VALID.1`, `DATA-PROV.1`, `UX-UNKNOWN.1` | Full |
| `GUARD-REPUTE.1` | No unverified parity claim | `NFR-FID.1` guardrail | Full |
| `GUARD-REPUTE.2` | No implied approval | `FR-CHECK.2` | Full |
| `GUARD-SCOPE.1` | Not generalised | *Governance* | N/A by design |
| `GUARD-SCOPE.2` | No map writing | `NFR-VALID.2` | Full |
| `GUARD-SCOPE.3` | No gadget authoring | `FR-SHOW.9`, `NFR-VALID.3`, `NFR-SEC.1` | Full |
| `GUARD-SCOPE.4` | No generation | *Scope boundary* | N/A by design |
| `GUARD-SCOPE.5` | No engine dependency | `NFR-DEP.1` | Full |

### 8.2 Backward traceability (PRD → BRD)

Every PRD requirement cites at least one BRD source in its **Traces to** field.
Spot-check of the categories:

| PRD ID | BRD source(s) |
|:---|:---|
| `FR-READ.6` | `OPER-MIGRATE.1` |
| `FR-CHECK.4` | `BIZ-EFF.1` (via `DR-10`, `ASSUM-7`) |
| `NFR-PERF.2` | `BIZ-EXP.2` (via `DR-7`) |
| `DATA-CONST.1` | `FR-CHECK.4`, `ASSUM-7` |
| `UX-UNKNOWN.1` | `FR-SHOW.11`, `NFR-VALID.1` ← `GUARD-ACC.1` |
| `NFR-A11Y.1` | **No BRD source** - see gap report |

### 8.3 Gap report

**BRD items with no PRD coverage:** none. Four are marked *N/A by design* -
`BIZ-SAT.1`, `COMP-LICENSE.1`, `GUARD-SCOPE.1`, `GUARD-SCOPE.4` - because they
are project, repository or governance obligations rather than product
behaviour. Each remains binding in the BRD.

**PRD items with no BRD source:** one.

- **`NFR-A11Y.1`** has no BRD source. The BRD contains no accessibility
  requirement, and no statutory regime applies to a volunteer desktop tool.
  It is retained as a **quality target** rather than removed, because the
  checklist view encodes results and colour-only encoding would fail the
  reviewing persona outright. Flagged here rather than silently justified: this
  is the one candidate for gold-plating in the document, and it is P1.

---

## 9. Quality Gates

| Gate | Status |
| --- | --- |
| All 6 PRD sections plus agent-readiness sections present | Met - 9 sections |
| Every FR has ID, traces-to, requirement and dual-format acceptance criteria | Met - 27 FRs |
| NFRs have quantified thresholds with three-layer metrics | Met - 14 NFRs, each with primary, secondary and guardrail. `NFR-PERF.4` states a falsifiable guardrail and defers its numeric ceiling to measurement rather than asserting an unsourced figure |
| Data requirements specify source, refresh, retention, privacy | Met - 5 DATA items |
| UX requirements include happy path, error states, edge cases | Met - 6 UX items, each with loading and empty states; every persona has at least one journey |
| Traceability matrix shows full BRD coverage | Met - 27 of 27 BRD items; 4 N/A by design with rationale; 1 PRD item with no BRD source, flagged |
| Semantic glossary defines all domain terms | Met - 21 terms |
| Protection patterns stated | Met - 9 locked decisions |
| Phase identity and boundary match the registry | Met |
| Exit criteria have verifiable product requirements | Met - 13-item verification checklist |
| No subjective language | Met - "fast", "intuitive" and "easy" do not appear as requirement language; every threshold is numeric or externally anchored |
