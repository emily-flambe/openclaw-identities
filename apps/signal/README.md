# Signal Setup for OpenClaw

This guide covers setting up Signal as a messaging channel for OpenClaw on Windows, using a dedicated Google Voice number.

## Overview

- **Use case**: Text your OpenClaw agent from your personal Signal, using a separate bot number
- **Architecture**: signal-cli daemon ↔ OpenClaw gateway ↔ your phone
- **Privacy**: Your personal Signal messages stay private — only messages to the bot number reach OpenClaw

## Prerequisites

- Windows 10/11
- Java 17+ (signal-cli requirement)
- A spare phone number (Google Voice works great)

## Step 1: Install signal-cli

Download from [GitHub releases](https://github.com/AsamK/signal-cli/releases) and extract somewhere (e.g., `C:\Users\<you>\signal-cli-<version>`).

Or use a package manager:
```powershell
scoop install signal-cli
# or
choco install signal-cli
```

## Step 2: Register your bot number

Signal requires a captcha for new registrations:

1. Go to https://signalcaptchas.org/registration/generate.html
2. Solve the captcha
3. **Right-click** "Open Signal" → **Copy Link** (don't click it!)
4. Run:

```powershell
signal-cli -a +1YOURNUMBER register --captcha "signalcaptcha://signal-recaptcha-v2.03AGdBq24..."
```

5. You'll get an SMS verification code. Verify it:

```powershell
signal-cli -a +1YOURNUMBER verify 123456
```

## Step 3: Set a profile name (optional but recommended)

```powershell
signal-cli -a +1YOURNUMBER updateProfile --given-name "YourBotName" --about "Your friendly AI assistant"
```

## Step 4: Configure OpenClaw

Add to your `openclaw.json` (or use `openclaw config set`):

```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "account": "+1YOURNUMBER",
      "cliPath": "signal-cli",
      "dmPolicy": "pairing",
      "allowFrom": ["+1YOURPERSONALNUMBER"]
    }
  }
}
```

**Config options:**
- `account`: Your bot's phone number (the one you registered)
- `cliPath`: Path to signal-cli (or just `"signal-cli"` if in PATH)
- `dmPolicy`: `"pairing"` (require approval), `"allowlist"` (only allowFrom), or `"open"` (anyone)
- `allowFrom`: Array of phone numbers that can message the bot

## Step 5: Start the daemon

signal-cli needs to run as an HTTP daemon for OpenClaw to connect:

```powershell
signal-cli -a +1YOURNUMBER daemon --http
```

**To run it hidden (detached):**
```powershell
Start-Process -WindowStyle Hidden -FilePath "path\to\signal-cli.bat" -ArgumentList "-a +1YOURNUMBER daemon --http"
```

## Step 6: Make it persistent (optional)

To auto-start on boot, create a scheduled task:

```powershell
$action = New-ScheduledTaskAction -Execute "C:\path\to\signal-cli.bat" -Argument "-a +1YOURNUMBER daemon --http"
$trigger = New-ScheduledTaskTrigger -AtStartup
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable
Register-ScheduledTask -TaskName "signal-cli-daemon" -Action $action -Trigger $trigger -Settings $settings -RunLevel Highest
```

## Troubleshooting

### "JAVA_HOME is not set"
Add Java to your PATH or set JAVA_HOME. signal-cli needs Java 17+.

### "signal daemon not ready"
The daemon isn't running. Start it with `signal-cli -a +1NUMBER daemon --http`.

### "Captcha required"
Signal requires captcha verification for new accounts. See Step 2.

### Profile name not showing
Signal profile sync can be slow. Try:
1. Close and reopen Signal on your phone
2. Or just save the bot number as a contact

### OpenClaw can't find signal-cli
Add the signal-cli bin directory to `tools.exec.pathPrepend` in your OpenClaw config:

```json
{
  "tools": {
    "exec": {
      "pathPrepend": [
        "C:\\path\\to\\signal-cli\\bin",
        "C:\\path\\to\\java\\bin"
      ]
    }
  }
}
```

## Example config (complete)

```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "account": "+17208932370",
      "cliPath": "signal-cli",
      "dmPolicy": "pairing",
      "allowFrom": ["+17165727513"],
      "groupPolicy": "allowlist"
    }
  },
  "tools": {
    "exec": {
      "pathPrepend": [
        "C:\\Users\\emily\\signal-cli-0.13.23\\bin",
        "C:\\Program Files\\Eclipse Adoptium\\jdk-21.0.6.7-hotspot\\bin"
      ]
    }
  }
}
```

## Sending messages

Once set up, OpenClaw can send Signal messages:

```
message action=send channel=signal target=+1PHONENUMBER message="Hello!"
```

Or the agent can use the message tool automatically when replying to Signal conversations.
