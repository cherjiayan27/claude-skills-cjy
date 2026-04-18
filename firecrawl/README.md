# Firecrawl Installation for Claude Code

Step-by-step record of installing the Firecrawl CLI skill on macOS and wiring it up to Claude Code so that it runs successfully end-to-end.

## What Firecrawl is

Firecrawl is a CLI that gives an agent real-time web access: search, scrape, map, crawl, interact. It returns clean markdown optimized for LLM context windows. Installed as a **Claude Code skill** (not an MCP server) — the skill provides the SKILL.md that Claude loads, and the `firecrawl` CLI binary does the actual work.

## Prerequisites

- macOS with zsh
- Node.js + npm installed (and `npm config get prefix` pointing at a writable dir, e.g. `~/.npm-global`)
- `gh` CLI (only needed if you want to push docs to GitHub afterwards)
- A Firecrawl API key — get one at https://firecrawl.dev

## Step-by-step

### 1. Confirm Firecrawl was not already installed

```bash
claude mcp list
```

Result: only Gmail, Google Drive, Google Calendar, and excalidraw were connected — no Firecrawl. (Note: Firecrawl installs as a *skill*, not an MCP server, so this check was mostly to confirm the starting state.)

### 2. Install the Firecrawl skill (SKILL.md + rules)

The skill docs live in a skill package hosted at `github.com/firecrawl/cli`. Install it globally, scoped to Claude Code only:

```bash
npx -y skills@1.5.1 add https://github.com/firecrawl/cli \
  --skill firecrawl \
  --agent claude-code \
  --global -y
```

Flags explained:
- `--skill firecrawl` — only install the `firecrawl` skill from that repo (the repo contains 8 skills total)
- `--agent claude-code` — only install for Claude Code, not every agent the tool knows about
- `--global` — install to `~/.claude/skills/` (user-level) rather than a project dir
- `-y` — skip interactive confirmation prompts

Result: skill copied to `~/.claude/skills/firecrawl/` (contains `SKILL.md` + a `rules/` dir).

### 3. Install the `firecrawl` CLI binary

The skill in step 2 only installs the instructions Claude reads. The actual CLI binary is a separate npm package:

```bash
npm install -g firecrawl-cli@1.14.8
```

This places the binary at `<npm-prefix>/bin/firecrawl`. On this machine: `/Users/cherjiayan/.npm-global/bin/firecrawl`.

### 4. Set the API key so Claude Code subprocesses can see it

Firecrawl reads `FIRECRAWL_API_KEY` from the environment. Add it to `~/.claude/settings.json` under a top-level `env` block so it's exposed to every Bash tool call Claude Code runs — and only to Claude Code (not every shell on the system).

Edit `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "linear": {
      "type": "http",
      "url": "https://mcp.linear.app/mcp"
    }
  },
  "env": {
    "FIRECRAWL_API_KEY": "fc-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

Replace the placeholder with your real key from firecrawl.dev.

Alternative locations considered (and rejected here):
- `~/.zshrc` → persistent but exposes the key to every process on the system
- Session-only `export` → lost on shell close, not useful for a persistent setup

### 5. Add npm global bin to PATH

After step 3, `firecrawl` existed at `~/.npm-global/bin/firecrawl` but `which firecrawl` said "not found" — the directory wasn't on PATH. Fix by appending one line to `~/.zshrc`:

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

Then reload the shell (or open a new terminal):

```bash
source ~/.zshrc
```

This is a one-time, system-wide fix — any future globally-installed npm CLI will also Just Work.

### 6. Verify

Two commands confirm install + auth + output pipeline are all healthy:

```bash
firecrawl --status
```

Expected output (values will differ):

```
  🔥 firecrawl cli v1.14.8

  ● Authenticated via FIRECRAWL_API_KEY
  Concurrency: 0/2 jobs (parallel scrape limit)
  Credits: 1,025 / 500 (205% left this cycle)
```

Then do a real request:

```bash
mkdir -p .firecrawl
firecrawl scrape "https://firecrawl.dev" -o .firecrawl/install-check.md
```

If both succeed, install is healthy.

## Gotchas hit during this install

1. **Interactive prompt on first `skills add` run.** Running without `--agent` and `-y` drops you into a TUI for picking which agents to install to. Use the flags shown in step 2 to avoid it.
2. **Skill install ≠ CLI install.** The `skills add` step only lays down SKILL.md. You still need `npm install -g firecrawl-cli` for the binary.
3. **npm global bin not on PATH.** `npm config get prefix` was `~/.npm-global`, but nothing had ever added `~/.npm-global/bin` to PATH, so `firecrawl` appeared "not installed" after step 3.
4. **settings.json `env` block is the right home for the API key** when the tool is only used from Claude Code — keeps the key off the global shell environment.

## Final state

- Skill: `~/.claude/skills/firecrawl/` (SKILL.md + rules/)
- Binary: `~/.npm-global/bin/firecrawl` (v1.14.8)
- Auth: `FIRECRAWL_API_KEY` set in `~/.claude/settings.json`
- PATH: `~/.npm-global/bin` prepended in `~/.zshrc`
- Verified with: `firecrawl --status` → `Authenticated via FIRECRAWL_API_KEY`
