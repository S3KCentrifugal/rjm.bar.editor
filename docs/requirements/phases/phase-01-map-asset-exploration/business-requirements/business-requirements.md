---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
document: Business Requirements Document
status: Approved - Baselined
scale: Major
authored: 2026-09-20
baselined: 2026-09-20
next: analyze-prd-readiness (phase P01)
---

# Business Requirements Document: P01 - Map Asset Exploration (MAE)

## 1. Document Control & Context

### 1.1 Control

| Field | Value |
|:---|:---|
| **Document status** | **Approved - Baselined 2026-09-20.** All ten open questions resolved by the stakeholder; validation conditions C2 and C3 closed. See [`validation-report.md`](validation-report.md). All subsequent changes go through the Change Log |
| **Owner** | Project owner (`rjm.bar.editor`), sole accountable business owner and maintainer |
| **Phase ID** | `P01` (immutable) |
| **Phase name** | Map Asset Exploration |
| **Phase acronym** | `MAE` |
| **Phase directory** | `docs/requirements/phases/phase-01-map-asset-exploration/` |
| **Prerequisite phases** | None |
| **Document scale** | **Major** - new product, new capability, multi-quarter, full 8 sections |
| **Decision date** | 2026-09-20 |
| **Review window** | Revalidate on completion of P01, or on any change to the phase registry |

### 1.2 Phase registry entry (copied from the Intent Brief, immutable)

- **Outcome.** Any BAR map in the local library opens in the editor and can be
  explored in three dimensions, with every asset class the archive contains
  represented, and with what is shown traceable to an engine rule rather than an
  approximation.
- **Included scope.** Reading `.sd7` archives and unpacked directories through
  one virtual file system; decoding SMF (header, heightmap, typemap, metalmap,
  minimap, feature block, vegetation extra-header) and the `.smt` tile atlas;
  evaluating `mapinfo.lua`, `mapoptions.lua` and overlays in a sandboxed Lua 5.1
  environment with a VFS shim; parsing the legacy `.smd` (TDF) definition;
  presenting every asset class including shading and override textures, sky,
  water, S3O feature models and textures, feature placements, start positions
  and map-side Lua as inspectable source; three-dimensional terrain with level
  of detail sufficient for a 32x32 map; camera and navigation components;
  visual parity with Recoil's SMF shading chain as the ranked-last commitment;
  and conformance verification across the installed corpus.
- **Entry criteria.** Intent Brief approved; P01 business, product and technical
  requirements complete and validated; licence decision recorded in the
  repository.
- **Exit criteria.** See `BIZ-EXP.1`, `BIZ-EXP.2`, `BIZ-EXP.3`, `GUARD-ACC.1`
  and `BIZ-SAT.1`, which restate them as measurable requirements.

### 1.3 Source bundle

All evidence is in
[`research-bundle.md`](research-bundle.md), gathered 2026-09-20. Reference key
namespaces: `MR-COMP` (competitors), `MR-CHK` (Map Checklist requirements
mining), `MR-CORP` (original corpus measurement), `MR-TREND`, `MR-REG`,
`MR-USER`, `MR-SIZE`. Upstream: [`../../../intent/intent-brief.md`](../../../intent/intent-brief.md),
[`readiness-analysis.md`](readiness-analysis.md).

### 1.4 Glossary

| Term | Definition |
|:---|:---|
| **Archive** | A `.sd7` file - a 7-Zip container holding one BAR map |
| **Unpacked map** | A directory holding the same contents unarchived, the form an author has mid-edit. One is present in the reference corpus (`supreme_isthmus_v1.8/`) |
| **Corpus** | The reference population: **254 `.sd7` archives plus 1 unpacked directory** in the local BAR installation, measured 2026-09-20 (`MR-CORP-01`) |
| **Asset class** | One of the eleven categories enumerated in the Intent Brief: compiled terrain, tile atlas, map definition, shading textures, override grids, sky, water, map objects, object placement, map-side Lua, start positions |
| **Derived value** | A value computed by reproducing a documented engine rule, with that rule cited |
| **Unknown value** | A value that cannot be derived exactly from map data and is therefore displayed as unknown rather than estimated |
| **Visual parity** | Indistinguishability, by human side-by-side comparison at matching camera positions, between this editor and Beyond All Reason running at a recorded engine commit |
| **Corner-to-corner traversal time** | Elapsed seconds to move the camera from one corner of a map to the diagonally opposite corner under normal camera control, by a defined procedure. The agreed measure of exploration speed |
| **Checklist property** | One of the twenty reviewer-verifiable map properties published in the BAR Map Checklist (`MR-CHK-01` to `MR-CHK-20`) |
| **Reference corpus run** | An automated pass that opens every corpus member and records pass or fail per map |
| **COMP-LICENSE** | New section code: licensing obligations, in place of a statutory regime |
| **COMP-CONTENT** | New section code: third-party content handling obligations |
| **GUARD-ACC** | New section code: accuracy and honesty guardrails |
| **GUARD-SCOPE** | New section code: scope-boundary guardrails |
| **justified-by** | New relationship type: requirement A exists *because* requirement B was set, but B does not have to be satisfied first. Distinct from `depends-on`, which implies ordering and can form cycles |

### 1.5 Change log

| Version | Date | Change | Author |
|:---|:---|:---|:---|
| 0.1 | 2026-09-20 | Initial draft | Project owner |
| 0.2 | 2026-09-20 | Validation fixes: broke the `BIZ-EXP.1` / `OPER-MIGRATE.1` dependency cycle (C1); narrowed `GUARD-SCOPE.2` to map files and declared its tension with `OPER-REPORT.1` and `MARK-PARITY.1` (C4). See `validation-report.md` | This review |
| **1.0** | 2026-09-20 | **Baselined.** All ten open questions resolved (`DR-7` to `DR-16`); validation conditions C2 and C3 closed. `BIZ-EXP.2` retargeted against in-game traversal; `BIZ-EFF.1` raised to all twenty checklist properties; six requirements added (`OPER-WORKFLOW.2`, `.3`, `.4`, `OPER-REPORT.2`, `GUARD-REPUTE.2`); `GUARD-SCOPE.3` and `.4` reclassified from project anti-goals to P01 scope boundaries; `ASSUM-7` added; `RISK-1` and `RISK-6` ratings raised | Project owner |

---

## 2. Executive Summary & Scope

### 2.1 Problem statement

Beyond All Reason's map toolchain runs one way. Authors produce images, a
compiler bakes them into a binary `.smf`/`.smt` pair, and the only routine way
to see the result is to launch the game. Every tool the official guides
recommend - World Machine, Photoshop or GIMP, Beherith's SMF compiler,
SpringBoard, PyMapConv, a pathability-checker script, 7-Zip - authors, converts,
compiles or packs. None of them shows a finished map (`MR-USER-02`,
`MR-COMP-01` to `MR-COMP-05`).

