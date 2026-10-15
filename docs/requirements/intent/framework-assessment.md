---
stage: 0.4 (assess)
status: assessed
confidence: inference (framework application from behavioural evidence) / research (comparable ventures)
assessed: 2026-09-20T20:35:00Z
source: docs/requirements/intent/intent-record.md (stages 0.1-0.3)
next: refine-intent
---

# Framework Assessment Report: BAR Map Editor (rjm.bar.editor)

> These frameworks are lenses, not verdicts. Nothing here invalidates the
> intent; the assessment reveals the pattern that gives it coherence and names
> where that pattern is thin. Personality and motivation inferences are drawn
> from observable artifacts and stated choices, not from any instrument. The
> Big Five was developed in W.E.I.R.D. contexts and the four-circle Ikigai is a
> Western adaptation of a broader Japanese idea; both are used here with that
> caveat.

## Evidence base

Framework inference draws on artifacts in the workspace rather than
self-report, which makes it unusually well grounded for this stage:

| Artifact | What it evidences |
| --- | --- |
| `rjm.bar.docs` | A layered knowledge base with per-document front-matter, four-tier confidence grading (`data`/`derived`/`community`/`theory`), generators, a link checker, and a published gap analysis |
| `rjm.bar.tools` | A decoder that reproduces engine rules with `file:line` citations, guarantees byte-identical reruns, and documents what it **deliberately does not derive** |
| `s3k-CircuitAI` | Canonical agent instructions with a full Repository Map, eleven vendor routers, repo skills, and a script that keeps role docs in sync with role code |
| `rjm.bar.ai` | A first-principles design that explicitly refused prior-art inheritance |
| This project | Specification demanded before any code, unprompted |

## Frameworks applied

Seven of eight, plus Value Dimensions.

| Framework | Why |
| --- | --- |
| PF-01 Big Five | Strong behavioural signal across five repositories; matrix lists it for craft and community intents |
| PF-03 SDT | Always recommended; the user describes personal motivation ("editing is the real goal") |
| PF-04 Ikigai | Always applied |
| PF-05 Logotherapy | User describes personal motivation; a distinctly-theirs assignment is visible |
| PF-06 Eudaimonia | Ongoing craft with a long horizon |
| PF-07 Flow | The intent involves sustained solo craft; skill claims are explicit |
| PF-08 VIA | Skills and capabilities are describable from artifacts |
| Value Dimensions | Always applied |

**PF-02 Maslow excluded.** The matrix triggers it for suffering-origin or
scattered intents. This intent is neither: the motivation is friction and
craft, not adversity, and the record is unusually well-ordered.

## Framework results

### PF-01: Big Five (OCEAN)

| Trait | Signal | Evidence |
| --- | --- | --- |
| **Openness** | High | Five parallel projects across Rust, C++, AngelScript, Lua, now Godot; first-principles AI design; willingness to discard prior art |
| **Conscientiousness** | Very high, orderliness-weighted | Provenance front-matter, confidence tiers, determinism guarantees, generated-file discipline, doc-code sync checkers, spec-before-code |
| **Extraversion** | Low to moderate | Solo work across every repository; no collaboration artifacts; audiences are served at arm's length, through a tool |
| **Agreeableness** | Moderate, tough-minded | Serving volunteer reviewers is a service orientation; "don't reference the old AI" and "purpose built" are competitive and boundaried |
| **Neuroticism** | Low | Calm, sequential decision-making; no risk-aversion language; large scope accepted without visible anxiety |

**Work-pattern fit.** High Openness plus very high Conscientiousness is close to
an ideal profile for a long, exacting format-reverse-engineering project. Both
the novelty appetite and the discipline to finish are evidenced.

