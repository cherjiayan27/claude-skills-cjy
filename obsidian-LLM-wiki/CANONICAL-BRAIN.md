# Canonical Brain — Multi-Source Infrastructure

Companion doc to [SKILL.md](SKILL.md). SKILL.md is the generic one-source ingestion recipe. This doc covers the **multi-source canonical-brain architecture** — how to organise `~/Obsidian/brain/` to hold multiple books + reports without them polluting each other or getting polluted by personal thinking.

Use SKILL.md when ingesting a new source. Use this doc to understand where the source goes, how it cross-links, and how sub-agents like pm-agent consult the whole brain.

---

## The core rule

**Canonical vs Personal separation.** The brain is *only* for fundamental truths from published, validated external sources. Your own thinking lives elsewhere.

| | Canonical | Personal |
|---|---|---|
| **What** | Books, reports, whitepapers, research papers | PRDs, decisions, ideas, customer calls, meeting notes |
| **Lifetime** | Stable | Messy, evolving |
| **Lives in** | `~/Obsidian/brain/` | **Anywhere else** — project folders, Linear, Notion, separate vault |
| **Write access** | Read-only; only updated via explicit ingest | User writes freely |

**Cross-direction linking:**
- Personal → Canonical: ✅ free. A PRD can cite `[[Net Revenue Retention]]`.
- Canonical → Personal: ❌ forbidden. The brain never references your PRDs.

**Why the separation:** if personal thinking pollutes the brain, every query returns mixed-signal answers. The brain's value is pristine, citable signal.

---

## Folder structure

```
~/Obsidian/brain/                              ← single Obsidian vault
├── CLAUDE.md                                  ← schema, canonical-only rules
├── Memory.md                                  ← about the brain (not the user)
├── _index.md                                  ← cross-source catalog
├── _log.md                                    ← top-level activity
│
├── books/
│   ├── <book-slug>/
│   │   ├── index.md       ← book-scoped catalog
│   │   ├── log.md         ← book-scoped ingest history
│   │   ├── raw/           ← immutable sources
│   │   │   ├── pdf/
│   │   │   ├── pdf-chapters/
│   │   │   └── markdown/
│   │   └── wiki/          ← AI-generated
│   │       ├── sources/
│   │       ├── frameworks/
│   │       ├── concepts/
│   │       ├── metrics/
│   │       └── roles/
│   └── <another-book-slug>/
│
└── reports/
    └── <report-slug-YYYY>/      ← year-versioned for annuals
        ├── index.md
        ├── log.md
        ├── raw/
        └── wiki/
            ├── sources/
            ├── concepts/
            ├── metrics/
            └── benchmarks/      ← per-industry data pages (reports only)
```

---

## Books vs Reports

| | Books | Reports |
|---|---|---|
| Lifetime | Stable across years | Often year-versioned |
| Content | Frameworks, concepts, roles | Data, benchmarks, industry snapshots |
| Re-ingest | Rarely | Periodically on new editions |
| Wiki subdirs | sources, frameworks, concepts, metrics, roles | sources, concepts, metrics, **benchmarks** |

Reports get `benchmarks/` for per-industry or per-segment data pages (e.g. "B2B SaaS Retention Benchmark"). Books typically don't need this.

**Naming convention:** year-stamp report folders (`reports/amplitude-product-benchmark-2025/`) so old and new editions sit side-by-side for delta tracking when the next edition drops.

---

## Adding a new source to an existing brain

Follow SKILL.md's 10 steps, but with the multi-source discipline layered on:

1. **Decide book or report.** Use the table above.
2. **Pick the slug.** `<type>/<name>` or `<type>/<name>-YYYY` for reports.
3. **Scaffold just the new source folder** — don't touch existing sources or root files:
   ```bash
   mkdir -p ~/Obsidian/brain/<type>/<slug>/{raw/{pdf,pdf-chapters,markdown},wiki/{sources,frameworks,concepts,metrics,roles,benchmarks,assets}}
   cp <path-to-pdf> ~/Obsidian/brain/<type>/<slug>/raw/pdf/
   ```
   (Omit `roles` for reports; omit `benchmarks` for books.)
4. **Run Steps 1–9 from SKILL.md** on the new source.
5. **Cross-source discipline during ingestion** — see next section.
6. **Update root `_index.md`** with the new source entry and any new convergence points.
7. **Append a root `_log.md` entry** for "source-added" + "ingest-complete".

---

## Cross-source linking (the critical discipline)

When ingesting a new source into a brain that already has sources, the #1 risk is **creating parallel pages for concepts that already exist**. Rules:

### Filename uniqueness across the entire vault