The cost falls hardest on review. Every map entering the game requires **at
least two volunteer reviewers** to approve it against a published checklist of
twenty properties, from pathability by movement class to sun angle and feature
reclaim values, with no tool that surfaces those properties together
(`MR-USER-01`, `MR-USER-03`, `MR-CHK-01` to `MR-CHK-20`). The one incumbent
editor runs inside the engine and inherits its limits: Spring saves only a delta
heightmap rather than a full map, and texture editing against live GL textures
is documented as unsolved (`MR-COMP-01`). Derived terrain analysis - the slope,
passability and metal-spot data that downstream tooling computes - has no visual
verification path at all.

No adopted external viewer exists. The single attempt found has 22 commits, zero
stars and zero forks (`MR-COMP-04`).

### 2.2 Business objectives summary

P01 shall deliver a standalone application that opens any map in the reference
corpus and lets a mapper explore it in three dimensions, showing every asset
class the map contains, with every displayed value either derived from a cited
engine rule or explicitly marked unknown.

### 2.3 Non-goals

Out of scope for P01, each carrying a guardrail in Section 5c:

1. **Editing of any kind.** P01 does not modify, write or export map data.
2. **Map generation.** *(P01 scope boundary, not a permanent project anti-goal - `DR-12`.)*
3. **Scenario, trigger, unit or gadget authoring.** *(P01 scope boundary, not a permanent project anti-goal - `DR-12`.)*
4. **Generality beyond Recoil.** Not a generic terrain editor, not a Godot
   terrain plugin, not multi-engine.
5. **Any dependency on a running engine.** The editor never launches Recoil and
   never requires a game installation to read a map.
6. **Reading external BAR repositories.** P01 does not consult `maps-metadata` or `mapfeatures`; it reads map files only (`DR-16`).
7. **Commercial exploitation.** No revenue model; every comparable tool in this
   domain is free and community maintained (`MR-COMP-01` to `MR-COMP-05`,
   `MR-COMP-07`).

### 2.4 Baseline and target metrics

| Metric | Baseline (2026-09-20) | Target at P01 exit |
|:---|:---|:---|
| Corpus members openable in an external tool | **0 of 255** | **255 of 255** (254 archives + 1 unpacked directory) |
| Asset classes represented outside the game | 0 of 11 | 11 of 11, for maps that contain them |
| Checklist properties visually verifiable without launching the game | 0 of 20 | **20 of 20** (`MR-CHK`, `DR-10`) |
| Corner-to-corner traversal time | Not achievable - no external tool exists | Equal to or faster than the same traversal in Beyond All Reason (`DR-7`) |
| Displayed values that are silent approximations | Not applicable | **0** |
| Platforms supported | 0 | Windows and Linux (`DR-13`) |
| Mappers who have used the tool on their own map | 0 | At least 1, with feedback recorded, after a runnable build exists (`DR-14`) |

---

## 3. Stakeholders & Personas

> **Role:** Project owner
> **Type:** Business Owner
> **Context:** Sole developer and maintainer. Holds the assets P01 depends on - an exact BAR archive decoder (`rjm.bar.tools`), a provenance-graded game knowledge base (`rjm.bar.docs`), and fluency in the Recoil source. Volunteer time, no budget.
> **Desired Outcome:** A tool that shows any BAR map faithfully, and a foundation on which layer and terrain editing can later be built.
> **Influence:** Decision Maker

> **Role:** BAR map reviewer
> **Type:** End User
> **Context:** Experienced mapmaker on the BAR team. Reviews submissions posted to the `#mapping` Discord channel; at least two reviewers must approve each map, with a single named tie-breaker. Verifies twenty published checklist properties as unpaid volunteer work. Currently must launch the game to inspect anything. Guidance asks reviewers to be efficient, avoid moving goalposts, and point at documented standards (`MR-USER-01`).
> **Desired Outcome:** To inspect a submitted map directly and answer checklist questions without starting BAR.
> **Influence:** Consulted

> **Role:** BAR map author
> **Type:** End User
> **Context:** Builds maps in World Machine or a paint tool, compiles with Beherith's SMF compiler, places objects in SpringBoard, packs with 7-Zip (`MR-USER-02`). Works with unpacked files mid-edit. Publicly credited on the BAR maps directory (`MR-USER-04`).
> **Desired Outcome:** To see their compiled map without launching the game, closing the one-way authoring loop.
> **Influence:** Consulted

> **Role:** Early-user mapper
> **Type:** End User
> **Context:** A single author or reviewer to be recruited before P01 exit. The persona above is reconstructed entirely from published process documents; no primary user research exists (Data Gap 1, `research-bundle.md`).
> **Desired Outcome:** To fly a map they authored themselves and judge instantly whether it looks right.
> **Influence:** Consulted - and the only validation of the persona

> **Role:** Tooling and AI developer
> **Type:** Operator
> **Context:** Maintains `rjm.bar.tools`, `rjm.bar.ai` and `s3k-CircuitAI`, which derive slope, passability, metal spots and start-position classification from the same map data, with no way to check those derivations visually.
> **Desired Outcome:** Visual confirmation of derived terrain facts against the source map.
> **Influence:** Informed

> **Role:** Recoil engine maintainers
> **Type:** External Partner
> **Context:** Own the SMF shader chain that is this project's parity target, and the GPL v2-or-later licence that governs its reuse (`MR-REG-01`).
> **Desired Outcome:** No misrepresentation of engine behaviour by a third-party renderer.
> **Influence:** Informed

---

## 4. Business Objectives & Outcomes

> **ID:** `BIZ-EXP.1`
> **Priority:** `P0`
> **Source / Evidence:** `MR-CORP-01` (corpus measured at 254 archives plus 1 unpacked directory); stakeholder decision 2026-09-20, `DR-4`
> **Requirement:** The editor shall open every member of the reference corpus without error. No exclusion list shall be maintained.
> **Success Metric:** Count of corpus members opened successfully in a reference corpus run.
> **Baseline / Target:** 0 of 255 → **255 of 255**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `depends-on: [OPER-WORKFLOW.1, OPER-MIGRATE.1, OPER-SCALE.1]`

> **ID:** `BIZ-EXP.2`
> **Priority:** `P0`
> **Source / Evidence:** Intent Brief priority stack rank 3; stakeholder decision 2026-09-20, `DR-1`
> **Requirement:** The editor shall permit a user to traverse a map corner to corner, under normal camera control, in no more time than the same traversal takes in Beyond All Reason.
> **Success Metric:** Corner-to-corner traversal time on each reference-set map, compared against the same traversal performed in Beyond All Reason at the pinned engine commit.
> **Baseline / Target:** No external tool exists → **equal to or faster than the in-game reference time**, on every reference-set map, at P01 exit. (`DR-7`)
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.1, OPER-REPORT.2]`

> **ID:** `BIZ-EXP.3`
> **Priority:** `P0`
> **Source / Evidence:** Intent Brief priority stack rank 2; asset enumeration from two archives inspected in full; `MR-CORP-06` to `MR-CORP-09`
> **Requirement:** The editor shall represent every asset class a map contains, and shall render correctly when an optional asset class is absent.
> **Success Metric:** Asset classes represented, and corpus run pass rate for maps lacking optional slots.
> **Baseline / Target:** 0 of 11 → **11 of 11**; and 100% pass for the 5 maps lacking a normal map, 9 lacking specular, 4 lacking splat, 166 lacking sky and 60 lacking features.
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.1]`

