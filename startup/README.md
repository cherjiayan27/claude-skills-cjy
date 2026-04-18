# New Laptop Startup Guide (macOS)

Step-by-step install list for setting up a fresh macOS laptop for dev + PM work. Run the commands top-to-bottom — later steps depend on earlier ones (e.g. Claude Code needs Node, which needs nvm, which needs Homebrew).

---

## 0. Xcode Command Line Tools (prerequisite)

Required by Homebrew, git, and many build steps.

```bash
xcode-select --install
```

A dialog will pop up — click Install. Wait until it finishes before continuing.

---

## 1. Homebrew

The macOS package manager. Everything else flows from this.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, add Homebrew to PATH (the installer prints the exact command; on Apple Silicon it's):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify:

```bash
brew --version
```

---

## 2. Git

Comes with Xcode CLI tools, but install the newer Homebrew version:

```bash
brew install git
```

Set your identity (used on every commit):

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

---

## 3. Terminal — iTerm2 or Warp (pick one)

**iTerm2** (classic, customizable):

```bash
brew install --cask iterm2
```

**Warp** (modern, AI-enhanced):

```bash
brew install --cask warp
```

Most people pick one. If unsure, start with Warp.

---

## 4. Oh My Zsh (shell framework)

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

This gives you themes, plugins, and a saner default zsh setup. Edit `~/.zshrc` to pick a theme (`ZSH_THEME="robbyrussell"` is the default).

---

## 5. SSH key + GitHub CLI

**Generate an SSH key** for GitHub:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Press Enter to accept the default path (`~/.ssh/id_ed25519`) and set a passphrase if you want one.

Add it to the ssh-agent:

```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

**Install `gh` CLI**:

```bash
brew install gh
```

Log in — this will also offer to upload your SSH key to GitHub:

```bash
gh auth login
```

Pick: GitHub.com → SSH → Upload your public key → Login with browser.

Verify:

```bash
gh auth status
ssh -T git@github.com
```

---

## 6. Node.js via nvm

Claude Code and many other CLIs need Node.

```bash
brew install nvm
```

Add nvm to your shell (Homebrew prints these; add to `~/.zshrc`):

```bash
mkdir -p ~/.nvm
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"' >> ~/.zshrc
source ~/.zshrc
```

Install the latest LTS Node:

```bash
nvm install --lts
nvm use --lts
nvm alias default lts/*
```

Verify:

```bash
node --version
npm --version
```

---

## 7. Python via pyenv

```bash
brew install pyenv
```

Add pyenv to shell:

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init - zsh)"' >> ~/.zshrc
source ~/.zshrc
```

Install a Python and set it as global default:

```bash
pyenv install 3.12
pyenv global 3.12
python --version
```

---

## 8. VS Code

```bash
brew install --cask visual-studio-code
```

Enable the `code` CLI command (so you can `code .` from any dir). In VS Code: `Cmd+Shift+P` → "Shell Command: Install 'code' command in PATH".

---

## 9. Claude Code (CLI)

Install globally via npm (Node must be installed first — see step 6):

```bash
npm install -g @anthropic-ai/claude-code
```

Verify and log in:

```bash
claude --version
claude
```

The first `claude` run will walk you through login.

---

## 10. Caveman (Claude Code plugin)

Cuts Claude output tokens ~75%. Same technical accuracy, less word.

Install the plugin:

```bash
claude plugin marketplace add JuliusBrussee/caveman && claude plugin install caveman@caveman
```

Make it always-on by adding to `~/.claude/CLAUDE.md` (create the file if it doesn't exist):

```
Terse like caveman. Technical substance exact. Only fluff die.
Drop: articles, filler (just/really/basically), pleasantries, hedging.
Fragments OK. Short synonyms. Code unchanged.
Pattern: [thing] [action] [reason]. [next step].
ACTIVE EVERY RESPONSE. No revert after many turns. No filler drift.
Code/commits/PRs: normal. Off: "stop caveman" / "normal mode".
```

Trigger: `/caveman` (lite / full / ultra). Stop: `stop caveman`.

---

## 11. Claude Desktop

```bash
brew install --cask claude
```

Launch and sign in.

---

## 12. Obsidian

```bash
brew install --cask obsidian
```

---

## 13. Docker Desktop

```bash
brew install --cask docker
```

Open Docker Desktop once from Applications to accept the license and start the daemon.

Verify:

```bash
docker --version
docker run hello-world
```

---

## 14. Notion

```bash
brew install --cask notion
```

---

## 15. Linear (desktop)

```bash
brew install --cask linear-linear
```

---

## 16. Figma

```bash
brew install --cask figma
```

---

## 17. API testing — Postman or Bruno (pick one)

**Postman** (full-featured, cloud sync):

```bash
brew install --cask postman
```

**Bruno** (local-first, git-friendly):

```bash
brew install --cask bruno
```

---

## 18. TablePlus (database GUI)

```bash
brew install --cask tableplus
```

---

## 19. Arc browser

```bash
brew install --cask arc
```

---

## Bonus: one-shot install with a Brewfile

Once Homebrew is installed (step 1), you can skip most individual `brew install` commands by dropping a `Brewfile` in your home dir and running `brew bundle`.

Save as `~/Brewfile`:

```ruby
# CLIs
brew "git"
brew "gh"
brew "nvm"
brew "pyenv"

# Terminal
cask "warp"          # or "iterm2"

# Dev
cask "visual-studio-code"
cask "docker"
cask "tableplus"
cask "postman"       # or cask "bruno"

# AI / Notes
cask "claude"
cask "obsidian"
cask "notion"

# PM / Design
cask "linear-linear"
cask "figma"

# Browser
cask "arc"
```

Then run:

```bash
brew bundle --file=~/Brewfile
```

This installs everything in one go. You'll still need to do these manually since they aren't brew-installable or need interactive auth:
- Xcode CLI tools (step 0)
- Oh My Zsh (step 4)
- SSH key + `gh auth login` (step 5)
- `nvm install --lts` (step 6)
- `pyenv install 3.12` (step 7)
- `npm install -g @anthropic-ai/claude-code` (step 9)
- Caveman plugin + `~/.claude/CLAUDE.md` (step 10)

---

## Recommended install order

1. Xcode CLI tools → Homebrew → git config
2. Terminal + Oh My Zsh
3. SSH key + gh CLI + login
4. nvm → Node
5. pyenv → Python
6. `brew bundle` for everything else
7. Claude Code (`npm install -g`)
8. Caveman plugin
9. Open each app once to log in

---

## Sanity check after setup

```bash
brew doctor
git --version
gh auth status
node --version
python --version
docker --version
claude --version
code --version
```

All should report a version and no errors.
