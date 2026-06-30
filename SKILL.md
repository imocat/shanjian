---
name: shanjian-cli
description: Install and safely operate the Shanjian CLI from GitHub Release binaries in Claude Code. Use when the user invokes /shanjian-cli, wants Claude Code to install shanjian, check login status, run Shanjian workflow commands, list templates/tasks/creations, download outputs, or package this skill for a GitHub repository that intentionally does not include CLI source code.
---

# Shanjian CLI

在 Claude Code 中安装和使用闪剪 CLI 时使用本技能。这个 GitHub 包只面向 Claude Code 安装使用，不包含 CLI 源码或工程文件。

## 默认行为

用户只输入 `/shanjian-cli` 或只点选本技能时，不要只解释本技能。直接执行以下流程：

1. 定位本技能目录，也就是当前 `SKILL.md` 所在目录。
2. 检查 `shanjian` 是否已在 PATH 中：运行 `shanjian --help` 或等价命令。
3. 如果已安装，运行 `shanjian auth status`，然后报告安装可用和登录状态。
4. 如果未安装，在 macOS/Linux 执行 `<skill_dir>/scripts/install-shanjian.sh --version latest --install-dir "$HOME/.local/bin"`；在 Windows PowerShell 执行 `<skill_dir>\scripts\install-shanjian.ps1 -Version latest -InstallDir "$HOME\.local\bin"`。
5. 安装后重新运行 `shanjian --help` 和 `shanjian auth status`。

如果执行脚本需要网络、写入用户目录或提升权限，使用工具请求许可；不要把安装命令作为最终答案停在说明层。

## 包内容

- `SKILL.md`：Claude Code 使用说明。
- `agents/openai.yaml`：技能 UI 元数据。
- `scripts/install-shanjian.sh`：macOS/Linux 安装脚本。
- `scripts/install-shanjian.ps1`：Windows PowerShell 安装脚本。

## 安装 CLI

优先使用本技能自带脚本安装 GitHub Release 二进制。默认仓库是 `shanjian-tv/shanjian-cli`；如果实际发布仓库不同，传 `--repo owner/name` 或设置 `SHANJIAN_CLI_REPO`。

macOS/Linux：

```bash
<skill_dir>/scripts/install-shanjian.sh --version latest --install-dir "$HOME/.local/bin"
```

Windows PowerShell：

```powershell
<skill_dir>\scripts\install-shanjian.ps1 -Version latest -InstallDir "$HOME\.local\bin"
```

安装脚本按当前系统下载匹配的 release asset，并在存在 `SHA256SUMS` 时校验哈希：

```text
shanjian_darwin_arm64.tar.gz
shanjian_darwin_amd64.tar.gz
shanjian_linux_amd64.tar.gz
shanjian_linux_arm64.tar.gz
shanjian_windows_amd64.zip
SHA256SUMS
```

安装完成后先检查：

```bash
shanjian --help
shanjian auth status
```

不要替用户执行 `shanjian auth login`，除非用户明确要求登录。

## 安全边界

- 把 `~/.shanjian/session.json` 和任何自定义 `--state-dir` 会话文件视为敏感文件。
- 不要打印、提交、复制或总结 `bhb-session-token`。
- `auth login` 会写入本地登录态；`auth logout` 会删除登录态。
- 所有 `create` 命令都会提交真实任务，除非带 `--dry-run`，否则可能消耗积分。
- 检查请求体时优先使用 `--dry-run`；对比接口结构时优先使用 `--json`。
- 下载命令会写文件；除非用户要求，不要把下载产物放进要发布的 skill 仓库。

## 常用命令

只读或低风险检查：

```bash
shanjian agent list
shanjian templates list --workflow-type shortVideo --json
shanjian creation short-video templates
shanjian creation ai-short-film models
shanjian creation ai-short-film prompts
shanjian tasks list
shanjian creations list
```

真实创建前先 dry-run：

```bash
shanjian creation moments create "朋友圈主题" --dry-run
shanjian creation article create "文章主题" --dry-run
shanjian creation image-text create "图文主题" --dry-run
shanjian creation short-video create "短视频主题" --duration 30 --dry-run
shanjian creation ai-short-film create "AI短剧主题" --image-url "https://example.com/ref.jpg" --dry-run
```

确认用户明确要提交后，再移除 `--dry-run`。

## 工作流注意

- 支持的工作流类型：`shortVideo`、`imageText`、`article`、`moments`、`aiShortFlim`。
- 服务端工作流拼写 `aiShortFlim` 是故意保留，不要改成 `aiShortFilm`。
- `--template-id` 指 `/ai_agent_user/template` 返回的记录 `id`，不是嵌套的 `templateId` 或 `styleId`。
- `ai-short-film create` 当前使用已有图片 URL；`--image-url` 可重复。

## 发布这个 Skill 包

发布到 GitHub 时只放这个 skill 包所需文件：

```text
SKILL.md
agents/openai.yaml
scripts/install-shanjian.sh
scripts/install-shanjian.ps1
```

不要把 CLI 源码、构建目录、下载产物或登录态放进该 GitHub 仓库。
