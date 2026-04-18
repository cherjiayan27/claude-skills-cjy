# obsidian-LLM-wiki

A step-by-step skill for turning any book or dense reference PDF into a compounding Obsidian knowledge base. Implements [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — instead of RAG re-reading raw documents on every query, the LLM incrementally builds and maintains a persistent wiki that compounds over time.

**Covers:** PDF inspection → vault scaffold → local chapter splitting → markdown extraction → schema authoring → pilot ingestion → quality review → full-book ingestion → ongoing compounding phase.

**The 10 steps:**

| Step | What it does |
|---|---|
| 1. Inspect PDF | Verify text-extractability, locate ToC, confirm page-number alignment |
| 2. Scaffold vault | Create `raw/` + `wiki/` directory tree; drop in CLAUDE.md, Memory.md, index.md, log.md |
| 3. Split PDF | Use qpdf to split source PDF into chapter PDFs by ToC page ranges |
| 4. Extract markdown | `pdftotext -layout` per chapter into `raw/markdown/` |
| 5. Author CLAUDE.md | Customise the schema template (page types, naming conventions, workflows) |
| 6. Seed Memory.md | 15–20 min user-context distillation |
| 7. Pilot ingest | Ingest ONE foundational chapter; review in Obsidian |
| 8. Review + tune | Adjust CLAUDE.md based on pilot quality before scaling |
| 9. Batch ingest | Process remaining chapters; forward-stub concepts mentioned but not yet defined |
| 10. Compounding | Layer your own material (calls, PRDs, podcasts) onto the framework lattice |

**Key gotchas documented:**
- Title Case filenames for wiki pages (kebab-case creates ghost nodes in graph view)
- Firecrawl can't accept local files — use `pdftotext` for one-time book ingestion
- Forward-stub pattern prevents ghost nodes + documents the gap
- 10–15 pages per chapter is Karpathy's sweet spot — tighten the schema if Claude overproduces

Includes full CLAUDE.md schema template, Memory.md template, index.md and log.md skeletons, and a copy-paste quick-start prompt. Refined via real-world ingestion of a 518-page framework book (127 interlinked wiki pages across 16 chapters).

---

## CANONICAL-BRAIN.md — multi-source extension

Once you have more than one source, use [CANONICAL-BRAIN.md](CANONICAL-BRAIN.md). It covers the architecture for holding multiple books + reports in a single vault without them polluting each other.

**Core rule — canonical vs personal separation:**

| | Canonical | Personal |
|---|---|---|
| What | Books, reports, whitepapers | PRDs, decisions, ideas, meeting notes |
| Lives in | `~/Obsidian/brain/` | Anywhere else |
| Write access | Read-only; updated via explicit ingest only | User writes freely |

**Vault structure:**
```
~/Obsidian/brain/
├── books/<book-slug>/          ← frameworks, concepts, metrics, roles
└── reports/<report-slug-YYYY>/ ← concepts, metrics, benchmarks (year-versioned)
```

**The 4 cross-source disciplines:**
1. **Grep-before-create** — search the entire vault before making a new page; extend existing pages rather than duplicating
2. **Filename uniqueness across the vault** — `[[Net Revenue Retention]]` resolves regardless of which folder you're in
3. **Extend, don't duplicate** — when a new source confirms an existing concept, update that page's Sources section
4. **Disambiguate collisions** — if two sources use the same word differently, decide: merge into one page or split with distinguishing titles + cross-links

**Sub-agent integration:** scope the brain reference to the specific agent (e.g. `pm-agent.md`), not the user-level `~/.claude/CLAUDE.md` — otherwise it fires on every Claude Code session regardless of topic.

Includes 3 copy-paste quick-start prompts: adding a new source, registering the brain with a sub-agent, and starting a PM conversation grounded in the brain.
