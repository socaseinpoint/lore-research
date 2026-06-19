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

## fetch
Tiered ladder — see [`core/principles.md`](../core/principles.md) → **Tiered fetch**.
Channel quirk: for Cloudflare/JS-only pages use
`firecrawl_scrape({proxy:"stealth", waitFor:6000})`. Check body length; "200 OK" ≠ content;
fail loudly on empty.

## extract
Per claim → URL. For key facts → a verbatim quote. Mark each source `primary` vs `blog/SEO`.
Numbers come from official pages only (SEO blogs gave wrong figures historically).

## caveats
- ~20% of the web sits behind Cloudflare, which blocks AI crawlers by default (since Jul 2025).
- JS-only SPAs return an empty shell to non-rendering fetchers.
- News sites block hardest (~79% block at least one AI bot).
- A real URL does not guarantee the claim is in it — verify the quote exists on the page.

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