> **ID:** `BIZ-EFF.1`
> **Priority:** `P1`
> **Source / Evidence:** `MR-CHK-01` to `MR-CHK-20`; `MR-USER-01`; inference `INF-02`
> **Requirement:** The editor shall make **all twenty** published Map Checklist properties verifiable without launching Beyond All Reason, presenting them together in one view with a pass or fail result wherever the checklist states a numeric rule.
> **Success Metric:** Count of the twenty checklist properties answerable using the editor alone, assessed against the property-to-capability matrix in the Source Bundle.
> **Baseline / Target:** 0 of 20 → **20 of 20**, at P01 exit. (`DR-10`)
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.3, OPER-WORKFLOW.4]`; `constrained-by: [GUARD-REPUTE.2]`
>
> **Note on the two externally-sourced properties.** Pathability by movement class (`MR-CHK-02`) and geovent buildability (`MR-CHK-13`) require movement-definition and unit data that a map archive does not contain. These shall be satisfied by constants embedded in the editor with a cited source and a recorded Beyond All Reason commit, **not** by reading an installed game - preserving `GUARD-SCOPE.5`. See `ASSUM-7`.

> **ID:** `BIZ-SAT.1`
> **Priority:** `P1`
> **Source / Evidence:** Intent Brief, refinement cycle 1 and 2 decisions; Data Gap 1 in the Source Bundle
> **Requirement:** At least one BAR mapper who is not the project owner shall use the editor on a map they authored, and their feedback shall be recorded before P01 is closed.
> **Success Metric:** Recorded feedback from a named early user.
> **Baseline / Target:** 0 mappers → **at least 1**, before P01 exit.
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.1, BIZ-EXP.3]`; validates the persona underlying `BIZ-EFF.1`

> **ID:** `BIZ-EXP.4`
> **Priority:** `P1`
> **Source / Evidence:** `MR-CORP-01`; presence of `supreme_isthmus_v1.8/` as an unpacked directory in the corpus
> **Requirement:** The editor shall open a map from an unpacked directory as readily as from an archive, so that an author can inspect work in progress.
> **Success Metric:** The unpacked corpus member opens, and any archive extracted to a directory opens equivalently.
> **Baseline / Target:** Not possible → **supported**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `depends-on: [OPER-WORKFLOW.1]`

> **ID:** `MARK-DIFF.1`
> **Priority:** `P0`
> **Source / Evidence:** `MR-COMP-01` (SpringBoard's engine-inherited limits), `MR-COMP-04` (no adopted external editor)
> **Requirement:** The editor shall operate as a standalone application that requires neither a running Recoil engine nor an installed copy of Beyond All Reason in order to open a map.
> **Success Metric:** A corpus map opens on a machine with no BAR installation, given the map file alone.
> **Baseline / Target:** Every existing inspection path requires the game → **independent operation**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `conflicts-with: [MARK-PARITY.1]` - see resolution note below

> **ID:** `MARK-PARITY.1`
> **Priority:** `P2`
> **Source / Evidence:** Intent Brief priority stack rank 4 (designated sacrifice); stakeholder decision 2026-09-20, `DR-2`; `MR-REG-01`
> **Requirement:** The editor shall reproduce the visual output of Recoil's SMF shading chain such that a human comparing side-by-side captures at matching camera positions cannot distinguish them.
> **Success Metric:** Side-by-side screenshot comparison against captures taken from Beyond All Reason at a recorded engine commit, over the reference map set established by `OPER-REPORT.2`.
> **Baseline / Target:** No comparison exists → **indistinguishable on the reference map set**, at an engine commit recorded in document front matter. (`DR-2`)
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.3]`; `conflicts-with: [MARK-DIFF.1, GUARD-SCOPE.2]`
>
> **Conflict resolution.** Parity is defined against a *pinned engine commit*,
> not against a live engine. Producing the reference captures requires the game
> once, as a development activity; operating the editor never does. `MARK-DIFF.1`
> governs the product, `MARK-PARITY.1` governs its verification. Where the two
> cannot both be satisfied, the Intent Brief priority stack applies and
> `MARK-PARITY.1` yields - it is `P2` precisely because it is the designated
> sacrifice.

> **ID:** `OPER-WORKFLOW.1`
> **Priority:** `P0`
> **Source / Evidence:** `MR-CORP-01`; Intent Brief P01 included scope
> **Requirement:** The editor shall accept both `.sd7` archives and unpacked map directories as input through a single access path, so that behaviour does not differ by input form.
> **Success Metric:** Identical presentation of the same map supplied in both forms.
> **Baseline / Target:** Not applicable → **both forms supported identically**, at P01 exit.
> **Owner:** Project owner

> **ID:** `OPER-MIGRATE.1`
> **Priority:** `P1`
> **Source / Evidence:** `MR-CORP-04`, `MR-CORP-05` (3 archives carry `.smd`; `mescaline_v2` depends on it alone); `rts/Map/MapParser.cpp:30`
> **Requirement:** The editor shall read the legacy `.smd` (TDF) map definition where `mapinfo.lua` is absent, as the engine does.
> **Success Metric:** `mescaline_v2.sd7` opens with its definition values correctly resolved.
> **Baseline / Target:** Unsupported by existing tooling → **supported**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `justified-by: [BIZ-EXP.1]` - this requirement exists only because `BIZ-EXP.1` admits no exclusions. Deliberately **not** `depends-on`, which would create a cycle with `BIZ-EXP.1`.

> **ID:** `OPER-SCALE.1`
> **Priority:** `P1`
> **Source / Evidence:** `MR-CORP-06`, `MR-CORP-07`, `MR-CORP-08`, `MR-CORP-11`
> **Requirement:** The editor shall handle the full observed variation of the corpus, including archives from 10 to 436 files and maps lacking any optional asset.
> **Success Metric:** Corpus run pass rate across the full range.
> **Baseline / Target:** Not applicable → **100% of corpus**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.1]`

> **ID:** `OPER-REPORT.1`
> **Priority:** `P1`
> **Source / Evidence:** `BIZ-EXP.1` requires a measurable corpus result; stakeholder elicitation question 6
> **Requirement:** The project shall produce a reference corpus run that records a pass or fail result for every corpus member, repeatably.
> **Success Metric:** A run artifact listing every corpus member with its result.
> **Baseline / Target:** No such report exists → **produced and repeatable**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** Provides the evidence for `BIZ-EXP.1`, `BIZ-EXP.3`, `OPER-SCALE.1`; `conflicts-with: [GUARD-SCOPE.2]` - resolved there, the run artifact is not map data

> **ID:** `OPER-WORKFLOW.2`
> **Priority:** `P1`
> **Source / Evidence:** `MR-CORP-10` (94.9% of archives ship `mapoptions.lua`); `rjm.bar.tools` finding that some options rewrite the height range; `DR-11`
> **Requirement:** The editor shall let the user select map options and shall re-evaluate the map definition against the selected set, so that option-dependent values can be seen as a player would see them.
> **Success Metric:** For a map whose options alter the height range, changing the option changes the displayed terrain accordingly.
> **Baseline / Target:** No tool exposes map options → **selectable and re-evaluated**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `depends-on: [BIZ-EXP.3]`

