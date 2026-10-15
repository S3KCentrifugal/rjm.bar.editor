---
phase_id: P01
phase_name: Map Asset Exploration
phase_acronym: MAE
phase_directory: docs/requirements/phases/phase-01-map-asset-exploration/
stage: BRD readiness analysis
status: elicitation-complete
input_maturity: L2
authored: 2026-09-20T21:35:00Z
source: docs/requirements/intent/intent-brief.md
next: research-brd-context (phase P01)
---

# BRD Readiness Analysis: P01 - Map Asset Exploration (MAE)

## Phase under analysis

| Field | Value |
| --- | --- |
| Phase ID | `P01` |
| Name | Map Asset Exploration |
| Acronym | `MAE` |
| Directory | `docs/requirements/phases/phase-01-map-asset-exploration/` |
| Outcome | Any BAR map in the local library opens and can be explored in three dimensions, with every asset class represented, and with what is shown traceable to an engine rule rather than an approximation |
| Prerequisite phases | None |

This analysis covers **P01 only**. P02-P04 are registered but out of scope here.

## Input maturity: L2 - Defined Goals

The Intent Brief is a discovery-pipeline product and carries structured
audience, motivation, value proposition, constraint and boundary data. It
classifies **L2 rather than L3** for one reason: the phase outcome is stated as
a capability, not as measurable business objectives with baselines, targets and
time windows. Four of the ten gap areas are Partial; none is Missing. Expected
effort is **moderate** - targeted elicitation plus focused research, not full
discovery.

## Quick competitive scan

Conducted before the gap analysis, to calibrate what "complete" looks like here.

| Solution | What it is | Business model | Position |
| --- | --- | --- | --- |
| **SpringBoard** | The in-engine Spring/Recoil map and scenario editor; Core updated March 2026 | Free, community, GPL ecosystem | The incumbent. Runs inside the engine and inherits its limits: delta-heightmap-only saves, live-texture editing unsolved |
| **PyMapConv / springrts_smf_compiler** | The SMF compiler the official BAR guide recommends; also decompiles to loose images | Free, community | Actively maintained. One-way batch tooling, no viewer |
| **SpringMapEdit** | Community-described "only complete map making solution", and "in no way visual" | Free, community | Names the gap in its own users' words |
| **tebeer/BARMapEdit** | A **Unity-based** map editor for Beyond All Reason | None stated | **Newly identified.** 22 commits, 0 stars, 0 forks, no releases, no documented format support. Early-stage and unadopted - but direct evidence that the external-engine editor bet is being made independently |
| **hendkai/bar-map-generator** | Browser procedural generator; builds full map assets and packages them | Free, community | Notably, it **delegates SMF compilation to PyMapConv** rather than writing the format itself - relevant precedent for P02 |
| **Terrain3D** | Godot 4 GPU clipmap terrain, MIT | Free, MIT | Not a competitor; an adopted dependency |
| **Tiled** | General-purpose level editor, 15+ years | ~200 sponsors, roughly $3,600/month, funding two days a week | The sustainability benchmark for a *broadly useful* editor. A BAR-only tool draws from a far smaller pool |

**Market changes.** No adopted external BAR map viewer was found. BARMapEdit is
the only comparable attempt and has no traction. The BAR infrastructure side has
moved, though: `maps-metadata` is now the source of truth for map data, exported
from a Rowy instance, carrying playercount, startboxes and `maxPlayersPerStartbox`,
with approved changes distributed automatically within minutes; and
`beyond-all-reason/mapfeatures` holds features certified to work with Recoil.
Both are potential integration surfaces and neither was known at intent stage.

**Business models.** Every comparable tool in this domain is free and community
maintained. Nothing monetises. This confirms the refinement decision to treat
viability as commons rather than revenue, and it means the BRD's objectives must
be adoption- and capability-shaped, not commercial.

## Gap analysis

