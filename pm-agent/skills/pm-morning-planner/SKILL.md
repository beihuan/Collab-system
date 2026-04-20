---
name: pm-morning-planner
description: Generates daily morning task assignments for each team member based on Feishu Bitable task status. Use when triggered by morning cron job (08:00) or when user asks to "plan today's tasks", "generate task assignments", "assign tasks", or "morning planning". Creates structured assignment documents and sends them via Feishu group chat.
---

# PM Morning Planner

## Overview

This skill generates personalized task assignment documents for each team member every morning. It reads the current task status from the Bitable, considers priorities, dependencies, and each person's workload, then creates assignment documents and sends them to the Feishu group chat.

## Prerequisites

- `pm-bitable-manager` skill must be available
- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for configuration values
- Read `references/project-context.md` for project background understanding

## Workflow

### Step 1: Read Current Task Status

Use `pm-bitable-manager` to read all tasks from the Bitable:

```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID}
```

Analyze the returned data to identify:
- Tasks in "进行中" status (continue working)
- Tasks in "待开始" status that are ready to start (dependencies met)
- Tasks in "已阻塞" status (need attention)
- Overdue tasks (due_date < today AND status != "已完成")
- Tasks assigned to each person

### Step 2: Generate Task Assignments

For each team member, create a personalized assignment document:

**Assignment Document Template:**

```markdown
📋 任务指派 — {PERSON_NAME} — {DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 🔴 今日最高优先级

### [{TASK_ID}] {TITLE}
- **项目**: {PROJECT}
- **优先级**: {PRIORITY}
- **状态**: {STATUS} → 目标: {TARGET_STATUS}
- **描述**: {DETAILED_DESCRIPTION}
- **验收标准**:
  1. {CRITERION_1}
  2. {CRITERION_2}
  3. ...
- **依赖**: {DEPENDENCIES_STATUS}
  - [{DEP_TASK_ID}] {DEP_TITLE}: {DEP_STATUS}
- **截止日期**: {DUE_DATE} {OVERDUE_WARNING}
- **关联PR**: {PR_LINK_OR_NONE}
- **备注**: {NOTES_FROM_PM}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 🟡 进行中的任务

### [{TASK_ID}] {TITLE}
- **当前进度**: {PROGRESS}
- **今日目标**: {TODAY_GOAL}
- **截止日期**: {DUE_DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 🟢 待开始的任务（如时间允许）

### [{TASK_ID}] {TITLE}
- **优先级**: {PRIORITY}
- **描述**: {BRIEF_DESCRIPTION}
- **依赖**: {DEPS}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## ⚠️ 需要关注

- [{TASK_ID}] 已阻塞: {BLOCKER_DESCRIPTION}
- [{TASK_ID}] 即将到期: 截止日期 {DUE_DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 📝 PM备注

{PM_NOTES_BASED_ON_PROJECT_CONTEXT}
```

### Step 3: Human Confirmation

Present the assignment documents to the human (you, the PM) for review:

```
今日任务指派已生成，请审核：

Person A (你自己):
  - 🔴 [TASK-0001] 实现用户登录API (P0)
  - 🟡 [TASK-0005] 设计权限模块 (P1)
  - 🟢 [TASK-0012] 优化首页加载 (P2)

Person B (全栈开发):
  - 🔴 [TASK-0003] 完成支付集成 (P0)
  - 🟡 [TASK-0008] 修复搜索bug (P1)
  - ⚠️ [TASK-0006] 已阻塞: 等待第三方API文档

Person C (后端开发):
  - 🔴 [TASK-0002] 数据库迁移脚本 (P1)
  - 🟡 [TASK-0009] 优化查询性能 (P2)

确认发送？(y/n/edit)
- y: 发送所有指派
- n: 取消
- edit: 修改特定指派
```

### Step 4: Send Assignment Documents

After human confirmation, create Feishu documents and send to group chat:

```bash
# Create assignment document for each person
lark-cli doc create --title "任务指派 — {PERSON_NAME} — {DATE}" \
  --content "$(cat /tmp/assignment-{PERSON}.md)" \
  --folder-token {FOLDER_TOKEN}

# Send to group chat
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type interactive \
  --content '{"config":{"wide_screen_mode":true},"header":{"title":{"tag":"plain_text","content":"📋 任务指派 — {PERSON_NAME} — {DATE}"}},"elements":[{"tag":"markdown","content":"{SUMMARY}"}]}'
```

### Step 5: Log and Update

After sending, log the assignments:
- Record which tasks were assigned today
- Note any changes from the original plan
- Update the Bitable if needed (e.g., status changes from "待开始" to "进行中")

## Decision Rules for Task Assignment

### Priority Assignment Logic

1. **P0 tasks** always go first, assigned to the most appropriate person
2. **Blocked tasks** are flagged but not assigned as primary work
3. **Tasks with met dependencies** are prioritized over those with pending dependencies
4. **Person A (PM/You)**: Gets product/coordination tasks + full-stack dev tasks
5. **Person B (Full-stack)**: Gets development tasks across both projects
6. **Person C (Backend)**: Gets backend/database/infrastructure tasks only

### Workload Balancing

- Check each person's current task count and estimated hours
- Avoid assigning more than 6 hours of estimated work per day
- If one person is overloaded, consider reassigning P2/P3 tasks

### Dependency Resolution

- Never assign a task whose dependencies are incomplete
- If a dependency is "in progress", note it in the assignment with expected completion
- If a dependency is "blocked", flag it and suggest the person follow up

## Error Handling

| Scenario | Action |
|----------|--------|
| Bitable read fails | Retry once, then notify human |
| No tasks found for a person | Send a "no assigned tasks" message, suggest picking up backlog items |
| Human rejects all assignments | Ask for guidance on what to change |
| Feishu doc creation fails | Retry with simpler formatting, fall back to plain text message |

## Troubleshooting

- If assignments seem wrong, check `references/project-context.md` for project understanding
- If dependency logic seems off, verify the `dependencies` field format in Bitable
- If a person consistently gets too many/few tasks, adjust workload balancing parameters
