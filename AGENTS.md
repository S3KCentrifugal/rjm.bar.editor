# rjm.bar.editor - Agent Instructions

These instructions apply to the whole repository and are the canonical guidance
for every coding agent (Claude Code, Codex, Copilot, Cursor, Gemini CLI, Aider,
Cline, and any other). Vendor files route here; they must not duplicate or
override this file.

## Project

A **Beyond All Reason map editor built in the Godot engine**. Two intents:

1. Load and edit map layers and terrain data.
2. **Version 1 (current scope):** load and explore *all* of a map's assets,
   with camera and navigation components for fast inspection. Editing is out of
   scope for v1.

**Status: requirements capture. No code exists and none is to be written yet.**
The current task is the specification chain, starting at intent. Do not scaffold
a Godot project, add dependencies, or write GDScript/C# until the task breakdown
says to.

## Documentation harness

This project uses the same spec chain as `../rjm.bar.ai`. Artifacts live under
`docs/requirements/`; each stage has a skill that produces it.

| Stage | Skill | Artifact |
| --- | --- | --- |
| Capture + classify | `capture-intent` | `docs/requirements/intent/intent-record.md` |
| Explore | `explore-intent` | same record, extended |
| Assess | `assess-intent` | psychological framework assessment |
| Refine | `refine-intent` | convergence cycles |
| Brief | `write-intent-brief` | `docs/requirements/intent/intent-brief.md` + phase registry |
| Business requirements | `analyze-brd-readiness` -> `research-brd-context` -> `draft-brd` -> `validate-brd` | `docs/requirements/brd/` |
| Product requirements | `analyze-prd-readiness` -> `research-prd-context` -> `draft-prd` -> `validate-prd` | `docs/requirements/prd/` |
| Technical design | `analyze-tdd-readiness` -> `research-tdd-context` -> `draft-tdd` -> `draft-information-architecture` -> `validate-tdd` | `docs/requirements/tdd/` |
| Tasks | `generate-tasks` -> `generate-task-summary` | `docs/requirements/tasks/` |
| Execution | `execute-task` -> `review-implementation` | source tree |

Run the stages in order. Do not skip ahead; each skill reads the previous
artifact. `audit-traceability` and `propagate-change` maintain the chain after
baselining.

## Workspace

Every project sits side by side under `C:\bardev`. Paths below are relative to
this repository's parent.

| Project | Path | What it is | Write policy |
| --- | --- | --- | --- |
| **rjm.bar.editor** | `.` | this repository - Godot BAR map editor | writable |
| **rjm.bar.docs** | `../rjm.bar.docs` | central game knowledge base for BAR + Recoil, shared by all projects | writable; **game facts only** |
| **rjm.bar.tools** | `../rjm.bar.tools` | Rust `smrt-bar-map`: reads `.sd7` archives, parses SMF, evaluates `mapinfo.lua`, derives terrain facts | writable |
| **rjm.bar.ai** | `../rjm.bar.ai` | Rust BAR skirmish AI, first-principles design; requirements capture only, no code | writable |
| **s3k-CircuitAI** | `../s3k-CircuitAI` | the user's CircuitAI/BARb fork (`S3KCentrifugal/CircuitAI`, branch `smrt`) | writable |
| **rlcevg-CircuitAI** | `../rlcevg-CircuitAI` | upstream CircuitAI (`rlcevg/CircuitAI`, branch `barb5`) for comparison | read-only by convention |
| **bar-Beyond-All-Reason** | `../bar-Beyond-All-Reason` | upstream BAR game mirror (`beyond-all-reason/Beyond-All-Reason`, `master`) | **strictly read-only** |
| **Beyond-All-Reason** | `../Beyond-All-Reason` | the user's BAR game fork (`S3KCentrifugal/Beyond-All-Reason`) | writable; game changes go here, never to the mirror |
| **bar-RecoilEngine** | `../bar-RecoilEngine` | upstream Recoil engine mirror (`beyond-all-reason/RecoilEngine`, `master`) | **strictly read-only** |
| **bar-RecoilEngine-s3k-build** | `../bar-RecoilEngine-s3k-build` | local build clone of the engine (origin = `C:\bardev\bar-RecoilEngine`); holds `build-amd64-windows/` | build tree; do not author source here |
| Game install | `%LOCALAPPDATA%\Programs\Beyond-All-Reason\data` | runtime logs (`infolog.txt`, `log/`), replays (`demos/`), 500+ installed `maps/` | read for evidence; never edit |

### Read-only policy

`bar-Beyond-All-Reason` and `bar-RecoilEngine` are authoritative local
references and are strictly read-only. Never edit, create, delete, rename,
format, generate, build, test, checkout, reset, clean, or update submodules in
either tree. Limit commands there to inspection: file reads, listings,
`git status`, `git log`, `git show`, `git diff`, `git grep`, `git ls-files`. If
a change belongs upstream, report it and name the target file without applying
it. Do not follow a path into a read-only tree and write through it.

Do not assume another machine has this layout; verify paths before use.

## Navigation

Where to look, by repository. Grep is fine - unit ids, gadget filenames and role
names are stable tokens.

### `../rjm.bar.docs` - game truth (search here first)

