---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** If subagent support is available (ZCode custom agents defined under `.zcode/cli/agents/`), aq-workflow:subagent-driven-development usually produces higher-quality results — fresh subagent per task with two-stage review. Use this inline executing-plans skill when you want same-session batch execution instead.

## The Process

### Step 0: Decision Gate (轻量任务豁免)

Before executing or creating any physical plan file, evaluate if it is truly necessary:
- **物理 Plan 禁用场景**：不修改代码的纯审查/调研/分析任务（如 RFC 审查、方案讨论）；单文件局部修改、参数微调等 Trivial 任务；单次会话即可闭环的轻量级任务。
- **执行规则**：
  1. 若属于上述禁用场景，**强制采用 Lightweight 模式**（禁止创建物理 plan 文件，只需在 Context/当前回复中以 Markdown Task 列表跟踪步骤并执行）。
  2. 若符合，直接跳过 Step 1（不创建/不读取物理 Plan 文件），直接在上下文中列出任务步骤并开始执行。

### Step 1: Load and Review Plan
*(Only applies to Artifact mode - when a physical plan is required)*
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Read its stated **verification tier** from the task header (TDD / run-and-verify / decision memo)
2. Mark as in_progress
3. Follow that tier's step template exactly — do NOT impose TDD steps on a run-and-verify task, and do NOT skip verification on a TDD task
4. Run verifications as specified BY THAT TIER (check AGENTS.md for exact commands: venv interpreter path, `-m "not slow"` flag, sanity criteria)
5. Mark as completed

### Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use aq-workflow:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **aq-workflow:using-git-worktrees** - Ensures isolated workspace (creates one or verifies existing)
- **aq-workflow:writing-plans** - Creates the plan this skill executes
- **aq-workflow:finishing-a-development-branch** - Complete development after all tasks
