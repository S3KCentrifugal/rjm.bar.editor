---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
stage: Architecture research
status: complete
gathered: 2026-09-20
source_plan: readiness-analysis.md
next: draft-tdd (phase P01)
---

# Architecture Research Bundle: P01 - Map Asset Exploration (MAE)

Architecture decisions are trade-offs, not best practices. Every
recommendation below names the constraint it optimises for and the trade-off it
accepts. Constraints come from the readiness analysis scalability checklist:
**one user, one process, read-only, local, no network, no personal data, one
untrusted-input security boundary.**

## Key findings

1. **`gdext` threading is experimental and explicitly warned as unsound.**
   Access to `Gd<T>` from a non-main thread panics without the experimental
   feature, and the maintainers describe a high risk of unsoundness with it. This
   settles AD-7: decoding threads must never touch Godot objects, and results
   cross to the main thread via deferred calls. The `mae-core` / `mae-gdext`
   split already proposed makes this natural rather than awkward.
2. **`Image.FORMAT_DXT1` exists (value 17)** and Godot's documentation warns
   only that *modifying* VRAM-compressed images is prohibited - which this
   product never does, since it creates and uploads textures without editing
   them. This makes "keep the atlas compressed" the leading option for AD-3,
   though it is inference from adjacent documentation rather than an explicit
   statement, so it stays a spike.
3. **Lua sandboxing must be whitelist-based, and blocking globals is not
   sufficient.** Resource exhaustion - infinite loops, unbounded memory - is a
   separate problem from capability denial, and needs instruction-count hooks
   and memory limits. `NFR-SEC.1` names four capability classes; only two of
   them are solved by clearing globals.
4. **Terrain3D's documented data path remains an editor importer.** No runtime
   data API is documented. AD-2 must be settled by reading the Terrain3D source,
   not its documentation.
5. **The conventional cloud architecture questions do not apply.** No storage
   engine, no compute model, no wire protocol, no deployment topology. The
   equivalent decisions for this system are in-process: data representation,
   threading, and the FFI boundary.

## Storage evaluation

**No database is required, and none should be introduced.** The readiness
checklist shows why: one user, one process, read-only inputs, session-lifetime
data, no personal data, no multi-user consistency question. CAP theorem does not
apply - there are no partitions and no replicas.

Per-data-type evaluation, against `DATA-*`:

| Data | Consistency need | Scale | Access pattern | Chosen store | Trade-off accepted |
|:---|:---|:---|:---|:---|:---|
| Decoded map (`DATA-MAP.1`) | None - single owner | Up to ~50 MB in, larger in memory | Read many, write once per open | **In-process memory** | Nothing persists across runs; reopen re-decodes unless AD-11 adds a cache |
| Value provenance (`DATA-PROV.1`) | None | Hundreds of records per map | Read on display | **In-process, alongside each value** | Pervasive type impact - the subject of AD-6 |
| Embedded constants (`DATA-CONST.1`) | Must match a recorded BAR commit | Tens of values | Read-only, compile-time | **Compiled-in Rust data with provenance** | Staleness risk, mitigated by `ASSUM-7` revalidation |
| Reference set (`DATA-REF.1`) | Versioned with the project | 6-10 maps, a few images each | Read on comparison | **Files in the repository** | Repository size grows with captures; acceptable |
| Session state (`DATA-SESS.1`) | None | Kilobytes | Read at start, write on change | **Single local config file** | No migration story needed at this size |
| Conformance results (`FR-CONF.1`) | None | One row per corpus member | Written once per run | **Plain text or JSON file** | None |

**Evidence:** `TR-STORE-01` - derived from the readiness scalability checklist,
which is itself derived from `DATA-*` and the BRD. No external benchmark
applies, because no candidate store is under consideration.

## Compute model evaluation

Also not the usual question: there are no services to place. The real decision
is **where each component runs relative to the Godot main thread**.

| Component | Execution pattern | Placement | Rationale |
|:---|:---|:---|:---|
| C1-C11 decode and derive | CPU-bound, bursty at open | **Worker threads in `mae-core`, never touching Godot** | Required by `TR-COMPUTE-01`; enables `NFR-PERF.3` staged loading |
| C12 extension bridge | Marshalling | **Main thread**, receiving deferred results | gdext threading is unsound; see `TR-COMPUTE-01` |
| C13-C16 rendering and UI | Frame-driven | **Main thread** | Godot requirement |
| C17 conformance runner | Batch | **Separate process, no Godot at all** | Links `mae-core` directly; proves `NFR-DEP.1` and runs headless |
| C18 reference tool | Batch plus rendering | **Godot headless** | Needs the renderer to capture |

