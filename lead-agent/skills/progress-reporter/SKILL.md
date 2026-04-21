---
name: progress-reporter
description: Generates structured daily progress reports and submits them to the Feishu group chat. Use when triggered by evening cron job (18:00), when user asks to "report progress", "submit daily report", "write progress update", or "end of day report". Collects work done today, current task status, blockers, and plans for next period.
---

# Dev Progress Reporter

## Overview

This skill helps developers generate structured progress reports at the end of each work period. It collects information from the current OpenClaw session, git activity, and human input to create a comprehensive report that references task IDs from the Bitable.

## Prerequisites

- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for configuration values
- Read `references/identity.md` for your name and role

## Workflow

### Step 1: Collect Information

#### 1a. Read Today's Task Assignments

Check the Feishu group chat for today's task assignment document:

```bash
lark-cli chat message list --chat-id {CHAT_ID} --limit 20
```

Identify the task assignment message for today.

#### 1b. Collect Git Activity

```bash
# Get today's commits
cd {REPO_PATH}
git log --author="{GIT_AUTHOR}" --since="today" --oneline

# Get today's changed files
git diff --stat HEAD~{N}

# Get open PRs
gh pr list --author="{GITHUB_USER}" --state open 2>/dev/null || echo "gh CLI not available"
```

#### 1c. Ask Human for Input

Present the collected information and ask the human to fill in the gaps:

```
我已收集到以下今日活动信息：

📝 Git提交:
  - abc1234: 实现用户登录JWT验证
  - def5678: 添加登录API单元测试

📂 变更文件:
  - src/auth/jwt.ts (+45/-12)
  - src/api/login.ts (+89/-0)
  - tests/auth/login.test.ts (+67/-0)

🔀 开放PR:
  - PR #42: feat: 用户登录API (待审核)

📋 今日任务指派:
  - [TASK-0001] 实现用户登录API (P0)
  - [TASK-0005] 设计权限模块 (P1)

请补充以下信息：
1. [TASK-0001] 当前进度百分比？(0%/25%/50%/75%/90%/100%)
2. [TASK-0005] 是否已开始？进度如何？
3. 有没有遇到阻塞或风险？
4. 明天计划做什么？
5. 有没有需要PM协调的事项？
```

### Step 2: Generate Progress Report

Based on the collected information, generate a structured report:

```markdown
📊 工作进度汇报 — {PERSON_NAME} — {DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 当前任务

### [TASK-0001] 实现用户登录API
- **进度**: 75% → 90%
- **状态**: 进行中
- **今日完成**: JWT验证逻辑、登录API单元测试
- **关联PR**: PR #42 (待审核)
- **Git提交**: abc1234, def5678

### [TASK-0005] 设计权限模块
- **进度**: 0% → 25%
- **状态**: 进行中
- **今日完成**: 完成RBAC模型初步设计文档

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 本周期已完成事项

- [TASK-0015] 修复支付回调bug — 已合并 PR #40
- 完成了数据库索引优化，查询性能提升3倍（计划外工作）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 下周期计划事项

- [TASK-0001] 完成用户登录API的集成测试，提交最终PR
- [TASK-0005] 完成权限模块数据模型设计，与后端同学对齐
- [TASK-0023] 开始设计权限管理API接口（如时间允许）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 风险/卡点

- [TASK-0001] 第三方OAuth服务商的沙箱环境不稳定，微信OAuth回调偶发超时
  - 影响：可能延迟联调完成时间
  - 需要PM协调：是否需要联系服务商确认沙箱稳定性？

- 数据库测试环境磁盘空间不足，影响本地测试
  - 需要 Person C 协助扩容

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 预计完成时间变更

- [TASK-0001] 原计划 4/20 完成 → 因OAuth联调问题预计延迟至 4/22
```

### Step 3: Human Confirmation

Present the report to the human for review:

```
进度报告已生成，请审核：

📊 今日完成: 2个任务推进, 1个任务完成
⚠️ 风险: 2项（OAuth沙箱不稳定、测试环境磁盘不足）
📅 时间变更: [TASK-0001] 延迟2天

确认发送到协作群组？(y/n/edit)
```

### Step 4: Submit Report

After human confirmation, send to the Feishu group chat:

```bash
# Create a Feishu document with the report
lark-cli doc create --title "进度汇报 — {PERSON_NAME} — {DATE}" \
  --content "$(cat /tmp/progress-report.md)" \
  --folder-token {FOLDER_TOKEN}

# Send summary to group chat
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type interactive \
  --content '{"config":{"wide_screen_mode":true},"header":{"title":{"tag":"plain_text","content":"📊 进度汇报 — {PERSON_NAME} — {DATE}"}},"elements":[{"tag":"markdown","content":"{SUMMARY}"}]}'
```

## Report Structure Guidelines

The report should be:
- **Task-centric**: Always reference Bitable task IDs
- **Factual**: State what was done, not what was intended
- **Quantitative**: Use progress percentages, commit counts, PR links
- **Honest**: Don't hide blockers or delays
- **Forward-looking**: Include clear plans for next period

## Auto-Detection Rules

| Signal | Auto-Detection |
|--------|---------------|
| Git commits today | Auto-associate with tasks based on branch name or commit message |
| PR created/updated | Auto-include in report with link |
| Task assignment received | Auto-include in "current tasks" section |
| No git activity | Ask human if they worked on non-code tasks |

## Error Handling

| Scenario | Action |
|----------|--------|
| No task assignment found | Report on whatever was worked on, flag as potentially unplanned |
| Git repo not accessible | Skip git activity collection, rely on human input |
| Human not available for confirmation | Save draft, remind human when they return |
| Feishu message send fails | Retry once, then save locally and try again in 5 minutes |
