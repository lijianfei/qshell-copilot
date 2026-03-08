<div align="center">

# qshell-copilot

**Manage Qiniu Cloud Storage with natural language in your AI-powered IDE.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![qshell](https://img.shields.io/badge/qshell-v2.x-green.svg)](https://github.com/qiniu/qshell)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](#-quick-start)
[![Benchmark](https://img.shields.io/badge/pass_rate-100%25-brightgreen.svg)](#-benchmark)

[English](README.md) | [中文](README.zh-CN.md)

</div>

---

An AI coding assistant prompt (Skill) that wraps [qshell](https://github.com/qiniu/qshell) — [Qiniu Cloud Storage (Kodo)](https://www.qiniu.com/products/kodo)'s official CLI — so you can manage cloud files through natural language instead of memorizing CLI syntax.

**Works with:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) · [Cursor](https://cursor.com) · [Windsurf](https://windsurf.com) · [Trae](https://trae.ai) · Any AI IDE that supports custom prompts

## 📋 Table of Contents

- [✨ Features](#-features)
- [⚙️ How It Works](#️-how-it-works)
- [🚀 Quick Start](#-quick-start)
- [📁 Project Structure](#-project-structure)
- [📊 Benchmark](#-benchmark)
- [📦 Requirements](#-requirements)
- [🤝 Contributing](#-contributing)

## ✨ Features

```
 "Upload this folder to my-bucket"            →  auto-selects qupload2, parallel threads
 "Show me what's in the images/ folder"       →  qshell listbucket2 with --prefix
 "Delete temp/debug.log from production"      →  shows file info first, waits for confirmation
 "Refresh CDN cache for these URLs"           →  writes URL file, runs cdnrefresh
 "Generate a private download link"           →  qshell privateurl with deadline
```

| Category | Operations |
|----------|-----------|
| 📤 **Upload** | Single file (auto fput/rput by size), batch directory upload, resume on failure |
| 📥 **Download** | Single file, batch download with config |
| 📂 **File Ops** | List, stat, delete (with safety confirmation), copy, move/rename |
| 🪣 **Bucket** | List all buckets, view domains, create new bucket with region |
| 🌐 **CDN** | Cache refresh (URLs & directories), prefetch, private download links |
| 🔗 **Network** | Fetch remote URL directly into bucket |

## ⚙️ How It Works

```
You say: "Upload ./assets/ to my-bucket with prefix v2/"
                           │
                           ▼
              ┌─────────────────────────┐
              │  1. Pre-flight Checks    │
              │  qshell --version  ✓     │
              │  qshell user ls    ✓     │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │  2. Smart Selection      │
              │  Directory detected      │
              │  → qshell qupload2       │
              │    --src-dir=./assets/   │
              │    --bucket=my-bucket    │
              │    --key-prefix=v2/      │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │  3. Result & Follow-up   │
              │  48/48 files uploaded ✓  │
              │  → qshell domains        │
              │  Access: https://...     │
              └─────────────────────────┘
```

### 🔒 Safety by Design

- **Delete protection** — Always runs `qshell stat` and shows file details before deletion. Waits for your explicit "yes".
- **Auth check** — Uses `qshell user ls` (read-only) instead of `qshell account` (which exposes credentials in terminal history).
- **Smart upload** — Checks file size to choose `fput` (<100MB) or `rput` (≥100MB, resumable). No wasted bandwidth on retry.

## 🚀 Quick Start

### 1. Install qshell

<details>
<summary><b>Windows</b></summary>

1. Download `qshell-windows-amd64.zip` from [GitHub Releases](https://github.com/qiniu/qshell/releases)
2. Extract and rename to `qshell.exe`, place in a directory (e.g. `C:\Tools\`)
3. Add to PATH: Win+S → search "Environment Variables" → Edit `Path` → Add the directory
4. Open a **new** terminal and verify:
   ```
   qshell --version
   ```

> ⚠️ Do NOT double-click `qshell.exe` — it's a command-line tool.

</details>

<details>
<summary><b>macOS</b></summary>

```bash
brew install qshell
```
Or download from [GitHub Releases](https://github.com/qiniu/qshell/releases).

</details>

<details>
<summary><b>Linux</b></summary>

```bash
wget https://github.com/qiniu/qshell/releases/download/v2.18.0/qshell-linux-amd64-v2.18.0.zip
unzip qshell-linux-amd64-v2.18.0.zip
chmod +x qshell && sudo mv qshell /usr/local/bin/
qshell --version
```

</details>

### 2. Configure credentials

```bash
qshell account <AccessKey> <SecretKey> <Name>
```

Get your AK/SK from [Qiniu Console → Key Management](https://portal.qiniu.com/user/key).

### 3. Add to your project

Copy `.claude/skills/qiniu/` into your project:

```
your-project/
└── .claude/
    └── skills/
        └── qiniu/
            ├── SKILL.md              ← Core skill (commands, workflows, error handling)
            └── references/
                ├── install-guide.md   ← Platform install instructions
                └── batch-download.md  ← Batch download config format
```

> 💡 **Using Cursor, Windsurf, or other AI IDEs?**
> Copy the content of `SKILL.md` into your project's custom rules file (`.cursorrules`, `.windsurfrules`, or equivalent).

## 📁 Project Structure

```
.claude/skills/qiniu/
├── SKILL.md              # Main skill — command reference, workflows, error handling
└── references/
    ├── install-guide.md   # Platform-specific installation instructions
    └── batch-download.md  # JSON config format for qdownload
```

## 📊 Benchmark

Evaluated across 4 real-world scenarios with 18 assertions:

```
                    With Skill          Without Skill
Upload File         ██████████ 100%     ██              20%
List Files          ██████████ 100%     ██▌             25%
Delete File         ██████████ 100%     ████            40%
Install Guide       ██████████ 100%     ███████▌        75%
────────────────────────────────────────────────────────────
Overall             ██████████ 100%     ████            40%
```

<details>
<summary><b>What does the skill fix?</b></summary>

| Without Skill (common mistakes) | With Skill (correct behavior) |
|---|---|
| Skips `qshell --version` check | Always verifies installation first |
| Uses `qshell account` (leaks creds) | Uses `qshell user ls` (safe read) |
| Deletes without confirmation | Shows file info, waits for explicit OK |
| Forgets `qshell domains` after upload | Auto-fetches domain, returns full URL |
| Uses wrong flags (`-v`, `--urls`) | Uses correct flags (`--version`, `-i`) |

</details>

## 📦 Requirements

- [qshell](https://github.com/qiniu/qshell/releases) v2.x+
- A [Qiniu Cloud](https://www.qiniu.com) account (AccessKey + SecretKey)
- An AI coding assistant that supports custom prompts/rules

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

## 📄 License

[MIT](LICENSE)
