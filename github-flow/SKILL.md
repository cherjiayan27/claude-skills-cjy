# GitHub Flow — End-to-End

A comprehensive GitHub workflow skill combining best practices from gstack, everything-claude-code, repomix, superpowers, and claude-skills into a single end-to-end flow.

**Covers:** Smart commits → pre-push review → push + PR creation → multi-perspective PR review → address feedback → CI + merge → post-deploy verification → branch cleanup.

---

## When to Use

- After completing implementation work and ready to ship
- When user says: "commit", "push", "ship it", "create a PR", "review this", "merge", "deploy", "clean up branches"
- Individual phases can be invoked standalone — you don't have to run the full flow every time

---

## Phase 1: Smart Commit

> Sources: repomix `contextual-commit`, ECC `/prp-commit`, claude-skills `/cm`

### 1a. Review changes before staging

Never use `git add .` or `git add -A`. Review each file individually.

```bash
git status --short
git diff -- <file>           # Review each file for secrets, debug code, unintended changes
```

### 1b. Stage with intent

Stage files intentionally. Support natural language targeting if user specifies:

| User says | Action |
|---|---|
| (nothing / all) | Stage all modified files individually |
| `staged` | Use pre-staged files as-is |
| `*.ts` (glob) | `git add '*.ts'` |
| `except tests` | Stage all, then `git reset -- '**/*.test.*' '**/*.spec.*'` |
| `only new files` | `git ls-files --others --exclude-standard \| xargs git add` |
| `the auth changes` | Parse status/diff, find relevant files, stage them |

```bash
git add <file1> <file2> ...
git diff --cached --stat     # Verify what's staged
```

### 1c. Write contextual commit message

Use Conventional Commits format with contextual action lines in the body.

**Subject line** (standard):
```
<type>(<scope>): <description>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

Rules:
- Imperative mood ("add" not "added")
- Lowercase after type prefix
- No period at end
- Max 72 characters
- Describe WHAT, not HOW

**Body** — add action lines ONLY when they carry signal beyond the diff:

```
intent(scope): what user wanted to achieve and why
decision(scope): chosen approach when alternatives existed
rejected(scope): considered but discarded + reason (highest value — always include WHY)
constraint(scope): hard limits that shaped the approach
learned(scope): something discovered that saves time later
```

**When you lack conversation context** (prior session work, another agent's changes):
- CAN infer: `decision()` if visible in diff (new dependency, pattern adopted)
- CANNOT infer: `intent()`, `rejected()`, `constraint()`, `learned()` — don't fabricate
- A clean subject with no action lines is better than fabricated context

**Commit:**
```bash
git commit -m "$(cat <<'EOF'
feat(auth): add Google OAuth login

intent(auth): social login starting with Google, then GitHub and Apple
decision(oauth-library): passport.js over auth0-sdk for multi-provider flexibility
rejected(oauth-library): auth0-sdk — locks into their session model, incompatible with redis store
constraint(google-oauth): requires verified domain for production credentials
learned(passport-google): requires explicit offline_access scope for refresh tokens
EOF
)"
```

---

## Phase 2: Pre-Push Review

> Sources: gstack `/review`, ECC `/review-pr` agents, superpowers `requesting-code-review`

Run a local code review BEFORE pushing. This catches issues early and cheaply.

### 2a. Get the diff scope

```bash
git fetch origin <base-branch> --quiet
git diff origin/<base-branch>...HEAD --stat
git log origin/<base-branch>..HEAD --oneline
```

### 2b. Dispatch specialist reviewer agents in parallel

Launch reviewers based on diff size and scope. Each agent reviews the diff and returns findings as structured output.

**Always-on reviewers** (any PR):
- **code-quality** — bugs, logic errors, edge cases, type safety
- **test-coverage** — missing tests, mock correctness, coverage gaps

**Conditional reviewers** (based on what changed):
- **security** — if auth files changed OR diff > 100 lines (injection, path traversal, secrets, SSRF)
- **performance** — if backend/frontend files changed (N+1 queries, bundle size, memory leaks)
- **conventions** — if > 5 files changed (naming, directory structure, commit format)
- **holistic** — if > 200 lines changed (architectural fit, cross-cutting concerns)

### 2c. Triage findings

Each finding must include: `severity` (CRITICAL / IMPORTANT / ADVISORY), `confidence` (1-10), `file`, `line`, `summary`, `suggested fix`.

Apply confidence gates:
- **7-10**: Show to user
- **5-6**: Show with caveat ("low confidence")
- **3-4**: Appendix only
- **1-2**: Suppress

Classify fixes:
- **AUTO-FIX**: Dead code, stale comments, obvious N+1, missing null checks → fix directly
- **ASK**: Design decisions, architectural changes, ambiguous trade-offs → present to user

If auto-fixes applied: commit them, then re-run tests before proceeding.

### 2d. Tests must pass

```bash
# Run the project test suite
npm test                     # or: cargo test, pytest, go test ./...
```

If tests fail: STOP. Fix failures before proceeding. Never push with failing tests.

---

## Phase 3: Push + Create PR

> Sources: gstack `/ship`, ECC `/prp-pr`, superpowers `finishing-a-development-branch`

### 3a. Pre-flight checks

```bash
CURRENT_BRANCH=$(git branch --show-current)
BASE_BRANCH="main"  # or detect from repo config

