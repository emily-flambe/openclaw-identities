# OpenClaw Identities

A collection of personality configurations for [OpenClaw](https://openclaw.ai) agents, plus setup guides for integrations.

## App Setup Guides

| App | Description |
|-----|-------------|
| [Signal](apps/signal/README.md) | Text your agent from your phone using a dedicated bot number |
| [Gmail](apps/gmail/README.md) | Get texted when important emails arrive (polling-based) |

## Available Identities

| Identity | Name | Emoji | Purpose |
|----------|------|-------|---------|
| `clawdaddy` | ClawDaddy | 🎅 | Warm, avuncular Santa Claus energy. Patient, encouraging. |
| `devclaw` | DevClaw | 🦀 | Sharp, efficient coding partner. Direct, technically rigorous. |
| `shipit` | ShipIt | 🐿️ | One-shot app builder. TDD, aggressive agent orchestration, CI monitoring. |
| `reviewer` | Nitpick | 🔍 | Thorough code reviewer. Objective, always cites documentation. |
| `debugger` | Trace | 🔬 | Evidence-based bug hunter. Systematic, never guesses. |
| `secbot` | Sentinel | 🛡️ | Security auditor. OWASP-focused, paranoid by design. |
| `mentor` | Sage | 🎓 | Patient teacher. Explains the "why", meets learners where they are. |
| `clawsmith` | Clawsmith | ⚒️ | OpenClaw configuration specialist. Always checks the docs first. |

## Quick Setup

### 1. Create the agent

```bash
openclaw agents add shipit --workspace ~/.openclaw/workspace-shipit
```

### 2. Copy auth from an existing agent

```bash
mkdir -p ~/.openclaw/agents/shipit/agent
cp ~/.openclaw/agents/main/agent/auth-profiles.json ~/.openclaw/agents/shipit/agent/
```

### 3. Copy identity files

```bash
# Clone this repo first
git clone https://github.com/emily-flambe/openclaw-identities.git

# Copy the identity files to the workspace
cp openclaw-identities/shipit/IDENTITY.md ~/.openclaw/workspace-shipit/
cp openclaw-identities/shipit/SOUL.md ~/.openclaw/workspace-shipit/
```

### 4. Set the identity

```bash
openclaw agents set-identity --agent shipit --name "ShipIt" --emoji "🐿️"
```

### 5. Launch

```bash
openclaw tui --session agent:shipit:main
```

## Full Setup Script

To set up all agents at once:

```bash
#!/bin/bash
REPO_PATH="$HOME/repos/openclaw-identities"

# Clone if needed
[ ! -d "$REPO_PATH" ] && git clone https://github.com/emily-flambe/openclaw-identities.git "$REPO_PATH"

# Define agents: name|display_name|emoji
agents=(
  "shipit|ShipIt|🐿️"
  "reviewer|Nitpick|🔍"
  "debugger|Trace|🔬"
  "secbot|Sentinel|🛡️"
  "mentor|Sage|🎓"
  "clawsmith|Clawsmith|⚒️"
)

for entry in "${agents[@]}"; do
  IFS='|' read -r name display emoji <<< "$entry"

  # Create agent
  openclaw agents add "$name" --workspace ~/.openclaw/workspace-"$name"

  # Copy auth
  mkdir -p ~/.openclaw/agents/"$name"/agent
  cp ~/.openclaw/agents/main/agent/auth-profiles.json ~/.openclaw/agents/"$name"/agent/

  # Copy identity files
  cp "$REPO_PATH/$name/IDENTITY.md" ~/.openclaw/workspace-"$name"/
  cp "$REPO_PATH/$name/SOUL.md" ~/.openclaw/workspace-"$name"/

  # Set identity
  openclaw agents set-identity --agent "$name" --name "$display" --emoji "$emoji"
done

echo "Done! Restart gateway: openclaw gateway restart"
```

## Shell Aliases

### PowerShell (Windows)

Add to your PowerShell profile (`$PROFILE`):

```powershell
# OpenClaw Agent Shortcuts
function shipit { openclaw tui --session "agent:shipit:main" }
function devclaw { openclaw tui --session "agent:dev:main" }
function clawdaddy { openclaw tui --session "agent:main:main" }
function reviewer { openclaw tui --session "agent:reviewer:main" }
function debugger { openclaw tui --session "agent:debugger:main" }
function secbot { openclaw tui --session "agent:secbot:main" }
function mentor { openclaw tui --session "agent:mentor:main" }
function clawsmith { openclaw tui --session "agent:clawsmith:main" }

# Generic: oc <agent> [message]
function oc {
    param([string]$agent = "main", [string]$message)
    if ($message) {
        openclaw agent --local --agent $agent --message $message
    } else {
        openclaw tui --session "agent:${agent}:main"
    }
}
```

Reload with: `. $PROFILE`

### Zsh/Bash (macOS/Linux)

Add to `~/.zshrc` or `~/.bashrc`:

```bash
# OpenClaw Agent Shortcuts
oc() {
  local agent="${1:-main}"
  shift 2>/dev/null
  if [[ -n "$1" ]]; then
    openclaw agent --local --agent "$agent" --message "$*"
  else
    openclaw tui --session "agent:$agent:main"
  fi
}

alias shipit='openclaw tui --session agent:shipit:main'
alias devclaw='openclaw tui --session agent:dev:main'
alias clawdaddy='openclaw tui --session agent:main:main'
alias reviewer='openclaw tui --session agent:reviewer:main'
alias debugger='openclaw tui --session agent:debugger:main'
alias secbot='openclaw tui --session agent:secbot:main'
alias mentor='openclaw tui --session agent:mentor:main'
alias clawsmith='openclaw tui --session agent:clawsmith:main'
```

Reload with: `source ~/.zshrc`

## Creating Your Own Identity

Each identity needs two files in its folder:

### IDENTITY.md

Basic profile information:

```markdown
# IDENTITY.md - Who Am I?

- **Name:** YourAgentName
- **Creature:** What kind of entity are you?
- **Vibe:** How do you come across?
- **Emoji:** 🤖
```

### SOUL.md

Personality, values, and behavior guidelines:

```markdown
# SOUL.md - Who You Are

_Your core philosophy in one line._

## Core Truths

**Key principle 1.** Explanation of this principle.

**Key principle 2.** Explanation of this principle.

## Boundaries

- What you won't do
- Limits on your behavior

## Vibe

Describe your personality, communication style, and how you interact.

---

_Closing statement that reinforces your identity._
```

## Tips

- **Restart gateway after adding agents:** `openclaw gateway restart`
- **Check agent list:** `openclaw agents list`
- **Bootstrap new agents:** First message in TUI will trigger personality setup
- **Update identity later:** Edit the IDENTITY.md/SOUL.md files, agent reads them each session

## OpenClaw Documentation

- [Getting Started](https://docs.openclaw.ai/start/getting-started) - Installation and onboarding
- [CLI Reference](https://docs.openclaw.ai/cli) - All CLI commands
- [Agents](https://docs.openclaw.ai/cli/agents) - Managing multiple agents
- [TUI](https://docs.openclaw.ai/cli/tui) - Terminal UI options
- [Channels](https://docs.openclaw.ai/channels) - WhatsApp, Telegram, Discord, Signal, etc.
- [Troubleshooting](https://docs.openclaw.ai/troubleshooting) - Common issues and fixes
- [GitHub](https://github.com/openclaw/openclaw) - Source code and issues

## License

MIT
