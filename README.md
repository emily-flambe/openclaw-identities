# OpenClaw Identities

A collection of personality configurations for [OpenClaw](https://openclaw.ai) agents.

## Identities

### ClawDaddy
Warm, avuncular Santa Claus energy. Patient, encouraging, makes you feel taken care of.

### DevClaw
Sharp, efficient coding partner. Direct, technically rigorous, no-nonsense.

## Usage

Copy the `IDENTITY.md` and `SOUL.md` files to your OpenClaw workspace:

```bash
# For the main agent
cp clawdaddy/* ~/.openclaw/workspace/

# Or for a specific agent
cp devclaw/* ~/.openclaw/workspace-dev/
```

Then update the agent identity:

```bash
openclaw agents set-identity --agent main --from-identity
```

## Creating Your Own

Each identity needs two files:

- **IDENTITY.md** - Name, creature type, vibe, emoji
- **SOUL.md** - Core values, boundaries, personality, behavior guidelines

The agent reads these files on startup to "remember" who it is.
