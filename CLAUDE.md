# qshell-copilot

A Claude Code Skill for managing Qiniu Cloud Storage (Kodo) via natural language.

## Features

- Upload, download, list, delete, copy, move files
- Bucket management (create, list, domains)
- CDN refresh and prefetch
- Private download link generation
- Automatic pre-flight checks (installation, authentication)

## Install

Place this project directory anywhere and open it with Claude Code.

Skill definition: `.claude/skills/qiniu/SKILL.md`

## Prerequisites

- **qshell**: Qiniu's official CLI tool. The skill guides you through installation on first use.
- **Qiniu account**: AccessKey and SecretKey from [Qiniu Console](https://portal.qiniu.com/user/key)

## Usage

Just describe what you need in natural language:

- "Upload ./image.png to my-bucket"
- "List all files in my-bucket"
- "Delete test.txt from my-bucket"
- "Copy a.jpg from my-bucket to backup-bucket"
- "Refresh CDN cache for this URL"
- "Batch upload this folder to my-bucket"

## Keywords

Qiniu, qiniu, qshell, Kodo, object storage, CDN, cloud storage, bucket