# Must not be on base branch
if [ "$CURRENT_BRANCH" = "$BASE_BRANCH" ]; then
  echo "ERROR: Cannot ship from $BASE_BRANCH. Switch to a feature branch."
  exit 1
fi

# Must have commits ahead
git log origin/$BASE_BRANCH..HEAD --oneline  # Must not be empty

# Must have no uncommitted changes
git status --short  # Must be empty (or warn user to commit/stash)

# Check for existing PR
gh pr list --head "$CURRENT_BRANCH" --json number  # Must be empty
```

### 3b. Merge latest base branch

```bash
git fetch origin $BASE_BRANCH
git merge origin/$BASE_BRANCH --no-edit
```

If conflicts: resolve them, run tests, commit the merge.

### 3c. Push

```bash
git push -u origin $CURRENT_BRANCH
```

If push fails due to divergence:
```bash
git fetch origin
git rebase origin/$BASE_BRANCH
git push -u origin HEAD
```

If rebase conflicts: stop and inform user.

**Rule:** Never use `git push --force`. Only `--force-with-lease` as last resort.

### 3d. Discover PR template

Search in order:
1. `.github/PULL_REQUEST_TEMPLATE/default.md`
2. `.github/PULL_REQUEST_TEMPLATE.md`
3. `.github/pull_request_template.md`
4. `docs/pull_request_template.md`

If found: fill in each section. Preserve all template sections — use "N/A" if not applicable.

### 3e. Analyze commits for PR title and body

```bash
git log origin/$BASE_BRANCH..HEAD --format="%h %s" --reverse
git diff origin/$BASE_BRANCH..HEAD --stat
git diff origin/$BASE_BRANCH..HEAD --name-only
```

- **PR title**: Use conventional commit format. Single commit → use its message. Multiple → dominant type prefix + summary.
- **PR body** (if no template found):

```markdown
## Summary
<1-3 sentence description of what this PR does and why>

## Changes
- <bulleted list grouped by area: backend, frontend, tests, config>

## Test plan
- [ ] Unit tests pass
- [ ] E2E tests pass (if applicable)
- [ ] Manual verification

## Related issues
<Closes #N, or Linear ticket ID, or "None">
```

### 3f. Create PR as draft

```bash
gh pr create \
  --draft \
  --title "<conventional commit title>" \
  --base "$BASE_BRANCH" \
  --body "<generated body>"
```

Creating as draft gives time for review before CI triggers (if CI is configured to skip drafts).

---

## Phase 4: Multi-Perspective PR Review

> Sources: repomix `pr-review`, gstack review army, ECC `/review-pr`

### 4a. Evaluate existing bot comments first

Before spawning reviewers, check for existing AI bot comments (CodeRabbit, Gemini, etc.):

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

- Skip Claude's own comments (avoid self-reply)
- Skip if Claude already replied (check `in_reply_to_id`)
- Classify each bot comment:
  - **Required**: Security, bugs, crashes → must address
  - **Recommended**: Code quality, best practices → should address
  - **Not needed**: Style, false positives, already addressed → skip

Reply with priority classification:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments/{id}/replies \
  -f body="\`Priority: Required\` — This is a valid security concern. Addressing."
```

### 4b. Spawn 6 parallel reviewer agents

Each agent reviews the full PR diff independently:

1. **reviewer-code-quality** — bugs, logic errors, edge cases, type safety
2. **reviewer-security** — injection, path traversal, secret exposure, SSRF
3. **reviewer-performance** — algorithmic complexity, resource leaks, memory, I/O
4. **reviewer-test-coverage** — test quality, mock correctness, coverage gaps
5. **reviewer-conventions** — style, naming, directory structure, commit format
6. **reviewer-holistic** — architectural fit, cross-cutting concerns, user impact

### 4c. Consolidate and deduplicate

- Merge findings across all reviewers
- Deduplicate by fingerprint (file + line + category)
- Keep highest confidence per fingerprint
- Boost confidence +1 for findings confirmed by multiple specialists

### 4d. Post review comments

