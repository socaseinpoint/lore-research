# Channel: web

Satisfies [`core/channel-contract.md`](../core/channel-contract.md).

## name / when-to-use
`web` — general/technical topics, official docs, repos, pricing, news, methodology. The
default channel; use it unless the question is specifically about video content.

## discover
Run BOTH in one message: `exa web_search_exa` AND native `WebSearch`. Dedupe by URL.
Prefer primary sources (official docs, arXiv, GitHub repos) over SEO listicles. Keep the
top sources by relevance + authority; calibrate count per `core/principles.md` → Fan-out
calibration.

**Freshness gate** (per `core/principles.md` → Freshness; levels: evergreen/current/fresh/bleeding):
at `current`/`fresh`/`bleeding`, drop sources older than the horizon BEFORE fan-out (report how
many dropped). Do NOT trust the year in title/URL — it is not the publication date. For
time-sensitive topics prefer dated sources over undated SEO. Metadata source: `exa web_search_exa`
carries `publishedDate`; native `WebSearch` does not — run the date pass through exa. A source
still undated at `fresh`/`bleeding` is held/dropped, not passed downstream undated.

## fetch
Tiered ladder — see [`core/principles.md`](../core/principles.md) → **Tiered fetch**.
Channel quirk: for Cloudflare/JS-only pages use
`firecrawl_scrape({proxy:"stealth", waitFor:6000})`. Check body length; "200 OK" ≠ content;
fail loudly on empty.

## extract
Per claim → URL. For key facts → a verbatim quote. Mark each source `primary` vs `blog/SEO`.
Capture each source's publication date when present (feeds the freshness gate and citations).
Numbers come from official pages only (SEO blogs gave wrong figures historically).

## caveats
- ~20% of the web sits behind Cloudflare, which blocks AI crawlers by default (since Jul 2025).
- JS-only SPAs return an empty shell to non-rendering fetchers.
- News sites block hardest (~79% block at least one AI bot).
- A real URL does not guarantee the claim is in it — verify the quote exists on the page.
- SEO "best <year>" pages are often old content reskinned under a new year in the title — the
  date in the headline is not the publication date.

## subagent-brief
```
OBJECTIVE: <one specific sub-question>
SUB-QUESTIONS: <2–4 concrete things to answer>
DISCOVER: exa web_search_exa + native WebSearch (one message); dedupe; primary sources first.
FETCH: tiered ladder; Cloudflare/JS → firecrawl_scrape stealth waitFor 6000; check body length.
OUTPUT: distilled ~1–2k tokens. Each claim cited (URL); key facts get a verbatim quote;
        mark sources primary/blog. State which fetch tier worked or that all failed.
BOUNDARIES: only this sub-question. If blocked/empty — SAY SO, never invent page contents.
RULES: obey core/principles.md → 10 hard rules (mandatory).
```