> **ID:** `OPER-WORKFLOW.3`
> **Priority:** `P1`
> **Source / Evidence:** `OQ-7` resolution; `DR-13`; `DEP-2` (Terrain3D requires Godot 4.4+)
> **Requirement:** The editor shall run on Windows and on Linux.
> **Success Metric:** A reference corpus run completes on both platforms.
> **Baseline / Target:** No platform support → **both platforms**, at P01 exit.
> **Owner:** Project owner

> **ID:** `OPER-WORKFLOW.4`
> **Priority:** `P1`
> **Source / Evidence:** `MR-CHK-01` to `MR-CHK-20`; `MR-USER-01` (two volunteer reviewers per map); `DR-10`
> **Requirement:** The editor shall present the twenty Map Checklist properties together in a single view, showing each property's value for the open map, and a pass or fail only where the checklist states a numeric rule.
> **Success Metric:** All twenty properties appear in one view; pass or fail shown for the properties with published numeric rules (map size, wind range, tidal range, splat resolution, normals resolution).
> **Baseline / Target:** No such view exists anywhere → **present**, at P01 exit.
> **Owner:** Project owner
> **Relationships:** `constrained-by: [GUARD-REPUTE.2]`; supports `BIZ-EFF.1`

> **ID:** `OPER-REPORT.2`
> **Priority:** `P1`
> **Source / Evidence:** Required by `BIZ-EXP.2` and `MARK-PARITY.1`; `DR-7`, `DR-2`; composition variation from `MR-CORP-05` to `MR-CORP-11`
> **Requirement:** The project shall establish a fixed reference map set of six to ten maps spanning the composition variation the corpus exhibits, and shall record for each, from Beyond All Reason at a pinned engine commit, both visual captures at defined camera positions and a corner-to-corner traversal time.
> **Success Metric:** Reference set defined, captures and timings recorded, engine commit noted in `verified_against`.
> **Baseline / Target:** No reference material exists → **established**, before any parity or traversal claim.
> **Owner:** Project owner
> **Relationships:** Required by `BIZ-EXP.2`, `MARK-PARITY.1`
>
> **Coverage rule.** The set shall include at minimum: a map with a skybox and one without; a map with features and one without; a map missing a normal, specular or splat texture; the legacy `.smd` map (`mescaline_v2`); an option-sensitive map whose options alter the height range; and the largest and smallest archives by file count.

---

## 5. Constraints, Policies & Guardrails

### 5a. Regulatory & Compliance

> **ID:** `COMP-LICENSE.1`
> **Priority:** `P0`
> **Source / Evidence:** `MR-REG-01`; stakeholder decision 2026-09-20, `DR-3`
> **Requirement:** The project shall be licensed under GPL v2-or-later and published publicly, in consequence of deriving from Recoil's GPL-licensed shader chain.
> **Success Metric:** Licence file present and repository public before P01 development begins.
> **Baseline / Target:** No licence recorded → **recorded**, at P01 entry.
> **Owner:** Project owner

> **ID:** `COMP-LICENSE.2`
> **Priority:** `P0`
> **Source / Evidence:** `MR-REG-02`
> **Requirement:** The project shall satisfy the attribution obligations of its MIT-licensed dependencies, including Terrain3D and Godot.
> **Success Metric:** Attribution present in distributed builds and repository.
> **Baseline / Target:** None → **complete**, at P01 exit.
> **Owner:** Project owner

> **ID:** `COMP-CONTENT.1`
> **Priority:** `P0`
> **Source / Evidence:** `MR-REG-03`
> **Requirement:** The project shall not redistribute third-party map content. Map archives shall be read from the user's own installation or supplied by the user.
> **Success Metric:** No map content in the repository or in any distributed build.
> **Baseline / Target:** None → **none**, maintained throughout.
> **Owner:** Project owner

### 5b. Operational Business Rules

Recorded as `OPER-` requirements in Section 4: `OPER-WORKFLOW.1`,
`OPER-MIGRATE.1`, `OPER-SCALE.1`, `OPER-REPORT.1`.

### 5c. Guardrails

> **ID:** `GUARD-ACC.1`
> **Priority:** `P0`
> **Source / Evidence:** Intent Brief non-goal 2; stakeholder decision 2026-09-20, `DR-5`; `MR-USER-01` (reviewers act on what they see)
> **Prohibited Outcome:** The editor **shall not** display an approximated, estimated or guessed value as though it were derived. A value that cannot be derived exactly from map data shall be marked unknown in the interface, and the map shall still open.
> **Detection Method:** Every displayed value is traceable in the codebase to a cited engine rule or to an explicit unknown state; reviewed at P01 exit against the displayed-value inventory.
> **Escalation Path:** Any value found displayed without a derivation or an unknown marker is a P0 defect blocking P01 closure.

> **ID:** `GUARD-REPUTE.1`
> **Priority:** `P1`
> **Source / Evidence:** Intent Brief resistor analysis - a parity claim that silently diverges is worse than no claim
> **Prohibited Outcome:** The project **shall not** claim visual parity with Recoil except against a recorded engine commit for which reference captures exist.
> **Detection Method:** Any public parity claim is checked against the recorded commit and reference set.
> **Escalation Path:** Claim withdrawn or qualified; `verified_against` updated.

> **ID:** `GUARD-REPUTE.2`
> **Priority:** `P1`
> **Source / Evidence:** `OPER-WORKFLOW.4` introduces pass/fail reporting; `MR-USER-01` (approval is a human decision by two reviewers)
> **Prohibited Outcome:** The editor **shall not** present its checklist output as an approval decision, nor show a pass or fail for any property the checklist does not govern by a numeric rule. For all other properties it shall report the value and leave the judgement to the reviewer.
> **Detection Method:** Review of the checklist view against the published checklist: every pass/fail indicator must map to a stated numeric rule.
> **Escalation Path:** Indicator removed and replaced with a reported value.

> **ID:** `GUARD-SCOPE.1`
> **Priority:** `P0`
> **Source / Evidence:** Intent Brief non-goal 1, stakeholder's own words - "it's a purpose built tool for Recoil"
> **Prohibited Outcome:** The editor **shall not** be generalised to other engines, other map formats, or general-purpose terrain editing.
> **Detection Method:** Requirement review at each phase gate.
> **Escalation Path:** Requirement rejected as out of scope; recorded in the change log.

> **ID:** `GUARD-SCOPE.2`
> **Priority:** `P0`
> **Source / Evidence:** Intent Brief non-goal 6; phase registry places editing in P03 and P04
> **Prohibited Outcome:** P01 **shall not** modify, write or export **map data**. The editor shall not open any map file - archive, `.smf`, `.smt`, `.smd`, `mapinfo.lua` or any other archive member - for writing.
> **Detection Method:** No write path to map files exists in the P01 codebase.
> **Escalation Path:** P0 defect blocking P01 closure.
> **Relationships:** `conflicts-with: [OPER-REPORT.1, MARK-PARITY.1]`
>
> **Conflict resolution.** This guardrail governs **map files only**. The boundary is the file being written, not the act of writing. Permitted non-map writes, confirmed by `DR-8`, are: corpus run reports (`OPER-REPORT.1`), parity reference captures (`OPER-REPORT.2`, `MARK-PARITY.1`), and session state such as window layout and last-opened map. No other writing is permitted in P01.