| Path | Content |
| --- | --- |
| `knowledge/README.md` | layer map and reading order - **start here** |
| `knowledge/00-glossary.md` | canonical terms and numeric bands |
| `knowledge/10-engine/` | Recoil rules: movement, sensors, damage, weapons, construction, air |
| `knowledge/20-game-mechanics/` | BAR rules; `20-lua-mechanics-catalog.md` indexes gadgets |
| `knowledge/30-units/<faction>/<id>.md` | one page per unit (BAR internal id, e.g. `armfast`) |
| `knowledge/40-roles-and-counters/` | role taxonomy, counter matrix |
| `knowledge/50-economy/`, `60-tactics/`, `70-strategy/`, `80-theory/` | economy arithmetic, doctrine, theory |
| `knowledge/img/units/` | 96 px unit thumbnails |
| `tools/knowledge/` | generators: `extract.py`, `thumbs.py`, `gen_units.py`, `gen_layers.py`, `check.py` |

Every page carries front-matter with `layer`, `sources`, `confidence`
(`data` / `derived` / `community` / `theory`) and `verified_against`. Never
hand-edit generated pages; change the generator and regenerate.

### `../rjm.bar.tools` - map parsing (the nearest prior art for this project)

| Path | Content |
| --- | --- |
| `src/archive.rs` | `.sd7` access; reads only needed entries, never the 100 MB tile file |
| `src/smf.rs` | Spring Map Format binary parser (height, type, metal grids) |
| `src/luaenv.rs` | sandboxed Lua 5.1 with a VFS shim and Spring globals |
| `src/mapinfo.rs` | typed view over the evaluated `mapinfo.lua` table |
| `src/terrain.rs` | per-cell elevation, slope, depth band, passability |
| `src/analysis.rs` | connectivity, metal spots, per-start classification |
| `src/emit.rs` | AngelScript `MapConfig` and JSON rendering |
| `generated/circuitai-maps/` | committed per-map output (227 maps); do not hand-edit |
| `README.md` | the engine rules it reproduces, with source citations |

This crate already answers "how is a BAR map decoded correctly". Consult it
before re-deriving any archive, SMF or `mapinfo.lua` behaviour.

### `../bar-RecoilEngine` - engine (read-only)

| Path | Content |
| --- | --- |
| `rts/Map/` | map loading, SMF reader, height/slope/metal maps |
| `rts/Rendering/` | renderers, including map and terrain rendering |
| `rts/Sim/` | simulation: units, movement, weapons, features |
| `rts/Lua/` | Lua interface; `LuaParser.cpp` defines `mapinfo` evaluation rules |
| `rts/System/` | core systems, file system, archive scanner |
| `rts/ExternalAI/Interface/` | skirmish AI C ABI: `AISEvents.h`, `AISCommands.h`, `SSkirmishAICallback.h` |
| `AI/Interfaces/`, `AI/Wrappers/`, `AI/Skirmish/` | AI interface, generated wrappers, bundled AIs |
| `doc/site/` | published documentation source |
| `AGENTS.md`, `test/AGENTS.md`, `coding-agents/` | upstream agent guidance, build and test recipes |

### `../bar-Beyond-All-Reason` - game (read-only)

| Path | Content |
| --- | --- |
| `units/**/*.lua` | UnitDefs |
| `weapons/`, `unitbasedefs/` | WeaponDefs and shared unit bases |
| `gamedata/` | `unitdefs.lua`, `unitdefs_post.lua`, `alldefs_post.lua`, `movedefs.lua`, `modoptions.lua`, `unitDefRenames.lua` |
| `luarules/gadgets/` | synced/unsynced game logic |
| `luaui/` | widgets and UI |
| `mapgenerator/` | procedural map generation |
| `unitpics/`, `objects3d/`, `unittextures/` | icons, models, textures |
| `language/` | localisation strings |

### `../s3k-CircuitAI` - the user's AI fork

| Path | Content |
| --- | --- |
| `AGENTS.md` | canonical instructions; its "Repository Map" describes every file |
| `src/circuit/` | C++20 implementation (`map/`, `module/`, `task/`, `terrain/`, `unit/`, `script/`) |
| `data/script/` | active AngelScript policy (`src/roles/`, `src/manager/`, `src/maps/`) |
| `data/config/` | JSON behaviour profiles, one directory per profile |
| `data_sample/` | reference-only sample tree; never the implementation target |
| `doc/` | design notes, `roles/`, `knowledge/`, `known-issues.md`, `angelscript-references.md` |
| `skills/` | repo skills: `convention-angelscript`, `maintain-changelog`, `troubleshoot-bar-logs` |
| `tools/knowledge/` | `barb_report.py`, doc/link/role checkers |

### `../rjm.bar.ai` - the Rust AI

`docs/requirements/intent/intent-record.md` is the only substantive file. Read
it as the worked example of the spec chain this project follows.

## Source-of-truth routing

Use the narrowest authoritative source:

- Editor behaviour, architecture and UX: this repository.
- **How a BAR map decodes** (archive layout, SMF fields, `mapinfo.lua`
  evaluation, slope and passability maths): `../rjm.bar.tools` first, then
  `../bar-RecoilEngine/rts/Map/` and `rts/Lua/LuaParser.cpp`.
- Game and engine facts, unit data, mechanics, doctrine:
  `../rjm.bar.docs/knowledge/` first; the read-only BAR and Recoil trees only
  for what it lacks - and record the fact in the knowledge base afterwards.
- Godot APIs and idioms: the Godot version's own documentation, pinned in the
  TDD once chosen.

Do not copy game facts into this repository; name the knowledge-base path
instead. Do not put editor-implementation detail into the knowledge base.

## Working rules

- Specification before code. Every stage artifact traces to the one above it.
- Keep the two read-only trees pristine; inspection commands only.
- Do not commit automatically. Commit only when asked, and only the change
  asked for.
- Record the BAR and Recoil commits used in artifact front-matter so staleness
  is detectable.
- When a fact here contradicts `../rjm.bar.docs`, that repository wins for game
  facts and this one wins for editor facts; fix the other side and say so.
