# Channel: deep-research

Satisfies [`core/channel-contract.md`](../core/channel-contract.md). **Wrapper channel:** instead
of running its own web fan-out, this channel delegates the heavy web research to Claude Code's
built-in **`/deep-research`** workflow, then lets lore's own verify layer (step 5) re-check the
result. You get the built-in's strong fan-out + lore's adversarial verify, which the built-in
does NOT do (its "voting" is intra-run consensus, not fresh-context per-claim source re-audit).

## name / when-to-use
`deep-research` — broad web questions where you want maximum fan-out without lore re-implementing
it. Reuses the first-party engine. Web-only (the built-in can't do youtube/files — use those
channels for non-web). Good as the web arm of a multi-channel run.

## discover
Delegated. Hand the sub-question(s) to the built-in `/deep-research` workflow (it fans web
searches across angles, cross-checks, votes, filters, returns a cited report — up to 16
concurrent / 1000 agents per run). lore does not pick sources here; the workflow does.

## fetch
Delegated to the workflow. lore does not run the tiered ladder for this channel — `/deep-research`
handles search + fetch + synthesis internally and returns a cited report.

## extract
Take the workflow's cited report as the channel's raw evidence → store it in the channel arc's
`workspace/`. Keep its citations (URL per claim) intact for the verify step.

## caveats
- **Web-only.** No youtube transcripts, no your-own-files. Pair with `youtube`/`files` channels.
- The built-in's "cross-checking / voting" is **intra-run consensus, not** per-claim verification
  against each claim's own source — so lore's step-5 adversarial fresh-context verify is **still
  mandatory** on top; that's the whole point of wrapping it.
- Treat the workflow's report as ONE channel's findings, not the final answer — it still flows
  through lore's verify + synthesis.
- It costs real fan-out tokens (it spawns its own agents); don't also run the plain `web` channel
  for the same sub-question — pick one web engine.

## subagent-brief
```
This channel is dispatched as a WORKFLOW, not a plain subagent.
INVOKE: the built-in /deep-research workflow on the sub-question(s).
CAPTURE: its cited report verbatim → channel arc workspace/ (keep all source URLs).
DISTILL: dispatcher writes the channel output/ distillate from that report.
THEN: lore step 5 adversarially re-verifies each load-bearing claim against its own source in a
      FRESH context (the built-in did not do this). Mandatory.
RULES: obey core/principles.md → 10 hard rules; the wrapped engine does not exempt them.
```
