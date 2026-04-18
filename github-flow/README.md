# github-flow

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

Not every change needs all 8 phases — a typo fix only needs phases 1, 3, 6. See the Phase Selection Guide in `SKILL.md` for details.