| # | Area | Assessment | Basis |
|:--|:---|:---|:---|
| 1 | Problem Statement | **Present** | Pain named (one-way chain; inspection requires launching the game), audience identified (the BAR mapper), impact described and partly quantified (two volunteer reviewers per map, a written checklist, 254 installed archives) |
| 2 | Business Objectives | **Partial** | Outcomes exist - all installed maps open; mappers reach for it - but are not expressed as measurable objectives with baseline, target and time window |
| 3 | Success Metrics | **Partial** | One binary metric is well formed (every archive opens). Missing: a measurable definition of "explore quickly", a completion test for parity, and any adoption measure |
| 4 | Stakeholders | **Partial** | Groups, persona and resistors are all identified, and the business owner is the user. But the early-user mapper is unnamed and unapproached, so the persona rests on published process documents rather than on a person |
| 5 | Constraints & Policies | **Present** | Technical constraints enumerated with evidence; licensing resolved (GPL v2-or-later, Terrain3D MIT compatible); resources and skills gap stated |
| 6 | Guardrails | **Partial** | Strong prohibited outcomes (six non-goals) and a priority stack that names the sacrifice. Missing: escalation triggers, and an operational definition of "does not guess" - what the tool actually does when a value cannot be derived |
| 7 | Competitive Context | **Present** | Seven comparable solutions identified with positioning and business models; differentiation clear; BARMapEdit added by this scan |
| 8 | Evidence Base | **Present** | Every material claim cites a source: the checklist and review guides, engine headers and shader paths, two archives inspected in full, corpus counts from `rjm.bar.tools` |
| 9 | Assumptions & Risks | **Partial** | Three risks named and knowingly accepted. Missing: an explicit assumption register with revalidation - notably that Terrain3D's shader override can carry parity, that a Rust extension can drive a C++ extension adequately, and that all 254 archives are decodable at all |
| 10 | Scope Boundaries | **Present** | Six non-goals with rationale, P01 included scope, entry and exit criteria, and a four-phase registry fixing what is deferred |

**Totals:** Present 6, Partial 4, Missing 0.

## Elicitation questions

Grouped by BRD section. Closed where partial information exists, per the L2
rule. Items the stakeholder cannot answer become assumptions with a
revalidation point, or Open Questions carried into the BRD.

### Business Objectives

1. Is "every one of the 254 installed archives opens, or fails with a documented
   and justified reason" the **primary** objective of P01, with all others
   secondary?
2. Is there a time window for P01, or is it explicitly untimed volunteer work?
3. Should adoption be an objective *of P01*, or only of the project - given the
   three-year success condition is mappers reaching for it, but P01's exit
   criteria mention only one early user?

### Success Metrics

4. What is the measurable form of "explore quickly"? Candidates: seconds from
   file selection to first navigable frame for a 32x32 map; sustained frames per
   second while moving; seconds to travel corner to corner. *(Open question 2 in
   the brief.)*
5. What is the completion test for visual parity, given it is ranked last and
   designated the sacrifice? Without a falsifiable definition it has no
   stopping point.
6. Is a per-map pass or fail across the corpus the reporting unit, and should it
   be produced automatically as a conformance run?

### Stakeholders

7. Who specifically is the early-user mapper, and is the approach through the
   BAR Discord `#mapping` channel? *(Open question 8.)*
8. Should the BAR maps team be treated as a stakeholder for P01, now that
   `maps-metadata` and `mapfeatures` are identified as live infrastructure?

### Constraints & Policies

9. Is a minimum Godot version pinned - Terrain3D requires Godot 4.4+ - and is
   Windows the only target platform for P01, or must Linux be supported?
10. Is the `gdext` version pinned at a specific release, with upgrades treated
    as deliberate work items?

### Guardrails

11. Operationally, when a value cannot be derived exactly, does the tool: refuse
    to open the map, open it and mark the value unknown in the interface, or log
    it and continue silently? "It does not guess" needs one of these.
12. What is the escalation trigger if a map in the corpus cannot be opened at
    all - is it a defect that blocks the phase, or a documented exclusion like
    `mescaline_v2`?

### Assumptions & Risks

13. Should the three technical assumptions - Terrain3D's override can carry
    parity, Rust can drive Terrain3D adequately, and all archives are decodable -
    be validated by a spike before the technical design, or accepted as risks?

## Elicitation responses

Four blocking questions were put to the stakeholder on 2026-09-20 and answered.

| # | Question | Answer | Consequence |
| --- | --- | --- | --- |
| 4 | Measurable form of "explore quickly" | **Corner-to-corner traversal time** under normal camera control | Tests navigation design rather than load or render speed. A numeric threshold must be set in the product requirements, and a standard traversal procedure defined so the measure is repeatable |
| 5 | Completion test for visual parity | **Side-by-side screenshot comparison against captures from a pinned engine commit** | Parity becomes falsifiable and versioned, resolving the PF-01 unbounded-target risk. Requires a reference capture set, recorded camera positions, and an engine commit in `verified_against`. Judgement remains human, so the test is repeatable but not automatic |
| 11 | Operational meaning of "does not guess" | **Open the map and mark the value unknown in the interface** | The honesty guarantee becomes a visible interface behaviour, not a silent log. Requires an unknown-value representation in the data model and in the UI. Compatible with the ranked-first commitment that every map opens |
| 12 | A map that cannot be opened | **Hard blocker - every map must open, no exceptions** | No exclusion list. See the scope consequence below |

