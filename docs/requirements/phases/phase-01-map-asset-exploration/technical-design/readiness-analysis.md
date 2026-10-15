---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
stage: TDD readiness analysis
status: complete
authored: 2026-09-20
source: ../product-requirements/product-requirements.md (validated)
next: research-tdd-context (phase P01)
---

# TDD Readiness Report: P01 - Map Asset Exploration (MAE)

## Phase boundary

`P01`, Map Asset Exploration, `MAE`,
`docs/requirements/phases/phase-01-map-asset-exploration/`. No prerequisite
phases. Verified against the Intent Brief registry; analysis stays within P01.

## Requirements extraction

- **Total PRD requirements:** 54 - 28 FR, 15 NFR, 5 DATA, 6 UX
- **By architecture domain:** Decode and data pipeline **21** | Rendering and
  presentation **14** | Frontend and interaction **11** | Security **2** |
  Tooling and verification **4** | Build and packaging **2**
- **Requirements spanning multiple domains:** 9

### Classification

| PRD ID | Architecture domain | New or extends | Depends on |
|:---|:---|:---|:---|
| `FR-READ.1` | Decode pipeline - virtual file system | **Extends** `rjm.bar.tools::archive` | - |
| `FR-READ.2` | Decode pipeline | **Extends** `rjm.bar.tools::smf` | `FR-READ.1` |
| `FR-READ.3` | Decode pipeline - **new** SMT tile atlas | **New** | `FR-READ.2` |
| `FR-READ.4` | Decode pipeline - **new** minimap | **New** | `FR-READ.2` |
| `FR-READ.5` | Decode pipeline + **security** | **Extends** `rjm.bar.tools::luaenv` | `FR-READ.1` |
| `FR-READ.6` | Decode pipeline - **new** TDF parser | **New** | `FR-READ.1` |
| `FR-READ.7` | Decode pipeline + frontend | **New** | `FR-READ.5` |
| `FR-READ.8` | Cross-cutting data model | **New** | `DATA-PROV.1` |
| `FR-SHOW.1` | Rendering - terrain mesh | **New** | `FR-READ.2`, `FR-READ.3` |
| `FR-SHOW.2` | Rendering - **new** texture decode and upload | **New** | `FR-READ.1` |
| `FR-SHOW.3` | Decode + rendering | **New** | `FR-READ.5` |
| `FR-SHOW.4` | Rendering - sky | **New** | `FR-READ.5` |
| `FR-SHOW.5` | Rendering - water | **New** | `FR-READ.5` |
| `FR-SHOW.6` | Rendering - **new** S3O model loader | **New** | `FR-READ.1` |
| `FR-SHOW.7` | Decode pipeline | **New** | `FR-READ.5` |
| `FR-SHOW.8` | Rendering + frontend | **New** | `FR-READ.5` |
| `FR-SHOW.9` | Frontend - text presentation | **New** | `FR-READ.1` |
| `FR-SHOW.10` | Rendering - overlays | **New** | `FR-READ.2` |
| `FR-SHOW.11` | Cross-cutting - provenance display | **New** | `DATA-PROV.1` |
| `FR-NAV.1` to `.3` | Frontend - camera | **New** | `FR-SHOW.1` |
| `FR-CHECK.1` to `.3` | Frontend - checklist view | **New** | most decode requirements |
| `FR-CHECK.4` | Decode + **embedded data** | **New** | `DATA-CONST.1` |
| `FR-CONF.1` | Tooling - headless runner | **Extends** `rjm.bar.tools` CLI | all decode |
| `FR-CONF.2` | Tooling - capture and compare | **New** | `FR-NAV.1`, rendering |
| `NFR-PERF.1`, `.2` | Rendering performance | **New** | `FR-SHOW.1` |
| `NFR-PERF.3` | Concurrency - staged loading | **New** | decode pipeline |
| `NFR-PERF.4` | Memory management | **New** | `FR-SHOW.2` |
| `NFR-PORT.1` | Build and packaging | **New** | - |
| `NFR-DEP.1` | Architectural constraint | **New** | - |
| `NFR-SEC.1` | **Security architecture** | **Extends** `luaenv` sandbox | `FR-READ.5` |
| `NFR-REL.1` | **Resilience** - error taxonomy | **New** | all decode |
| `NFR-VALID.1` to `.3` | Architectural constraints | **New** | - |
| `NFR-LIC.1`, `.2` | Build and packaging | **New** | - |
| `NFR-A11Y.1` | Frontend | **New** | `FR-CHECK.1` |
| `NFR-FID.1` | Rendering - shader port | **New** | `FR-SHOW.2` |
| `DATA-PROV.1` | **Core data model** | **New** | - |
| `DATA-MAP.1` | Core data model | **Extends** | `FR-READ.*` |
| `DATA-CONST.1` | Embedded data + build | **New** | - |
| `DATA-REF.1` | Tooling data | **New** | `FR-CONF.2` |
| `DATA-SESS.1` | Frontend persistence | **New** | - |
| `UX-OPEN.1`, `UX-EXPLORE.1`, `UX-CHECK.1`, `UX-RELOAD.1`, `UX-UNKNOWN.1`, `UX-VERIFY.1` | Frontend interaction | **New** | corresponding FRs |