- **Inline comments** for specific code issues (use GitHub inline comment API)
- **Overall summary** as a PR comment with findings grouped by severity:
  - **Critical** (bugs, security, data loss)
  - **Important** (missing tests, quality problems)
  - **Advisory** (suggestions — only when explicitly requested)
- Only report findings with confidence >= 7 (80%)
- Wrap detailed feedback in `<details><summary>` tags

### 4e. Adversarial review (for large PRs)

If diff > 200 lines OR any CRITICAL finding: dispatch an adversarial reviewer that specifically tries to find what the other reviewers missed — failure modes, race conditions, edge cases.

---

## Phase 5: Address PR Feedback

> Sources: repomix `pr-address-feedback`, superpowers `receiving-code-review`

### 5a. Fetch all feedback

Run in parallel:
```bash
gh pr diff {pr_number}
gh pr view --comments
```

Plus GraphQL for full thread context with pagination:
```bash
gh api graphql -f query='
  query($pr: Int!, $owner: String!, $repo: String!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100) {
          nodes {
            id, isResolved, isOutdated, comments(first: 50) {
              nodes { id, body, author { login }, path, line }
            }
          }
        }
      }
    }
  }'
```

### 5b. Classify feedback

Skip already resolved or minimized threads.

**For human feedback:**

| Category | Condition | Action |
|---|---|---|
| **Fix** | Clear defects, bugs, security, logic errors | Must fix code |
| **Improve** | Valid quality suggestions | Fix unless conflicts with conventions |
| **Discuss** | Ambiguous, design disagreements, scope questions | Present to user — don't act yet |
| **Skip** | Already addressed, out of scope, false positives | Reply with reason + resolve |

**When uncertain between Improve and Discuss → choose Discuss** (safer).

**For bot feedback:**

| Condition | Action |
|---|---|
| `isOutdated: true` or code changed/removed | Reply + resolve + minimize with `OUTDATED` |
| Newer version from same bot exists | Minimize with `OUTDATED` |
| Still relevant | Leave untouched |

### 5c. Present plan before acting

Show summary table:

| # | Type | Category | File | Comment (summary) | Planned Action |
|---|---|---|---|---|---|

Wait for user confirmation before proceeding.

### 5d. Apply code fixes

For **Fix** and **Improve** items:
- Read relevant file + context
- Apply minimal change addressing feedback
- Do NOT refactor surrounding code
- Do NOT modify files outside the PR diff

### 5e. Handle feedback with technical rigor

From superpowers — when receiving review feedback:

**Forbidden responses:**
- "You're absolutely right!"
- "Great point!"
- "Excellent feedback!"

**Instead:**
- Restate the technical requirement
- Verify against codebase reality before implementing
- Push back with technical reasoning if reviewer is wrong
- YAGNI check: if suggested feature is unused, flag for removal

### 5f. Verify

```bash
npm run lint
npm run test
```

Retry up to 3 times if tests fail. If still failing after 3 attempts: stop, present errors to user.

### 5g. Commit and push

```bash
git add <changed files>
git commit -m "fix(<scope>): address PR review feedback

- <brief list of what was addressed>"
git push
```

**Rule:** Always push BEFORE resolving threads.

### 5h. Reply and resolve threads

After push is confirmed:

**Addressed comments (Fix/Improve):**
```
Addressed in `<commit_sha>` — <brief description>. 🤖
```
Then resolve thread.

**Skipped comments:**
```
No action needed — <brief explanation>. 🤖
```
Then resolve thread.

**Outdated bot threads:**
```
No longer applicable — the referenced code has been updated. 🤖
```
Then resolve + minimize with `OUTDATED`.

**API for resolving:**
```bash
# Reply to thread
gh api graphql -f query='mutation {
  addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: "PRRT_xxx", body: "REPLY"}) {
    comment { id }
  }
}'

# Resolve thread
gh api graphql -f query='mutation {
  resolveReviewThread(input: {threadId: "PRRT_xxx"}) {
    thread { isResolved }
  }
}'

# Minimize outdated bot comments
gh api graphql -f query='mutation {
  minimizeComment(input: {subjectId: "ID_xxx", classifier: OUTDATED}) {
    minimizedComment { isMinimized }
  }
}'
```

### 5i. Final report to user

| Section | Content |
|---|---|
| **Addressed** | Table of code changes + commit SHA |
| **No action needed** | Table of skipped items + reason |
| **Needs your input** | Discuss items — user chooses: Address / Skip / Leave |

---

## Phase 6: CI + Merge

> Sources: gstack `/land-and-deploy`, claude-skills `/cp`

### 6a. Mark PR ready (triggers CI)

```bash
gh pr ready
```

### 6b. Wait for CI

```bash
gh pr checks --watch --fail-fast    # 15-minute timeout
```