`Net Revenue Retention.md` exists only once. It sits wherever the concept was *first introduced* (typically the book's `wiki/metrics/`). Other sources link to it via `[[Net Revenue Retention]]`, which resolves regardless of their folder path.

### Grep-before-create (entire vault)

Before creating a new page for a concept the new source mentions:

```bash
grep -r -l "Concept Name" ~/Obsidian/brain/books ~/Obsidian/brain/reports
```

If it exists → extend the existing page. If not → create new.

### Extend, don't duplicate

When a new source confirms or adds data to an existing concept, **update the existing page's Sources section** (and add any new content like empirical benchmarks, worked examples, or extensions). Do NOT create a second page.

Example — pattern from a real ingest:

Before (existing page in books/):
```markdown
# Retention
...
## Sources
- [[02-first-principles]] — §2.3 Principle 2
```

After (same page, extended by a report ingest):
```markdown
# Retention
...

## Revenue retention vs user retention
<new section added by report ingest explaining the distinction>

## Sources
- [[02-first-principles]] — §2.3 Principle 2
- [[05-retention]] — Amplitude Product Benchmark Report 2025 — empirical benchmarks
```

### When two sources genuinely define different concepts with the same name

Sometimes two sources use the same word for different things (e.g. Jacco's "Retention" = revenue retained; Amplitude's "Retention" = users returning). In that case:

- Decide: merge (one page with both angles) or split (rename one)
- If splitting: give the new concept a distinguishing title — `User Retention.md` in addition to `Retention.md`
- Cross-link them in each other's "Related" section with the distinction explicitly noted

Document the decision in the source's `log.md` entry for the ingest.

---

## Sub-agent integration (how pm-agent reads the brain)

**Principle:** scope the brain-reference instruction to the sub-agent that needs it, not to the whole Claude Code machine.

### Recommended: per-agent integration

Most sub-agents have a definition file (e.g. `~/.claude/agents/pm-agent.md`). Add two things:

**1. A main block** (place it near any existing domain-knowledge callout):

```markdown
**Canonical Brain:** Throughout ALL steps, consult `~/Obsidian/brain/` —
the user's canonical knowledge brain — whenever the conversation touches
<topics relevant to this agent>. Read the relevant pages and ground
discussions in those frameworks and empirical benchmarks.
Cite as `(brain: <page-name>)`.

Current brain contents:
- `books/<slug>/` — <book summary>
- `reports/<slug>/` — <report summary>

Start by reading `~/Obsidian/brain/_index.md` to orient, then drill into
relevant pages.

**Hard rules for the brain:**
- Read-only. Never write into `~/Obsidian/brain/`.
- PRDs, decisions, ideas go elsewhere.
- When observation contradicts the brain, surface as a note —
  don't edit the brain.
```

**2. A one-liner in the agent's Global Rules:**

```markdown
N. Ground in the canonical brain. When a topic overlaps
   `~/Obsidian/brain/` contents, read the relevant brain page(s)
   and cite them. Treat the brain as read-only reference.
```

### Avoid: user-level CLAUDE.md

Putting the brain reference in `~/.claude/CLAUDE.md` loads it into every Claude Code session on the machine — including ones that have nothing to do with the brain's domain (debugging TypeScript, writing a shell script, etc.). Scope it to the sub-agent.

---

## What lives inside vs outside the brain

| Inside the brain | Outside the brain |
|---|---|
| `books/revenue-architecture/wiki/metrics/Net Revenue Retention.md` | Your product's PRD |
| `reports/amplitude-product-benchmark-2025/wiki/benchmarks/Retention Benchmarks (All Products).md` | Customer call transcript |
| `books/<next-book>/wiki/frameworks/JTBD.md` | Decision log for killing a feature |
| Published whitepaper ingests | Your roadmap |
| Gartner Magic Quadrant data | Sprint retrospective notes |

**Rule of thumb:** if your opinion of it might change, it doesn't belong in the brain. If it's a fact attributable to a named external source, it might.

---

## Real example: the current state

This pattern was developed by building one ourselves. Current state at time of writing:

```
~/Obsidian/brain/
├── CLAUDE.md          (canonical-only schema)
├── Memory.md          (about the brain)
├── _index.md          (cross-source catalog)
├── _log.md            (top-level activity)
├── books/
│   └── revenue-architecture/    (Jacco, 127 wiki pages)
└── reports/
    └── amplitude-product-benchmark-2025/    (22 wiki pages, 2 metrics extended into existing RA pages)
```

Total: 181 interlinked wiki pages, with cross-source citations where theory (Jacco) meets empirical data (Amplitude). `[[Retention]]` (Jacco) now cites both sources. `[[User Retention]]` is a new page that disambiguates Amplitude's user-activity-based metric from Jacco's revenue-based one. Both resolve from anywhere in the vault.

pm-agent (`~/.claude/agents/pm-agent.md`) is the current sub-agent consumer. When invoked on a PM task, it reads relevant brain pages and cites them as `(brain: <page-name>)`.

---

## Quick-start prompts

### Adding a new source to an existing brain

```
I want to add a new source to my canonical brain at ~/Obsidian/brain/.
New source: <path-to-pdf>
Type: <book or report>

Read ~/.claude/skills/obsidian-LLM-wiki/CANONICAL-BRAIN.md for the
multi-source discipline. Follow SKILL.md for the ingestion steps.

When ingesting, grep the entire existing vault (books AND reports) for
overlapping concepts. Extend existing pages' Sources sections rather
than creating parallels.
```

### Registering the brain with a new sub-agent

```
Add a Canonical Brain reference to ~/.claude/agents/<agent>.md following
the pattern in ~/.claude/skills/obsidian-LLM-wiki/CANONICAL-BRAIN.md
§ "Sub-agent integration".

Current brain contents:
- books/<slug>/
- reports/<slug>/
```

### Starting a PM conversation that should consult the brain

```
<Your PM question>

Ground this in ~/Obsidian/brain/. Read the relevant canonical pages
first. Cite as (brain: <page-name>). Don't file any output into the
brain — that stays in the project.
```

---

## Attribution

- Pattern: [Andrej Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Obsidian: [obsidian.md](https://obsidian.md)
- Extended for multi-source canonical brains + sub-agent integration via real-world use.
