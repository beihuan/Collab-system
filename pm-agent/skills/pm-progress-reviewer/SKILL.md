---
name: pm-progress-reviewer
description: Reviews team progress reports from Feishu group chat, analyzes progress vs plan, identifies deviations and risks, and updates the Bitable accordingly. Use when triggered by evening cron job (19:00), when user asks to "review progress", "check status", "analyze progress", or when new progress reports are detected in the group chat.
---

# PM Progress Reviewer

## Overview

This skill reviews daily progress reports submitted by each developer agent, compares actual progress against the plan in the Bitable, identifies deviations and risks, and proposes Bitable updates for human confirmation.

## Prerequisites

- `pm-bitable-manager` skill must be available
- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for configuration values
- Read `references/project-context.md` for project understanding

## Workflow

### Step 1: Collect Progress Reports

Read the latest progress reports from the Feishu group chat:

```bash
# List recent messages in the group chat
lark-cli chat message list --chat-id {CHAT_ID} --limit 20
```

Identify progress report messages (they follow the structured format defined in `dev-progress-reporter` skill).

For each person, extract:
- Current tasks with progress
- Completed items
- Planned items for next period
- Risks/blockers
- Due date changes

### Step 2: Read Current Bitable Status

```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID}
```

### Step 3: Compare and Analyze

For each task mentioned in the progress reports:

1. **Find the matching Bitable record** by task_id
2. **Compare reported progress vs Bitable status**
3. **Identify deviations**:
   - Progress slower than expected
   - Unexpected blockers
   - Due date changes
   - Tasks not in the plan but being worked on
   - Planned tasks not mentioned in the report

4. **Risk assessment**:
   - Is any task likely to miss its deadline?
   - Are there cascading dependency risks?
   - Is anyone overloaded or underloaded?

### Step 4: Generate Analysis Report

Create a progress analysis report:

```markdown
📊 进度分析报告 — {DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 整体状态

| 项目 | 总任务 | 已完成 | 进行中 | 已阻塞 | 逾期 |
|------|--------|--------|--------|--------|------|
| 项目A | {N} | {N} | {N} | {N} | {N} |
| 项目B | {N} | {N} | {N} | {N} | {N} |

## 进度偏差

### ✅ 符合预期
- [TASK-0001] 进度 80%，按计划推进
- [TASK-0005] 已完成，符合预期

### ⚠️ 轻微偏差
- [TASK-0003] 进度 40%（预期 60%），原因：第三方API文档延迟
  - 建议：调整截止日期至 {NEW_DATE}

### 🔴 严重偏差
- [TASK-0007] 已阻塞，等待数据库迁移完成
  - 影响：[TASK-0012] 无法开始
  - 建议：将 Person C 的数据库迁移任务提升为 P0

### 🆕 计划外工作
- Person B 报告修复了一个紧急线上bug（未在任务看板中）
  - 建议：补充记录到任务看板

## 风险预警

1. **[高风险]** 项目A的登录模块可能延期，影响后续3个任务的启动
2. **[中风险]** Person B 本周工作量偏大（预估 45h），建议重新分配部分任务

## 建议操作

1. 将 [TASK-0007] 状态更新为"已阻塞"
2. 将 [TASK-0003] 截止日期从 {OLD_DATE} 调整为 {NEW_DATE}
3. 新增任务记录：修复线上bug
4. 将 [TASK-0012] 的依赖 [TASK-0007] 标记为阻塞中
```

### Step 5: Propose Bitable Updates

Based on the analysis, propose specific Bitable updates:

```
基于进度分析，建议更新以下多维表格记录：

状态变更:
  - [TASK-0001] 进度: 50% → 80%
  - [TASK-0003] 进度: 0% → 40%
  - [TASK-0005] 状态: 进行中 → 已完成
  - [TASK-0007] 状态: 进行中 → 已阻塞, 阻塞原因: 等待数据库迁移

⚠️ 截止日期变更:
  - [TASK-0003] 截止日期: 2026-04-20 → 2026-04-25

新增记录:
  - 修复线上支付bug (Person B, P1, 项目A)

确认以上更新？(y/n/edit)
```

### Step 6: Execute Updates

After human confirmation, use `pm-bitable-manager` to execute all approved updates.

### Step 7: Follow-up Questions

If any information is unclear or incomplete, generate Socratic questions:

```
我注意到以下信息需要进一步确认：

1. Person B 报告 [TASK-0003] 因"第三方API文档延迟"而进度落后。
   → 这个API文档是哪个服务商提供的？我们是否有备选方案？
   → 如果文档持续延迟，对项目A的里程碑有什么影响？
   → 是否需要我联系对方催促，或者调整技术方案？

2. [TASK-0007] 的阻塞原因是"等待数据库迁移"。
   → 数据库迁移的具体阻塞点是什么？是技术问题还是审批流程？
   → 迁移预计什么时候能完成？有没有临时绕过方案？
```

## Deviation Detection Rules

| Deviation Type | Detection Logic | Severity |
|---------------|----------------|----------|
| Progress behind | reported progress < expected progress by >20% | Medium |
| Progress far behind | reported progress < expected progress by >40% | High |
| Unplanned work | task mentioned but not in Bitable | Low |
| Missing report | no report from a person by 19:30 | High |
| New blocker | blocker mentioned not in Bitable | Medium |
| Due date change | reported ETA != Bitable due_date | Medium |
| Task not mentioned | Bitable task "进行中" but not in report | Medium |

## Error Handling

| Scenario | Action |
|----------|--------|
| No progress reports found | Notify human, suggest reminding the person |
| Task ID not found in Bitable | Flag as potential new task, ask human |
| Conflicting information | Ask human for clarification |
| Bitable update fails | Log error, retry once, then notify human |