## Scalability & constraints checklist

This checklist assumes a multi-user networked service. **P01 is a single-user
local desktop application with no server, no network and no accounts.** Each
dimension is answered honestly, including where it does not apply, rather than
given a service-shaped answer.

| Dimension | Answer | Source |
|:---|:---|:---|
| **Scale** | One user, one process, one map open at a time. No concurrency across users exists | Product shape; `NFR-DEP.1` |
| **Volume** | Per map: archives 10-436 files, mean 96, up to ~50 MB of which ~25 MB is the tile atlas; heightmap up to 2049x2049 `u16`. Library: 254 archives plus 1 directory | `DATA-MAP.1`, `MR-CORP-01`, `MR-CORP-11` |
| **Throughput** | **Read-only.** Zero write throughput to map data by architectural prohibition (`NFR-VALID.2`). The only writes are reports, captures and session state | `NFR-VALID.2`, `DR-8` |
| **Latency** | Terrain navigable within 5 s of open; all textures within 20 s; no interface block longer than 1 s; 60 fps sustained, 30 fps hard floor; corner-to-corner traversal at or faster than in-game | `NFR-PERF.1` to `.3` |
| **Freshness** | A map file is static once opened. The only freshness concern is the authoring persona's recompile loop changing the file underneath an open view | `DATA-MAP.1`, `UX-RELOAD.1` |
| **Retention** | Decoded map: session only. Session state: until user clears. Reference captures: life of project, versioned. **Nothing about map data is retained** | `DATA-*` |
| **Availability** | **Not applicable.** No service, no SLA, no uptime concept. The equivalent guarantee is `NFR-REL.1`: zero crashes on any corpus member, including malformed input | `NFR-REL.1` |
| **Security** | **One boundary, and it is real.** Map archives are untrusted third-party files containing executable Lua. `NFR-SEC.1` requires a sandbox denying filesystem, network, process creation and unbounded execution. No auth, no encryption, no PII - there is no personal data anywhere in the system | `NFR-SEC.1`, `DATA-*` privacy classifications |
| **Geo** | **Not applicable.** Local application, no regions | - |
| **Budget** | **Zero.** No infrastructure. Cost is developer time only | BRD 5d |

**Gaps needing stakeholder clarification: none.** Every dimension is answered
from the PRD or BRD, or is structurally inapplicable with a stated reason.

## Component inventory

| # | Component | Responsibility | Serves | Technology category |
|:--|:---|:---|:---|:---|
| C1 | **VFS** | Uniform read access to a map, whether archived or a directory | `FR-READ.1`, `BIZ-EXP.4` | Rust library, extends `archive.rs` |
| C2 | **SMF decoder** | Decode header, heightmap, typemap, metalmap, feature block, vegetation header | `FR-READ.2` | Rust, extends `smf.rs` |
| C3 | **SMT decoder** | Decode the tile atlas and resolve per-square tile indices | `FR-READ.3` | Rust, **new** |
| C4 | **Minimap decoder** | Decode the DXT1 minimap and its mip chain | `FR-READ.4` | Rust, **new** |
| C5 | **Definition evaluator** | Evaluate `mapinfo.lua` and overlays in a sandbox; parse legacy `.smd` | `FR-READ.5`, `FR-READ.6`, `FR-READ.7`, `NFR-SEC.1` | Rust + sandboxed Lua 5.1, extends `luaenv.rs` |
| C6 | **Terrain deriver** | Slope, passability by movement class, metal spots, depth bands | `FR-SHOW.10`, `FR-CHECK.4` | Rust, extends `terrain.rs` |
| C7 | **Texture decoder** | Decode DDS, TGA, BMP and the tile atlas into GPU-uploadable form | `FR-SHOW.2`, `FR-SHOW.3` | Rust, **new** - required because Godot cannot load DDS at runtime |
| C8 | **Model loader** | Parse S3O piece hierarchies, vertices and index tables | `FR-SHOW.6` | Rust, **new** |
| C9 | **Provenance model** | Represent every value as derived with a citation, unknown, or absent | `DATA-PROV.1`, `FR-SHOW.11`, `NFR-VALID.1` | Rust type system - **the most pervasive component** |
| C10 | **Checklist evaluator** | Compute all twenty checklist properties; judge the five with numeric rules | `FR-CHECK.1`, `FR-CHECK.2`, `FR-CHECK.4` | Rust, **new** |
| C11 | **Embedded constants** | BAR movement and geothermal constants with their source commit | `DATA-CONST.1`, `ASSUM-7` | Build-time generated Rust data |
| C12 | **Extension bridge** | Expose the core to Godot across the GDExtension boundary | all rendering and frontend | `godot-rust/gdext` |
| C13 | **Terrain renderer** | Present the heightmap as a clipmap surface with the map's shading | `FR-SHOW.1`, `NFR-FID.1` | Terrain3D with shader override |
| C14 | **Scene presenter** | Sky, water, feature instances, start position markers, overlays | `FR-SHOW.4` to `FR-SHOW.8`, `FR-SHOW.10` | Godot scene tree |
| C15 | **Camera controller** | Ground-anchored pan, orbit, zoom; altitude-scaled speed; named positions | `FR-NAV.1` to `.3` | Godot |
| C16 | **Inspection UI** | Checklist view, option picker, asset inventory, Lua source viewer | `FR-CHECK.1` to `.3`, `FR-READ.7`, `FR-SHOW.9`, `NFR-A11Y.1` | Godot Control nodes |
| C17 | **Conformance runner** | Non-interactive pass or fail over a whole library | `FR-CONF.1`, `OPER-REPORT.1` | Rust binary, headless |
| C18 | **Reference tool** | Capture and compare parity screenshots and traversal timings | `FR-CONF.2`, `DATA-REF.1` | Godot headless plus Rust |
| C19 | **Session store** | Window layout, last map, overlay and option selections | `DATA-SESS.1` | Local file |

