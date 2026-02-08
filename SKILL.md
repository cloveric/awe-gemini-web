---
name: awe-gemini-web
aliases: ["awepic"]
description: Image generation skill using Gemini Web. Generates images from text prompts via Google Gemini. Also supports text generation. Use when user asks to generate, create, or draw images. Triggers on requests like "生图", "生个图片", "出图", "画个图", "画一张", "generate image", "create image", "draw". Use as the image generation backend for other skills like aweppt.
---

# Gemini Web Client

Supports:
- Text generation
- Image generation (download + save)
- Reference images for vision input (attach local images)
- Multi-turn conversations via persisted `--sessionId`

## Script Directory

**Important**: All scripts are located in the `scripts/` subdirectory of this skill.

**Agent Execution Instructions**:
1. Determine this SKILL.md file's directory path as `SKILL_DIR`
2. Script path = `${SKILL_DIR}/scripts/<script-name>.ts`
3. Replace all `${SKILL_DIR}` in this document with the actual path

**Script Reference**:
| Script | Purpose |
|--------|---------|
| `scripts/main.ts` | CLI entry point for text/image generation |
| `scripts/gemini-webapi/*` | TypeScript port of `gemini_webapi` (GeminiClient, types, utils) |

## Windows Platform (CRITICAL)

**On Windows, you MUST use PowerShell to run bun commands.** The bash environment does not correctly capture stdout from bun.

**Correct execution method on Windows:**
```powershell
# Use powershell.exe -Command to wrap bun commands
powershell.exe -Command "bun '${SKILL_DIR}/scripts/main.ts' 'Your prompt here'"
powershell.exe -Command "bun '${SKILL_DIR}/scripts/main.ts' --promptfiles 'prompt.md' --image 'output.png'"
powershell.exe -Command "bun '${SKILL_DIR}/scripts/main.ts' --login"
```

**DO NOT use these on Windows (they will produce no output):**
```bash
# WRONG - bash does not capture bun stdout correctly on Windows
npx -y bun ${SKILL_DIR}/scripts/main.ts "prompt"
bun ${SKILL_DIR}/scripts/main.ts "prompt"
```

## Disclaimer (REQUIRED)

**Before using this skill**, the consent check MUST be performed.

### Consent Check Flow

**Step 1**: Check consent file

```bash
# macOS
cat ~/Library/Application\ Support/awe-skills/gemini-web/consent.json 2>/dev/null

# Linux
cat ~/.local/share/awe-skills/gemini-web/consent.json 2>/dev/null

# Windows (from Bash/Git Bash - use single quotes to prevent $env expansion)
powershell.exe -NoProfile -Command 'if (Test-Path "$env:APPDATA\awe-skills\gemini-web\consent.json") { Get-Content "$env:APPDATA\awe-skills\gemini-web\consent.json" } else { Write-Output "NOT_FOUND" }'
```

**Step 2**: If consent exists and `accepted: true` with matching `disclaimerVersion: "1.0"`:

Print warning and proceed:
```
Warning: Using reverse-engineered Gemini Web API (not official). Accepted on: <acceptedAt date>
```

**Step 3**: If consent file doesn't exist or `disclaimerVersion` mismatch:

Display disclaimer and ask user:

```
DISCLAIMER

This tool uses a reverse-engineered Gemini Web API, NOT an official Google API.

Risks:
- May break without notice if Google changes their API
- No official support or guarantees
- Use at your own risk

Do you accept these terms and wish to continue?
```

Use `AskUserQuestion` tool with options:
- **Yes, I accept** - Continue and save consent
- **No, I decline** - Exit immediately

**Step 4**: On acceptance, create consent file:

```bash
# macOS
mkdir -p ~/Library/Application\ Support/awe-skills/gemini-web
cat > ~/Library/Application\ Support/awe-skills/gemini-web/consent.json << 'EOF'
{
  "version": 1,
  "accepted": true,
  "acceptedAt": "<ISO timestamp>",
  "disclaimerVersion": "1.0"
}
EOF

# Linux
mkdir -p ~/.local/share/awe-skills/gemini-web
cat > ~/.local/share/awe-skills/gemini-web/consent.json << 'EOF'
{
  "version": 1,
  "accepted": true,
  "acceptedAt": "<ISO timestamp>",
  "disclaimerVersion": "1.0"
}
EOF

# Windows (from Bash/Git Bash - use single quotes)
powershell.exe -NoProfile -Command 'New-Item -ItemType Directory -Force -Path "$env:APPDATA\awe-skills\gemini-web" | Out-Null; @{version=1;accepted=$true;acceptedAt="<ISO timestamp>";disclaimerVersion="1.0"} | ConvertTo-Json | Set-Content "$env:APPDATA\awe-skills\gemini-web\consent.json"'
```

**Step 5**: On decline, output message and stop:
```
User declined the disclaimer. Exiting.
```

---

## Quick start

```bash
npx -y bun ${SKILL_DIR}/scripts/main.ts "Hello, Gemini"
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "Explain quantum computing"
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "A cute cat" --image cat.png
npx -y bun ${SKILL_DIR}/scripts/main.ts --promptfiles system.md content.md --image out.png

# Multi-turn conversation (agent generates unique sessionId)
npx -y bun ${SKILL_DIR}/scripts/main.ts "Remember this: 42" --sessionId my-unique-id-123
npx -y bun ${SKILL_DIR}/scripts/main.ts "What number?" --sessionId my-unique-id-123
```

## Commands

### Text generation

