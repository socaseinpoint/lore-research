# Channel: files

Satisfies [`core/channel-contract.md`](../core/channel-contract.md). **Given-source channel:**
the source is already in hand — the user supplies a path. There is no discovery step.

## name / when-to-use
`files` — when you ALREADY HAVE the source: a PDF, a book, your own notes, a local knowledge
base / folder. Extract knowledge from material you point at, not from the open web.

## discover
**Skipped.** The user supplies the path(s): a single file, a list, or a folder ("take
everything under here"). If a folder, enumerate readable files (`.md`, `.txt`, `.pdf`); for a
large set, ask which / cap and say what was capped — never silently drop sources.

## fetch
Read the file directly with the Read tool (no extra dependency):
- `.md` / `.txt` → read as text.
- `.pdf` → read with the Read tool's `pages` param (page ranges; large PDFs in chunks).
Out of scope for this simple tier: scanned/image PDFs (need OCR), complex tables. If a PDF
looks scanned (empty/garbled text extraction), SAY SO and skip — do not invent contents.

## extract
Per claim → a **locus** instead of a URL: `<filename>:p.<page>` (PDF) or `<filename> › <heading>`
(md/txt). For key facts include a SHORT verbatim quote. The file IS the primary source.

## caveats
- Scanned PDFs without an OCR layer extract as empty/garbage — flag and skip, don't guess.
- Tables/figures/equations may extract poorly — flag when a claim leans on one.
- Big books → chunk by page range/chapter; note coverage (which pages were read).
- Locus replaces URL: cite `file:page`, not a link.

## subagent-brief
```
ONE SUBAGENT PER FILE (or per chapter/section of a large file).
OBJECTIVE: extract what <file path> says about <topic / sub-question>.
FETCH: Read the file directly (.md/.txt as text; .pdf via Read pages=). Large → chunk by pages.
EXTRACT: concrete findings + 2–4 SHORT verbatim quotes, each with a locus (file:p.N or file › heading).
OUTPUT: distilled ~300–600 words. Note which pages/sections were actually read (coverage).
BOUNDARIES: stay within the given file(s). If a PDF is scanned/empty/garbled — SAY SO and skip.
RULES: obey core/principles.md → 10 hard rules (mandatory); cite locus instead of URL.
```