> **ID:** `GUARD-SCOPE.3`
> **Priority:** `P1`
> **Source / Evidence:** Phase registry - editing is scoped to P03 and P04; `DR-12`. **Scope boundary for P01, not a project-level anti-goal** - the stakeholder declined to confirm it as permanent.
> **Prohibited Outcome:** **P01** shall not provide scenario, trigger, unit or gadget authoring. Map-side Lua is presented as inspectable source only (`DR-9`).
> **Detection Method:** Requirement review at each phase gate.
> **Escalation Path:** Requirement rejected as SpringBoard's ground.

> **ID:** `GUARD-SCOPE.4`
> **Priority:** `P1`
> **Source / Evidence:** Phase registry - no phase includes generation; `DR-12`. **Scope boundary for P01, not a project-level anti-goal** - the stakeholder declined to confirm it as permanent.
> **Prohibited Outcome:** **P01** shall not generate maps procedurally or otherwise.
> **Detection Method:** Requirement review at each phase gate.
> **Escalation Path:** Requirement rejected; `MR-COMP-05` already serves that need.

> **ID:** `GUARD-SCOPE.5`
> **Priority:** `P0`
> **Source / Evidence:** Intent Brief non-goal 5; `MARK-DIFF.1`
> **Prohibited Outcome:** The editor **shall not** require a running Recoil engine or an installed copy of Beyond All Reason in order to open a map.
> **Detection Method:** Corpus run executed on a machine with no BAR installation.
> **Escalation Path:** P0 defect blocking P01 closure.

### 5d. Budget & Timeline

**Budget: zero.** One developer, volunteer time, no funding sought. Every
comparable tool in this domain is free and community maintained; the only
monetised comparable, Tiled, funds two days a week from roughly $3,600/month of
donations while serving a far broader market (`MR-COMP-07`). No revenue
assumption is made.

**Timeline: none set.** P01 is explicitly untimed volunteer work. The phase is
bounded by its exit criteria, not by a date. The Intent Brief records that the
work proceeds in strict dependency order - decoder complete and correct before
rendering - with the resulting absence of early visible progress knowingly
accepted. See `OQ-3`.

### 5e. AI Governance

**Not applicable.** P01 involves no artificial intelligence, machine learning,
or automated decision-making affecting users. The editor decodes and displays
file data deterministically.

---

## 6. Market & Competitive Context

> **Competitor:** SpringBoard - the in-engine Spring/Recoil map and scenario editor
> **Source:** `MR-COMP-01`
> **Relevance:** Informs `MARK-DIFF.1` and `GUARD-SCOPE.5`.
> **Implication:** **Opportunity.** SpringBoard runs inside the engine and inherits its limits - Spring saves only a delta heightmap rather than a full map, and texture editing against live GL textures is documented as unsolved. An external reader owns the file and inherits neither constraint. It is also actively maintained, so this is differentiation, not replacement, reinforced by `GUARD-SCOPE.3`.

> **Competitor:** PyMapConv and Beherith's SMF compiler
> **Source:** `MR-COMP-02`
> **Relevance:** Informs the problem statement and `BIZ-EFF.1`.
> **Implication:** **Parity requirement.** These prove the format is fully round-trippable, which P02 depends on. They provide no interactive inspection, which is the gap P01 fills.

> **Competitor:** tebeer/BARMapEdit - a Unity-based BAR map editor
> **Source:** `MR-COMP-04`
> **Relevance:** Informs `MARK-DIFF.1` and `RISK-5`.
> **Implication:** **Threat, currently negligible.** 22 commits, zero stars, zero forks, no releases, no documented scope. It demonstrates that the external-game-engine editor approach is being reached for independently, which validates the direction while showing that building it is not sufficient for adoption.

> **Competitor:** hendkai/bar-map-generator
> **Source:** `MR-COMP-05`
> **Relevance:** Informs `GUARD-SCOPE.4` and, later, P02.
> **Implication:** **Boundary marker.** It generates rather than inspects, and notably delegates SMF compilation to PyMapConv rather than writing the format itself - a precedent worth weighing when P02 designs the write path.

> **Trend:** BAR map infrastructure has centralised on `maps-metadata`, exported from a Rowy instance and distributed automatically within minutes; `mapfeatures` holds features certified for Recoil
> **Source:** `MR-TREND-02`, `MR-TREND-03`
> **Relevance:** Informs `OQ-4`.
> **Implication:** **Opportunity, unscoped for P01.** A defined integration surface that did not exist at intent stage. Not required for the P01 outcome, so it is recorded as an open question rather than a requirement.

> **Trend:** Godot does not load DDS at runtime; DDS is converted to CTEX at import
> **Source:** `MR-TREND-05`
> **Relevance:** Constrains `BIZ-EXP.3`.
> **Implication:** **Constraint, already accommodated.** Map textures arrive from arbitrary archives at runtime, so Godot's import pipeline is unusable and texture decoding must be owned by the project. This strengthens rather than undermines the decision to build the decoder as a native extension.

> **Trend:** BAR commercialising - Hooded Horse deal signed June 2026, Steam release pending
> **Source:** `MR-TREND-01`
> **Relevance:** Informs `BIZ-SAT.1` and the project-level adoption goal.
> **Implication:** **Opportunity.** A larger and less expert author population raises the value of inspection tooling over the project horizon.

---

## 7. Assumptions, Dependencies & Risks

### Assumptions

> **ID:** `ASSUM-1`
> **Assumption:** The 254-archive local library is representative of BAR maps generally.
> **Impact if Wrong:** `BIZ-EXP.1` would certify against an unrepresentative population; maps outside it could fail.
> **Revalidation:** Re-measure against the official maps directory (~130+ live maps, `MR-SIZE-01`) before P01 exit, and re-run the corpus scan on 2027-03-20.

> **ID:** `ASSUM-2`
> **Assumption:** Terrain3D's `shader_override_enabled` can carry a full Recoil-parity shader, despite its material model assuming 32 splatted texture sets rather than a tile-indexed atlas.
> **Impact if Wrong:** `MARK-PARITY.1` would require a custom clipmap renderer, materially enlarging the weakest-skill portion of the work.
> **Revalidation:** Technical spike before the technical design is baselined; no later than P01 design completion.

> **ID:** `ASSUM-3`
> **Assumption:** A Rust GDExtension can drive Terrain3D's C++ GDExtension classes adequately through dynamic access.
> **Impact if Wrong:** Either Terrain3D is dropped, or a language boundary is introduced.
> **Revalidation:** Same spike as `ASSUM-2`.

