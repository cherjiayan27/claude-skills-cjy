# Claude Skills — CJY

Personal collection of [Claude Code](https://claude.ai/claude-code) skills, portable across machines.

---

## Skills

### github-flow

End-to-end GitHub workflow skill — 8 phases from commit to cleanup, sourced from 5 skill libraries. See [github-flow/README.md](github-flow/README.md).

### prd-writing

A Spec-Driven Development (SDD) skill for product managers. Guides you through writing a formal behavioral specification with traceable requirement IDs before any code is written. Combined from:

- **gstack** `/office-hours` — demand validation, forcing questions, anti-sycophancy
- **everything-claude-code** `/prp-prd` — interactive PRD phases, hypothesis format, codebase grounding
- **superpowers** `brainstorming` — hard gate (no implementation before spec), spec self-review
- **SDD article** (technomanagers.com) — behavioral spec format (FR/NFR/EC with IDs)
- **claude-skills** — MoSCoW priorities, Given-When-Then format

**The 5 phases:**

| Phase | What it does |
|---|---|
| 1. Problem Validation | 5 forcing questions — kill bad ideas early, push back on vague answers |
| 2. Strategic Alignment | Define primary job, out of scope, success metrics, testable hypothesis |
| 3. Behavioral Specification | Draft formal spec with US-XX, FR-XX, NFR-XX, EC-XX — all traceable |
| 4. Codebase Grounding | Validate spec against actual codebase — reuse, new work, risks |
| 5. Approval & Handoff | PM approves → spec saved → next step is `/ticket-writer` for Linear tickets |

**Core principle:** Debug the spec, not the code. When a bug is found downstream, trace it to the FR/EC ID and fix the spec first.

### obsidian-LLM-wiki

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

## Installation

### First time setup

```bash
# 1. Clone this repo
git clone https://github.com/cherjiayan27/claude-skills-cjy.git ~/claude-skills-cjy

# 2. Create the skills directory (if it doesn't exist)
mkdir -p ~/.claude/skills

# 3. Symlink the skills you want
ln -s ~/claude-skills-cjy/github-flow ~/.claude/skills/github-flow
ln -s ~/claude-skills-cjy/prd-writing ~/.claude/skills/prd-writing
ln -s ~/claude-skills-cjy/obsidian-LLM-wiki ~/.claude/skills/obsidian-LLM-wiki
```

### On another machine

Same 3 steps. The symlink makes Claude Code pick up the skill automatically — no config changes needed.

### Updating

```bash
cd ~/claude-skills-cjy
git pull
```

The symlink means updates are picked up immediately — no need to re-copy files.

---

## Adding new skills

Each skill lives in its own folder with a `SKILL.md` file:

```
claude-skills-cjy/
  github-flow/
    SKILL.md
  your-new-skill/
    SKILL.md
```

After adding a new skill folder, symlink it on each machine:

```bash
ln -s ~/claude-skills-cjy/your-new-skill ~/.claude/skills/your-new-skill
```
