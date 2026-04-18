# Obsidian LLM Wiki — Book → Compounding Knowledge Base

A step-by-step skill for turning **any book (or dense reference PDF)** into a living, interlinked Obsidian wiki that Claude Code maintains. Implements [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): instead of RAG re-reading raw documents on every query, the LLM **incrementally builds and maintains a persistent wiki** that compounds over time.

**Covers:** PDF inspection → vault scaffold → local chapter splitting → markdown extraction → schema authoring → pilot ingestion → quality review → full-book ingestion → ongoing compounding phase.

---

## When to Use

- User wants to turn a book, dense PDF, or body of long-form content into a structured, queryable Obsidian vault
- User says: "ingest this book", "make a second brain from this PDF", "build an LLM wiki", "Karpathy wiki pattern", "compounding knowledge base"
- User wants to layer their own material (customer calls, PRDs, podcasts, other books) onto a framework-heavy reference

---

## The pattern in one picture

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────┐
│   raw/      │ →   │   wiki/           │ →   │  Obsidian    │
│ (immutable) │     │ (Claude owns)     │     │  graph view  │
│             │     │                   │     │              │
│ Source PDF  │     │ sources/          │     │ You browse   │
│ Chapter PDFs│     │ frameworks/       │     │ Claude edits │
│ Chapter MDs │     │ concepts/         │     │              │
│             │     │ metrics/          │     │              │
│             │     │ roles/            │     │              │
└─────────────┘     │ Memory.md         │     └──────────────┘
                    │ index.md          │
                    │ log.md            │
                    │ CLAUDE.md (schema)│
                    └──────────────────┘
```

**Hard rules:**
- `raw/` is immutable. Never modify.
- `wiki/` is Claude-owned. User reads; Claude writes.
- `CLAUDE.md` is the schema. Co-evolve; don't silently edit.

---

## Prerequisites

| What | Install command | Why |
|---|---|---|
| **Obsidian** | [obsidian.md](https://obsidian.md) | The viewer; not strictly required for ingestion but essential for day-to-day use |
| **Claude Code** | [claude.com/product/claude-code](https://claude.com/product/claude-code) | The engine |
| **qpdf** | `brew install qpdf` | Local PDF splitting by page range |
| **poppler** | `brew install poppler` (bundles `pdftotext`, `pdfinfo`) | Local PDF→text extraction + metadata inspection |

**Optional (skip for most cases):**
- Firecrawl CLI + API key — gives richer markdown (LaTeX formulas, neural layout), but **requires a publicly-reachable URL**. Can't accept local file paths. For a one-time book ingestion, `pdftotext` is usually sufficient. See §"Firecrawl tradeoff" below.

---

## Step-by-step

> These steps are what Claude Code should execute when invoked with a book. Each step has a checkpoint — don't advance until it passes.

### Step 1 — Inspect the PDF

Before splitting, verify three things:

```bash
# File size, page count
pdfinfo "<path-to.pdf>"

# Text-extractability — extract first 3 pages, eyeball the output
pdftotext -layout -f 1 -l 3 "<path-to.pdf>" -

# Find the table of contents
pdftotext -layout "<path-to.pdf>" - | grep -n -i -E "^(chapter|contents|part [0-9ivx]+)" | head -40
```

**Checkpoint:**
- ✅ File size is reasonable for page count (< ~10KB/page → text-based; > ~100KB/page → probably scanned, OCR needed)
- ✅ Text extraction looks clean (proper ligatures `fi`/`fl`, smart quotes preserved, no garbled mojibake)
- ✅ ToC is readable and page numbers match PDF page numbers (check a couple: "Chapter 2 starts on p. 42" → open PDF page 42 in viewer, verify)

**If text is bad / PDF is scanned:** you'll need OCR (Firecrawl, or `ocrmypdf` locally). Don't proceed until extraction is clean.

---

### Step 2 — Scaffold the vault

Pick a target directory (new folder, not inside an existing project):

```bash
VAULT=~/Obsidian/<book-slug>-brain
mkdir -p "$VAULT"/{raw/{pdf,pdf-chapters,markdown},wiki/{sources,frameworks,concepts,metrics,roles,assets}}
```

Copy the source PDF into the vault (preserves the original in Downloads):

```bash
cp "<path-to.pdf>" "$VAULT/raw/pdf/"
```

Create top-level files:
- `CLAUDE.md` — the schema (see template below)
- `Memory.md` — user context (see template below)
- `index.md` — skeleton catalog (see template below)
- `log.md` — skeleton activity log
- `.gitignore` — at minimum `.DS_Store\n.obsidian/\n*.log`

**Checkpoint:** Open the vault in Obsidian. File tree shows the directory structure. `CLAUDE.md` is readable in Obsidian preview.

---

### Step 3 — Split the PDF into chapters

Using the ToC you found in Step 1, build a page-range map:

```bash
cd "$VAULT"
SRC=raw/pdf/<book-name>.pdf
DEST=raw/pdf-chapters

