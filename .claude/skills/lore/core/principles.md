# Research principles

Condensed operating core for the `/lore` tool. Source of facts: `RESEARCH-KNOWLEDGE-EXPORT.md`
(§1, §6, §9, §10, §12). Do not contradict it.

## Loop

Run every research as: **Scope → Research → Verify → Write.**

- **Scope** — one-paragraph brief + explicit sub-questions + chosen `freshness` (level + horizon) BEFORE searching. Never hand a complex question to the model as one monolithic retrieval.
- **Research (fan-out)** — a lead spawns parallel subagents, each with a specific objective, output format, source guidance, boundaries. Each burns tokens, returns a distilled 1–2k-token summary with citations.
- **Verify (separate, adversarial)** — a reviewer in a FRESH context checks each load-bearing claim against its cited source: full / partial / not-supported. Self-grading in the same context inherits the same blind spots.
- **Write once** — synthesize the final product in one pass from verified files ("write last").

## Fan-out calibration

- 1 agent = a single fact.
- 2–4 agents = a comparison.
- >10 agents = broad synthesis only.
- Multi-agent costs ~15× single-agent tokens (Anthropic). Reserve breadth for breadth.

## Tiered fetch

Escalate only on failure; report which tier worked or that all failed.

```
1. Cheap:    WebFetch / exa web_fetch_exa
2. Blocked/JS → Firecrawl (proxies, anti-bot, JS render) / Jina (r.jina.ai)
3. Real browser needed → Playwright MCP
4. Dead/paywalled → Wayback Machine + cache
```

Detect failure by body length and block-pages/CAPTCHA, not status code. **"200 OK" ≠ content** (a CSR SPA returns a 200 with an empty shell). **Fail loudly on empty — never invent page contents.** Past Cloudflare: `firecrawl_scrape({proxy:"stealth", waitFor:6000})`.

## 11 hard rules

Embed in every channel and subagent brief:

1. No claim without a cited source; for a key fact, a verbatim quote + URL.
2. If a fetch is blocked/empty/near-empty: say so, don't invent the page.
3. Escalate fetch by tiers; report which tier worked or that all failed.
4. Verify the quoted text actually exists on the page (real URL ≠ supported claim).
5. Cross-check important facts across 2+ independent sources.
6. Prefer primary sources (official docs, arXiv, repos) over listicles/SEO.
7. Allow "I don't know" / "unverified"; flag low confidence.
8. Verify with an adversarial reviewer in a FRESH context (full/partial/not-supported).
9. For decisions, prefer self-consistency (majority vote) over naive debate.
10. YouTube transcript = speech only (flag visual gaps); fetch via exa, never the youtube-transcript MCP.
11. cheap-gate-before-expensive-fanout: when freshness≠evergreen, gate candidates by cheap metadata (publishedDate / view-velocity) BEFORE fan-out; never discover staleness post-dispatch; never trust the year in a title — confirm against publishedDate; report how many candidates were dropped and why.

## Depth

Depth is orthogonal to channels: channels = WHERE to look, depth = HOW HARD to dig. The user
picks one per run; it scales fan-out, sources, verification, and synthesis length.

| depth | for | agents/channel | sources/agent | verify | synthesis |
|-------|-----|----------------|---------------|--------|-----------|
| **quick** | yes/no, a single fact, "is there a tool for X" | 1 (across channels) | 1–2 | light (one pass over the answer, may skip per-claim) | 2–4 sentences |
| **standard** *(default)* | a normal question / comparison | 2–4 as needed | 2–3 | 1 adversarial reviewer, fresh context | full |
| **deep** | broad question, "dig deep" | 3–5+ across sub-questions | 3+ | multi-vote, **perspective-diverse** (≥2 reviewers per load-bearing claim) + completeness critic / loop-until-dry | long, structured |

- Map onto Fan-out calibration above — depth just sets the knobs.
- `deep` is where a deterministic orchestration engine (Workflow) earns its place — for now
  `deep` = more subagents via the skill; route to Workflow later (v2).