**Misalignment flagged for discussion, not prohibition.** Very high
orderliness combined with an explicitly unbounded quality target ("visual
parity with Recoil") is the classic amplifier of scope creep. The trait that
guarantees the work will be *right* is the same trait that resists declaring it
*done*. Parity has no natural stopping point unless one is defined externally.
This is the personality-level restatement of tension 1 in the intent record,
and it argues that the phase cut must be written down, not held as intention.

### PF-03: Self-Determination Theory

**Motivation classification: Integrated regulation, shading into Intrinsic.**

Not "I need to make money" (external), not "I'd feel like a failure otherwise"
(introjected). The evidence points to *this is who I am*: five repositories in
the same domain, each built to a standard well beyond what its immediate use
required, sustained without external demand. The move from "inspecting maps is
painful" to building a reference renderer is the signature of integrated
motivation — the value is congruent with self-concept, not instrumental.

| Need | Fulfilment | Evidence |
| --- | --- | --- |
| **Autonomy** | Very high | "Purpose built tool for Recoil"; first-principles design; rejection of inherited architecture; solo, self-directed, self-specified |
| **Competence** | Very high | Mastery of a niche binary format is the substance of the work; the checklist gives an external standard to be measurably good against |
| **Relatedness** | Moderate, and the weakest leg | Four audiences named, two of them communal (reviewers, authors) — but relatedness is mediated entirely by an artifact; no evidence of engaging that community directly |

**Sustainability signal: strong.** Intrinsic and integrated motivation predicts
persistence without external reward, which matters because the research found no
revenue path in this niche. The corroborating evidence is domain-specific and
positive: PyMapConv and SpringBoard are both still maintained and documented,
inside an ecosystem that has run on community energy since Spring's GPL release
in 2005. Niche RTS tooling here is sustained by people who care, not by budgets.

**Watch item.** Relatedness is the need least served. If the reviewer and author
audiences stay hypothetical, this reverts to a tool of one — which is a
legitimate outcome, but it should be chosen rather than defaulted into.

### PF-04: Ikigai

| Dimension | Score | Basis |
| --- | --- | --- |
| **Love** | **Strong** | Unprompted, sustained, multi-year-shaped investment in this exact domain; depth well beyond task requirements; "editing is the real goal" states a long horizon |
| **Skill** | **Strong, with a located weak spot** | Strong: binary format decoding, Rust, sandboxed Lua, systems architecture, technical documentation, engine-source reading. Weak or unknown: Godot 4, `gdext`, GLSL-to-Godot shader porting, DXT1 decoding, large-terrain LOD |
| **Need** | **Strong — externally validated** | Every map needs two volunteer reviewers to approve it against a *written* checklist; the entire recommended toolchain authors, converts or packs; no external viewer exists; 254 archives installed locally; BAR commercialising into a larger author population |
| **Viability** | **Moderate — and probably the wrong question** | No revenue intent stated. Benchmark: Tiled, a famous general-purpose editor serving many engines, sustains ~200 sponsors at roughly $3,600/month, funding two days a week. A BAR-only tool draws from a far smaller pool. But the value exchange here is adoption and reciprocal utility, not money |

**Intersections.** Love + Skill = **Passion**, present and strong. Love + Need =
**Mission**, present and evidence-backed. The profile is *Mission with a Passion
core* — three strong legs. The fourth is undefined rather than absent.

**On Viability.** Ikigai's "paid for" leg translates badly to community
infrastructure. The honest reframing: viability here is whether the effort is
**personally sustainable** and whether the tool earns **adoption**. On the first,
SDT says yes. On the second, the answer depends on choices not yet made —
distribution, licensing, whether reviewers ever see it. Scored Moderate because
the path exists but is unchosen, not because it is blocked.

**The one substantive skill finding.** The fidelity bar was set at visual parity
with Recoil — which lands precisely in the weakest quadrant of an otherwise very
strong skill profile. Everything the user has demonstrated mastery of (exact
decoding, provenance, determinism, documentation) is *upstream* of the renderer;
everything the parity goal requires (GLSL porting, LOD, real-time rendering) is
new. This is not an argument against parity. It is an argument that parity is
the part to schedule last, learn deliberately, and phase — and that the decoder
work, where the user is strongest, is what should reach "done" first.

### PF-05: Logotherapy

**Primary pathway: Creative** — meaning through building a work and doing a deed.
Unambiguous: the intent is to make a thing that did not exist.

**The unique assignment.** Frankl's question is what makes this task
irreducibly *this person's*. The answer here is unusually concrete. Doing this
well requires, simultaneously: an exact BAR archive decoder with engine-rule
citations; a provenance-graded game knowledge base to check derived facts
against; fluency in the Recoil source; and downstream projects that consume the
same terrain data and would expose errors. One person holds all four. The
combination is not merely convenient — it is close to a precondition, and
almost nobody else in the community has assembled it.

That is a strong meaning signal: the work is not interchangeable, and it would
not obviously get done by someone else.

**Secondary register.** The "it does not guess" ethic is a moral stance carried
into engineering, not a technical preference. It reappears in every artifact —
confidence tiers, `file:line` citations, documenting what is *deliberately not*
derived. Meaning here attaches to fidelity to truth as much as to the tool.

### PF-06: Eudaimonia

| Dimension | Assessment |
| --- | --- |
| **Self-Discovery** | Moderate. The work expresses an already-settled identity more than it explores one |
| **Best Potentials** | **Strong.** Deliberately reaches into unfamiliar territory (Godot, shaders, real-time rendering) from a secure base |
| **Purpose & Meaning** | **Strong.** Serves a community need the user can articulate and has evidence for |
| **Pursuit of Excellence** | **Very strong.** The defining trait across every artifact; parity over convenience, whole corpus over showcase |
| **Intense Involvement** | **Strong.** Sustained engagement across five repositories in one domain |
| **Personal Expression** | **Strong.** The tool embodies a stance about how software should relate to truth; that stance is authentically the user's |

**Meta-check — will the user flourish, not merely succeed?** The profile says
yes, with one condition. Five of six dimensions are strong, and the work is
authentically self-expressive. The condition is Pursuit of Excellence: at very
high intensity against an unbounded target, excellence stops being a source of
flourishing and becomes a source of never-finishing. Flourishing here depends
on the project having a **defined edge** — which is, again, the phase cut.

### PF-07: Flow

Assessing flow conditions for *the user's work*, not the product's UX.

| Condition | Assessment |
| --- | --- |
| **Clear Goals** | **Excellent.** "Open any of the 254 installed maps and fly around it" is crisp, binary, externally verifiable and not self-graded. Most specification-heavy projects never achieve a goal this clean |
| **Immediate Feedback** | **Excellent once rendering exists; absent before it.** A renderer is the most immediately-feedback-rich work there is — you see it. And a 254-map corpus yields a pass/fail count on every run. But the phase before first pixel (Rust `.smt` and minimap decoding, `gdext` plumbing, Godot project setup) is a **feedback desert** |
| **Challenge-Skill Balance** | **At risk on the anxiety side.** Parity shaders, terrain LOD, `gdext`, DXT1 decoding, S3O parsing and two VFS backends are all new, all at once. Demands are high and concentrated in the unfamiliar half of the skill profile |

**The clearest actionable finding in this assessment.** Flow is where the
phasing argument stops being prudential and becomes concrete: the work should be
ordered so that **something renders early**, even something crude and wrong.
A greybox heightmap with a flying camera, reached quickly, converts the feedback
desert into a feedback loop and rebalances challenge against a growing skill
base. Ordering parity first inverts both conditions at once.

### PF-08: VIA Character Strengths

Inferred signature strengths, 3-5 as the method prescribes:

| Strength | Virtue | Evidence |
| --- | --- | --- |
| **Honesty** | Courage | "It does not guess." Confidence tiers, documented non-derivations, citations over assertions. Integrity as an engineering principle, not a slogan |
| **Love of Learning** | Wisdom | Five languages and domains; reads engine source directly rather than documentation about it |
| **Judgment** | Wisdom | Grading evidence by confidence; distinguishing what the data supports from what it does not; refusing prior-art assumptions on purpose |
| **Perseverance** | Courage | Whole-corpus determinism, 227 maps, sustained multi-repository investment |
| **Appreciation of Beauty** | Transcendence | Visual parity chosen over data-honest rendering when the cheaper option was offered and defensible |

**Alignment: strong.** The intent activates every one of these. Honesty and
Judgment are load-bearing for a reference renderer specifically — the product's
entire claim is that what it shows is true.

**Gap: the Justice virtues.** Teamwork, Fairness and Leadership are
un-evidenced. Every artifact is solo. Yet two of the four stated audiences —
reviewers and authors — are a *community*, and reaching them requires
engagement the profile does not yet show: posting in the mapping channel,
responding to reviewer needs, sustaining users. The method's guidance is to
suggest complementary partners or deliberate development. Concretely: either
someone in the BAR mapping community becomes a collaborator or early user, or
the community audiences should be acknowledged as aspirational and the tool
scoped honestly as personal-first.

## Value dimension assessment

| Dimension | Score | Rationale |
|:---|:---|:---|
| Physical | **2/5** | A digital tool with no bodily, health or material impact; the only indirect register is reduced unpaid labour for volunteer reviewers |
| Emotional | **3/5** | Relieves tedious volunteer review work and lets authors *see* their own creation — an esteem and identity moment — but no feature yet addresses the relational side |
| Intellectual | **5/5** | Central and inseparable from the intent: it makes an opaque binary format legible, solves a genuinely hard decoding and rendering problem, and raises the craft ceiling for every map author |
| Spiritual | **4/5** | Contribution to a commons, a moral stance about fidelity to truth carried into engineering, and legacy through an ecosystem of interlocking repositories |

**Composite profile:** Intellectual-dominant, Spiritual-strong, moderate
Emotional, peripheral Physical.

**Balance assessment.** The profile is internally consistent and cross-checks
cleanly: Intellectual maps to the Wisdom virtues and to SDT Competence and
Autonomy, both scored very high; Spiritual maps to Logotherapy's Creative
pathway and the unique assignment. Physical at 2/5 is **expected and not a
gap** — a developer tool has no obligation there.

**Emotional at 3/5 is a genuine coverage gap.** The stakeholder map names
volunteer reviewers and map authors — relational audiences — while the intent
currently delivers to them only by rendering pixels accurately. The gap
coincides exactly with the thin Relatedness need (PF-03) and the missing Justice
virtues (PF-08). Three frameworks independently point at the same place, which
is the strongest signal in this assessment.

## Strengths

- **Motivation is structurally sustainable.** Integrated-to-intrinsic
  regulation with very high Autonomy and Competence fulfilment (PF-03), in a
  domain where the research confirms tools survive on community energy rather
  than revenue — PyMapConv and SpringBoard both still maintained, inside an
  ecosystem running since 2005. Sustainability does not depend on a business
  model that does not exist.
- **Need is externally validated, not assumed** (PF-04 Need: Strong). Two
  volunteer reviewers per map, a written checklist of verifiable properties, a
  recommended toolchain that exclusively authors and packs, and no external
  viewer. The gap was documented by the community itself.
- **Three strong Ikigai legs** — Love, Skill and Need — which is more than most
  intents carry at this stage (PF-04).
- **The goal is flow-shaped** (PF-07). "All 254 maps open and are navigable" is
  binary, externally verifiable, and immune to self-grading.
- **The assignment is genuinely unique** (PF-05). Exact decoder plus
  provenance-graded knowledge base plus engine fluency plus downstream
  consumers, held by one person. The work is not interchangeable.
- **Personality fits the task** (PF-01). High Openness with very high
  orderliness is close to optimal for long, exacting reverse-engineering.
- **Signature strengths are load-bearing, not incidental** (PF-08). Honesty and
  Judgment are precisely what a reference renderer's claim rests on.
- **Eudaimonic profile is strong on five of six dimensions** (PF-06), with
  authentic personal expression.

## Gaps

- **The fidelity bar sits in the weakest skill quadrant** (PF-04 Skill; PF-07
  Challenge-Skill). Visual parity requires GLSL-to-Godot porting, LOD and
  real-time rendering — the only areas with no demonstrated track record —
  while the user's proven mastery is all upstream in decoding. *Remedy:*
  sequence decoder work to done first; treat parity as a named, later,
  deliberately-learned phase.
- **A feedback desert before first pixel** (PF-07 Immediate Feedback). `.smt`
  decoding, `gdext` plumbing and project setup all precede anything visible.
  *Remedy:* order the work so a crude greybox heightmap with a flying camera
  arrives as early as possible.
- **Very high orderliness against an unbounded target** (PF-01; PF-06 Pursuit of
  Excellence). Parity has no natural stopping point; the trait that guarantees
  correctness resists declaring completion. *Remedy:* the phase edge must be
  written into the intent brief, not held as intention.
- **Relatedness, Justice virtues and Emotional value all thin at the same
  point** (PF-03 + PF-08 + Value Dimensions). Four audiences are named; two are
  communities; nothing in the intent yet engages them, and no artifact evidences
  community collaboration. *Remedy:* either recruit one reviewer or author as an
  early user, or scope the tool honestly as personal-first and let the community
  audiences be a later phase.
- **Viability is undefined rather than weak** (PF-04). No revenue intent, no
  licence decision, no distribution plan — and shader parity likely forces
  GPL v2-or-later anyway. Benchmark: even Tiled, far more broadly useful, funds
  only two days a week. *Remedy:* decide in refinement what "sustainable" means
  here — personal tool, community commons, or funded project. They imply
  different v1s.
- **Serving reviewers may require more than rendering** (Value Dimensions
  Emotional; PF-03 Relatedness). If checklist verification is a real use, a
  panel that surfaces the verifiable properties together is a different product
  surface than a viewer. Already carried forward as an open question; this
  assessment raises its priority.

## Research evidence used

| Finding | Source |
| --- | --- |
| Two-reviewer approval, `@mapper` role, written standards, volunteer reviewers | [BAR Map Review Process](https://www.beyondallreason.info/guide/map-reviews-process) |
| Checklist of verifiable map properties (pathability, metal distribution, normals resolution, sun angle, wind 0-30, tidal 0-25, feature reclaim, geovent buildability, minimap brightness) | [BAR Map Checklist](https://www.beyondallreason.info/guide/map-checklist) |
| Recommended toolchain authors, converts and packs; only validation tool is a SpringBoard pathability script | [BAR Mapmaking Resources](https://www.beyondallreason.info/guide/mapmaking-resources) |
| Tiled sustains ~200 sponsors at roughly $3,600/month, funding two days a week | [Tiled on Open Collective](https://opencollective.com/tiled) |
| PyMapConv and SpringBoard both actively maintained; ecosystem GPL since 2005 | [springrts_smf_compiler](https://github.com/Beherith/springrts_smf_compiler), [SpringBoard documentation](https://springboard-core.readthedocs.io/) |
| `gdext` usable for tools and editor plugins, most Godot APIs mapped, occasional breaking changes | [godot-rust book](https://godot-rust.github.io/book/) |
| Scope creep the most-cited failure cause; stated cause of death "infinitely growing scope" | [ModDB scope tutorial](https://www.moddb.com/tutorials/scope-your-video-game-and-eliminate-feature-creep) |
| S3O fully documented as pieces with vertex and index tables at file offsets | `bar-RecoilEngine/rts/Rendering/Models/s3o.h` |
| SMF shader chain location for the parity target | `bar-RecoilEngine/cont/base/springcontent/shaders/GLSL/SMF*.glsl` |
| Engine GPL v2-or-later | `bar-RecoilEngine/LICENSE` |

## Quality gates

| Gate | Status |
| --- | --- |
| At least 4 of 8 frameworks applied | Met - 7 of 8, with PF-02's exclusion justified |
| Ikigai and Value Dimensions always included | Met |
| Value dimension scores assigned for all four | Met - 2 / 3 / 5 / 4 |
| Strengths and gaps identified with framework citations | Met - 8 strengths, 6 gaps, all cited |
| Research evidence supports key assessments | Met - 10 sources, Need and Viability both externally grounded |
| Comparable ventures investigated for Need, Viability and Flow | Met - Tiled funding model; PyMapConv and SpringBoard longevity; ecosystem history |