# Edit this list with your book's actual chapter pages.
# Format: "filename-slug:startPage-endPage"
declare -a SPLITS=(
  "00-prologue:8-9"
  "01-introduction:10-41"
  "02-first-principles:42-69"
  # ... etc
)

for entry in "${SPLITS[@]}"; do
  name="${entry%%:*}"
  pages="${entry##*:}"
  qpdf "$SRC" --pages . "$pages" -- "$DEST/${name}.pdf" && echo "✓ ${name}.pdf ($pages)"
done
```

Verify page counts:

```bash
for f in "$DEST"/*.pdf; do
  echo -n "$(basename $f): "
  pdfinfo "$f" | grep "^Pages:" | awk '{print $2}'
done
```

**Checkpoint:** All chapter PDFs exist, page counts match expected. If any chapter is 0 pages, the split range was wrong.

**Gotcha:** Printed page numbers in the ToC don't always match PDF page numbers (some books have prefaces/covers that offset). Verify the offset on one chapter before running the whole loop.

---

### Step 4 — Extract markdown (chapter → .md)

**Default: use `pdftotext -layout`** (local, free, works offline):

```bash
for f in raw/pdf-chapters/*.pdf; do
  name=$(basename "$f" .pdf)
  pdftotext -layout "$f" "raw/markdown/${name}.md"
  echo "✓ ${name}.md ($(wc -l < "raw/markdown/${name}.md") lines)"
done
```

**Spot-check the pilot chapter:** open one `.md` file. Verify:
- Prose is clean and readable
- Section headings / numbered subsections are recognisable
- Bullet structure is preserved
- Running headers (e.g. `CHAPTER 02 | TITLE`) and page numbers appear regularly — these will be stripped later during wiki ingestion

**Firecrawl tradeoff:**
- **Pro:** better markdown (LaTeX formulas, multi-column layout detection, diagram OCR)
- **Con:** requires public URL — can't send local files. Need cloudflared/ngrok tunnel or upload step.
- **Verdict:** Skip Firecrawl for one-time book ingestion. `pdftotext` quality is sufficient because the real structure gets built in the wiki pages, not in raw/markdown.

**Checkpoint:** `raw/markdown/` has one `.md` per chapter PDF.

---

### Step 5 — Author the CLAUDE.md schema

Copy the [CLAUDE.md template](#claudemd-template) below into `$VAULT/CLAUDE.md` and customise for the book's domain.

**What to customise:**
- The vault purpose paragraph (who the user is, what this book is)
- The five page-type examples under "Page templates" (if the book's domain isn't business/frameworks — e.g. for a physics textbook, replace "frameworks" with "laws", "concepts" with "phenomena", "metrics" with "equations")
- The "Initial seed" pointer (which chapter to ingest first as pilot)

**Critical rule to enforce in CLAUDE.md:**

> - **Wikilink style:** `[[Page Name]]` (Obsidian default). Link text matches filename exactly so links resolve without pipe syntax.
> - **File naming:**
>   - **Source pages** — `wiki/sources/NN-slug.md` (kebab-case, numbered, mirrors `raw/markdown/NN-slug.md`). Linked as `[[NN-slug]]`.
>   - **Framework / concept / metric / role pages** — Title Case with spaces, e.g. `wiki/concepts/Risk Shift.md`. Linked as `[[Risk Shift]]`.
>   - Hyphens preserved inside tokens (e.g. `Customer-Centric Growth.md`).

This prevents ghost nodes in graph view — the single biggest gotcha we hit.

---

### Step 6 — Seed Memory.md

Open Claude chat with the user. Spend 15–20 minutes capturing:
- Their role
- Why this book matters to them
- What they plan to layer on top (customer calls, PRDs, podcasts, other books)
- Their preferences for how Claude Code should work in this vault

Save the distillation as `Memory.md`. See [Memory.md template](#memorymd-template) below.

**Checkpoint:** User sees their own context reflected back. Adjust if it's wrong before proceeding.

---

### Step 7 — Pilot: ingest one chapter

Pick a **foundational chapter** (not necessarily Chapter 1 — often Chapter 2 or 3 is richer). Tell Claude Code:

```
Ingest raw/markdown/<NN-slug>.md per the CLAUDE.md workflow.

Process:
1. Read the file end-to-end
2. Strip running headers/footers
3. Present top 3–5 takeaways for my review
4. Wait for my confirmation or redirect
5. Write wiki/sources/<NN-slug>.md
6. Create framework/concept/metric/role pages for entities introduced
7. Update index.md
8. Append an entry to log.md
9. Report every file touched

Aim for 10–15 new wiki pages per chapter (Karpathy's sweet spot).
```

**Checkpoint:** Open Obsidian. Switch to graph view. Verify:
- ✅ The source page sits at the centre of a dense cluster
- ✅ 10–15 pages linked to it
- ❌ **No ghost nodes** — every link resolves to a real file

**If you see ghost nodes** (paired dark + light circles with similar names): filenames don't match link text. Either rename the file (preferred — keep Obsidian-native Title Case) or update the links. See the "Title Case vs kebab-case" gotcha below.

---

### Step 8 — Review and tune the schema

Before scaling to the rest of the book, pressure-test the pilot:

- **Tone:** are pages mechanical and reference-style, or are they bloated / marketing-voiced? Adjust CLAUDE.md's tone section.
- **Depth:** 10–15 pages is the target. If Claude produced 30 pages for one chapter, tighten the rule. If it produced 3, the chapter is under-ingested.
- **Page types:** are frameworks, concepts, metrics, and roles clearly differentiated? If everything became "concept", the type system is too loose.
- **Cross-linking:** does each page link to 3+ others? Isolated pages don't compound.

Adjust CLAUDE.md. Re-ingest the pilot chapter if needed (delete its wiki pages first, re-run).

**Checkpoint:** You'd be happy to scale this pattern to the remaining chapters.

---

### Step 9 — Batch ingest remaining chapters

Once the schema is stable, ingest the rest. Two strategies:

**A. Strategic (recommended for books with layered frameworks):** Ingest in an order that promotes stubs first. For Revenue Architecture this was Ch 5 (Revenue Model) + Ch 7 (Mathematical Model) before the Ch 8/9/10 dynamic-layer chapters, because the metric-defining chapters promote ARR/NRR/LTV stubs that later chapters reference.

**B. Sequential:** Just go in order. Simpler but sometimes creates more stubs that get promoted later.

**For each chapter, paste the same ingest prompt from Step 7.** Between chapters, spot-check the index and graph.

**Forward-stub rule:** If Claude encounters a term it needs to link to but that hasn't been defined yet (e.g. Ch 5 mentions "Operating Model" which won't be defined until Ch 8), create a **stub page** immediately with an `*Stub — full treatment in Ch N. Expand on that ingestion.*` note. This prevents ghost nodes and guides future ingestions.

**Checkpoint:** All chapters ingested. index.md lists every page. log.md has one entry per ingest.

---

### Step 10 — Compounding phase (ongoing)

This is where the Karpathy pattern pays off. The book's framework is now the **lattice** onto which everything else you consume layers.

Clip articles with Obsidian Web Clipper → `raw/`. Record customer call transcripts → `raw/`. Save podcast notes → `raw/`. For each new source, tell Claude Code:

```
Ingest <path> per the CLAUDE.md workflow. Link it into the existing wiki framework — don't create new isolated concepts if an existing page covers the same idea.
```

Periodically run a lint:

```
Lint the wiki per CLAUDE.md. Find:
- Contradictions between pages
- Orphan pages (no inbound links)
- Concepts mentioned repeatedly but lacking their own page
- Stub pages that could now be promoted from newer sources
Write the report to wiki/lint-report.md.
```

---

## CLAUDE.md template

Paste this into `$VAULT/CLAUDE.md`, then customise the two commented sections.

~~~markdown
# CLAUDE.md — Wiki Schema

You are the maintainer of a personal knowledge wiki. Follow this schema whenever you work in this vault.

## Purpose

<!-- CUSTOMISE: one paragraph on who the user is, what book/corpus seeds this vault, and what they plan to layer on top. -->

A personal second brain for {{USER_ROLE}}, seeded with {{BOOK_TITLE}} by {{AUTHOR}}. The goal is a compounding, interlinked knowledge base where {{BOOK_DOMAIN}} becomes a living reference the user layers real-world material onto (e.g. customer calls, PRDs, podcasts, articles, other books).

Pattern reference: Karpathy's LLM Wiki.

## Architecture (three layers)

1. **`raw/`** — Source documents. **Immutable.** Never modify files here. Read-only.
   - `raw/pdf/` — original PDFs
   - `raw/pdf-chapters/` — split-by-chapter PDFs
   - `raw/markdown/` — extracted markdown (one file per chapter, naming `NN-slug.md`)
2. **`wiki/`** — AI-generated markdown. **You own this entirely.**
   - `wiki/sources/` — one page per source (book chapter, article, call transcript)
   - `wiki/frameworks/` — named frameworks
   - `wiki/concepts/` — ideas and constructs
   - `wiki/metrics/` — specific measurements with formulas
   - `wiki/roles/` — human roles (where applicable)
   - `wiki/assets/` — images, screenshots
3. **This file (`CLAUDE.md`)** — the schema. Co-evolve with the user. Don't modify silently; propose changes.

## Top-level files

- `Memory.md` — user context, recurring preferences
- `index.md` — content-oriented catalog of every wiki page
- `log.md` — chronological, append-only record

## Page templates

Every wiki page starts with YAML frontmatter:

```yaml
---
type: source | framework | concept | metric | role
tags: [tag1, tag2]
sources: ["[[02-source-slug]]"]
updated: YYYY-MM-DD
---
```

### Source page (`wiki/sources/NN-slug.md`, kebab-case with number)
```markdown
# {Source Title}

**Source:** `raw/markdown/NN-slug.md`

## Thesis
One paragraph summary.

## Key claims
- Each claim links to a concept/framework/metric via [[wikilink]]

## Frameworks introduced
- [[Framework Name]] — one-line

## Concepts introduced
- [[Concept Name]] — one-line

## Metrics introduced
- [[Metric Name]] — one-line

## Notable diagrams
- Figure N.N: description

## My notes
*Reserved for user.*
```

### Framework / Concept / Metric / Role page (`wiki/<type>/Title Case.md`)

Generic shape:
```markdown
# {Name}

## Definition
Precise, unambiguous. Cite source.

## {Type-specific body}
- Framework → Components, When to use
- Concept → Mechanism, Consequences
- Metric → Formula, Benchmarks, Interpretation
- Role → Scope, Metrics owned, Reports to

## Related
- [[Related Page]]

## Sources
- [[NN-slug]] — section reference
```

## Workflows

### `ingest <file>`

1. Read the file end-to-end.
2. Strip running headers/footers (boilerplate from PDF conversion).
3. Discuss top 3–5 takeaways with the user. Pause for confirmation.
4. Write `wiki/sources/NN-slug.md`.
5. Create or update framework/concept/metric/role pages. **Prefer updating an existing page over creating a duplicate.** Grep the wiki for the concept name + variants first.
6. Update `index.md` with every new page.
7. Append a log.md entry:
   ```
   ## [YYYY-MM-DD] ingest | Chapter NN: Title
   Files touched:
   - wiki/sources/NN-slug.md (created)
   - wiki/frameworks/foo.md (created)
   - ...
   ```
8. Report the full file list to the user.

### `query <question>`

1. Read `index.md` first. Identify candidate pages.
2. Read those pages. Don't guess; only cite what's in the wiki.
3. Synthesize an answer with `[[wikilink]]` citations.
4. If the answer is substantive, offer to file it as a new page (e.g. `wiki/sources/exploration-YYYY-MM-DD-topic.md`) so explorations compound too.

### `lint`

Find contradictions, orphan pages, stubs worth promoting, missing cross-refs, stale claims. Write to `wiki/lint-report.md` with severity.

## Tone and style

- **Mechanical, not marketing.** No "unlock", no "supercharge", no flourish.
- **Definitions before examples.**
- **Formulas as code or LaTeX**, never prose.
- **Cite or omit.** If a claim isn't in a source, don't write it. If it's your inference, mark it `*Inference:* ...`.
- **Terse.** Prefer bullets to paragraphs.

## Hard rules

- Never modify anything under `raw/`.
- Never invent facts not grounded in a source.
- Never delete a wiki page without proposing the deletion first.
- Never change `CLAUDE.md` silently; surface the proposed change.
- **Wikilink style:** `[[Page Name]]`. Link text matches filename exactly so links resolve without pipe syntax.
- **File naming:**
  - **Source pages** — `wiki/sources/NN-slug.md` (kebab-case, chapter-numbered). Linked as `[[NN-slug]]`.
  - **Everything else** — `wiki/<type>/Title Case.md` (Title Case with spaces; hyphens preserved inside tokens). Linked as `[[Title Case]]`.
  - **Rationale:** Obsidian resolves `[[Foo Bar]]` to `Foo Bar.md` by default. Matching filenames to link text eliminates ghost nodes without needing pipe syntax.

## Linking conventions

- A source page links OUT to every framework/concept/metric/role it introduces.
- A framework/concept/metric/role page lists source pages under `## Sources`.
- Cross-link related entities. Graph view should reveal clusters, not orphans.
- **Before creating a new page**, grep the wiki for the name + variants (singular/plural, abbreviation). Prefer updating over duplicating.

## Initial seed

<!-- CUSTOMISE: which chapter to ingest first as the pilot. Pick a foundational one — not always Ch 1. Often Ch 2 or 3 is richer. -->

The first source ingested will be `raw/markdown/{{PILOT_CHAPTER}}.md`. Use it to bootstrap the initial set of framework/concept/metric/role pages. After pilot review, the user will direct ingestion of the remaining chapters.
~~~

---

## Memory.md template

~~~markdown
# Memory

Persistent context about the user. Apply when working in this vault.

## Role
{{USER_ROLE — e.g. product manager, research analyst, founder}}

## Why this vault exists
{{1–2 sentences on the problem this vault solves for the user, and what the seed book/corpus is}}

## Goals for the wiki
- {{Goal 1}}
- {{Goal 2}}
- {{Goal 3}}

## Preferences
- **Never modify files without explicit approval.** Present the plan first, wait.
- {{Other preferences — terseness, trade-off analysis, etc.}}

## Seed sources
{{Book title, split into N chapter markdowns in raw/markdown/}}

## Future sources likely to come
{{Customer calls, PRDs, podcasts, other books — whatever the user plans to layer on}}

## Open questions (user can fill in)
- {{Question 1}}
- {{Question 2}}
~~~

---

## index.md skeleton

~~~markdown
# Index

Content-oriented catalog of every page in the wiki. Updated on every ingest.

## Sources
*(empty — first ingest pending)*

## Frameworks
*(empty)*

## Concepts
*(empty)*

## Metrics
*(empty)*

## Roles
*(empty)*
~~~

---

## log.md skeleton

~~~markdown
# Log

Chronological, append-only record of work done in this vault.

Format: `## [YYYY-MM-DD] {ingest | query | lint} | {subject}`
Parseable via: `grep "^## \[" log.md | tail -N`

---
~~~

---

## Key gotchas (learned the hard way)

### 1. Title Case vs kebab-case for wiki filenames

**Problem:** Using `[[Page Name]]` links with kebab-case filenames (`page-name.md`) creates **ghost nodes** in Obsidian graph view — you see two circles per entity, one the real file and one the unresolved link.

**Fix:** Use Title Case filenames with spaces for framework/concept/metric/role pages. Keep kebab-case **only** for source pages (because they mirror `raw/markdown/` which uses slugs).

```
✅ wiki/concepts/Risk Shift.md       ← linked as [[Risk Shift]]
❌ wiki/concepts/risk-shift.md       ← linked as [[Risk Shift]] → ghost node
```

### 2. Firecrawl can't accept local files

**Problem:** Firecrawl requires a publicly-reachable URL. `file://` doesn't work. Multipart uploads return `BAD_REQUEST`.

**Fix:** For one-time book ingestion, use `pdftotext -layout`. The raw markdown doesn't need to be beautiful because the real structure is built in the wiki pages.

If you must use Firecrawl: install `cloudflared`, spin up a local HTTP server for `raw/pdf-chapters/`, tunnel it, feed the tunneled URLs to Firecrawl. Tear down after extraction.

### 3. Forward stubs prevent ghost nodes AND document the gap

When Chapter 5 references "Operating Model" (defined in Chapter 8), don't skip the link. Create a stub:

~~~markdown
# Operating Model

*Stub — referenced in Ch 5 as Layer 4 of the stack. Full treatment in Ch 8. Expand on that ingestion.*

## Role in the stack
...

## Open items
- [ ] Full framework from Ch 8
- [ ] Metrics owned
~~~

When Chapter 8 gets ingested, the stub is promoted to a full page. Meanwhile the graph stays clean.

### 4. Don't over-ingest

Karpathy's rule: **10–15 wiki pages per source.** If Claude is producing 30+ per chapter, the type system is too loose — bullet points are being elevated to full pages. Tighten the CLAUDE.md rules. Aim for one page per *first-class entity* (something that will be referenced 3+ times across chapters), not one page per noun.

### 5. Pilot before scaling

Ingest ONE chapter, review in Obsidian, tune CLAUDE.md, THEN scale. Scaling with a broken schema means every chapter's pages need fixing. Scaling with a tuned schema means the rest is mechanical.

### 6. Source-page naming is special

Source pages use `NN-slug.md` (kebab-case with chapter number) because they mirror `raw/markdown/NN-slug.md`. The chapter number keeps the file list sorted. Linked as `[[NN-slug]]`, not `[[Chapter Title]]`.

Everything **else** uses Title Case. This split is intentional — source pages are ordered/numbered artifacts; entity pages are semantic artifacts.

### 7. Promote stubs when new chapters arrive

Each new chapter's ingest should include a step: *"Check existing stubs; promote any that now have enough content."* Otherwise the wiki accumulates stubs that never get filled in.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Graph view shows paired dark + light circles | Kebab-case filenames with Title Case links | Rename files to Title Case |
| Some nodes are completely disconnected orphans | No inbound links AND the page wasn't added to index.md | Add to index.md; add cross-references from related pages |
| Wiki pages read like marketing copy | CLAUDE.md tone rules too soft | Tighten "Tone and style" — explicitly forbid words like "unlock", "supercharge", "powerful" |
| Every chapter produces 30+ wiki pages | Type system too loose; bullets being elevated | Tighten criteria: an entity gets its own page ONLY if it's referenced 3+ times across chapters OR has a formula OR has a named definition |
| Claude creates duplicate pages for the same concept | Grep step skipped | Add an explicit "grep wiki for the concept name + variants before creating" to the ingest workflow |
| Running headers clutter every wiki page | Stripping step skipped | Add explicit regex examples in CLAUDE.md ingest workflow (e.g. `^CHAPTER \d+ \| .+ \d+$`) |
| Stub pages never get promoted | No trigger in the ingest workflow | Add "Check and promote stubs" as step 0 of each new chapter's ingest |

---

## Quick-start prompt (paste into Claude Code)

If you've cloned this repo and want to start fresh on a new book:

```
I want to build an Obsidian LLM Wiki from this PDF: <path-to-pdf>
I'm a <role> and this book is <why it matters to you>.
I plan to layer <other material types> onto this framework over time.

Follow the SKILL.md at ~/.claude/skills/obsidian-LLM-wiki/SKILL.md step by step.
Start with Step 1 (inspect the PDF). Don't scaffold or split anything until
we've verified text-extractability and the ToC.
```

Claude Code should then:
1. Inspect the PDF (Step 1) and report findings
2. Propose a vault location and split plan
3. Wait for your approval before creating files
4. Scaffold, split, extract, write schema
5. Pilot-ingest one chapter
6. Wait for you to review in Obsidian
7. Tune CLAUDE.md based on feedback
8. Scale to the rest

---

## Attribution

- Pattern: [Andrej Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Obsidian: [obsidian.md](https://obsidian.md)
- Refined via real-world ingestion of a 518-page framework book (Revenue Architecture by Jacco van der Kooij). Resulted in 127 interlinked wiki pages across 16 chapters.
