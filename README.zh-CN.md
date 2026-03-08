<div align="center">

# qshell-copilot

**在 AI 编程助手中用自然语言管理七牛云存储。**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![qshell](https://img.shields.io/badge/qshell-v2.x-green.svg)](https://github.com/qiniu/qshell)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](#-快速开始)
[![Benchmark](https://img.shields.io/badge/通过率-100%25-brightgreen.svg)](#-评测结果)

[English](README.md) | [中文](README.zh-CN.md)

</div>

---

一个 AI 编程助手的 Prompt（Skill），封装了七牛云存储官方命令行工具 [qshell](https://github.com/qiniu/qshell)，让你用自然语言管理 [七牛 Kodo](https://www.qiniu.com/products/kodo) 云存储文件，不用记命令语法。

**兼容：** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) · [Cursor](https://cursor.com) · [Windsurf](https://windsurf.com) · [Trae](https://trae.ai) · 任何支持自定义 Prompt 的 AI IDE

## 📋 目录

- [✨ 功能一览](#-功能一览)
- [⚙️ 工作原理](#️-工作原理)
- [🚀 快速开始](#-快速开始)
- [📁 项目结构](#-项目结构)
- [📊 评测结果](#-评测结果)
- [📦 环境要求](#-环境要求)
- [🤝 参与贡献](#-参与贡献)

## ✨ 功能一览

```
 "把这个文件夹上传到 my-bucket"                →  自动选择 qupload2 批量上传
 "看看 images/ 目录下有什么文件"               →  qshell listbucket2 --prefix
 "删掉 production 里的 temp/debug.log"         →  先展示文件信息，等你确认
 "刷新这几个 URL 的 CDN 缓存"                  →  写入 URL 文件，执行 cdnrefresh
 "生成一个私有下载链接"                         →  qshell privateurl + deadline
 "帮我安装 qshell"                              →  根据系统引导安装配置
```

| 分类 | 支持的操作 |
|------|-----------|
| 📤 **上传** | 单文件（自动按大小选 fput/rput）、批量目录上传、覆盖上传、断点续传 |
| 📥 **下载** | 单文件下载、批量下载（配置文件方式） |
| 📂 **文件操作** | 列出、查看文件详情（stat）、删除（安全确认）、复制、移动/重命名 |
| 🪣 **Bucket** | 列出所有空间、查看绑定域名、创建新空间（支持 6 大区域） |
| 🌐 **CDN** | 缓存刷新（URL 和目录）、预热、生成私有下载链接 |
| 🔗 **网络** | 抓取远程 URL 直接存入 bucket |
| 🛠️ **环境配置** | 引导安装 qshell（Windows/macOS/Linux）、引导配置认证凭证 |
| 🧠 **智能辅助** | 远程路径自动推断、按文件大小选上传方式、错误诊断与修复引导 |

## ⚙️ 工作原理

```
你说: "把 ./assets/ 上传到 my-bucket，前缀用 v2/"
                           │
                           ▼
              ┌─────────────────────────┐
              │  1. 前置检查             │
              │  qshell --version  ✓     │
              │  qshell user ls    ✓     │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │  2. 智能选择命令         │
              │  检测到目录              │
              │  → qshell qupload2       │
              │    --src-dir=./assets/   │
              │    --bucket=my-bucket    │
              │    --key-prefix=v2/      │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │  3. 结果处理             │
              │  48/48 文件上传完成 ✓    │
              │  → qshell domains        │
              │  访问链接: https://...   │
              └─────────────────────────┘
```

### 🔒 安全设计

- **删除保护** — 删除前先用 `qshell stat` 展示文件信息，等你明确确认后才执行
- **认证安全** — 用 `qshell user ls`（只读）检查认证，不用 `qshell account`（会在终端历史中暴露密钥）
- **智能上传** — 自动判断文件大小：< 100MB 用 `fput`，≥ 100MB 用 `rput`（支持断点续传）
- **错误诊断** — 识别 7 种常见错误模式（认证失效、空间不存在、文件已存在等），自动引导修复而非展示原始报错
- **路径推断** — 未指定远程路径时自动用本地文件名；指定了目录前缀（如 `images/`）时自动拼接为 `images/文件名`

## 🚀 快速开始

### 1. 安装 qshell

<details>
<summary><b>Windows</b></summary>

1. 从 [GitHub Releases](https://github.com/qiniu/qshell/releases) 下载 `qshell-windows-amd64.zip`
2. 解压，将 exe 重命名为 `qshell.exe`，放到固定目录（如 `C:\Tools\`）
3. 加入 PATH：Win+S 搜索「编辑系统环境变量」→ 编辑 Path → 新建 → 添加目录
4. 打开**新终端**验证：
   ```
   qshell --version
   ```

> ⚠️ 不要双击 `qshell.exe` 运行，它是命令行工具。

</details>

<details>
<summary><b>macOS</b></summary>

```bash
brew install qshell
```
或从 [GitHub Releases](https://github.com/qiniu/qshell/releases) 下载。

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
            ├── SKILL.md              ← 核心 Skill（命令参考、工作流、错误处理）
            └── references/
                ├── install-guide.md   ← 各平台安装指南
                └── batch-download.md  ← 批量下载配置格式
```

> 💡 **用的是 Cursor、Windsurf 或其他 AI IDE？**
> 把 `SKILL.md` 的内容复制到项目的自定义规则文件中（`.cursorrules`、`.windsurfrules` 等）。

## 📁 项目结构

```
.claude/skills/qiniu/
├── SKILL.md              # 核心 Skill — 命令参考、工作流、错误处理
└── references/
    ├── install-guide.md   # 各平台安装说明
    └── batch-download.md  # qdownload 批量下载的 JSON 配置格式
```

## 📊 评测结果

在 4 个真实场景中进行了 18 项断言测试：

```
                    有 Skill            无 Skill
上传文件            ██████████ 100%     ██              20%
列出文件            ██████████ 100%     ██▌             25%
删除文件            ██████████ 100%     ████            40%
安装引导            ██████████ 100%     ███████▌        75%
────────────────────────────────────────────────────────────
总计                ██████████ 100%     ████            40%
```

<details>
<summary><b>Skill 解决了哪些问题？</b></summary>

| 无 Skill（常见错误） | 有 Skill（正确行为） |
|---|---|
| 跳过 `qshell --version` 检查 | 每次先验证安装状态 |
| 用 `qshell account` 检查认证（泄露密钥） | 用 `qshell user ls`（安全只读） |
| 不确认直接删除文件 | 展示文件信息，等待明确确认 |
| 上传后不获取访问链接 | 自动 `qshell domains` 拼出完整 URL |
| 用错参数（`-v`、`--urls`） | 用正确参数（`--version`、`-i`） |

</details>

## 📦 环境要求

- [qshell](https://github.com/qiniu/qshell/releases) v2.x+
- [七牛云](https://www.qiniu.com)账号（AccessKey + SecretKey）
- 任何支持自定义 Prompt / Rules 的 AI 编程助手

## 🤝 参与贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

[MIT](LICENSE)
