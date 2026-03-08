# qshell-copilot

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill for managing [Qiniu Cloud Storage (Kodo)](https://www.qiniu.com/products/kodo) through natural language.

Instead of memorizing `qshell` CLI syntax, just describe what you want — the skill handles pre-flight checks, command selection, error recovery, and result formatting automatically.

## What it does

| Capability | Example prompt |
|------------|---------------|
| Upload files | "Upload ./photo.jpg to my-bucket" |
| Batch upload | "Upload this entire folder to my-bucket" |
| Download files | "Download logo.png from my-bucket" |
| List files | "List all images in my-bucket" |
| Delete files | "Delete temp/debug.log from my-bucket" |
| Copy / Move | "Copy banner.png from bucket-a to bucket-b" |
| Bucket management | "List all my buckets" |
| CDN refresh | "Refresh CDN cache for this URL" |
| Private links | "Generate a private download link for this file" |

## How it works

Every request goes through a 3-step flow:

```
1. Pre-flight checks
   qshell --version  → installed?
   qshell user ls    → authenticated?

2. Smart command selection
   Single file < 100MB  → qshell fput
   Single file ≥ 100MB  → qshell rput (resumable)
   Directory             → qshell qupload2 (batch)

3. Result formatting
   Upload → auto-fetch domain → return access URL
   Delete → show file info → wait for confirmation
   Error  → diagnose + guide recovery
```

## Install

### 1. Install qshell

**Windows:**
1. Download `qshell-windows-amd64.zip` from [GitHub Releases](https://github.com/qiniu/qshell/releases)
2. Extract and rename to `qshell.exe`
3. Add to PATH (Win+S → "Environment Variables" → Edit Path → Add directory)
4. Verify: `qshell --version`

**macOS:**
```bash
brew install qshell
# or download from GitHub Releases
```

**Linux:**
```bash
# Download from GitHub Releases, choose your architecture
chmod +x qshell && sudo mv qshell /usr/local/bin/
```

### 2. Configure credentials

```bash
qshell account <AccessKey> <SecretKey> <Name>
```

Get your AK/SK from [Qiniu Console → Key Management](https://portal.qiniu.com/user/key).

### 3. Add the skill to your project

Copy the `.claude/skills/qiniu/` directory into your project:

```
your-project/
└── .claude/
    └── skills/
        └── qiniu/
            ├── SKILL.md
            └── references/
                ├── install-guide.md
                └── batch-download.md
```

Then open your project with Claude Code — the skill triggers automatically when you mention Qiniu, qshell, bucket, or cloud storage operations.

## Skill structure

```
.claude/skills/qiniu/
├── SKILL.md              # Main skill file (command reference, workflows, error handling)
└── references/
    ├── install-guide.md   # Platform-specific installation instructions
    └── batch-download.md  # JSON config format for qdownload
```

## Benchmark

Evaluated with 4 test scenarios (Upload, List, Delete, Install Guide), comparing with-skill vs without-skill performance:

| Metric | With Skill | Without Skill | Delta |
|--------|-----------|---------------|-------|
| Pass Rate | **100%** (18/18) | 40% (7/18) | **+60%** |

Key improvements the skill provides:
- **Authentication**: Uses `qshell user ls` instead of raw `qshell account` (avoids credential exposure)
- **Upload safety**: Checks file size to choose `fput` vs `rput`, auto-fetches access URL via `qshell domains`
- **Delete safety**: Always runs `qshell stat` first and waits for explicit user confirmation
- **Correct flags**: Uses `--version` (not `-v`), `-i` for CDN refresh (not `--urls`)

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI or IDE extension)
- [qshell](https://github.com/qiniu/qshell/releases) v2.x+
- Qiniu Cloud account with AccessKey and SecretKey

## License

MIT
