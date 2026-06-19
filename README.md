# lore

A small, channel-selectable research tool for Claude Code. You pick **where** to search
(web, youtube, your own files) and **how deep** to dig; it fans out subagents, adversarially
verifies every load-bearing claim against its source, and synthesizes a cited answer.

## Core idea

`research` is two different things, kept separate:

- **the verb** — doing research: scope → fan-out → verify → synthesize. Has a start and an end.
- **the noun** — the knowledge produced. The answer lives next to the question; promotion to a
  durable knowledge doc is optional and you choose where.

No buried product: question and answer stay together.

## How a run works (`/lore`)

1. **Scope** — a one-paragraph brief + explicit sub-questions.
2. **Pick channels** (interactive) — where to search.
3. **Pick depth** (interactive) — `quick` / `standard` / `deep`.
4. **Set freshness** (auto-detected, overridable) — `evergreen` / `current` / `fresh` / `bleeding`. Stale candidates are gated out by cheap metadata **before** fan-out.
5. **Fan out** — a subagent per source/channel, following that channel's brief + the hard rules.
6. **Verify** — a fresh-context adversarial reviewer rules each claim full / partial / not-supported.
7. **Synthesize** — one cross-channel pass from verified claims only.
8. **Promote (optional)** — lift the answer to a knowledge doc where you choose, or leave it.

## Channels

| channel | kind | what |
|---------|------|------|
| `web` | search | exa + native search; tiered anti-bot fetch (Firecrawl stealth, Playwright); primary sources |
| `youtube` | search | transcripts via exa; one subagent per video; flags visual-only content |
| `files` | given-source | your PDFs / books / notes / local KB; cite `file:page`; no discovery step |
| `deep-research` | wrapper | delegates web fan-out to Claude Code's built-in `/deep-research`, then runs lore's adversarial verify on top (which the built-in lacks) |

Add a channel by dropping a `channels/<name>.md` that satisfies `core/channel-contract.md`.

## Depth

`quick` (yes/no, a fact) · `standard` (default) · `deep` (broad — multi-source, perspective-diverse
multi-vote verify, completeness critic). Depth scales fan-out, sources, and verification — never rigor.

## Freshness

How current the answer must be — a third knob, orthogonal to channels and depth. Auto-detected from
the question at scope, overridable. `evergreen` (timeless — age ignored) · `current` (default, soft
~24mo) · `fresh` (`latest`/`2025`/fast-moving — hard ~12mo) · `bleeding` (newest, days–weeks). When
freshness ≠ evergreen, channels **gate stale candidates with cheap metadata (publishedDate,
view-velocity) before fan-out** — never burning subagent tokens on stale sources, never trusting the
year in a title over its real publish date.

## Layout

The skill is **self-contained** — everything lives under one directory, so symlinking it
carries the whole tool:

```
.claude/skills/lore/
  SKILL.md                # the /lore dispatcher (flow: scope → channels → depth → freshness → fan-out → verify → synth)
  core/
    principles.md         # the loop, fan-out calibration, tiered fetch, 11 hard rules, depth, freshness
    channel-contract.md   # the schema every channel must satisfy
  channels/
    web.md  youtube.md  files.md  deep-research.md  _template.md
```

## Install

The skill auto-loads when Claude Code runs inside this repo. To use `/lore` from **any**
project, symlink the self-contained skill dir into your user skills dir (it carries core/ +
channels/ with it):

```bash
ln -s "$(pwd)/.claude/skills/lore" ~/.claude/skills/lore
```

Tooling the channels expect (install what you need):

```bash
# Playwright (browser / JS render, no key)
claude mcp add playwright -s user -- npx @playwright/mcp@latest
npx playwright install chromium

# Firecrawl (anti-bot / Cloudflare; free key from firecrawl.dev; kept in the OS keychain)
security add-generic-password -s firecrawl-api-key -a "$USER" -w 'fc-YOUR_KEY' -U
claude mcp add firecrawl -s user -- /bin/sh -c 'FIRECRAWL_API_KEY=$(security find-generic-password -s firecrawl-api-key -w) npx -y firecrawl-mcp'
```

Search is also covered by exa + native WebSearch. Tiered fetch: cheap (WebFetch/exa) →
Firecrawl (anti-bot/JS, `{proxy:"stealth"}`) → Playwright (browser) → Wayback.

## The hard rules (anti-fabrication)

Every claim cited; if a fetch is blocked/empty, say so — never invent a page; verify the quote
actually exists; cross-check key facts across 2+ sources; primary sources over SEO; "unverified"
beats a confident guess; verify adversarially in a fresh context. Full set: `core/principles.md`.

## License

MIT.
