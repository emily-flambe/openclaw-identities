# Gmail Polling for OpenClaw

This guide covers setting up Gmail notifications for OpenClaw using polling (cron-based checking).

## Overview

- **Use case**: Get texted when important emails arrive
- **Architecture**: Cron job → isolated agent → checks Gmail via `gog` → texts you via Signal
- **Filtering**: Only primary inbox, only actionable emails

## Prerequisites

- Windows 10/11
- [gog](https://github.com/openclaw/gog) (Gmail CLI tool)
- A Google Cloud project with Gmail API enabled
- Signal channel configured (see `../signal/README.md`)

## Step 1: Install gog

Download from [gog releases](https://github.com/openclaw/gog/releases) and extract to a folder (e.g., `C:\Users\<you>\gogcli`).

Add to your OpenClaw PATH in `openclaw.json`:

```json
{
  "tools": {
    "exec": {
      "pathPrepend": [
        "C:\\Users\\<you>\\gogcli"
      ]
    }
  }
}
```

## Step 2: Authenticate gog with Gmail

```powershell
gog auth add your.email@gmail.com
```

This opens a browser for OAuth. Grant access to your Gmail.

## Step 3: Enable Gmail API

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create or select a project
3. Enable the Gmail API:
   ```powershell
   gcloud services enable gmail.googleapis.com --project YOUR_PROJECT_ID
   ```

## Step 4: Test gog

```powershell
gog gmail search "is:unread" --account your.email@gmail.com
```

You should see your unread emails listed.

## Step 5: Set up the polling cron job

The key insight: use `sessionTarget: "isolated"` with `payload.kind: "agentTurn"` so the agent actually runs the check autonomously.

Using the OpenClaw cron tool:

```json
{
  "name": "gmail-poll",
  "schedule": {
    "kind": "every",
    "everyMs": 300000
  },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Check for new unread emails using: gog gmail search 'is:unread category:primary' --account your.email@gmail.com. If there are actionable emails (requiring a response, decision, or action), text <YOUR_PHONE> via Signal with a brief, human-friendly summary. Never send raw command output, error messages, or technical details. Skip newsletters, FYIs, and automated notifications. If nothing actionable, stay completely silent - no reply at all.",
    "deliver": true,
    "channel": "signal",
    "to": "+1YOURNUMBER"
  }
}
```

**Important settings:**
- `sessionTarget: "isolated"` — spawns a separate agent that actually does the work
- `payload.kind: "agentTurn"` — runs a full agent turn, not just a text reminder
- `everyMs: 300000` — 5 minutes (adjust as needed)
- `deliver: true` — send output to the specified channel

### Why isolated + agentTurn?

If you use `sessionTarget: "main"` with `payload.kind: "systemEvent"`, it just injects a reminder text into your main session. You (the agent) have to be actively paying attention to see and act on it.

With `isolated` + `agentTurn`, a fresh agent spawns, runs the check, and texts you directly — no human-in-the-loop required.

## Step 6: Configure email preferences

Tell your agent what emails matter. Add to `USER.md` in your workspace:

```markdown
## Email Preferences
- **Only primary inbox matters** — use `category:primary` not just `in:inbox`
- No promotions, updates, social, forums, or old archived mail
- Search: `is:unread category:primary`
- **Only actionable emails** — don't text about newsletters, FYIs, or automated notifications
- Text only if something requires a response, decision, or action
```

## Gmail Search Filters

Common filters for the `gog gmail search` command:

| Filter | Description |
|--------|-------------|
| `is:unread` | Unread emails only |
| `category:primary` | Primary inbox (not promotions/social/updates) |
| `newer_than:1d` | Last 24 hours |
| `newer_than:1h` | Last hour |
| `in:inbox` | In inbox (includes promotions tab) |
| `-category:promotions` | Exclude promotions |
| `from:someone@example.com` | From specific sender |

Combine them: `is:unread category:primary`

**Note:** The `newer_than` filter is usually unnecessary — `is:unread` is sufficient since read emails won't keep showing up.

## Cost Considerations

Each poll spawns an isolated agent turn:
- ~500-1000 tokens per check
- At 5-minute intervals: ~288 checks/day
- Roughly 150K-300K tokens/day

This is modest, but consider longer intervals (15-30 min) if cost is a concern.

## Troubleshooting

### "gog: command not found"
Add gog's directory to `tools.exec.pathPrepend` in your OpenClaw config.

### Cron runs but no texts
Check that:
1. `sessionTarget` is `"isolated"` (not `"main"`)
2. `payload.kind` is `"agentTurn"` (not `"systemEvent"`)
3. Signal channel is working: `openclaw message send --channel signal --to +1NUMBER --message "test"`

### Old emails showing up
Use `is:unread` — once emails are read/archived, they won't reappear.

### Promotions showing up
Use `category:primary` instead of just `in:inbox`.

### Agent checks but never texts
The agent only texts when there are actionable results. Send yourself a test email to verify.

### Archiving doesn't work (emails still show up)
The search returns **thread IDs**, not message IDs. Use `gog gmail thread modify <threadId> --remove "INBOX,UNREAD"` instead of batch modify for reliable archiving.

### CATEGORY_UPDATES emails showing in primary
Gmail's `category:primary` search may return emails that have `CATEGORY_UPDATES` labels but still appear in the Primary tab. Don't over-filter based on labels — trust what the user actually sees in their inbox.

## Alternative: Push Notifications (Advanced)

OpenClaw supports Gmail push notifications via Pub/Sub, but this requires:
- Google Cloud Pub/Sub setup
- Tailscale Funnel for public endpoint
- The `gog gmail watch serve` command (currently buggy on Windows)

For most users, polling every 5 minutes is simpler and reliable enough.

## Example: Complete Setup

1. **gog installed and authenticated**
2. **Signal channel working**
3. **Cron job created:**

```powershell
# Via OpenClaw CLI (if available) or directly in config
```

4. **Test it:**
```powershell
gog gmail send --to your.email@gmail.com --subject "Test" --body "Testing polling!" --account your.email@gmail.com
```

Wait up to 5 minutes — you should get a text!

## See Also

- [Signal Setup](../signal/README.md) — Required for receiving texts
- [OpenClaw Cron Docs](https://docs.openclaw.ai/cron) — Full cron configuration
- [gog Documentation](https://github.com/openclaw/gog) — Gmail CLI tool