### Scope consequence of the no-exceptions decision

The corpus was re-counted during this analysis and contains **254 `.sd7`
archives** plus one unpacked map directory - the earlier figure of 510 was the
directory entry count, which includes one `.md5.gz` per archive. The unpacked
directory (`supreme_isthmus_v1.8/`) is direct evidence that folder input is a
real-world requirement rather than a hypothetical one.

Within those 254, `mescaline_v2.sd7` carries a legacy **`.smd` (TDF) map
definition instead of `mapinfo.lua`**, alongside a normal `.smf`, `.smt`,
feature set and models. `rjm.bar.tools` skips this archive today. The engine
still supports the format as a fallback (`rts/Map/MapParser.cpp:30`), so
honouring it is engine-backed rather than a workaround.

**Therefore the no-exceptions decision adds a legacy TDF/`.smd` parser to P01
scope.** This is recorded as a consequence of the stakeholder's decision, not as
an objection to it; the Intent Brief's P01 included scope and exit criteria have
been updated to match.

## Research plan

### Input maturity: L2

### Required research

1. **Customer / user research** - *Priority: **High***
   The persona is inferred from published process documents, not from a mapper.
   Needed: what reviewers and authors actually do when inspecting a map, what
   they check first, and what would make an external tool worth adopting. Fills
   gap areas 3, 4 and the adoption objective. Deliverable: `MR-USER` reference
   entries grounded in community sources, plus the named early user if
   identifiable.

2. **Requirements mining from the BAR Map Checklist** - *Priority: **High***
   The checklist is the closest thing to a pre-written requirements source this
   project has. Needed: the full text captured verbatim and mapped
   property-by-property to what an editor must display to make each verifiable.
   Fills gap areas 1, 2, 3. Deliverable: a checklist-to-capability matrix.

3. **Corpus characterisation** - *Priority: **High***
   P01's exit criterion is that every installed archive opens, so the shape of
   the corpus is a requirement input, not a test detail. Needed: how many of the
   254 carry unusual shapes - missing texture slots, option-rewritten height
   ranges, multiple SMT files, pre-`mapinfo.lua` archives such as
   `mescaline_v2`. Source: `rjm.bar.tools` over the local library. Fills gap
   areas 3, 9.

4. **Competitor analysis, depth pass** - *Priority: **Medium***
   Mostly complete. Remaining: what BARMapEdit actually implements, and whether
   any in-community tool surfaced only on Discord or the forums performs
   external inspection. Fills gap area 7.

5. **Audience sizing** - *Priority: **Medium***
   The commercial TAM question does not apply; the equivalent is how many people
   author or review BAR maps, and at what rate maps are submitted. This bounds
   what adoption could mean. Fills gap areas 2, 3.

6. **Integration-surface research** - *Priority: **Medium***
   `maps-metadata` schemas and the `mapfeatures` certified set were unknown at
   intent stage. Needed: what they record and whether the editor should read or
   align with them. Fills gap areas 4, 9.

7. **Regulatory and licensing** - *Priority: **Low***
   Effectively resolved - GPL v2-or-later accepted, Terrain3D MIT compatible,
   Godot MIT. Needs confirmation only, plus the rule on not redistributing
   third-party map content.

### Elicitation required

The thirteen questions above. The **four blocking ones - 4, 5, 11 and 12 -
are now answered** (see Elicitation responses). The remaining nine carry into
the BRD as open questions or as assumptions with a revalidation point; none of
them blocks drafting.

### Items already sufficient

- **Problem Statement** - named, evidenced, quantified.
- **Constraints & Policies** - technical and licensing constraints are
  enumerated with citations; no regulatory regime applies beyond licence.
- **Competitive Context** - seven solutions with positioning and models.
- **Evidence Base** - every material claim already carries a source.
- **Scope Boundaries** - six non-goals, phase registry, entry and exit criteria.

## Quality gates

| Gate | Status |
| --- | --- |
| Input maturity classified | Met - L2, with justification |
| All 10 gap areas assessed | Met - 6 Present, 4 Partial, 0 Missing |
| Elicitation questions generated for Missing/Partial areas | Met - 13 questions across 6 BRD sections |
| Research plan specifies areas to investigate | Met - 7 areas with priority and gap mapping |
| Quick competitive scan conducted | Met - 7 solutions; one new entrant found (BARMapEdit) and two live BAR infrastructure repositories |
| Stakeholder elicitation complete | Met - four blocking questions answered; the remaining nine carry into the BRD as open questions or assumptions |
| Report identifies and remains within one registered phase | Met - P01 only |