## C4 container candidates

| Container | Contents | Runtime |
|:---|:---|:---|
| **`mae-core`** - map decoding library | C1-C11 | Rust library (`cdylib` + `rlib`), no Godot dependency |
| **`mae-gdext`** - Godot extension | C12, and the core's exposed surface | Rust GDExtension loaded by Godot |
| **`mae-app`** - the editor | C13-C16, C19 | Godot 4.4+ project |
| **`mae-conform`** - verification CLI | C17 | Rust binary, links `mae-core` directly, **never loads Godot** |
| **`mae-reference`** - parity tooling | C18 | Godot headless plus Rust |
| **Terrain3D** *(external)* | Clipmap terrain, LOD | C++ GDExtension, MIT |
| **Map library** *(external)* | `.sd7` archives and unpacked directories | User's filesystem - **read-only** |

**A deliberate structural point.** `mae-core` has no Godot dependency, so
`mae-conform` can verify the entire corpus without a renderer, a window or a
GPU. That serves `FR-CONF.1`, makes `NFR-DEP.1` provable, and means the decode
pipeline is testable long before anything renders - which directly addresses
the feedback desert recorded as an accepted risk in the intent record.

## Architecture decision points

Each becomes an ADR in the technical design.

| # | Decision | Candidate options | Driver |
|:--|:---|:---|:---|
| AD-1 | **Rust-to-Terrain3D access** | Direct dynamic calls from Rust; a GDScript glue layer; abandon Terrain3D for a custom clipmap | `PR-TECH-01` - gdext has no typed API for external GDExtension classes (issue #372). **Highest-risk decision in the phase** |
| AD-2 | **Terrain data injection into Terrain3D** | Programmatic region API; synthesise the format its importer expects; custom mesh | `PR-TECH-02` - the documented path is an editor importer, not a runtime API. Threatens `ASSUM-2` |
| AD-3 | **Tile atlas memory strategy** | Hand pre-compressed DXT1 blocks to Godot; decode to RGBA in Rust; stream tiles on demand | `PR-TECH-05` unresolved. Determines `NFR-PERF.4`'s achievable ceiling |
| AD-4 | **Texture upload path** | `Image` plus `ImageTexture`; `RenderingDevice` directly | `FR-SHOW.2`, `NFR-PERF.3` |
| AD-5 | **Lua runtime and sandbox mechanism** | `mlua` with restricted globals; a custom interpreter; reuse `rjm.bar.tools::luaenv` as-is | `NFR-SEC.1` - must deny filesystem, network, process and unbounded execution |
| AD-6 | **Provenance representation** | Generic `Provenanced<T>` wrapper; enum per value; parallel metadata map | `DATA-PROV.1` - pervasiveness makes ergonomics decisive |
| AD-7 | **Concurrency model for staged loading** | Background thread pool with progressive publication; async tasks; synchronous with yields | `NFR-PERF.3` - no interface block over 1 second |
| AD-8 | **Shader porting approach** | Port `SMFFragProg.glsl` to Godot shading language; reimplement from the engine's C++ render state; defer | `NFR-FID.1`, P2. Carries the GPL consequence (`COMP-LICENSE.1`) |
| AD-9 | **Error taxonomy and failure policy** | Typed error enum per decode stage; string errors; panic-free contract | `NFR-REL.1`, `BIZ-EXP.1` no-exceptions |
| AD-10 | **Code sharing with `rjm.bar.tools`** | Extract a shared crate; vendor the modules; depend on the existing crate directly | `DEP-1`. Affects both repositories |
| AD-11 | **Decoded-map caching between reopens** | None; in-memory by path and mtime; on-disk cache | `UX-RELOAD.1`, persona P-AUT's recompile loop |
| AD-12 | **Embedded constants generation** | Build script reading `bar-Beyond-All-Reason`; checked-in generated file with commit | `DATA-CONST.1`, `ASSUM-7`. Must record provenance, and must not require BAR at runtime |
| AD-13 | **Packaging and distribution** | Godot export templates per platform; separate CLI binary; single bundle | `NFR-PORT.1`, `NFR-LIC.1` |

## Quick technology landscape scan

- **Standard stack for this system type** - a native decoding core with a game
  engine as the presentation shell is the conventional shape for format viewers
  and modding tools. The unusual element here is two GDExtensions in one
  project (ours in Rust, Terrain3D in C++), which is where AD-1 bites.
- **Cloud services:** none apply. There is no server component.
- **Emerging patterns:** none of serverless, edge or AI-native has any bearing
  on a local read-only desktop tool. Recorded so the design does not import
  irrelevant architecture.
- **Known scaling ceilings:** Terrain3D is documented to 65.5 km against a
  16,384-unit requirement, so terrain extent has roughly 4x headroom. The real
  ceiling is GPU memory for the tile atlas, which AD-3 decides.

## Research plan

| # | Area | Priority | What to investigate | Decides |
|:--|:---|:---|:---|:---|
| 1 | **Godot integration and interop** | **High** | Whether Terrain3D accepts terrain data programmatically at runtime; whether Rust can drive it acceptably or a GDScript layer is needed; cost of each | AD-1, AD-2 |
| 2 | **Texture and memory strategy** | **High** | Whether Godot accepts pre-compressed DXT1 at runtime; RGBA fallback cost; streaming viability; upload path | AD-3, AD-4, `NFR-PERF.4` |
| 3 | **Sandbox threat model** | **High** | Lua runtime options and their isolation guarantees; instruction and time limiting; known escape classes; what `rjm.bar.tools::luaenv` already guarantees and what it does not | AD-5, `NFR-SEC.1` |
| 4 | **Provenance ergonomics** | **High** | How comparable systems carry per-value provenance without poisoning every signature | AD-6 |
| 5 | **Failure mode analysis** | Medium | FMEA across the decode pipeline; panic-free guarantees in Rust; behaviour on truncated, hostile and malformed input | AD-9, `NFR-REL.1` |
| 6 | **Concurrency and staged loading** | Medium | Threading across the GDExtension boundary; progressive publication; what gdext permits off the main thread | AD-7, `NFR-PERF.3` |
| 7 | **Shader port feasibility** | Medium | Mapping Recoil's GLSL to Godot's shading language; Terrain3D's generated shader as a starting point; which of the ten texture slots have no direct equivalent | AD-8 |
| 8 | **Build, packaging and code sharing** | Low | Cross-compiling a Rust GDExtension for Windows and Linux; Godot export templates; crate extraction versus vendoring | AD-10, AD-12, AD-13 |

**Storage, compute, protocol, caching-layer and infrastructure evaluation are
not applicable** in their usual sense - there is no database, no service
topology, no network protocol and no deployment environment. The equivalent
questions for this system are areas 1, 2 and 6.

## Gaps needing clarification

**None requiring stakeholder input.** All PRD open questions were resolved
before baseline. Three technical unknowns remain, and all three are research
tasks rather than questions for the stakeholder:

1. Terrain3D runtime data injection (`ASSUM-2`) - research area 1.
2. Rust-to-Terrain3D interop cost (`ASSUM-3`) - research area 1.
3. Runtime compressed-texture support (`PR-TECH-05`) - research area 2.

The first two are already registered as `DEP-2` "Pending" and must resolve
before the technical design is baselined.

## Quality gates

| Gate | Status |
| --- | --- |
| Every PRD requirement classified by architecture domain | Met - 54 of 54 |
| Scalability checklist complete, gaps flagged | Met - 10 dimensions; 3 marked structurally N/A with reasons; no stakeholder gaps |
| Major components identified with responsibility and traceability | Met - 19 components |
| C4 container candidates grouped | Met - 5 owned containers, 2 external |
| Architecture decision points listed | Met - 13, each with candidate options and a driver |
| Research plan specifies priority areas | Met - 8 areas, 4 High; inapplicable areas named rather than silently dropped |
| Analysis remains within the selected phase | Met - P01 only |