> **ID:** `ASSUM-4`
> **Assumption:** Every corpus archive is decodable in principle - no archive is corrupt or uses an undocumented variant beyond `.smd`.
> **Impact if Wrong:** `BIZ-EXP.1`'s no-exceptions target becomes unachievable and the stakeholder decision `DR-4` must be revisited.
> **Revalidation:** First full corpus decode attempt; immediately on completion of the reader.

> **ID:** `ASSUM-5`
> **Assumption:** The reviewer persona, reconstructed from published process documents, reflects what reviewers actually need.
> **Impact if Wrong:** `BIZ-EFF.1`'s checklist coverage target may address the wrong properties.
> **Revalidation:** `BIZ-SAT.1` - early-user recruitment, which by `DR-14` happens **once a runnable build exists**, not during design. The persona therefore remains unvalidated throughout requirements and design; this is the single largest evidential weakness in this document (Data Gap 1), knowingly accepted.

> **ID:** `ASSUM-6`
> **Assumption:** Exactly one `.smf` and one `.smt` per map, as measured across all 254 archives (`MR-CORP-02`, `MR-CORP-03`).
> **Impact if Wrong:** A multi-SMT map would fail to open, breaching `BIZ-EXP.1`.
> **Revalidation:** The tile-file count field shall be parsed correctly even though only the single-file case is exercised; re-measure on corpus refresh.

> **ID:** `ASSUM-7`
> **Assumption:** The Beyond All Reason movement-definition and geothermal constants embedded to satisfy `MR-CHK-02` and `MR-CHK-13` remain valid for the BAR version a user's maps target.
> **Impact if Wrong:** Pathability or geovent results shown to a reviewer would be wrong while appearing derived, breaching `GUARD-ACC.1` - the most damaging failure mode this document contains.
> **Revalidation:** Constants carry the BAR commit they were read from; re-checked against `bar-Beyond-All-Reason` at each P01 milestone and on any BAR release. First check no later than P01 exit.

### Dependencies

> **ID:** `DEP-1`
> **Dependency:** `rjm.bar.tools` - the existing Rust decoder for `.sd7`, SMF grids and sandboxed `mapinfo.lua` evaluation.
> **Required By:** `BIZ-EXP.1`, `BIZ-EXP.3`, `OPER-WORKFLOW.1`
> **Status:** Confirmed - exists, under the same owner
> **Expected Resolution:** Available now; requires extension for `.smt` and minimap decoding.

> **ID:** `DEP-2`
> **Dependency:** Terrain3D (MIT), requiring Godot 4.4 or later.
> **Required By:** `BIZ-EXP.2`, `BIZ-EXP.3`, `MARK-PARITY.1`
> **Status:** Pending - adopted in principle, unvalidated
> **Expected Resolution:** `ASSUM-2` and `ASSUM-3` spike, before technical design baseline.

> **ID:** `DEP-3`
> **Dependency:** `godot-rust/gdext`, which ships occasional breaking changes.
> **Required By:** All rendering requirements
> **Status:** Confirmed, with a maintenance cost
> **Expected Resolution:** Version pinned at technical design; upgrades treated as deliberate work items.

> **ID:** `DEP-4`
> **Dependency:** A recruited early-user mapper from the BAR community.
> **Required By:** `BIZ-SAT.1`, and revalidation of `ASSUM-5`
> **Status:** At Risk - deliberately deferred until a runnable build exists (`DR-14`)
> **Expected Resolution:** After the first build that can open and fly a map; before P01 exit.

> **ID:** `DEP-5`
> **Dependency:** Reference captures from Beyond All Reason at a pinned engine commit.
> **Required By:** `MARK-PARITY.1`
> **Status:** Pending
> **Expected Resolution:** Before any parity claim is made.

### Risks

> **ID:** `RISK-1`
> **Risk:** Scope expansion prevents P01 from ever closing. Parity has no natural stopping point, and the framework assessment flagged very high orderliness aimed at an unbounded target as the project's central personality-level risk.
> **Probability:** Medium | **Impact:** High
> **Affected Requirements:** `MARK-PARITY.1`, and P01 as a whole
> **Mitigation:** The Intent Brief priority stack ranks parity last and designates it the first sacrifice; `MARK-PARITY.1` is `P2`; `DR-2` gives parity a falsifiable completion test for the first time. **Residual risk increased at baseline:** `DR-15` accepted an option picker, a checklist view, all twenty checklist properties and dual-platform support into P01 rather than deferring any of them. The floor protecting closure is now the three P0 exploration requirements alone.
> **Owner:** Project owner

> **ID:** `RISK-2`
> **Risk:** The no-exceptions corpus target is breached by a single malformed or undocumented archive, blocking phase closure on one file.
> **Probability:** Medium | **Impact:** Medium
> **Affected Requirements:** `BIZ-EXP.1`, `OPER-MIGRATE.1`
> **Mitigation:** `ASSUM-4` is revalidated at first full decode, early enough to revisit `DR-4` before it blocks closure.
> **Owner:** Project owner

> **ID:** `RISK-3`
> **Risk:** The parity target moves. Tracking Recoil's shaders couples the project to a live upstream.
> **Probability:** High | **Impact:** Low
> **Affected Requirements:** `MARK-PARITY.1`, `GUARD-REPUTE.1`
> **Mitigation:** Parity is defined against a pinned engine commit recorded in `verified_against`, so divergence is visible and dated rather than silent.
> **Owner:** Project owner

> **ID:** `RISK-4`
> **Risk:** Sole-maintainer discontinuity. The framework research identified single-maintainer abandonment, made terminal by licensing or documentation that blocks continuation, as a top failure pattern for comparable tools.
> **Probability:** Low | **Impact:** High
> **Affected Requirements:** All
> **Mitigation:** `COMP-LICENSE.1` - GPL v2-or-later and public from the start, so continuation by others is legally and practically possible.
> **Owner:** Project owner

> **ID:** `RISK-5`
> **Risk:** The tool is built and not adopted. `MR-COMP-04` shows a comparable effort with zero adoption, and no adoption-path evidence was found (Data Gap 4).
> **Probability:** Medium | **Impact:** Medium for P01, High for the project
> **Affected Requirements:** `BIZ-SAT.1`
> **Mitigation:** Early-user recruitment during P01 rather than after it; adoption itself is a project-level goal, not a P01 exit criterion.
> **Owner:** Project owner

> **ID:** `RISK-6`
> **Risk:** The persona is wrong. Every reviewer- and author-facing requirement rests on published process documents rather than on any conversation with a mapper.
> **Probability:** **Medium-High** (raised at baseline) | **Impact:** **High** (raised at baseline)
> **Affected Requirements:** `BIZ-EFF.1`, `BIZ-SAT.1`, `OPER-WORKFLOW.4`
> **Mitigation:** `ASSUM-5` revalidation through `DEP-4`. **Both ratings rose at baseline**: `DR-10` committed to all twenty checklist properties and `DR-15` accepted a dedicated checklist view into P01, so more of the phase now rests on the unvalidated persona; and `DR-14` deferred recruitment until a build exists, so the correction arrives after design rather than before it. This is the clearest accepted risk in the document.
> **Owner:** Project owner

