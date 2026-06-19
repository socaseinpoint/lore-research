# Channel contract

A **channel** is a place to search (web, youtube, …). Every `channels/<name>.md` MUST
provide exactly these six H2 sections, in this order. The `/lore` skill reads them to
present channel choices and to brief subagents.

## name / when-to-use
Channel id + what topics it is good for. The skill shows this line at selection time so
the user can pick where to search.

## discover
How to find candidate sources in this channel (which search tools, how to dedupe, how to
rank, how many to keep). Each channel MUST state how it applies the freshness gate (per
`core/principles.md` → Freshness) — or explicitly that age is not applicable (e.g. files).

## fetch
How to get the actual content. Reference the tiered ladder in
[`core/principles.md`](principles.md) → **Tiered fetch**. State any channel-specific fetch
quirk (e.g. stealth proxy, transcript endpoint).

## extract
What to pull from each source and the citation form — URL always; verbatim quote for key
facts; mark each source primary vs blog/SEO.

## caveats
Channel-specific failure modes (what silently returns nothing, what is invisible, what is
hype).

## subagent-brief
A copy-pasteable brief template the dispatcher hands each subagent for this channel:
objective, sub-questions, required output format, source guidance, boundaries.

---

**Mandatory for every channel:** the subagent-brief MUST embed the anti-fabrication rules
by reference to [`core/principles.md`](principles.md) → **10 hard rules**. No channel may
relax them.
