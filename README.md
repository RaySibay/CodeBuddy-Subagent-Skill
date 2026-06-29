# CodeBuddy Subagent Skill

中文 | [English](#english)

## 简介

`codebuddy-subagent` 是一个 Codex Desktop 自定义 Skill，用于让 Codex 将边界清晰、低风险、消耗上下文较多的工作委托给本地 CodeBuddy CLI 作为子 agent 执行。

适合场景包括代码库搜索、日志分析、CI 失败分析、diff review、文档迁移、结构化信息抽取、并行探索、简单隔离修改和后台长任务。

## 前置条件

- 已安装 Codex Desktop。
- 已安装 CodeBuddy CLI，并且命令行中可以运行 `codebuddy` 或 `cbc`。
- CodeBuddy CLI 已完成登录或可正常使用。

验证 CodeBuddy CLI：

```powershell
codebuddy --version
codebuddy -p "Reply with OK only." --output-format json --max-turns 1
```

## 安装

### Windows / PowerShell

将本仓库中的 `codebuddy-subagent` 文件夹复制到 Codex 的个人技能目录：

```powershell
$src = ".\codebuddy-subagent"
$dst = "$env:USERPROFILE\.codex\skills\codebuddy-subagent"

New-Item -ItemType Directory -Force $dst | Out-Null
Copy-Item -Recurse -Force "$src\*" $dst
```

然后重启 Codex Desktop，或新建一个 Codex 线程，让技能索引重新加载。

### macOS / Linux

```bash
mkdir -p "$HOME/.codex/skills/codebuddy-subagent"
cp -R ./codebuddy-subagent/* "$HOME/.codex/skills/codebuddy-subagent/"
```

然后重启 Codex Desktop，或新建一个 Codex 线程。

## 验证安装

确认目录结构如下：

```text
~/.codex/skills/codebuddy-subagent/
  SKILL.md
  agents/
    openai.yaml
```

在 Codex Desktop 中使用类似提示触发：

```text
用 CodeBuddy CLI 子 agent 帮我分析这个 CI 日志，主 agent 只汇总结论和下一步。
```

也可以让 Codex 检查已加载技能列表中是否出现 `codebuddy-subagent`。

## 使用示例

让 CodeBuddy 做只读调查：

```text
用 CodeBuddy CLI 子 agent 总结 src/auth 的入口、数据流、关键风险和需要人工检查的文件。不要修改代码。
```

让 CodeBuddy 分析大日志：

```text
用 CodeBuddy CLI 子 agent 分析 build-error.txt，返回根因、疑似文件和下一条最小验证命令。
```

让 CodeBuddy 做 diff review：

```text
用 CodeBuddy CLI 子 agent review 当前 git diff，只报告正确性 bug、缺失测试、安全风险和高风险行为变化。
```

让 CodeBuddy 执行简单隔离修改：

```text
用 CodeBuddy CLI 子 agent 给 parser 的这个边界条件补一个聚焦测试，只允许修改 tests/parser，运行目标测试后报告 diff。
```




---

## English

## Overview

`codebuddy-subagent` is a Codex Desktop custom skill that lets Codex delegate bounded, low-risk, token-heavy work to the local CodeBuddy CLI as a subagent.

Good fits include broad repository search, log analysis, CI failure analysis, diff review, documentation migration, structured extraction, parallel exploration, simple isolated edits, and background tasks.

## Prerequisites

- Codex Desktop is installed.
- CodeBuddy CLI is installed and available as `codebuddy` or `cbc`.
- CodeBuddy CLI is authenticated and usable from your terminal.

Verify CodeBuddy CLI:

```powershell
codebuddy --version
codebuddy -p "Reply with OK only." --output-format json --max-turns 1
```

## Installation

### Windows / PowerShell

Copy the `codebuddy-subagent` folder from this repository into your personal Codex skills directory:

```powershell
$src = ".\codebuddy-subagent"
$dst = "$env:USERPROFILE\.codex\skills\codebuddy-subagent"

New-Item -ItemType Directory -Force $dst | Out-Null
Copy-Item -Recurse -Force "$src\*" $dst
```

Restart Codex Desktop, or start a new Codex thread, so the skill index can reload.

### macOS / Linux

```bash
mkdir -p "$HOME/.codex/skills/codebuddy-subagent"
cp -R ./codebuddy-subagent/* "$HOME/.codex/skills/codebuddy-subagent/"
```

Restart Codex Desktop, or start a new Codex thread.

## Verify Installation

The installed structure should look like this:

```text
~/.codex/skills/codebuddy-subagent/
  SKILL.md
  agents/
    openai.yaml
```

Trigger it in Codex Desktop with a prompt like:

```text
Use CodeBuddy CLI as a subagent to analyze this CI log. The main agent should only summarize conclusions and next steps.
```

You can also ask Codex whether `codebuddy-subagent` appears in the loaded skills list.

## Usage Examples

Read-only investigation:

```text
Use CodeBuddy CLI as a subagent to summarize src/auth: entrypoints, data flow, key risks, and files to inspect. Do not edit files.
```

Large log analysis:

```text
Use CodeBuddy CLI as a subagent to analyze build-error.txt. Return the root cause, suspect files, and the smallest next verification command.
```

Diff review:

```text
Use CodeBuddy CLI as a subagent to review the current git diff. Report only correctness bugs, missing tests, security risks, and risky behavior changes.
```

Simple isolated edit:

```text
Use CodeBuddy CLI as a subagent to add a focused test for this parser edge case. Modify only tests/parser, run the targeted test, and report the diff.
```


