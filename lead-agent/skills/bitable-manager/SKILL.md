---
name: bitable-manager
description: Manages Feishu Bitable (多维表格) CRUD operations for project task management. Use when you need to read, create, update, or delete task records in the project management bitable. Supports tiered human confirmation (L0 auto-read, L2 confirm for updates, L3 double-confirm for high-risk changes).
---

# PM Bitable Manager

## Overview

This skill encapsulates all Feishu Bitable operations for the project management system. It provides a structured interface for reading and writing task data, with built-in human confirmation mechanisms based on operation risk level.

## Prerequisites

- Feishu CLI (`lark-cli`) installed and authenticated with robot identity
- Bitable app created with the correct table structure (see `references/bitable-schema.md`)
- Configuration in `references/feishu-config.md` must be completed before use

## Configuration

Before using this skill, read `references/feishu-config.md` to get:
- `APP_TOKEN`: Bitable application token
- `TABLE_ID`: Task table ID
- `CHAT_ID`: Collaboration group chat ID

## Operations

### Read Operations (L0 - Auto, No Confirmation)

#### List All Tasks
```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID}
```

#### Filter Tasks by Assignee
```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --filter '{"assignee": "{PERSON_NAME}"}'
```

#### Filter Tasks by Status
```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --filter '{"status": "已阻塞"}'
```

#### Filter Overdue Tasks
```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --filter '{"due_date": {"$lt": "{today}"}, "status": {"$ne": "已完成"}}'
```

#### Get Single Task
```bash
lark-cli bitable record get --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --record-id {RECORD_ID}
```

### Create Operations (L2 - Confirm Before Execution)

When creating a new task record, you MUST:

1. Present the full task details to the human for review
2. Wait for explicit confirmation (y/n/edit)
3. Only execute after confirmation

```bash
lark-cli bitable record create --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --fields '{
    "project": "{PROJECT}",
    "title": "{TITLE}",
    "description": "{DESCRIPTION}",
    "assignee": "{ASSIGNEE}",
    "status": "待开始",
    "priority": "{PRIORITY}",
    "dependencies": "{DEPS}",
    "due_date": "{DUE_DATE}",
    "acceptance_criteria": "{CRITERIA}",
    "milestone": "{MILESTONE}",
    "estimated_hours": {HOURS},
    "progress": "0%",
    "risk_level": "无风险"
  }'
```

**Confirmation Template:**
```
即将创建新任务：
  标题: {TITLE}
  项目: {PROJECT}
  负责人: {ASSIGNEE}
  优先级: {PRIORITY}
  截止日期: {DUE_DATE}
  预估工时: {HOURS}小时
确认创建？(y/n/edit)
```

### Update Operations (L2/L3 - Confirm Before Execution)

#### L2 Updates (Standard Confirmation)
- Status changes (待开始 → 进行中, 进行中 → 待评审, etc.)
- Progress updates
- PR link updates
- Actual hours updates

```bash
lark-cli bitable record update --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --record-id {RECORD_ID} --fields '{"status": "{NEW_STATUS}", "progress": "{NEW_PROGRESS}"}'
```

**Confirmation Template:**
```
即将更新多维表格中的以下记录：
  - [{TASK_ID}] {FIELD}: {OLD_VALUE} → {NEW_VALUE}
确认？(y/n/edit)
```

#### L3 Updates (Double Confirmation Required)
- Due date changes
- Priority changes
- Assignee changes
- Marking tasks as blocked
- Deleting records

**L3 Confirmation Template:**
```
⚠️ 高风险操作 — 需要二次确认
  - [{TASK_ID}] {FIELD}: {OLD_VALUE} → {NEW_VALUE}
  原因: {REASON}
  影响: {IMPACT_DESCRIPTION}
确认此变更？(yes/no)
```

### Delete Operations (L3 - Double Confirmation)

```bash
lark-cli bitable record delete --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --record-id {RECORD_ID}
```

**Always explain WHY the record is being deleted and what the impact is.**

## Batch Operations

When updating multiple records (e.g., after reviewing all progress reports):

1. Collect all pending changes
2. Present a summary of ALL changes together
3. Get single confirmation for the batch (or allow editing individual items)
4. Execute all confirmed changes

**Batch Confirmation Template:**
```
即将批量更新多维表格（共{N}条记录）：

状态变更:
  - [TASK-0001] 状态: 进行中 → 待评审
  - [TASK-0015] 状态: 待开始 → 已完成
  - [TASK-0023] 状态: 待开始 → 进行中, 进度: 20%

⚠️ 高风险变更:
  - [TASK-0007] 截止日期: 2026-04-20 → 2026-04-25 (延迟5天)

确认以上所有变更？(y/n/edit/split)
- y: 确认全部
- n: 取消全部
- edit: 修改特定项
- split: 分别确认每项
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `Connection refused` | Feishu CLI not authenticated | Run `lark-cli auth login` |
| `Permission denied` | Robot lacks bitable access | Add robot as bitable collaborator |
| `Record not found` | Invalid record_id | Re-query to get correct ID |
| `Field type mismatch` | Wrong data type for field | Check `references/bitable-schema.md` for field types |
| `Rate limit exceeded` | Too many API calls | Wait 60 seconds and retry |

## Troubleshooting

1. **If bitable operations fail**: First verify authentication with `lark-cli auth status`
2. **If field values are rejected**: Check that enum values match exactly (e.g., "P0-紧急" not "P0")
3. **If batch operations partially fail**: Log which records succeeded and failed, report to human
