---
name: lore
description: Channel-selectable research — pick where to search (web, youtube, your own files) and how deep, fan out a subagent per source, adversarially verify each claim, synthesize a cited answer. The run is self-contained in an arc (question + answer together); promotion to a durable knowledge doc is optional. Invoke on "наресёрчь", "research X", "/lore", "изучи тему", "build lore about X", "есть ли тула под".
argument-hint: "<topic or research question>"
user-invocable: true
disable-model-invocation: false
---

# /lore — grow lore (research → durable knowledge)

A research run is **self-contained in its arc**: the question lives in `input/`, the evidence
in `workspace/`, the answer in `output/`. Question and answer stay together in one place.

Promotion to a durable knowledge shelf (e.g. a `lore/` doc) is **optional and user-directed** —
the user decides whether to lift the answer out and where it should land. By default the
answer just lives in its arc.

**Tool files live in THIS skill's base directory** (the dir holding this SKILL.md), regardless
of which project you invoke from: `core/principles.md`, `core/channel-contract.md`,
`channels/*.md`. Read them from there directly — do not go hunting one level up or resolving
symlink targets; the skill is self-contained.

## Flow

### 1. Scope
Take the topic from `$ARGUMENTS`, or ask for it. Write a one-paragraph brief + explicit
sub-questions. (core/principles.md → Loop.)

### 2. Resolve the knobs (channels · depth · freshness)
Three knobs steer the run. Resolve each from the question at Scope — don't reflexively ask, and
don't blindly default. See `core/principles.md` → **Knob resolution** for the full model.

**Per knob, classify the source of its value:**
- **explicit** — given in `$ARGUMENTS` or by the upstream caller/orchestrator → use as-is, NEVER
  re-prompt. (This is the contract for agent callers — skip-if-provided.)
- **inferable** — the question gives a clear signal → infer a confident value.
- **ambiguous** — two values are equally defensible; the question doesn't settle it.

**Decide ask-vs-infer by (confidence × cost-of-wrong).** Cost-of-wrong, which sets how eager to
ask: `channels > freshness > depth`.

- **channels** — cost HIGH (wrong channel = wrong universe of sources). Infer ONLY on a clear
  signal; else it's ambiguous. Heuristics: a file path / "my notes/PDF/book/local KB" → `files`;
  "in youtube / video / talk / demo" → `youtube`; "library/tool/docs/pricing/API/repo" or a plain
  technical question → `web` (default); an explicit **cross-medium** signal ("across articles and
  talks", "everything out there") → `web`+`youtube`. Note: bare "go deep / разберись глубоко" is a
  **depth** signal, NOT a channel-width one — don't widen channels on it; if the topic fits both
  web and youtube with no medium cue, channels is *ambiguous* (ask / headless-default web).
- **depth** — cost LOW (only scales effort). Almost always infer silently; default `standard`.
  "quick/just check/is there/yes-no/one fact" → `quick`; "deep/thorough/comprehensive/landscape"
  → `deep`; else `standard`. (`core/principles.md` → Depth.)
- **freshness** — cost MEDIUM. Auto-detect: "самое свежее/breaking/this week" → `bleeding`;
  year/"latest"/"now"/"актуальн" + fast-moving domain → `fresh`; "how X works"/theory/history →
  `evergreen`; else → `current`. Tie-breaker: when the FORM is theory-shaped ("how X works") but
  the SUBSTANCE is a fast-moving subfield (e.g. LLM agents, JS tooling), substance wins → `current`
  (or `fresh`), not `evergreen`. (`core/principles.md` → Freshness.) When ≠ evergreen, channels
  gate stale candidates at discover (cheap metadata BEFORE fan-out — rule 11).

Confidence test: *would a reasonable person reading the question land on the same value?* Yes →
infer. Two values equally defensible → ambiguous.

**Interaction mode:**
- **interactive** (a human is at the prompt): batch EVERY knob that needs the user into ONE
  `AskUserQuestion` (multi-question), with the inferred value listed first and marked
  "(Recommended)". Never fire three separate prompts. Knobs you resolved confidently are not asked.
- **headless / agent-invoked** (you are running as a subagent/tool, no human to ask): NEVER block.
  Take the best inferred value per knob; for any still-ambiguous knob use the conservative default
  (channels→`web`, depth→`standard`, freshness→`current`); and SURFACE every assumption in the
  run's `output/` so the caller can correct and re-run. Rule of thumb: if you cannot put an
  `AskUserQuestion` in front of a live human in this context, infer and flag — do not stall.

**Always echo** the resolved knobs before fan-out, with the source of each:
`channels=web [inferred] · depth=standard [default] · freshness=fresh [inferred]` — so the choice
is visible in both modes and the user can intervene.

Channel kinds (a channel states which via its `discover` section):
- **search channels** (`web`, `youtube`) — the tool discovers sources itself.
- **given-source channels** (`files`) — `discover` is SKIPPED; the source is in hand. If `files`
  is chosen, get the path(s) — a file, a list, or a folder — before fanning out.
- **wrapper channels** (`deep-research`) — delegate the fan-out to an external engine (the
  built-in `/deep-research` workflow) and run lore's verify on top. lore's adversarial verify
  (step 5) is still mandatory — that's the value the wrapped engine lacks.

### 3. Open the arc container (process)
- `arcs new-goal <topic-slug>` — the run.
- For each chosen channel: `arcs new-arc -g <topic-slug> <channel>`.
- `arcs new-arc -g <topic-slug> synthesis`.
- Write the brief + sub-questions into the goal's / each channel arc's `input/`.

### 4. Fan out (research)
For each channel arc, dispatch subagent(s) using that `channels/<name>.md` **subagent-brief**
+ `core/principles.md`. Count per the chosen **depth** (principles → Depth / Fan-out
calibration). **Subagents write numbered reports → the channel arc's `workspace/`** (and may
return their distillate to you). The channel arc's **`output/` is written by YOU (the
dispatcher) after that channel's subagents return** — a short distillate of the channel's
findings. Do not expect subagents to populate `output/`; that's the dispatcher's step, so the
close-guard finds it non-empty.

