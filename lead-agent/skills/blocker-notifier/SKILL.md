---
name: blocker-notifier
description: Detects and reports blockers encountered during development work, notifying the team via Feishu group chat and updating the Bitable directly. Use when you encounter a blocker, when user says "I'm blocked", "stuck on", "can't proceed", "need help", or when the heartbeat detects no progress on an in-progress task for an extended period.
---

# Blocker Notifier

## Overview

This skill handles blocker detection and reporting for Person A's development work. Since you are both PM and developer, you can **simultaneously** notify the team via group chat AND update the Bitable — no need to wait for a separate PM Agent to process the blocker.

## Prerequisites

- Feishu CLI authenticated and configured
- Read `references/feishu-config.md` for CHAT_ID, APP_TOKEN, TABLE_ID
- Read `references/identity.md` for your name and role

## Workflow

### Manual Trigger

When Person A says something like "I'm blocked on [TASK-XXXX]" or "I can't proceed because...":

#### Step 1: Collect Blocker Information

Ask Person A for details:

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

#### Step 3: Combined Confirmation

Since you are both PM and developer, confirm both actions at once:

```
阻塞报告已生成，确认执行以下操作？(y/n/edit)

1. 📤 发送阻塞报告到协作群组（通知 Person B/C）
2. 📋 在多维表格中将 [TASK-XXXX] 标记为"已阻塞"
```

#### Step 4: Execute

```bash
# 1. Send to group chat
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type interactive \
  --content '{"config":{"wide_screen_mode":true},"header":{"title":{"tag":"plain_text","content":"🚫 阻塞报告 — {PERSON_NAME}"}},"elements":[{"tag":"markdown","content":"{BLOCKER_SUMMARY}"}]}'

# 2. Update Bitable directly (you have PM authority)
lark-cli bitable record update --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --record-id {RECORD_ID} \
  --fields '{"status": "已阻塞", "blocker": "{BLOCKER_DESCRIPTION}"}'
```

### Automatic Detection (Heartbeat)

The heartbeat checks for stalled tasks every 30 minutes:

#### Detection Rules

| Signal | Condition | Action |
|--------|-----------|--------|
| No git commits | No commits on an in-progress task for 4+ hours | Ask Person A if they're stuck |
| No progress update | Task progress unchanged for 2+ days | Check with Person A |
| PR stuck | PR open with no review for 24+ hours | Remind Person A |
| Build failure | CI build failing on a task branch | Alert Person A |

#### Heartbeat Check Flow

```
1. Check git activity for each of Person A's in-progress tasks
2. If no activity detected for 4+ hours:
   → Ask Person A: "我注意到 [TASK-XXXX] 最近没有代码提交，是否遇到了阻塞？"
3. If Person A confirms blocker:
   → Trigger manual blocker notification flow
4. If Person A says no blocker:
   → Log and check again in next heartbeat
```

## Blocker Resolution

When the blocker is resolved:

```bash
# 1. Notify group chat
lark-cli chat message send --chat-id {CHAT_ID} \
  --msg-type text \
  --content "✅ [TASK-XXXX] 阻塞已解除 — {PERSON_NAME} 可以继续工作"

# 2. Update Bitable directly
lark-cli bitable record update --app-token {APP_TOKEN} --table-id {TABLE_ID} \
  --record-id {RECORD_ID} \
  --fields '{"status": "进行中", "blocker": ""}'
```

Confirm both actions with Person A before executing.

## Error Handling

| Scenario | Action |
|----------|--------|
| Person A not responding | Retry in 30 minutes |
| Feishu message send fails | Retry once, then alert in terminal |
| Bitable update fails | Send group chat notification first, retry Bitable later |
| Multiple blockers at once | Send consolidated blocker report |