---

## 8. Open Questions & Decisions

### Open questions

**All open questions were resolved by the stakeholder on 2026-09-20.** None
remains outstanding; the document is therefore eligible for baseline. Each
resolution is recorded as a Decision Record below.

| ID | Question | Status | Resolution | Decision |
|:---|:---|:---|:---|:---|
| `OQ-1` | Traversal threshold and procedure | **Resolved** | Match or beat the same traversal performed in Beyond All Reason at the pinned engine commit | `DR-7` |
| `OQ-2` | What non-map writing is permitted | **Resolved** | Corpus run reports, parity reference captures, and session state. Nothing else | `DR-8` |
| `OQ-3` | Map-option handling at load | **Resolved** | Expose an option picker and re-evaluate the map definition | `DR-11` |
| `OQ-4` | Use of `maps-metadata` and `mapfeatures` | **Resolved - deferred** | P01 reads map files only; no external repository dependency | `DR-16` |
| `OQ-5` | Early-user recruitment channel and timing | **Resolved** | Approach a mapper once a runnable build exists, not during design | `DR-14` |
| `OQ-6` | `LuaGaia` representation | **Resolved** | Inspectable source only; no execution, no effect simulation | `DR-9` |
| `OQ-7` | Platform coverage | **Resolved** | Windows and Linux | `DR-13` |
| `OQ-8` | Which checklist properties are committed | **Resolved** | All twenty, with the two externally-sourced ones satisfied by embedded cited constants | `DR-10` |
| `A-1` | How the parity reference map set is chosen | **Resolved** | A fixed set of six to ten maps spanning the corpus composition variation | `DR-7` |
| `Q-7` | Confirmation of the three derived anti-goals | **Resolved** | Only "it does not guess" confirmed as a project anti-goal; the other two retained as P01 scope boundaries | `DR-12` |

**P0 approval check: PASSES.** No `P0` requirement depends on an unresolved
Open Question. The two conditions raised in validation are closed: `OQ-1` now
gives `BIZ-EXP.2` a measurable, externally anchored target (C2), and `OQ-3` and
`OQ-6` now fix the meaning of `BIZ-EXP.3` (C3).

### Decision records

