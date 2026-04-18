# Claude Skills — CJY

Personal collection of [Claude Code](https://claude.ai/claude-code) skills, portable across machines.

---

## Skills

### github-flow

End-to-end GitHub workflow skill — 8 phases from commit to cleanup, sourced from 5 skill libraries. See [github-flow/README.md](github-flow/README.md).

### prd-writing

Spec-Driven Development (SDD) skill — 5 phases from problem validation to approved spec, with traceable FR/NFR/EC IDs. See [prd-writing/README.md](prd-writing/README.md).

### obsidian-LLM-wiki

Karpathy LLM Wiki pattern — turn any PDF into a compounding Obsidian knowledge base, 10 steps from inspection to batch ingest. See [obsidian-LLM-wiki/README.md](obsidian-LLM-wiki/README.md).

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