```bash
# Simple prompt (positional)
npx -y bun ${SKILL_DIR}/scripts/main.ts "Your prompt here"

# Explicit prompt flag
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "Your prompt here"
npx -y bun ${SKILL_DIR}/scripts/main.ts -p "Your prompt here"

# With model selection
npx -y bun ${SKILL_DIR}/scripts/main.ts -p "Hello" -m gemini-2.5-pro

# Pipe from stdin
echo "Summarize this" | npx -y bun ${SKILL_DIR}/scripts/main.ts
```

### Image generation

```bash
# Generate image with default path (./generated.png)
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "A sunset over mountains" --image

# Generate image with custom path
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "A cute robot" --image robot.png

# Shorthand
npx -y bun ${SKILL_DIR}/scripts/main.ts "A dragon" --image=dragon.png
```

### Vision input (reference images)

```bash
# Text + image -> text
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "Describe this image" --reference a.png

# Text + image -> image
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "Generate a variation" --reference a.png --image out.png
```

### Output formats

```bash
# Plain text (default)
npx -y bun ${SKILL_DIR}/scripts/main.ts "Hello"

# JSON output
npx -y bun ${SKILL_DIR}/scripts/main.ts "Hello" --json
```

## Options

| Option | Description |
|--------|-------------|
| `--prompt <text>`, `-p` | Prompt text |
| `--promptfiles <files...>` | Read prompt from files (concatenated in order) |
| `--model <id>`, `-m` | Model: gemini-3-pro (default), gemini-2.5-pro, gemini-2.5-flash |
| `--image [path]` | Generate image, save to path (default: generated.png) |
| `--reference <files...>`, `--ref <files...>` | Reference images for vision input |
| `--sessionId <id>` | Session ID for multi-turn conversation (agent generates unique ID) |
| `--list-sessions` | List saved sessions (max 100, sorted by update time) |
| `--json` | Output as JSON |
| `--login` | Refresh cookies only, then exit |
| `--cookie-path <path>` | Custom cookie file path |
| `--profile-dir <path>` | Chrome profile directory |
| `--help`, `-h` | Show help |

CLI note: `scripts/main.ts` supports text generation, image generation, reference images (`--reference/--ref`), and multi-turn conversations via `--sessionId`.

## Models

- `gemini-3-pro` - Default, latest model
- `gemini-2.5-pro` - Previous generation pro
- `gemini-2.5-flash` - Fast, lightweight

## Authentication

First run opens Chrome to authenticate with Google. Cookies are cached for subsequent runs.

### Login Flow (IMPORTANT)

**Agent MUST directly execute the login command** when authentication is needed. Do NOT ask the user to run it manually.

1. **Check if cookies exist**: Look for cookies file at `%APPDATA%/baoyu-skills/gemini-web/cookies.json` (Windows)
2. **If no cookies or auth fails**: Execute login command directly:
   ```powershell
   # Windows - agent executes this directly
   powershell.exe -Command "bun '${SKILL_DIR}/scripts/main.ts' --login"
   ```
   ```bash
   # macOS/Linux
   npx -y bun ${SKILL_DIR}/scripts/main.ts --login
   ```
3. **Chrome will open** for the user to complete Google login
4. **After login completes**, cookies are saved and subsequent commands will work

**DO NOT** stop and ask the user to manually run login commands. Execute them directly.

## Environment variables

| Variable | Description |
|----------|-------------|
| `GEMINI_WEB_DATA_DIR` | Data directory |
| `GEMINI_WEB_COOKIE_PATH` | Cookie file path |
| `GEMINI_WEB_CHROME_PROFILE_DIR` | Chrome profile directory |
| `GEMINI_WEB_CHROME_PATH` | Chrome executable path |

## Examples

### Generate text response
```bash
npx -y bun ${SKILL_DIR}/scripts/main.ts "What is the capital of France?"
```

### Generate image
```bash
npx -y bun ${SKILL_DIR}/scripts/main.ts "A photorealistic image of a golden retriever puppy" --image puppy.png
```

### Get JSON output for parsing
```bash
npx -y bun ${SKILL_DIR}/scripts/main.ts "Hello" --json | jq '.text'
```

### Generate image from prompt files
```bash
# Concatenate system.md + content.md as prompt
npx -y bun ${SKILL_DIR}/scripts/main.ts --promptfiles system.md content.md --image output.png
```

### Multi-turn conversation
```bash
# Start a session with unique ID (agent generates this)
npx -y bun ${SKILL_DIR}/scripts/main.ts "You are a helpful math tutor." --sessionId task-abc123

# Continue the conversation (remembers context)
npx -y bun ${SKILL_DIR}/scripts/main.ts "What is 2+2?" --sessionId task-abc123
npx -y bun ${SKILL_DIR}/scripts/main.ts "Now multiply that by 10" --sessionId task-abc123

# List recent sessions (max 100, sorted by update time)
npx -y bun ${SKILL_DIR}/scripts/main.ts --list-sessions
```

Session files are stored in platform-specific data directories and contain:
- `id`: Session ID
- `metadata`: Gemini chat metadata for continuation
- `messages`: Array of `{role, content, timestamp, error?}`
- `createdAt`, `updatedAt`: Timestamps

## Extension Support

Custom configurations via EXTEND.md.

**Check paths** (priority order):
1. `.awe-skills/awe-gemini-web/EXTEND.md` (project)
2. `~/.awe-skills/awe-gemini-web/EXTEND.md` (user)

If found, load before workflow. Extension content overrides defaults.