> **ID:** `DR-1`
> **Decision:** Exploration speed shall be measured as corner-to-corner traversal time under normal camera control.
> **Alternatives Rejected:** Time to first navigable frame - measures load, not navigation. Sustained frame rate - measures rendering, not usability. Both together - rejected as more than needed.
> **Rationale:** Camera and navigation were named by the stakeholder as a component requirement; traversal time tests navigation design, which is what was asked for.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-2`
> **Decision:** Visual parity shall be tested by side-by-side screenshot comparison against captures from a pinned engine commit.
> **Alternatives Rejected:** Shader-input equivalence - testable but does not prove the picture matches. Automated image diff - brittle across drivers and expensive to build. Subjective judgement by the early user - not repeatable.
> **Rationale:** Gives the ranked-last, unbounded commitment a falsifiable completion test and a versioned reference point, directly addressing `RISK-1`.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-3`
> **Decision:** The project shall be GPL v2-or-later and public from the start.
> **Alternatives Rejected:** Deferring the licence until parity work begins - risks deciding under pressure. Avoiding shader porting to keep licence freedom - weakens parity, which is the product's distinguishing claim. Private repository - forecloses adoption and continuity.
> **Rationale:** Porting Recoil's shaders makes the work a derivative; the ecosystem has been GPL since 2005; and public licensing mitigates `RISK-4`.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-4`
> **Decision:** Every corpus map shall open. No exclusion list.
> **Alternatives Rejected:** Documented exclusions with justification - the approach `rjm.bar.tools` already takes with `mescaline_v2`. A failure budget of a small number of maps.
> **Rationale:** Stakeholder decision. Consequence accepted and recorded: this adds legacy `.smd` (TDF) parsing to P01 scope (`OPER-MIGRATE.1`), because `mescaline_v2` depends on it and the engine still supports the format.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-5`
> **Decision:** Where a value cannot be derived exactly, the map shall still open and the value shall be marked unknown in the interface.
> **Alternatives Rejected:** Refusing to open the map - conflicts directly with `BIZ-EXP.1`. Logging silently without surfacing - weakens the guarantee for the reviewer audience, who act on what they see. Classifying values by importance - adds a classification with no evidence to ground it.
> **Rationale:** Makes the honesty principle a visible interface behaviour rather than an internal convention, and keeps it compatible with the ranked-first commitment.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-6`
> **Decision:** Terrain3D shall be adopted for terrain mesh and level of detail.
> **Alternatives Rejected:** A plain heightmap mesh deferring the decision. A custom clipmap renderer - highest cost, squarely in the weakest skill area.
> **Rationale:** Proven GPU clipmap covering 64 m to 65.5 km with 10 LOD levels; MIT and GPL-compatible; `shader_override_enabled` preserves the parity option. Carries `ASSUM-2` and `ASSUM-3`.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-7`
> **Decision:** Exploration speed shall be measured against Beyond All Reason itself - the editor must equal or beat the same corner-to-corner traversal performed in game - over a fixed reference map set of six to ten maps spanning the corpus composition variation.
> **Alternatives Rejected:** A fixed threshold of 10, 20 or 30 seconds - any number would have been arbitrary. Maps chosen by the early user - maximises their judgement but gives poor edge-case coverage. One map per size class - blind to the asset-composition variation that actually breaks readers.
> **Rationale:** Self-calibrating, needs no invented figure, and ties the speed claim to the same pinned-commit discipline as the parity claim. Resolves validation condition C2.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-8`
> **Decision:** P01 may write corpus run reports, parity reference captures and session state. It may not write map data of any kind.
> **Alternatives Rejected:** Reports and captures only, with no session state - stricter than needed, and the tool would forget its layout every launch. User-initiated export of on-screen content - a new product surface not required by any business outcome.
> **Rationale:** Keeps the no-editing guarantee exact while letting the requirements that need artifacts function. The boundary is the file written, not the act of writing.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-9`
> **Decision:** Map-side Lua shall be presented as inspectable source only.
> **Alternatives Rejected:** Source plus a static summary of declared gadgets and placement calls - more useful, but carries a risk of misreading that `GUARD-ACC.1` would not tolerate. Executing gadgets to show runtime effect - contradicts the no-engine-dependency guardrail and heads toward scenario emulation.
> **Rationale:** Gadget behaviour is runtime state, not map data. Showing source satisfies the asset class without the editor asserting anything it cannot derive.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-10`
> **Decision:** `BIZ-EFF.1` shall commit to all twenty Map Checklist properties, presented together in one view, with pass or fail shown only where the checklist states a numeric rule.
> **Alternatives Rejected:** The eighteen derivable from map data alone - would have kept `GUARD-SCOPE.5` trivially clean but left reviewers still launching the game for pathability. The twelve that are pure data display. Dropping the numeric target entirely. Populating the view without any pass or fail.
> **Rationale:** The checklist is the externally validated need; partial coverage leaves the reviewer workflow unchanged. The two externally-sourced properties are satisfied by embedded constants carrying a cited BAR commit, so no game installation is required. Creates `ASSUM-7` and raises `RISK-6`.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-11`
> **Decision:** P01 shall expose a map-option picker and re-evaluate the map definition against the selection.
> **Alternatives Rejected:** Engine defaults with option-sensitive maps flagged - what `rjm.bar.tools` does today, cheaper and disclosure-honest, but leaves the user unable to see what an option actually changes. Defaults now, picker in a later phase.
> **Rationale:** 94.9% of archives ship options and some rewrite the height range itself; a viewer that cannot show those states shows only one of several truths.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-12`
> **Decision:** Of the three anti-goals derived during intent exploration, only **"it does not guess"** is confirmed as a project-level anti-goal. "Not a scenario or gadget editor" and "not a map generator" are retained as **P01 scope boundaries**, not permanent project positions.
> **Alternatives Rejected:** Confirming all three as permanent. Dropping both unconfirmed ones entirely. Keeping the scenario boundary while opening generation.
> **Rationale:** The stakeholder confirmed one of three when offered all three. Treating silence as assent would itself have been a guess. The phase registry already excludes both from P01, so nothing in this phase changes; what changes is that a future phase is not foreclosed.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-13`
> **Decision:** P01 shall support Windows and Linux.
> **Alternatives Rejected:** Windows only - matches the development machine and the map library location, but narrows adoption in an ecosystem with substantial Linux presence. Unrestricted export tested only on Windows - honest but leaves the Linux claim unverified.
> **Rationale:** Both Godot and Rust cross-compile; the cost is a second conformance run rather than a second implementation.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-14`
> **Decision:** The early-user mapper shall be approached once a runnable build exists, not during requirements or design.
> **Alternatives Rejected:** Posting in the `#mapping` Discord thread now. Approaching a credited author directly now. Two contacts, one before design and one at P01 exit.
> **Rationale:** Stakeholder decision, made with the consequence stated: the persona stays unvalidated through the period when correcting it would be cheapest. `RISK-6` raised to Medium-High probability and High impact in consequence, and `ASSUM-5` revalidation moved accordingly.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-15`
> **Decision:** The scope added by these resolutions - option picker, checklist view, all twenty properties, dual-platform - is accepted into P01 rather than deferred or split into a further phase.
> **Alternatives Rejected:** Accepting the additions but ranking them below parity so they too become sacrificial. Registering reviewer tooling as a fifth phase. Trimming back by reverting the option picker.
> **Rationale:** Each addition serves the externally validated reviewer need. Recorded against `RISK-1`: the floor that protects phase closure is now the three P0 exploration requirements alone, with everything above them able to yield.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

> **ID:** `DR-16`
> **Decision:** P01 shall not read `maps-metadata` or `mapfeatures`.
> **Alternatives Rejected:** Reading `mapfeatures` to flag uncertified features. Reading both and cross-checking a map's start positions against BAR shipped startboxes.
> **Rationale:** Neither is needed for a P01 outcome, and both would add an external repository dependency to a tool whose distinguishing claim is that it reads map files and nothing else. Revisit when editing arrives and validation matters.
> **Date:** 2026-09-20 | **Decision Maker:** Project owner

---

## Requirements index

| ID | Category | Priority | Title |
|:---|:---|:---|:---|
| `BIZ-EXP.1` | Business | P0 | Every corpus member opens |
| `BIZ-EXP.2` | Business | P0 | Corner-to-corner traversal within threshold |
| `BIZ-EXP.3` | Business | P0 | Every asset class represented |
| `BIZ-EXP.4` | Business | P1 | Unpacked directories open as readily as archives |
| `BIZ-EFF.1` | Business | P1 | Checklist properties verifiable without the game |
| `BIZ-SAT.1` | Business | P1 | Early-user validation by a mapper |
| `MARK-DIFF.1` | Market | P0 | Standalone - no engine or game required |
| `MARK-PARITY.1` | Market | P2 | Visual parity against a pinned engine commit |
| `OPER-WORKFLOW.1` | Operational | P0 | Archives and directories through one access path |
| `OPER-WORKFLOW.2` | Operational | P1 | Map-option picker with re-evaluation |
| `OPER-WORKFLOW.3` | Operational | P1 | Windows and Linux support |
| `OPER-WORKFLOW.4` | Operational | P1 | All twenty checklist properties in one view |
| `OPER-MIGRATE.1` | Operational | P1 | Legacy `.smd` definition supported |
| `OPER-SCALE.1` | Operational | P1 | Full observed corpus variation handled |
| `OPER-REPORT.1` | Operational | P1 | Repeatable reference corpus run |
| `OPER-REPORT.2` | Operational | P1 | Reference map set with in-game captures and timings |
| `COMP-LICENSE.1` | Compliance | P0 | GPL v2-or-later, public |
| `COMP-LICENSE.2` | Compliance | P0 | MIT dependency attribution |
| `COMP-CONTENT.1` | Compliance | P0 | No redistribution of map content |
| `GUARD-ACC.1` | Guardrail | P0 | No approximated value shown as derived |
| `GUARD-REPUTE.1` | Guardrail | P1 | No unverified parity claim |
| `GUARD-REPUTE.2` | Guardrail | P1 | Checklist output is not an approval decision |
| `GUARD-SCOPE.1` | Guardrail | P0 | Not generalised beyond Recoil |
| `GUARD-SCOPE.2` | Guardrail | P0 | P01 writes no map data |
| `GUARD-SCOPE.3` | Guardrail | P1 | No scenario or gadget authoring *(P01 boundary)* |
| `GUARD-SCOPE.4` | Guardrail | P1 | No map generation *(P01 boundary)* |
| `GUARD-SCOPE.5` | Guardrail | P0 | No engine or game dependency |

**27 requirements** - 12 P0, 14 P1, 1 P2. Within the 15-40 range for a Major BRD.

Five were added at baseline by the open-question resolutions: `OPER-WORKFLOW.2`
(option picker, `DR-11`), `OPER-WORKFLOW.3` (dual platform, `DR-13`),
`OPER-WORKFLOW.4` (checklist view, `DR-10`), `OPER-REPORT.2` (reference map
set, `DR-7`), and `GUARD-REPUTE.2` (no implied approval, `DR-10`). None is P0:
the phase-closure floor remains the three P0 exploration requirements, as
`DR-15` records.

## Quality gates

| Gate | Status |
| --- | --- |
| All required sections present | Met - all 8, with 5e marked N/A and justified |
| Every requirement has ID, priority, source/evidence, owner | Met - 27 of 27 |
| "Shall" language used consistently | Met |
| Evidence citations reference the Source Bundle | Met - every requirement cites an `MR-` key, a decision record, or a named engine source |
| Inter-requirement relationships declared | Met - including one declared conflict (`MARK-DIFF.1` / `MARK-PARITY.1`) with a written resolution |
| AI governance included if applicable | Met - N/A, with justification |
| Open Questions capture all unresolved items | Met - all 10 resolved at baseline; P0 approval check passes |
| Scope and output path match the phase registry entry | Met - P01 only |
