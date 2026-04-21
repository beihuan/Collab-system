---
name: dev-task-receiver
description: Receives and presents task assignments from PM Agent to developers. Use when triggered by morning cron job (09:00), when a new task assignment is detected in the Feishu group chat, or when user asks to "check my tasks", "show today's tasks", or "what should I work on". Reads assignment documents and presents them in a clear, actionable format.
---

# Dev Task Receiver

## Overview

This skill receives task assignments from the PM Agent (sent via Feishu group chat) and presents them to the developer in a clear, actionable format. It also helps the developer acknowledge receipt and ask clarifying questions.

## Prerequisites

- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for configuration values
- Read `references/identity.md` for your name and role

## Workflow

### Step 1: Check for New Task Assignments

```bash
# Check recent messages in the group chat
lark-cli chat message list --chat-id {CHAT_ID} --limit 20
```

Look for messages from the PM Agent bot that contain task assignments for today.

### Step 2: Read Assignment Document

If the message contains a document link, read the full document:

```bash
lark-cli doc read --doc-id {DOC_ID}
```

### Step 3: Present Tasks to Human

Display the tasks in a clear, prioritized format:

```
📬 今日任务指派 — {DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 P0 — 紧急 (必须今日完成)

  [TASK-0001] 实现用户登录API
  ├─ 项目: 项目A
  ├─ 截止: 4/20 (今天!)
  ├─ 验收标准:
  │  1. 邮箱登录返回有效JWT
  │  2. 微信OAuth回调正常
  │  3. 单测覆盖率>80%
  ├─ 依赖: TASK-0005 ✅已完成
  └─ 备注: 客户4/25要看demo，此任务是前置条件

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🟡 P1 — 高优先级

  [TASK-0005] 设计权限模块
  ├─ 项目: 项目A
  ├─ 截止: 4/25
  ├─ 验收标准:
  │  1. RBAC数据模型文档已评审
  │  2. API接口列表已定义
  └─ 依赖: TASK-0001 (进行中)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔵 P2 — 中优先级

  [TASK-0023] 数据导出功能
  ├─ 项目: 项目B
  ├─ 截止: 4/28
  └─ 备注: 如时间允许再开始

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️ 注意事项
- TASK-0001 是今日最高优先级
- 后端同学已确认测试环境数据库就绪

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

请确认是否接受以上任务安排？(y/n/疑问)
```

### Step 4: Handle Human Response

**If accepted (y):**
- Send acknowledgment to the Feishu group chat
- No further action needed

**If rejected (n):**
- Ask which tasks are problematic and why
- Send feedback to PM Agent via group chat:
  ```
  @PM-Agent 关于今日任务指派，{PERSON_NAME} 有以下反馈：
  - [TASK-XXXX] 无法接受，原因: {REASON}
  请重新评估任务分配。
  ```

**If questions (疑问):**
- Help the developer formulate questions
- Send questions to PM Agent via group chat

### Step 5: Acknowledge Receipt

Send a brief acknowledgment to the group chat:

```bash
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type text \
  --content "✅ {PERSON_NAME} 已收到今日任务指派，共{N}个任务，预计可完成{M}个"
```

## Task Presentation Guidelines

1. **Always sort by priority**: P0 first, then P1, P2, P3
2. **Show dependencies clearly**: Indicate if a dependency is complete, in progress, or blocked
3. **Highlight deadlines**: Especially if a task is due today or tomorrow
4. **Include acceptance criteria**: So the developer knows exactly what "done" looks like
5. **Include context**: Why this task matters, how it connects to the bigger picture

## Integration with Other Skills

- After receiving tasks, the developer can start working
- `dev-code-activity-tracker` will monitor git activity against assigned tasks
- `dev-blocker-notifier` will trigger if a blocker is encountered
- `dev-progress-reporter` will reference these tasks in the evening report