**If CI fails:**
1. Read the failure log: `gh run view <run-id> --log-failed`
2. Determine: was the behavior change intentional?
   - **Yes** → update tests to match new behavior
   - **No** → fix code, not the test
3. Commit fix → push → CI re-runs automatically
4. Repeat until green

**Rules:**
- Never delete or skip a failing test to make CI pass
- Never use `--no-verify` to bypass hooks

### 6c. Merge

```bash
gh pr merge --squash --delete-branch
```

Squash merge keeps `main` history clean. The contextual commit body from Phase 1 is preserved in the squash commit.

If merge queue is enabled: poll PR state every 30 seconds (30-min timeout):
```bash
gh pr view --json state,mergeStateStatus
```

---

## Phase 7: Post-Deploy Verification

> Source: gstack `/canary`

After merge triggers auto-deploy (Railway, Vercel, Fly.io, etc.):

### 7a. Wait for deploy

Detect platform and poll:
- **Vercel/Netlify**: Check deployment status via dashboard or CLI
- **Railway/Fly.io/Render**: Check via platform CLI
- **GitHub Actions deploy**: `gh run list --workflow deploy.yml --limit 1`

### 7b. Canary check

Visit the production URL and verify:

1. **Page loads** — no 500 errors, no blank screens
2. **Console errors** — no new JavaScript errors
3. **Performance** — page load time within acceptable range (not >2x baseline)
4. **Content** — key content renders correctly (not error pages)

If any CRITICAL issue found: consider reverting:
```bash
git revert <merge-commit-sha> --no-edit
git push origin main
```

### 7c. Verify on mobile

If frontend changes: test on mobile viewport sizes (360px–430px) in addition to desktop.

---

## Phase 8: Cleanup

> Sources: claude-skills `/clean`, superpowers worktree cleanup

### 8a. Clean up branches

```bash
# Delete local merged branches (keep main, dev, gh-pages)
git branch --merged main | grep -v -E '^\*|main$|dev$|gh-pages$' | xargs -n 1 git branch -d

# Prune stale remote references
git fetch -p

# Delete remote branches (confirm with user first)
git branch -r --merged origin/main | grep -v -E 'origin/main$|origin/dev$|origin/gh-pages$|origin/HEAD' 
# Show list → get confirmation → then delete:
git push origin --delete <branch-name>
```

### 8b. Clean up worktrees (if used)

```bash
git worktree list
git worktree remove <path>   # For merged/discarded branches
```

### 8c. Report final state

```bash
git branch          # Local branches
git branch -r       # Remote branches
```

Present summary:
| Item | Count |
|---|---|
| Local branches deleted | N |
| Remote branches deleted | N |
| Remaining | main, dev |

---

## Quick Reference: The 8 Phases

```
1. COMMIT       → Review diffs → stage with intent → contextual commit message
2. PRE-REVIEW   → Specialist agents → confidence scoring → auto-fix or ask
3. PUSH+PR      → Pre-flight → merge base → push → discover template → create draft PR
4. PR REVIEW    → Evaluate bot comments → 6 parallel reviewers → adversarial review
5. FEEDBACK     → Fetch all → classify → fix → verify → push → resolve threads
6. CI+MERGE     → Mark ready → wait for CI → squash merge
7. POST-DEPLOY  → Wait for deploy → canary check → mobile verify
8. CLEANUP      → Delete merged branches → prune remotes → remove worktrees
```

---

## Phase Selection Guide

Not every change needs every phase. Use this to decide which phases to run:

| Change Type | Phases to Run |
|---|---|
| **Typo fix / 1-line change** | 1 → 3 → 6 |
| **Bug fix (small)** | 1 → 2 (code-quality only) → 3 → 6 |
| **New feature** | 1 → 2 → 3 → 4 → 5 (if feedback) → 6 → 7 |
| **Large refactor** | 1 → 2 (all reviewers) → 3 → 4 (+ adversarial) → 5 → 6 → 7 |
| **Hotfix (production down)** | 1 → 3 (skip draft, create ready PR) → 6 → 7 |
| **Post-merge cleanup** | 8 |

---

## Attribution

Combined from the best practices of:
- **gstack** — `/ship`, `/land-and-deploy`, `/canary`, `/review` (by Garry Tan)
- **everything-claude-code** — PRP workflow, `/prp-commit`, `/prp-pr`, `/review-pr`
- **repomix** — `contextual-commit`, `pr-review`, `pr-address-feedback`
- **superpowers** — `finishing-a-development-branch`, `requesting-code-review`, `receiving-code-review` (by Jesse Vincent)
- **claude-skills** — `/cm`, `/cp`, `/pr`, `/clean`, `git-workflow-standards`
