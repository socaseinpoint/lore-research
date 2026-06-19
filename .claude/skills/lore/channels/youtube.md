# Channel: youtube

Satisfies [`core/channel-contract.md`](../core/channel-contract.md). Canonical
implementation = the installed `yt-research` skill; this file is the channel spec for `/lore`.

## name / when-to-use
`youtube` — practitioner pitfalls, mental models, demos, conference talks, "how it feels in
production". Complements `web`; weak for exact numbers/specs (use web for those).

## discover
Run BOTH in one message: `WebSearch(allowed_domains:["youtube.com"])` AND `exa web_search_exa`.
Dedupe by video id. Prefer full videos over Shorts. Channel-scoped discovery: Firecrawl scrape
of `youtube.com/@channel/search?query=...` (renders the JS list).

**Freshness gate** (per `core/principles.md` → Freshness; levels: evergreen/current/fresh/bleeding):
cheap metadata pass for `publishedDate` BEFORE fan-out — at `current`/`fresh`/`bleeding`, drop
candidates older than the horizon (report how many dropped). At `fresh`/`bleeding` apply YouTube's
date filter (this year / this month). Rank by **view-velocity** (views ÷ age in months), NOT raw
views. Confirm any year-in-title against `publishedDate`; mismatch → flag. Keep top 5–6 of survivors.
Metadata source: `exa web_search_exa` carries `publishedDate`+views; `WebSearch` returns title+URL
ONLY (no dates) — so run the metadata pass through exa. A candidate still undated at `fresh`/
`bleeding` is HELD/DROPPED, never passed downstream undated (an undatable candidate FAILS the gate,
it does not bypass it).

## fetch
Transcript via `exa web_fetch_exa(url)` ONLY — returns transcript + metadata
(channel/length/views). **NEVER use `mcp__youtube-transcript__get-transcript`** — it is dead
(YouTube bot-blocks the server IP). Verified working path: exa, 0 failures on 7+ videos.

## extract
Concrete insights + 2–4 SHORT verbatim quotes per video. Capture title / channel / url /
views. ASR quotes may be rough — keep them short.

## caveats
- A transcript is SPOKEN WORDS ONLY — on-screen code/demos/slides are invisible. **Flag
  visual-heavy videos as incomplete.**
- Flag signal-vs-hype (affiliates/sponsors/clickbait).
- **Views-trap:** an old video accumulates views over time and looks more authoritative than it
  is — rank by view-velocity, not raw views.
- **The year in a title lies** — it is not the `publishedDate`; a "2025" title can be a 2022 upload.
- On empty/failed fetch: SAY SO and skip; never invent.

## subagent-brief
```
ONE SUBAGENT PER VIDEO (concurrent).
OBJECTIVE: distill <video url> for <topic>.
FETCH: exa web_fetch_exa(url) → transcript + metadata. NEVER the youtube-transcript MCP.
EXTRACT: concrete insights + 2–4 short verbatim quotes; title/channel/url/views.
OUTPUT: distilled ~300–500 words, NOT the raw transcript.
FLAGS: mark if value is VISUAL (transcript incomplete); flag affiliate/sponsor hype.
BOUNDARIES: if fetch empty/errored — SAY SO and skip; never invent.
RULES: obey core/principles.md → 10 hard rules (mandatory).
```