- Always honor the hard rules regardless of depth; quick trades breadth for speed, NOT rigor —
  a quick answer still cites its source and still fails loudly on a blocked fetch.
- deep's "≥2 perspectives per load-bearing claim": independent cross-channel corroboration
  (web AND youtube confirming a claim from independent sources) counts as one perspective, so
  one dedicated adversarial reviewer on top meets the bar. A claim resting on a SINGLE source /
  single channel gets a dedicated 2nd reviewer regardless — no cross-channel angle to lean on.

## Freshness

Freshness is the 3rd orthogonal knob — a mirror of depth: channels = WHERE to look, depth =
HOW HARD to dig, freshness = HOW CURRENT to keep. The user picks one per run; it's auto-detected
at Scope and the user overrides.

Freshness ≠ just a date. It's a composite actuality signal: publishedDate + view-velocity
(views/age) + whether the topic's SUBSTANCE has decayed + year-in-title as a FALSE signal.

| freshness | for | gate horizon | discover behavior |
|-----------|-----|--------------|-------------------|
| **evergreen** | timeless topic: "how TCP works", theory, history, math | none | ignore age; rank by relevance + authority |
| **current** *(default, auto)* | a normal question / comparison | soft ~24mo | prefer fresh; keep old ONLY if clearly authoritative and the substance is still alive |
| **fresh** | "latest", "2025/now", fast-moving domain (prices, job market, model releases) | hard ~12mo | DROP stale before fan-out; confirm dates; rank by velocity |
| **bleeding** | "newest", breaking, "what shipped this week", just-released | days–weeks | tightest date filter (this week/month); rank by pure RECENCY (newest first), velocity secondary; sources: news/release-notes/changelog/social; authority yields to speed — flag as unverified-fresh |

- **Auto-detect at Scope** from question signals: "самое свежее"/breaking/"this week"/
  "just now" → `bleeding`; year/"latest"/"now"/"актуальн" + fast-moving domain → `fresh`;
  "how X works"/theory/history → `evergreen`; else → `current`.
- **Form-vs-substance tie-breaker**: a theory-shaped question ("how X works") on a fast-moving
  subfield (LLM agents, JS tooling, prices) is NOT evergreen — substance decay wins → `current`/
  `fresh`. evergreen is only for genuinely timeless substance (math, protocols, history).
- **User overrides** (like depth) — an explicit choice beats auto-detect.
- **Horizon is tunable per topic at Scope, not magic**: default by level, but Scope can tighten
  it to the domain (news=weeks, tooling=12mo, theory=∞).

## Knob resolution

The three knobs (channels, depth, freshness) are often left unspecified — especially when `/lore`
is called by another agent. Resolve each from the question; never reflexively ask, never blindly
default.

- **Provenance** per knob: **explicit** (in args / from the caller — use as-is, never re-prompt) ·
  **inferable** (clear signal in the question — infer confidently) · **ambiguous** (two values
  equally defensible).
- **Ask vs infer = confidence × cost-of-wrong.** Cost-of-wrong ranks `channels > freshness >
  depth`, so be most willing to ask about channels (wrong channel = wrong source universe) and
  least about depth (only scales effort; default standard). Confidence test: would a reasonable
  reader land on the same value?
- **Interaction mode.** Interactive (human present): batch all uncertain knobs into ONE question,
  inferred value first/recommended. Headless (running as a subagent/tool, no human): never block —
  infer the best, fall back to conservative defaults (channels→web, depth→standard,
  freshness→current) for the still-ambiguous, and surface every assumption in the run's output.
- **Always echo** the resolved knobs (with the source of each) before fan-out, both modes.

## Decisions

Self-consistency (majority vote over sampled reasoning) > multi-agent debate / mixture-of-agents (inconsistent, hyperparameter-sensitive). Dominant failure of all consensus methods: **agents converge on agreement, not truth** (conformity/sycophancy). Design for diversity, not just voting.
