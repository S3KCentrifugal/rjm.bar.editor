# rjm.bar.editor

A Beyond All Reason map editor built in the [Godot](https://godotengine.org)
engine.

Two intents:

1. Load and edit map layers and terrain data.
2. **Version 1:** load and explore *all* of a map's assets, with camera and
   navigation components for fast inspection. Editing is out of scope for v1.

## Status

Requirements capture. Nothing is implemented yet.

| Stage | Artifact | State |
| --- | --- | --- |
| Intent - capture and classify | [`docs/requirements/intent/intent-record.md`](docs/requirements/intent/intent-record.md) | Done |
| Intent - explore | same record, stage 0.3 | Done |
| Intent - assess | [`docs/requirements/intent/framework-assessment.md`](docs/requirements/intent/framework-assessment.md) | Done |
| Intent - refine + converge | same record, stages 0.5-0.6 | Done |
| Intent - brief + phase registry | [`docs/requirements/intent/intent-brief.md`](docs/requirements/intent/intent-brief.md) | Done |
| Business requirements (P01 MAE) | [`docs/requirements/phases/phase-01-map-asset-exploration/business-requirements/`](docs/requirements/phases/phase-01-map-asset-exploration/business-requirements/) | **Baselined v1.0** |
| Product requirements (P01 MAE) | [`.../product-requirements/`](docs/requirements/phases/phase-01-map-asset-exploration/product-requirements/) | **Validated** |
| Technical design + IA (P01 MAE) | [`.../technical-design/`](docs/requirements/phases/phase-01-map-asset-exploration/technical-design/) | **Validated** - 3.9/5 |
| Task breakdown (P01 MAE) | [`.../tasks/summary.md`](docs/requirements/phases/phase-01-map-asset-exploration/tasks/summary.md) | **32 tasks, graph validated** |
| Implementation | - | Next - 3 tasks ready |

Agent and contributor rules are in [`AGENTS.md`](AGENTS.md).

## Related repositories

All under `C:\bardev`, side by side.

| Path | What it is | Use here |
| --- | --- | --- |
| `../rjm.bar.tools` | Rust `smrt-bar-map`: `.sd7`, SMF and `mapinfo.lua` decoding | Nearest prior art; source of truth for how a map decodes |
| `../rjm.bar.docs` | Shared BAR and Recoil game knowledge base | Source of game truth; read, never duplicated here |
| `../bar-RecoilEngine` | Recoil engine | Read-only reference for map loading and rendering |
| `../bar-Beyond-All-Reason` | Beyond All Reason game | Read-only reference for unit, gadget and asset data |
| `../rjm.bar.ai` | Rust BAR skirmish AI | Worked example of the specification chain used here |

Game facts live in `rjm.bar.docs`. Editor implementation knowledge lives here.
