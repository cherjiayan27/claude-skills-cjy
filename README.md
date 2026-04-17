# Claude Skills — CJY

Personal collection of [Claude Code](https://claude.ai/claude-code) skills, portable across machines.

---

## Skills

### github-flow

An end-to-end GitHub workflow skill covering the full lifecycle from commit to cleanup. Built by combining the best practices from 5 open-source skill libraries:

- **gstack** — `/ship`, `/land-and-deploy`, `/canary`, `/review`
- **everything-claude-code** — PRP workflow, `/prp-commit`, `/prp-pr`, `/review-pr`
- **repomix** — `contextual-commit`, `pr-review`, `pr-address-feedback`
- **superpowers** — `finishing-a-development-branch`, `requesting-code-review`, `receiving-code-review`
- **claude-skills** — `/cm`, `/cp`, `/pr`, `/clean`, `git-workflow-standards`

**The 8 phases:**

| Phase | What it does |
|---|---|
| 1. Smart Commit | Review diffs, natural language staging, contextual commit messages with intent/decision/rejected lines |
| 2. Pre-Push Review | Specialist agents (security, performance, test-coverage, etc.) with confidence scoring and auto-fix |
| 3. Push + PR | Pre-flight checks, merge base, push, PR template discovery, create draft PR |
| 4. PR Review | Evaluate bot comments, 6 parallel reviewer agents, adversarial review for large PRs |
| 5. Address Feedback | Fetch via GraphQL, classify (Fix/Improve/Discuss/Skip), fix, resolve threads automatically |
| 6. CI + Merge | Mark ready, wait for CI, squash merge, handle merge queue |
| 7. Post-Deploy | Canary checks — page load, console errors, performance, mobile viewports |
| 8. Cleanup | Delete merged branches, prune remotes, remove worktrees |

Not every change needs all 8 phases — a typo fix only needs phases 1, 3, 6. See the Phase Selection Guide in the skill for details.

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
