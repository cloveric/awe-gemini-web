<div align="center">

# awe-gemini-web

**Unofficial TypeScript client for Google Gemini's web interface**

文本生成 &bull; 图片生成 &bull; 图片理解 &bull; 多轮对话

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-runtime-f472b6?logo=bun&logoColor=white)](https://bun.sh/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**[English](#features)** | **[中文](#功能特性)**

</div>

---

> [!WARNING]
> This project uses a **reverse-engineered** Gemini Web API, not an official Google API. It may break without notice if Google changes their internal endpoints. Use at your own risk.
>
> 本项目使用的是**逆向工程**的 Gemini Web API，并非 Google 官方 API。如果 Google 修改了内部接口，本项目可能会随时失效。使用风险自负。

## Features

- **Text Generation** — Send prompts, get responses. Simple as that.
- **Image Generation** — Generate images from text descriptions and save them locally.
- **Vision Input** — Attach reference images for multimodal prompts.
- **Multi-turn Conversations** — Persist chat sessions across multiple invocations via `--sessionId`.
- **Multiple Models** — Switch between `gemini-3-pro`, `gemini-2.5-pro`, and `gemini-2.5-flash`.
- **Auto Cookie Refresh** — Keeps your session alive with automatic `__Secure-1PSIDTS` rotation.
- **Prompt Files** — Concatenate multiple files as a single prompt with `--promptfiles`.
- **JSON Output** — Machine-readable output for scripting and pipelines.

## 功能特性

- **文本生成** — 发送提示词，获取回复，就这么简单。
- **图片生成** — 通过文字描述生成图片并保存到本地。
- **图片理解** — 附加参考图片，实现多模态提示。
- **多轮对话** — 通过 `--sessionId` 在多次调用之间保持上下文。
- **多模型切换** — 支持 `gemini-3-pro`、`gemini-2.5-pro` 和 `gemini-2.5-flash`。
- **自动刷新 Cookie** — 自动轮换 `__Secure-1PSIDTS`，保持会话有效。
- **文件提示词** — 使用 `--promptfiles` 将多个文件拼接为一个提示词。
- **JSON 输出** — 机器可读的输出格式，方便脚本和流水线集成。

## Architecture / 项目结构

```
scripts/
├── main.ts                        # CLI 入口 / CLI entry point
└── gemini-webapi/                 # gemini_webapi 的 TypeScript 移植
    ├── client.ts                  # GeminiClient 和 ChatSession
    ├── constants.ts               # 端点、请求头、模型定义
    ├── exceptions.ts              # 类型化的错误层次结构
    ├── components/
    │   └── gem-mixin.ts           # Gems（自定义聊天机器人）支持
    ├── types/
    │   ├── candidate.ts           # 响应候选项解析
    │   ├── gem.ts                 # Gem 数据结构
    │   ├── image.ts               # WebImage 和 GeneratedImage
    │   └── modeloutput.ts         # 统一的模型输出
    └── utils/
        ├── cookie-file.ts         # Cookie 持久化
        ├── get-access-token.ts    # 认证令牌提取
        ├── load-browser-cookies.ts# Chrome Cookie 读取
        ├── rotate-1psidts.ts      # 会话 Cookie 轮换
        ├── upload-file.ts         # 文件上传（用于图片理解）
        └── ...
```

## Prerequisites / 环境要求

- [Bun](https://bun.sh/)（推荐）或 Node.js 18+
- 拥有 [Gemini](https://gemini.google.com/) 访问权限的 Google 账号
- Chrome 浏览器（用于首次认证）

## Quick Start / 快速开始

### 1. 认证 / Authenticate

首次运行时，Chrome 会自动打开供你登录 Google 账号：

```bash
bun scripts/main.ts --login
```

登录后 Cookie 会被缓存到本地，后续无需重复登录。

### 2. 文本生成 / Generate Text

```bash
# 简单提示词
bun scripts/main.ts "法国的首都是哪里？"

# 从文件读取提示词
bun scripts/main.ts --promptfiles system-prompt.md user-query.md

# 指定模型
bun scripts/main.ts -p "解释量子计算" -m gemini-2.5-pro
```

### 3. 图片生成 / Generate Images

```bash
# 保存到默认路径（generated.png）
bun scripts/main.ts "山间日落的风景画" --image

# 自定义输出路径
bun scripts/main.ts "一个可爱的机器人在画画" --image robot.png
```

### 4. 图片理解 / Vision Input

```bash
# 描述一张图片
bun scripts/main.ts "这张图片里有什么？" --ref photo.jpg

# 基于参考图生成变体
bun scripts/main.ts "把这张图转换成水彩风格" --ref original.png --image watercolor.png
```

### 5. 多轮对话 / Multi-turn Conversations

```bash
# 开始一个会话
bun scripts/main.ts "你是一个数学辅导老师。" --sessionId math-tutor

# 继续对话（上下文自动保持）
bun scripts/main.ts "2+2等于多少？" --sessionId math-tutor
bun scripts/main.ts "再乘以10呢？" --sessionId math-tutor

# 列出最近的会话
bun scripts/main.ts --list-sessions
```

## CLI Reference / 命令行参数

| 参数 | 说明 |
|---|---|
| `-p, --prompt <text>` | 提示词文本 |
| `--promptfiles <files...>` | 从多个文件读取并拼接为提示词 |
| `-m, --model <id>` | 选择模型（见下表） |
| `--image [path]` | 生成图片，可指定保存路径 |
| `--ref, --reference <files...>` | 附加参考图片（图片理解） |
| `--sessionId <id>` | 多轮对话的会话 ID |
| `--list-sessions` | 列出已保存的会话（最多 100 条） |
| `--json` | 以 JSON 格式输出 |
| `--login` | 刷新认证 Cookie |
| `--cookie-path <path>` | 自定义 Cookie 文件路径 |
| `--profile-dir <path>` | Chrome 用户数据目录 |

## Models / 可用模型

| 模型 ID | 说明 |
|---|---|
| `gemini-3-pro` | 最新模型 *（默认）* |
| `gemini-2.5-pro` | 上一代 Pro 模型 |
| `gemini-2.5-flash` | 快速轻量模型 |

## Environment Variables / 环境变量

| 变量 | 说明 |
|---|---|
| `GEMINI_WEB_DATA_DIR` | 数据目录 |
| `GEMINI_WEB_COOKIE_PATH` | Cookie 文件路径 |
| `GEMINI_WEB_CHROME_PROFILE_DIR` | Chrome 用户数据目录 |
| `GEMINI_WEB_CHROME_PATH` | Chrome 可执行文件路径 |

## Platform Notes / 平台说明

<details>
<summary><strong>Windows</strong></summary>

在 Windows 上请使用 PowerShell 运行 bun 命令，bash 环境可能无法正确捕获 stdout：

```powershell
powershell.exe -Command "bun 'scripts/main.ts' '你的提示词'"
powershell.exe -Command "bun 'scripts/main.ts' --image 'output.png' -p '一条龙'"
```

</details>

## Acknowledgements / 致谢

本项目的 TypeScript 实现参考了 [gemini_webapi](https://github.com/HanaokaYuzu/Gemini-API)（Python 版）。

## License / 许可证

[MIT](LICENSE)
