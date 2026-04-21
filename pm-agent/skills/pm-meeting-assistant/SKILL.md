---
name: pm-meeting-assistant
description: Assists with daily standup meetings by generating pre-meeting progress summaries and processing post-meeting minutes. Use when triggered by pre-meeting cron (09:30), when user asks to "prepare meeting", "generate meeting summary", "process meeting notes", or "meeting agenda", or when meeting minutes are provided.
---

# PM Meeting Assistant

## Overview

This skill supports the daily standup meeting workflow in three phases:
1. **Pre-meeting**: Generate a concise progress summary and agenda
2. **During meeting**: (Human-led, no Agent action needed)
3. **Post-meeting**: Process meeting minutes, update tasks and project context

## Prerequisites

- `pm-bitable-manager` skill must be available
- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for configuration values

## Workflow

### Phase 1: Pre-Meeting Summary (09:30 Cron)

#### Step 1: Read Current Status

```bash
lark-cli bitable record list --app-token {APP_TOKEN} --table-id {TABLE_ID}
```

#### Step 2: Generate Meeting Agenda

```markdown
📋 每日对齐会议 — {DATE} {TIME}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 项目A 进度概览

### 昨日完成
- ✅ [TASK-0001] 用户登录API — Person A
- ✅ [TASK-0005] 权限模块设计 — Person B

### 进行中
- 🔄 [TASK-0003] 支付集成 (60%) — Person B
- 🔄 [TASK-0002] 数据库迁移 (40%) — Person C

### 阻塞/风险
- 🚫 [TASK-0007] 等待第三方API文档 — Person B
- ⚠️ [TASK-0012] 可能受 [TASK-0007] 影响

### 今日重点
- [TASK-0003] 支付集成需完成沙箱联调
- [TASK-0002] 数据库迁移需在下午前完成

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 项目B 进度概览

[同上格式]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 需要讨论的事项

1. [TASK-0007] 第三方API文档延迟的处理方案
2. Person B 工作量是否需要重新分配
3. [其他需要讨论的事项]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 各人今日计划

### Person A
- 继续开发 [TASK-0001] 的单元测试
- 产品评审会议 14:00

### Person B
- 完成 [TASK-0003] 支付集成
- 开始 [TASK-0008] 搜索功能修复

### Person C
- 完成 [TASK-0002] 数据库迁移
- 协助 [TASK-0009] 查询性能优化
```

#### Step 3: Send to Group Chat

```bash
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type interactive \
  --content '{"config":{"wide_screen_mode":true},"header":{"title":{"tag":"plain_text","content":"📋 每日对齐会议 — {DATE}"}},"elements":[{"tag":"markdown","content":"{AGENDA_SUMMARY}"}]}'
```

Also present the full agenda to the human for review before the meeting.

### Phase 2: Post-Meeting Processing

After the meeting, the human provides meeting minutes (text or document). Process them as follows:

#### Step 1: Receive Meeting Minutes

The human can provide minutes in any format:
- Direct text input in the OpenClaw session
- A Feishu document link
- A Feishu message forwarded to the PM Agent

#### Step 2: Extract Action Items

Parse the minutes to identify:
- **Task changes**: New tasks, status changes, priority changes
- **Decisions**: Important decisions made during the meeting
- **Action items**: Specific follow-up actions with owners
- **Context updates**: New information about the project
- **Deadline changes**: Adjusted due dates
- **Blocker resolutions**: How blockers were resolved or escalated

#### Step 3: Propose Updates

Present all extracted action items for human confirmation:

```
从会议纪要中提取到以下待执行事项：

📋 新增任务:
  1. [项目A] 调研微信支付V3接口 — Person B, P1, 截止 4/25
  2. [项目B] 设计数据导出功能 — Person A, P2, 截止 4/28

📝 状态变更:
  - [TASK-0007] 解除阻塞: 改用备选API方案
  - [TASK-0003] 截止日期: 4/20 → 4/23

🧠 项目理解更新:
  - 项目A决定放弃Google OAuth，仅支持微信登录
  - 项目B的数据导出需要支持CSV和Excel两种格式

确认执行？(y/n/edit)
```

#### Step 4: Execute Updates

After confirmation:
1. Use `pm-bitable-manager` to update/create task records
2. Update `references/project-context.md` with new project understanding
3. Send a summary of changes to the Feishu group chat

#### Step 5: Socratic Follow-up

If the meeting minutes contain ambiguous or incomplete information:

```
会议纪要中有几点我需要进一步了解：

1. 纪要提到"改用备选API方案"解决 [TASK-0007] 的阻塞。
   → 这个备选方案具体是什么？与原方案相比有什么取舍？
   → 是否需要调整验收标准？
   → 对项目A的上线时间线有什么影响？

2. 纪要提到"Person B工作量需要调整"。
   → 具体哪些任务需要转移？转移给谁？
   → 调整后的优先级排序是什么？
```

## Meeting Minutes Format Guide

When the human provides meeting minutes, accept any format but prefer structured input:

**Preferred format:**
```markdown
## 会议纪要 — {DATE}

### 讨论事项
1. [话题1]: [结论]
2. [话题2]: [结论]

### 决策
1. [决策1]
2. [决策2]

### 行动项
1. [ ] [行动1] — 负责人: [Person] — 截止: [Date]
2. [ ] [行动2] — 负责人: [Person] — 截止: [Date]

### 遗留问题
1. [问题1]
```

**Free-form format is also accepted** — the skill will extract structured information from any text.

## Error Handling

| Scenario | Action |
|----------|--------|
| No meeting minutes provided | Skip post-meeting processing, remind human next morning |
| Ambiguous action items | Ask human for clarification via Socratic questions |
| Conflicting information with Bitable | Flag the conflict, ask human which is correct |
| Human not available for confirmation | Save proposed updates as draft, retry next morning |
