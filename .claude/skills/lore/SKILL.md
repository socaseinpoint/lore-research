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

Find the tool's home (this repo) to read `core/principles.md`, `core/channel-contract.md`,
`channels/*.md`. If invoked from another project, those live where the `lore` skill source
is installed — read them there.

## Flow

### 1. Scope
Take the topic from `$ARGUMENTS`, or ask for it. Write a one-paragraph brief + explicit
sub-questions. (core/principles.md → Loop.)

### 2. Pick channels (INTERACTIVE)
Read each `channels/*.md` `name / when-to-use` line. Present the available channels and flag
which fit this topic. Ask the user to pick 1+ (use AskUserQuestion). Never auto-decide — the
user chooses where to search.

Channels are of two kinds (a channel states which via its `discover` section):
- **search channels** (`web`, `youtube`) — the tool discovers sources.
- **given-source channels** (`files`) — `discover` is SKIPPED; the source is in hand. If the
  user picks `files`, ask for the path(s) — a file, a list, or a folder — before fanning out.

### 2b. Pick depth (INTERACTIVE)
Ask how deep to dig (use AskUserQuestion): **quick** / **standard** / **deep**. See
`core/principles.md` → **Depth**. Depth is orthogonal to channels and sets the knobs for
steps 4–6: agents per channel, sources per agent, verify intensity, synthesis length.
Default = standard. quick trades breadth for speed, never rigor (hard rules always hold).

### 3. Open the arc container (process)
- `arcs new-goal <topic-slug>` — the run.
- For each chosen channel: `arcs new-arc -g <topic-slug> <channel>`.
- `arcs new-arc -g <topic-slug> synthesis`.
- Write the brief + sub-questions into the goal's / each channel arc's `input/`.

### 4. Fan out (research)
For each channel arc, dispatch subagent(s) using that `channels/<name>.md` **subagent-brief**
+ `core/principles.md`. Count per the chosen **depth** (principles → Depth / Fan-out
calibration). Subagents write numbered reports → the channel arc's `workspace/`, distilled
findings → its `output/`.

### 5. Verify (adversarial, fresh context)
For each channel `output/`, dispatch FRESH-context reviewer subagent(s). Check every
load-bearing claim against its cited source: **full / partial / not-supported**. Verify
intensity per **depth**: quick = one light pass; standard = one adversarial reviewer; deep =
≥2 perspective-diverse reviewers per load-bearing claim + a completeness critic. Write the
verdict into the channel arc's `workspace/`. (principles → rule 8 + Depth.)

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