### 5. Verify (adversarial, fresh context)
For each channel `output/`, dispatch FRESH-context reviewer subagent(s). Check every
load-bearing claim against its cited source: **full / partial / not-supported**. Verify
intensity per **depth**: quick = one light pass; standard = one adversarial reviewer; deep =
**≥2 independent perspectives** per load-bearing claim + a completeness critic.

The "≥2 perspectives" for deep can be met two ways: (a) two dedicated fresh-context reviewers,
or (b) **independent cross-channel corroboration counts as one perspective** — a claim
confirmed by both `web` and `youtube` from independent sources already has a second angle, so
one dedicated adversarial reviewer on top satisfies the bar. **But a load-bearing claim that
rests on a SINGLE source / single channel gets a dedicated 2nd reviewer regardless** — no
cross-channel angle exists to lean on. Write verdicts into the channel arc's `workspace/`.
(principles → rule 8 + Depth.)

### 6. Synthesize
In the synthesis arc: input = the verified channel outputs. Write a cross-channel synthesis —
citations, disagreements, cross-validation — into the synthesis `output/`. Build only from
verified claims ("write last").

### 7. Offer promotion (OPTIONAL, user-directed)
The answer already lives in the synthesis arc's `output/` (next to the question). Now ASK the
user whether to promote it to a durable knowledge shelf, and WHERE — e.g. a `lore/<topic>.md`
in this project, a central knowledge home, or nowhere (leave it in the arc). Default = leave
it in the arc; do not exile by default.

If the user promotes: write/merge the synthesis into the chosen destination with frontmatter
`updated:`, `sources:`, `produced-by:` (the goal + arc paths). If a doc already exists there,
MERGE (grow it, keep prior cited claims) and append a history line. A grown doc may become a
folder (`README.md` + `sources/` + `log.md`). For `files`-channel sources cite a locus
(`file:p.N`) instead of a URL.

### 8. Close + report
`arcs close -g <topic-slug> <channel>` for each child + `synthesis`; then close the goal
(outputs are non-empty). Print the synthesis + the arc output path (where the answer lives) +
the promoted doc path if the user promoted it.

## Hard rules
Obey `core/principles.md` → **10 hard rules** throughout (no uncited claims; fail loudly on
blocked/empty fetch; verify quotes exist; primary sources; adversarial fresh-context verify).
Never finish with an empty or unclosed arc. Never invent page contents on a failed fetch.
