<div align="center">

# qshell-copilot

**Manage Qiniu Cloud Storage with natural language in your AI-powered IDE or terminal.**

[English](#english) | [中文](#中文)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![qshell](https://img.shields.io/badge/qshell-v2.x-green.svg)](https://github.com/qiniu/qshell)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](#install-qshell)

</div>

---

<a name="english"></a>

## What is this?

An AI coding assistant prompt (Skill) that wraps [qshell](https://github.com/qiniu/qshell) — [Qiniu Cloud Storage (Kodo)](https://www.qiniu.com/products/kodo)'s official CLI tool — so you can manage cloud files through natural language instead of memorizing CLI syntax.

**Works with:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | [Cursor](https://cursor.com) | [Windsurf](https://windsurf.com) | [Trae](https://trae.ai) | Any AI IDE or terminal that supports custom prompts/rules

> Stop copy-pasting `qshell` commands from docs. Just say what you need.

## Features

```
 "Upload this folder to my-bucket"            →  auto-selects qupload2, parallel threads
 "Show me what's in the images/ folder"       →  qshell listbucket2 with --prefix
 "Delete temp/debug.log from production"      →  shows file info first, waits for confirmation
 "Refresh CDN cache for these URLs"           →  writes URL file, runs cdnrefresh
 "Generate a private download link"           →  qshell privateurl with deadline
```

| Category | Operations |
|----------|-----------|
| **Upload** | Single file (auto fput/rput by size), batch directory upload, resume on failure |
| **Download** | Single file, batch download with config |
| **File Ops** | List, stat, delete (with safety confirmation), copy, move/rename |
| **Bucket** | List all buckets, view domains, create new bucket with region |
| **CDN** | Cache refresh (URLs & directories), prefetch, private download links |
| **Network** | Fetch remote URL directly into bucket |

## How it works

```
You say: "Upload ./assets/ to my-bucket with prefix v2/"
                           │
                           ▼
              ┌─────────────────────────┐
              │   1. Pre-flight Checks   │
              │   qshell --version  ✓    │
              │   qshell user ls    ✓    │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │  2. Smart Selection      │
              │  Directory detected      │
              │  → qshell qupload2       │
              │    --src-dir=./assets/    │
              │    --bucket=my-bucket     │
              │    --key-prefix=v2/       │
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

### Safety by design

- **Delete protection** — Always runs `qshell stat` and shows file details before deletion. Waits for your explicit "yes" before executing.
- **Auth check** — Uses `qshell user ls` (read-only) instead of `qshell account` (which exposes credentials in terminal history).
- **Smart upload** — Checks file size to choose `fput` (<100MB) or `rput` (≥100MB, resumable). No wasted bandwidth on retry.

## Quick Start

### Install qshell

<details>
<summary><b>Windows</b></summary>

1. Download `qshell-windows-amd64.zip` from [GitHub Releases](https://github.com/qiniu/qshell/releases)
2. Extract and rename to `qshell.exe`, place in a directory (e.g. `C:\Tools\`)
3. Add to PATH: Win+S → search "Environment Variables" → Edit `Path` → Add the directory
4. Open a **new** terminal and verify:
   ```
   qshell --version
   ```

> Do NOT double-click `qshell.exe` — it's a command-line tool.

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

### Configure credentials

```bash
qshell account <AccessKey> <SecretKey> <Name>
```

Get your AK/SK from [Qiniu Console → Key Management](https://portal.qiniu.com/user/key).

### Add to your project

Copy `.claude/skills/qiniu/` into your project's `.claude/skills/` directory:

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

For **non-Claude** AI tools (Cursor, Windsurf, etc.), copy the content of `SKILL.md` into your project's custom rules file (`.cursorrules`, `.windsurfrules`, or equivalent).

## Benchmark

Evaluated across 4 real-world scenarios with 18 assertions total:

```
                    With Skill          Without Skill
Upload File         ██████████ 100%     ██              20%
List Files          ██████████ 100%     ██▌             25%
Delete File         ██████████ 100%     ████            40%
Install Guide       ██████████ 100%     ███████▌        75%
────────────────────────────────────────────────────────────
Overall             ██████████ 100%     ████            40%
```

**What the +60% delta comes from:**

| Without Skill (common mistakes) | With Skill (correct behavior) |
|---|---|
| Skips `qshell --version` check | Always verifies installation first |
| Uses `qshell account` (leaks creds) | Uses `qshell user ls` (safe read) |
| Deletes without confirmation | Shows file info, waits for explicit OK |
| Forgets `qshell domains` after upload | Auto-fetches domain and returns full URL |
| Uses wrong flags (`-v`, `--urls`) | Uses correct flags (`--version`, `-i`) |

## Requirements

- [qshell](https://github.com/qiniu/qshell/releases) v2.x+
- A [Qiniu Cloud](https://www.qiniu.com) account with AccessKey and SecretKey
- An AI coding assistant that supports custom prompts/rules

## License

[MIT](LICENSE)

---

<a name="中文"></a>

<div align="center">

## 中文说明

</div>

## 这是什么？

一个 AI 编程助手的 Prompt（Skill），封装了七牛云存储的官方命令行工具 [qshell](https://github.com/qiniu/qshell)，让你用自然语言管理云存储文件，不用记命令。

**兼容:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | [Cursor](https://cursor.com) | [Windsurf](https://windsurf.com) | [Trae](https://trae.ai) | 任何支持自定义 Prompt / Rules 的 AI IDE 或终端

## 功能一览

| 分类 | 支持的操作 |
|------|-----------|
| **上传** | 单文件（自动按大小选 fput/rput）、批量目录上传、断点续传 |
| **下载** | 单文件下载、批量下载（配置文件方式） |
| **文件操作** | 列出文件、查看详情、删除（安全确认）、复制、移动/重命名 |
| **Bucket** | 列出所有空间、查看绑定域名、创建新空间（指定区域） |
| **CDN** | 缓存刷新（URL 和目录）、预热、生成私有下载链接 |
| **网络** | 抓取远程 URL 直接存入 bucket |

## 使用示例

```
你说: "把 ./assets/ 文件夹上传到 my-bucket，前缀用 v2/"

Skill 自动执行:
  1. 检查 qshell 安装状态
  2. 检查认证状态
  3. 识别为目录 → 选择 qupload2 批量上传
  4. 上传完成 → 自动获取域名 → 返回访问链接
```

### 安全设计

- **删除保护** — 删除前先用 `qshell stat` 展示文件信息，等你确认后才执行
- **认证安全** — 用 `qshell user ls`（只读）检查认证，而不是 `qshell account`（会在终端历史中暴露密钥）
- **智能上传** — 自动判断文件大小选择上传方式：< 100MB 用 `fput`，≥ 100MB 用 `rput`（支持断点续传）

## 快速开始

### 1. 安装 qshell

**Windows:**
1. 从 [GitHub Releases](https://github.com/qiniu/qshell/releases) 下载 `qshell-windows-amd64.zip`
2. 解压，将 exe 重命名为 `qshell.exe`，放到固定目录（如 `C:\Tools\`）
3. 加入 PATH：Win+S 搜索「编辑系统环境变量」→ 编辑 Path → 新建 → 添加目录
4. 打开**新终端**验证：`qshell --version`

**macOS:** `brew install qshell` 或从 Releases 下载

**Linux:** 从 Releases 下载对应架构版本，`chmod +x` 后移到 `/usr/local/bin/`

### 2. 配置账号

```bash
qshell account <AccessKey> <SecretKey> <账号名>
```

AK/SK 在 [七牛控制台 → 密钥管理](https://portal.qiniu.com/user/key) 获取。

### 3. 添加到项目

将 `.claude/skills/qiniu/` 目录复制到你的项目中：

```
your-project/
└── .claude/
    └── skills/
        └── qiniu/
            ├── SKILL.md              ← 核心 Skill 文件
            └── references/
                ├── install-guide.md   ← 安装指南
                └── batch-download.md  ← 批量下载配置格式
```

**其他 AI 工具**（Cursor、Windsurf 等）：将 `SKILL.md` 的内容复制到项目的自定义规则文件中（`.cursorrules`、`.windsurfrules` 或同类文件）。

## 评测结果

在 4 个真实场景（上传、列表、删除、安装引导）中进行了 18 项断言测试：

| 指标 | 有 Skill | 无 Skill | 提升 |
|------|---------|---------|------|
| 通过率 | **100%** (18/18) | 40% (7/18) | **+60%** |

Skill 带来的关键改进：认证安全检查、删除前确认、正确的命令参数、上传后自动返回访问链接。

## 环境要求

- [qshell](https://github.com/qiniu/qshell/releases) v2.x+
- [七牛云](https://www.qiniu.com)账号（AccessKey + SecretKey）
- 任何支持自定义 Prompt / Rules 的 AI 编程助手

## 许可证

[MIT](LICENSE)