**`TR-COMPUTE-01`.** gdext's own documentation describes threading support as
experimental, adding synchronisation and disabling single-thread checks, "not
ironed out yet with a high risk of unsoundness"; without it, non-main-thread
access to `Gd<T>` panics before the call. `call_deferred` is the documented
route from a worker thread to the main thread.
Source: [gdext docs](https://godot-rust.github.io/docs/gdext/master/godot/index.html),
[gdext issue #1131](https://github.com/godot-rust/gdext/issues/1131), 2026-09-20.

**Trade-off accepted:** the decode pipeline cannot construct Godot resources
directly. It produces plain data - byte buffers, vertex arrays, grids - which
the bridge converts on the main thread. This adds a marshalling step and is the
price of soundness.

## Communication protocol evaluation

No network protocols. The interfaces are in-process boundaries.

| Interface | Direction | Mechanism | Rationale |
|:---|:---|:---|:---|
| `mae-app` → `mae-gdext` | Request/response | **GDExtension method calls** | The only mechanism available |
| `mae-gdext` → `mae-core` | Request/response | **Direct Rust calls** | Same process, no boundary cost |
| `mae-core` worker → main thread | Completion notification | **Channel plus `call_deferred`** | `TR-COMPUTE-01` |
| `mae-app` → Terrain3D | Request/response | **Dynamic calls via `Object`** or a GDScript layer | `TR-PROTO-01` - no typed API exists |
| `mae-conform` → `mae-core` | Direct | **Rust crate dependency** | Deliberately excludes Godot |

**`TR-PROTO-01`.** godot-rust issue #372 requests a type-safe API for classes
defined in other GDExtensions; it remains open. Calls into Terrain3D from Rust
are therefore untyped and unchecked at compile time, with errors surfacing at
runtime.
Source: [gdext issue #372](https://github.com/godot-rust/gdext/issues/372), 2026-09-20.

## Security architecture research

The product has **one** security boundary, and it is genuinely exposed: map
archives are files users obtain from other people and from the internet, and
they contain Lua that the product must execute to read the map correctly.

**`TR-SEC-01` - sandboxing is whitelist-based, and capability denial is only
half the problem.** Practitioner guidance is consistent: start by trusting
nothing and permit only what is demonstrably safe; clearing `io`, `os`,
`loadstring`, `require` and similar globals handles capability, but **resource
exhaustion is a separate axis** requiring instruction-count hooks and memory
limits. Long-standing issues in the Rust Lua bindings note that running
untrusted code is inadvisable without execution and memory limits exposed.
Sources: [lua-users SandBoxes](https://lua-users.org/wiki/SandBoxes),
[rlua issue #38](https://github.com/mlua-rs/rlua/issues/38),
[Luau sandboxing](https://luau.org/sandbox/), 2026-09-20.

Mapping to `NFR-SEC.1`'s four capability classes:

| Capability to deny | Mechanism | Sufficient alone? |
|:---|:---|:---|
| Filesystem | Whitelist globals; VFS shim exposes only archive contents | Yes, if whitelist not blacklist |
| Network | No networking library exposed | Yes |
| Process creation | `os.execute` and equivalents absent from the whitelist | Yes |
| Unbounded execution | **Instruction-count hook plus memory limit** | **No - requires a distinct mechanism** |

**`TR-SEC-02` - no other security surface exists.** No authentication, no
authorization, no encryption at rest or in transit, no secrets, no PII, no
compliance framework. Recorded explicitly so the technical design does not
import auth architecture the product has no use for. The only `COMP-*`
requirements are licensing (`NFR-LIC.1`, `NFR-LIC.2`), which are packaging
obligations rather than security controls.

## Resilience research and FMEA

`NFR-REL.1` requires zero crashes on any input, and `BIZ-EXP.1` admits no
exclusions, so a single unhandled failure blocks phase closure. That raises the
severity of every decode failure mode.

| # | Component | Failure mode | Sev | Occ | Det | RPN | Mitigation |
|:--|:---|:---|:---:|:---:|:---:|:---:|:---|
| F1 | C5 definition evaluator | Malicious or defective map Lua loops forever or exhausts memory | 9 | 3 | 4 | **108** | Instruction-count hook and memory ceiling (`TR-SEC-01`); evaluation on a worker thread so the UI stays responsive |
| F2 | C2/C3 decoders | Truncated or malformed file causes a panic | 9 | 4 | 3 | **108** | Panic-free contract: all parsing returns typed errors; bounds-checked slicing; fuzz the decoders |
| F3 | C7 texture decoder | Atlas decode exhausts memory on the largest archive | 8 | 4 | 3 | **96** | AD-3 memory strategy; stream tiles if compressed upload proves unavailable |
| F4 | C12 bridge | Worker thread touches a Godot object, causing unsoundness | 9 | 3 | 3 | **81** | Architectural: `mae-core` has no Godot dependency, so this is prevented by the type system rather than by discipline |
| F5 | Terrain3D | Runtime data injection proves unsupported | 7 | 5 | 2 | **70** | AD-2 spike before design baseline; fallback to a custom mesh |
| F6 | C5 | Legacy `.smd` map fails to parse | 6 | 3 | 2 | **36** | `FR-READ.6`; one known map, tested explicitly |
| F7 | C11 | Embedded constants drift from BAR | 7 | 4 | 6 | **168** | **Highest RPN.** Constants carry their source commit and it is displayable; revalidation at each milestone (`ASSUM-7`). Detection is poor because a stale constant produces a plausible wrong answer |
| F8 | C9 | A value is displayed without provenance | 8 | 3 | 5 | **120** | AD-6: make provenance structural so an un-provenanced value cannot be constructed |

**Two findings from the FMEA.**

- **F7 is the highest-risk failure in the system** and the hardest to detect,
  because a stale movement constant yields a pathability answer that looks
  derived and is wrong - precisely what `GUARD-ACC.1` exists to prevent, arriving
  through the one door the guardrail does not watch. Detection, not prevention,
  is where the design effort belongs.
- **F4 and F8 should be eliminated by construction rather than mitigated.** Both
  are avoidable with the right type boundaries, which is what AD-6 and the
  container split are for.

RTO and RPO do not apply: there is no service to recover and no data to lose.
The equivalent guarantee is that a failed map leaves the application usable
(`NFR-REL.1` secondary metric).

## Infrastructure research

- **Deployment topology:** none. A desktop application, distributed as a
  download.
- **Environments:** none. No dev, staging or production split.
- **CI/CD:** a build matrix producing Windows and Linux artifacts, plus the
  conformance run as the principal gate. `NFR-PORT.1` requires identical
  per-map results on both platforms, which makes the corpus run a
  cross-platform differential test rather than merely a smoke test.
- **Observability:** no telemetry. The product is offline and collects nothing.
  The diagnostic surface is the conformance report and local logs.
  `TR-INFRA-01`: recorded explicitly, because "add OpenTelemetry" would be a
  reflex here and would contradict the product's offline, no-collection shape.

## ADR drafts

### ADR-01: Rust-to-Terrain3D access

**Status:** Proposed
**Context:** `mae-gdext` is a Rust GDExtension; Terrain3D is a C++ GDExtension.
`TR-PROTO-01` establishes there is no typed API for external extension classes.
**Options:**
1. **Direct dynamic calls from Rust** - no extra layer; untyped, unchecked, errors at runtime.
2. **GDScript glue layer** - dynamic dispatch is idiomatic in GDScript rather than a workaround; adds a thin layer and a language boundary.
3. **Drop Terrain3D for a custom clipmap** - full control, full cost, squarely in the weakest skill area (`RISK`: PF-04 skill finding).
**Decision:** **Option 2**, with option 1 used where a call is trivial.
**Rationale:** The interaction surface is small - supply terrain data, set a shader override, configure regions. Confining untyped calls to a layer whose language is dynamic anyway keeps the Rust side honest, and keeps option 3 open.
**Consequences:** One more component and language in the build. Terrain interaction is not compile-time checked.
**Evidence:** `TR-PROTO-01`, `INF-P4`.

### ADR-02: Terrain data injection

**Status:** Proposed - **blocked on a spike**
**Context:** `TR-TERRAIN-01`: Terrain3D's documented heightmap path is an editor importer expecting EXR or R16/RAW. No runtime data API is documented.
**Options:** programmatic region API if one exists in the source; synthesising the importer's expected format in memory; a custom mesh bypassing Terrain3D for terrain while keeping it for nothing.
**Decision:** Deferred pending the spike. `DEP-2` is "Pending" for this reason.
**Rationale:** Documentation is insufficient; the source must be read.
**Consequences:** If no runtime path exists, `DR-6` (adopt Terrain3D) must be revisited, which would move LOD back into scope as new development.
**Evidence:** `TR-TERRAIN-01` ([Terrain3D import docs](https://terrain3d.readthedocs.io/en/stable/docs/import_export.html), 2026-09-20).

### ADR-03: Tile atlas memory strategy

**Status:** Proposed
**Context:** The atlas is ~25 MB of DXT1 per map; decoding to RGBA multiplies it roughly six to eight times. `NFR-PERF.4` has no numeric ceiling pending this decision.
**Options:** hand pre-compressed DXT1 to Godot; decode to RGBA in Rust; stream tiles on demand.
**Decision:** **Compressed upload preferred**, RGBA fallback, streaming only if both fail.
**Rationale:** `TR-TEX-01`: `Image.FORMAT_DXT1` exists as format 17, and the documented restriction is only that VRAM-compressed images cannot be *modified* - which this product never does. Keeping the atlas compressed preserves GPU memory and avoids a decode pass.
**Consequences:** Rests on inference from adjacent documentation, not an explicit statement. Must be proven by spike before `NFR-PERF.4` can be given a ceiling.
**Evidence:** `TR-TEX-01` ([Godot Image class](https://docs.godotengine.org/en/stable/classes/class_image.html), 2026-09-20).

### ADR-04: Texture upload path

**Status:** Proposed
**Context:** Ten shader texture slots plus the atlas, all decoded by `mae-core`.
**Options:** `Image` + `ImageTexture`; `RenderingDevice` directly.
**Decision:** **`Image` + `ImageTexture`**, with `RenderingDevice` reserved for cases it cannot express.
**Rationale:** Higher-level, sufficient for static textures, and compatible with ADR-03. `RenderingDevice` would add renderer-backend coupling for no present gain.
**Consequences:** Some control over upload timing is given up.
**Evidence:** `TR-TEX-01`.

### ADR-05: Lua runtime and sandbox mechanism

**Status:** Proposed
**Context:** `NFR-SEC.1` requires denial of filesystem, network, process creation and unbounded execution over untrusted map Lua.
**Options:** reuse `rjm.bar.tools::luaenv` unchanged; extend it with resource limits; a different runtime.
**Decision:** **Reuse and extend.** Keep the existing VFS shim and global whitelist; add an instruction-count hook and a memory ceiling.
**Rationale:** `TR-SEC-01`: capability denial and resource exhaustion are separate axes, and the existing implementation addresses only the first. The whitelist approach is already correct.
**Consequences:** Map Lua that legitimately runs long may be terminated; the limit must be generous enough for real maps and is a tuning parameter, measured across the corpus.
**Evidence:** `TR-SEC-01`.

### ADR-06: Provenance representation

**Status:** Proposed
**Context:** `DATA-PROV.1` requires every displayed value to be derived with a citation, unknown, or absent. F8 in the FMEA rates an un-provenanced value at RPN 120.
**Options:** a generic `Provenanced<T>` wrapper; an enum per value type; a parallel metadata map keyed by value identity.
**Decision:** **Generic wrapper**, so that an un-provenanced value cannot be constructed.
**Rationale:** Makes F8 a compile error rather than a review item. A parallel map keeps signatures clean but allows the two to drift, which is exactly the failure mode.
**Consequences:** Pervasive signature impact across `mae-core`; unwrapping ergonomics matter and should be designed once, deliberately.
**Evidence:** F8; `TR-STORE-01`.

### ADR-07: Concurrency model

**Status:** Proposed
**Context:** `NFR-PERF.3` requires navigable terrain within 5 seconds and no interface block over 1 second, over archives up to ~50 MB.
**Options:** worker threads with deferred publication; async tasks; synchronous decode with yields.
**Decision:** **Worker threads inside `mae-core`, results marshalled to the main thread by deferred call.**
**Rationale:** `TR-COMPUTE-01`: gdext threading is experimental and warned as unsound; non-main-thread access to `Gd<T>` panics. Because `mae-core` has no Godot dependency, its threads cannot touch a Godot object even by mistake.
**Consequences:** A marshalling step on every completion; decode results must be plain data.
**Evidence:** `TR-COMPUTE-01`.

### ADR-08: Shader porting approach

**Status:** Proposed
**Context:** `NFR-FID.1` (P2, designated sacrifice) requires reproducing Recoil's SMF shading.
**Options:** port `SMFVertProg.glsl` and `SMFFragProg.glsl` to Godot's shading language; reimplement from the C++ render state; defer entirely.
**Decision:** **Port, starting from Terrain3D's generated shader as the scaffold.**
**Rationale:** `TR-TERRAIN-02`: enabling `shader_override` with a blank shader emits the generated shader, giving a working baseline to modify rather than a blank file. Porting preserves behaviour more reliably than reimplementation.
**Consequences:** Makes the project a derivative of GPL v2-or-later engine code - already accepted in `COMP-LICENSE.1`. Parity remains P2 and may be cut.
**Evidence:** `TR-TERRAIN-02`.

### ADR-09: Error taxonomy and failure policy

**Status:** Proposed
**Context:** `NFR-REL.1` requires zero crashes; F2 rates malformed input at RPN 108.
**Options:** typed error enum per stage; string errors; panic-free contract enforced by lint.
**Decision:** **Typed errors per decode stage, plus a panic-free contract** in `mae-core`, enforced by denying panicking operations in library code and fuzzing the decoders.
**Rationale:** The conformance run must report *which stage* failed for a given map, which string errors make awkward and panics make impossible.
**Consequences:** More error types to maintain; indexing must be written defensively throughout.
**Evidence:** F2, F6.

### ADR-10: Code sharing with `rjm.bar.tools`

**Status:** Proposed
**Context:** `DEP-1`. Both projects need archive access, SMF decoding, Lua evaluation and terrain derivation. This project additionally needs SMT, minimap, textures, S3O and provenance.
**Options:** extract a shared crate used by both; vendor the modules here; depend on `rjm.bar.tools` directly.
**Decision:** **Extract a shared crate**, owned by `rjm.bar.tools`, consumed here.
**Rationale:** Provenance (ADR-06) changes signatures pervasively; vendoring would fork that change and guarantee drift between two decoders that must agree. A shared crate keeps one decoder, which is also what makes P-TOOL's verification workflow meaningful.
**Consequences:** Cross-repository coordination; `rjm.bar.tools` must accept the provenance change or the crate must make it optional.
**Evidence:** `DEP-1`; persona P-TOOL.

### ADR-11: Decoded-map caching

**Status:** Proposed
**Context:** `UX-RELOAD.1` and persona P-AUT's compile-look-adjust loop make *reopen* cost dominant for the authoring persona.
**Options:** none; in-memory keyed by path and modification time; on-disk cache.
**Decision:** **In-memory only for P01.**
**Rationale:** An on-disk cache introduces an invalidation problem and a second source of truth for map data, against a persona whose importance is unvalidated (`RISK-6`). In-memory reuse covers reopening within a session, which is the common case.
**Consequences:** A cold restart re-decodes. Revisit once the early user confirms the workflow.
**Evidence:** `PR-UX-07`, `INF-P3`.

### ADR-12: Embedded constants generation

**Status:** Proposed
**Context:** `DATA-CONST.1` and `ASSUM-7`. F7 is the highest-RPN failure mode at 168, with poor detectability.
**Options:** build script reading `bar-Beyond-All-Reason` at compile time; a checked-in generated file carrying its source commit; runtime read from an installed game.
**Decision:** **Checked-in generated file with its source commit recorded and displayable**, produced by a script that is run deliberately.
**Rationale:** The runtime option is prohibited by `NFR-DEP.1`. A build script would make builds depend on a sibling checkout and would silently change values between builds - which is F7 happening quietly. A checked-in file makes every constant change a reviewable diff.
**Consequences:** Updating constants is a manual step that can be forgotten - which is exactly why the commit is displayed in the interface, turning a silent staleness into a visible one.
**Evidence:** F7, `ASSUM-7`, `NFR-DEP.1`.

### ADR-13: Packaging and distribution

**Status:** Proposed
**Context:** `NFR-PORT.1` (Windows and Linux), `NFR-LIC.1` (attribution), `COMP-LICENSE.1` (GPL v2-or-later, public).
**Options:** Godot export templates per platform with the extension bundled; separate CLI distribution; a single combined bundle.
**Decision:** **Two artifacts per platform** - the Godot application with `mae-gdext` bundled, and `mae-conform` as a standalone binary.
**Rationale:** `mae-conform` must run without Godot to prove `NFR-DEP.1` and to make the corpus run cheap in CI. Bundling it inside the application would defeat both.
**Consequences:** Two build outputs and two attribution surfaces.
**Evidence:** `TR-INFRA-01`, `NFR-DEP.1`.

## Reference key index

| Key | Area | Subject | Source | Date |
|:---|:---|:---|:---|:---|
| TR-STORE-01 | Storage | No database required; per-data-type placement | Readiness scalability checklist | 2026-09-20 |
| TR-COMPUTE-01 | Compute | gdext threading experimental, unsound; `call_deferred` is the route | [gdext docs](https://godot-rust.github.io/docs/gdext/master/godot/index.html), [issue #1131](https://github.com/godot-rust/gdext/issues/1131) | 2026-09-20 |
| TR-PROTO-01 | Protocol | No typed API for external GDExtension classes | [gdext issue #372](https://github.com/godot-rust/gdext/issues/372) | 2026-09-20 |
| TR-SEC-01 | Security | Whitelist sandboxing; resource limits are a separate axis | [lua-users SandBoxes](https://lua-users.org/wiki/SandBoxes), [rlua #38](https://github.com/mlua-rs/rlua/issues/38), [Luau sandbox](https://luau.org/sandbox/) | 2026-09-20 |
| TR-SEC-02 | Security | No other security surface exists | Readiness checklist; `DATA-*` privacy classifications | 2026-09-20 |
| TR-TEX-01 | Texture | `FORMAT_DXT1` exists; only modification of compressed images is prohibited | [Godot Image class](https://docs.godotengine.org/en/stable/classes/class_image.html) | 2026-09-20 |
| TR-TERRAIN-01 | Terrain | Documented import path is an editor importer | [Terrain3D import docs](https://terrain3d.readthedocs.io/en/stable/docs/import_export.html) | 2026-09-20 |
| TR-TERRAIN-02 | Terrain | Blank shader override emits the generated shader | [Terrain3DMaterial docs](https://terrain3d.readthedocs.io/en/stable/api/class_terrain3dmaterial.html) | 2026-09-20 |
| TR-INFRA-01 | Infrastructure | No deployment, no telemetry; conformance run is the CI gate | This analysis | 2026-09-20 |

## Data gaps

1. **Terrain3D runtime data injection** - undocumented; must be settled by
   reading the source. Blocks ADR-02 and `DEP-2`.
2. **Compressed-texture upload** - `FORMAT_DXT1` exists, but acceptance by
   `create_from_data` is inferred rather than documented. Blocks the numeric
   ceiling in `NFR-PERF.4`.
3. **Instruction-hook limits for real maps** - no data on how long legitimate
   map Lua runs. The sandbox limit must be calibrated against the corpus rather
   than guessed; ADR-05 says so.
4. **Shader slot correspondence** - which of the ten engine texture slots have
   no direct Godot equivalent has not been enumerated. Affects ADR-08 effort,
   which is P2.

## Quality gates

| Gate | Status |
| --- | --- |
| Every storage decision backed by a benchmark at projected scale | **Adapted** - no store is under consideration; each data type is placed against the checklist, with the reasoning stated rather than a benchmark cited for a database that does not exist |
| Compute model selected per component with rationale | Met - placement relative to the main thread, driven by `TR-COMPUTE-01` |
| Communication protocol justified per interface | Met - five in-process interfaces; no network protocols exist |
| Security architecture covers auth, authz, encryption, compliance | **Partially adapted** - no auth, authz or encryption surface exists (`TR-SEC-02`, stated rather than omitted); the one real boundary, untrusted Lua, is covered by `TR-SEC-01` and ADR-05 |
| FMEA identifies top failure modes with RPN and mitigations | Met - 8 modes; highest is F7 at 168 |
| ADR drafts exist for each decision point | Met - 13 of 13 |
| All research cites specific sources | Met - 9 reference keys, each with a URL or a named derivation |
