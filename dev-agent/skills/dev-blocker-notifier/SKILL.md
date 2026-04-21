---
name: dev-blocker-notifier
description: Detects and reports blockers encountered during development work, proactively notifying the PM Agent via Feishu group chat. Use when a developer encounters a blocker, when user says "I'm blocked", "stuck on", "can't proceed", "need help", or when the heartbeat detects no progress on an in-progress task for an extended period.
---

# Dev Blocker Notifier

## Overview

This skill enables developer agents to proactively notify the PM Agent when they encounter blockers. It supports both manual triggering (when the developer tells their agent they're blocked) and automatic detection (when no progress is detected on an in-progress task).

**重要原则**：开发者Agent不直接操作多维表格。所有表格更新（如标记任务为"已阻塞"）由PM Agent负责。开发者Agent只通过飞书群组向PM Agent发送阻塞报告，PM Agent收到后会更新表格。

## Prerequisites

- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for CHAT_ID
- Read `references/identity.md` for your name and role

## Workflow

### Manual Trigger

When the developer says something like "I'm blocked on [TASK-XXXX]" or "I can't proceed because...":

#### Step 1: Collect Blocker Information

Ask the developer for details:

```
我检测到你遇到了阻塞，请提供以下信息：

1. 哪个任务被阻塞？(任务ID或描述)
2. 阻塞的具体原因是什么？
3. 你已经尝试了哪些解决方案？
4. 你认为需要什么帮助来解除阻塞？
5. 这个阻塞对其他任务有什么影响？
6. 如果阻塞无法很快解除，有没有临时绕过方案？
```

#### Step 2: Generate Blocker Report

```markdown
🚫 阻塞报告 — {PERSON_NAME} — {TIME}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**任务**: [TASK-0003] 支付集成
**项目**: 项目A
**优先级**: P0

**阻塞原因**: 第三方支付API的沙箱环境返回500错误，无法完成联调

**已尝试的解决方案**:
1. 重新阅读API文档，确认参数格式正确
2. 联系了服务商技术支持，等待回复中
3. 尝试使用mock数据绕过，但无法验证真实流程

**需要的帮助**:
1. PM协调：联系服务商确认沙箱环境状态
2. 技术支持：Person C 协助检查网络配置

**影响范围**:
- [TASK-0003] 直接影响：无法继续联调
- [TASK-0012] 间接影响：依赖支付完成的订单流程无法测试
- 里程碑影响：如2天内无法解除，项目A demo可能受影响

**临时方案**:
- 使用mock支付完成前端开发，但无法验证真实支付流程

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Step 3: Human Confirmation

```
阻塞报告已生成，确认发送到协作群组？(y/n/edit)
```

#### Step 4: Send Notification

```bash
# Send to group chat — PM Agent will update the Bitable accordingly
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type interactive \
  --content '{"config":{"wide_screen_mode":true},"header":{"title":{"tag":"plain_text","content":"🚫 阻塞报告 — {PERSON_NAME}"}},"elements":[{"tag":"markdown","content":"{BLOCKER_SUMMARY}"}]}'
```

> ⚠️ 不要直接更新多维表格。PM Agent 收到阻塞报告后会自动将任务标记为"已阻塞"。

### Automatic Detection (Heartbeat)

The heartbeat checks for stalled tasks every 30 minutes:

#### Detection Rules

| Signal | Condition | Action |
|--------|-----------|--------|
| No git commits | No commits on an in-progress task for 4+ hours | Ask developer if they're stuck |
| No progress update | Task progress unchanged for 2+ days | Check with developer |
| PR stuck | PR open with no review for 24+ hours | Notify PM Agent |
| Build failure | CI build failing on a task branch | Notify developer and PM Agent |

#### Heartbeat Check Flow

```
1. Check git activity for each in-progress task (based on local repo)
2. If no activity detected for 4+ hours:
   → Ask developer: "我注意到 [TASK-XXXX] 最近没有代码提交，是否遇到了阻塞？"
3. If developer confirms blocker:
   → Trigger manual blocker notification flow
4. If developer says no blocker:
   → Log and check again in next heartbeat
```

## Blocker Resolution Tracking

When a blocker is reported, track its resolution:

1. **Reported**: Blocker notification sent to group chat
2. **Acknowledged**: PM Agent or team member responds in group chat
3. **In Progress**: Someone is working on resolving the blocker
4. **Resolved**: Blocker is removed, task can continue

When the blocker is resolved:

```bash
# Only notify the group chat — PM Agent will update the Bitable
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type text \
  --content "✅ [TASK-XXXX] 阻塞已解除 — {PERSON_NAME} 可以继续工作"
```

> ⚠️ 不要直接更新多维表格。PM Agent 收到阻塞解除消息后会自动将任务状态改回"进行中"。

## Error Handling

| Scenario | Action |
|----------|--------|
| Developer not responding | Retry in 30 minutes, then notify PM Agent via group chat |
| Feishu message send fails | Retry once, then alert developer in terminal |
| Multiple blockers at once | Send consolidated blocker report |
