<div align="center">

# awe-gemini-web

**Unofficial TypeScript client for Google Gemini's web interface**

Text generation &bull; Image generation &bull; Vision input &bull; Multi-turn conversations

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-runtime-f472b6?logo=bun&logoColor=white)](https://bun.sh/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

</div>

---

> [!WARNING]
> This project uses a **reverse-engineered** Gemini Web API, not an official Google API. It may break without notice if Google changes their internal endpoints. Use at your own risk.

## Features

- **Text Generation** — Send prompts, get responses. Simple as that.
- **Image Generation** — Generate images from text descriptions and save them locally.
- **Vision Input** — Attach reference images for multimodal prompts.
- **Multi-turn Conversations** — Persist chat sessions across multiple invocations via `--sessionId`.
- **Multiple Models** — Switch between `gemini-3-pro`, `gemini-2.5-pro`, and `gemini-2.5-flash`.
- **Auto Cookie Refresh** — Keeps your session alive with automatic `__Secure-1PSIDTS` rotation.
- **Prompt Files** — Concatenate multiple files as a single prompt with `--promptfiles`.
- **JSON Output** — Machine-readable output for scripting and pipelines.

## Architecture

```
scripts/
├── main.ts                    # CLI entry point
└── gemini-webapi/             # TypeScript port of gemini_webapi
    ├── client.ts              # GeminiClient & ChatSession
    ├── constants.ts           # Endpoints, headers, model definitions
    ├── exceptions.ts          # Typed error hierarchy
    ├── components/
    │   └── gem-mixin.ts       # Gems (custom chatbots) support
    ├── types/
    │   ├── candidate.ts       # Response candidate parsing
    │   ├── gem.ts             # Gem data structures
    │   ├── image.ts           # WebImage & GeneratedImage classes
    │   └── modeloutput.ts     # Unified model output
    └── utils/
        ├── cookie-file.ts     # Cookie persistence
        ├── get-access-token.ts# Auth token extraction
        ├── load-browser-cookies.ts # Chrome cookie extraction
        ├── rotate-1psidts.ts  # Session cookie rotation
        ├── upload-file.ts     # File upload for vision
        └── ...
```

## Prerequisites

- [Bun](https://bun.sh/) (recommended) or Node.js 18+
- A Google account with access to [Gemini](https://gemini.google.com/)
- Chrome browser (for initial authentication)

## Quick Start

### 1. Authenticate

On first run, Chrome will open for you to sign in to your Google account:

```bash
bun scripts/main.ts --login
```

Cookies are cached locally for subsequent runs.

### 2. Generate text

```bash
# Simple prompt
bun scripts/main.ts "What is the capital of France?"

# From files
bun scripts/main.ts --promptfiles system-prompt.md user-query.md

# With model selection
bun scripts/main.ts -p "Explain quantum computing" -m gemini-2.5-pro
```

### 3. Generate images

```bash
# Save to default path (generated.png)
bun scripts/main.ts "A sunset over mountains" --image

# Custom output path
bun scripts/main.ts "A cute robot painting" --image robot.png
```

### 4. Vision input

```bash
# Describe an image
bun scripts/main.ts "What's in this image?" --ref photo.jpg

# Generate a variation
bun scripts/main.ts "Create a watercolor version" --ref original.png --image watercolor.png
```

### 5. Multi-turn conversations

```bash
# Start a session
bun scripts/main.ts "You are a helpful math tutor." --sessionId math-tutor

# Continue the conversation (context is preserved)
bun scripts/main.ts "What is 2+2?" --sessionId math-tutor
bun scripts/main.ts "Now multiply that by 10" --sessionId math-tutor

# List recent sessions
bun scripts/main.ts --list-sessions
```

## CLI Reference

| Option | Description |
|---|---|
| `-p, --prompt <text>` | Prompt text |
| `--promptfiles <files...>` | Read and concatenate multiple files as prompt |
| `-m, --model <id>` | Model selection (see below) |
| `--image [path]` | Generate image, optionally specify save path |
| `--ref, --reference <files...>` | Attach reference images for vision input |
| `--sessionId <id>` | Session ID for multi-turn conversations |
| `--list-sessions` | List saved sessions (max 100) |
| `--json` | Output as JSON |
| `--login` | Refresh authentication cookies |
| `--cookie-path <path>` | Custom cookie file path |
| `--profile-dir <path>` | Chrome profile directory |

## Models

| Model ID | Description |
|---|---|
| `gemini-3-pro` | Latest model *(default)* |
| `gemini-2.5-pro` | Previous generation pro |
| `gemini-2.5-flash` | Fast, lightweight |

## Environment Variables

| Variable | Description |
|---|---|
| `GEMINI_WEB_DATA_DIR` | Base data directory |
| `GEMINI_WEB_COOKIE_PATH` | Cookie file path override |
| `GEMINI_WEB_CHROME_PROFILE_DIR` | Chrome profile directory |
| `GEMINI_WEB_CHROME_PATH` | Chrome executable path |

## Platform Notes

<details>
<summary><strong>Windows</strong></summary>

Use PowerShell to run bun commands — bash on Windows may not correctly capture stdout:

```powershell
powershell.exe -Command "bun 'scripts/main.ts' 'Your prompt here'"
powershell.exe -Command "bun 'scripts/main.ts' --image 'output.png' -p 'A dragon'"
```

</details>

## Acknowledgements

This project is a TypeScript port inspired by [gemini_webapi](https://github.com/HanaokaYuzu/Gemini-API) (Python).

## License

MIT
