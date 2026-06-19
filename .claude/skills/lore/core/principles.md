# Research principles

Condensed operating core for the `/lore` tool. Source of facts: `RESEARCH-KNOWLEDGE-EXPORT.md`
(§1, §6, §9, §10, §12). Do not contradict it.

## Loop

Run every research as: **Scope → Research → Verify → Write.**

- **Scope** — one-paragraph brief + explicit sub-questions BEFORE searching. Never hand a complex question to the model as one monolithic retrieval.
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

## 10 hard rules

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

## Decisions

Self-consistency (majority vote over sampled reasoning) > multi-agent debate / mixture-of-agents (inconsistent, hyperparameter-sensitive). Dominant failure of all consensus methods: **agents converge on agreement, not truth** (conformity/sycophancy). Design for diversity, not just voting.
