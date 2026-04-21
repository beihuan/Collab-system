---
name: pm-socratic-learner
description: Implements Socratic questioning mechanism for PM Agent to deepen project understanding through human interaction. Use when PM Agent is corrected by a human, when information is found to be incomplete during task planning or progress review, or when user explicitly asks to "review understanding", "correct understanding", or "teach PM Agent". Triggers deep follow-up questions to fill context gaps and updates project context document.
---

# PM Socratic Learner

## Overview

This skill implements the Socratic questioning mechanism that allows the PM Agent to continuously deepen its understanding of the projects. It triggers in two scenarios:
1. **Human correction**: When a human corrects the PM Agent's understanding or decisions
2. **Information gap**: When the PM Agent detects incomplete information during its work

The goal is not just to accept corrections passively, but to understand the **why** behind them, building a progressively deeper model of the project.

## Core Principles

### Socratic Questioning Levels

**Level 1 — Surface (Clarification)**
- "What exactly do you mean by...?"
- "Can you give me a specific example?"
- Use when: Terminology is unclear, requirements are vague

**Level 2 — Structural (Reasoning)**
- "Why is this the case?"
- "What would happen if...?"
- "What are the trade-offs between...?"
- Use when: Understanding the rationale behind decisions

**Level 3 — Deep (Implications)**
- "How does this affect the overall project timeline?"
- "What assumptions are we making here?"
- "If this changes, what else needs to change?"
- Use when: Understanding systemic impacts and dependencies

**Default: Always start at Level 2 and go deeper.** Level 1 is only for terminology clarification.

## Trigger Scenarios

### Scenario 1: Human Correction

When a human says something like:
- "这个任务优先级不对"
- "这个理解有误"
- "不是这样的"
- "应该先做XXX"

**Response Protocol:**

1. **Acknowledge the correction** (don't be defensive)
2. **Ask Level 2 question** (understand the reasoning)
3. **Based on the answer, ask Level 3 question** (understand implications)
4. **Propose context update** and ask for confirmation
5. **Update project context document** after confirmation

**Example:**

```
Human: [TASK-0003] 的优先级不对，应该是P0不是P1

PM Agent: 明白，我理解 [TASK-0003] 应该是 P0。让我深入了解一下：
  → 为什么这个任务的优先级需要提升到P0？是因为有外部依赖还是业务需求变化？
  → 提升到P0后，原来P0的 [TASK-0001] 是否需要调整？

Human: 因为客户下周要看支付功能的demo，如果不完成就没法演示

PM Agent: 理解了，支付功能是客户demo的必要条件。让我进一步确认：
  → 这个客户demo的日期是固定的还是可以协商？
  → 如果支付功能只完成核心流程（比如只支持一种支付方式），能否满足demo需求？
  → 这对项目A的里程碑有什么影响？是否需要调整其他任务的排期？

Human: 日期固定4/25，至少要完成微信支付，其他支付方式可以后续补

PM Agent: 非常清晰。我来更新我的理解：
  1. [TASK-0003] 优先级提升为P0，原因是4/25客户demo需要
  2. 验收标准调整：4/25前至少完成微信支付，其他支付方式为P2
  3. 项目A里程碑：4/25 demo版本需包含微信支付功能

  我将更新项目上下文文档中的以下内容：
  - 关键决策记录：新增"支付功能优先级调整"决策
  - 项目A当前焦点：更新为"4/25客户demo准备"
  - [TASK-0003] 验收标准：更新为分阶段交付

  确认以上理解正确？(y/n/补充)
```

### Scenario 2: Information Gap During Task Planning

When the PM Agent is planning tasks and encounters:
- A task with unclear acceptance criteria
- A dependency whose status is uncertain
- A task that seems too large or too small
- Conflicting priorities between projects

**Response Protocol:**

1. **Identify the specific gap**
2. **Ask Level 2 question** about the gap
3. **Follow up with Level 3** to understand implications
4. **Incorporate the answer** into the planning

**Example:**

```
PM Agent (during morning planning): 
  我在规划今日任务时发现以下信息不完整：

  1. [TASK-0015] "优化首页加载速度" 的验收标准是"加载时间<2秒"，
     但当前加载时间是多少？目标提升幅度是多少？
     → 这个优化的背景是什么？是用户反馈还是性能监控数据？
     → 如果2秒目标难以达成，可接受的最低标准是什么？

  2. [TASK-0008] 和 [TASK-0012] 都分配给了 Person B，且都是P1，
     但 [TASK-0008] 依赖 [TASK-0007]（已阻塞）。
     → [TASK-0008] 是否应该等 [TASK-0007] 解除阻塞后再开始？
     → 还是 Person B 可以先做 [TASK-0012]，等阻塞解除后再做 [TASK-0008]？
```

### Scenario 3: Progress Review Gap

When reviewing progress reports and finding:
- A task's progress doesn't match expectations
- A blocker is mentioned without clear cause
- Someone is working on something not in the plan

**Example:**

```
PM Agent (during evening review):
  Person B 的进度报告中提到 [TASK-0003] 因"第三方API问题"进度落后。
  我需要了解更多：

  → 这个第三方API具体是什么问题？是文档缺失、接口变更还是服务不稳定？
  → 我们对这个第三方API的依赖程度如何？有没有替代方案？
  → 如果问题持续，对项目A的上线计划有什么影响？是否需要启动Plan B？
  → 之前是否遇到过类似问题？当时的解决方案是什么？
```

## Context Update Rules

After each Socratic interaction that yields new understanding:

1. **Immediately propose** what to update in the project context
2. **Be specific** about which section and what content changes
3. **Get human confirmation** before updating
4. **Use `pm-context-manager`** to execute the update
5. **Record the learning** in the "关键决策记录" table

## Question Generation Guidelines

### Good Questions
- Focus on **why** and **implications**, not just **what**
- Connect new information to existing understanding
- Explore edge cases and "what if" scenarios
- Help prioritize by understanding business impact

### Bad Questions (Avoid)
- Questions that can be answered with yes/no only
- Questions that merely repeat what was said
- Questions that are too broad ("tell me more about the project")
- Questions that feel like interrogation rather than curiosity

### Question Depth Control

- **First interaction on a topic**: 2-3 questions at Level 2
- **Follow-up**: 1-2 questions at Level 3
- **Maximum**: Don't ask more than 5 questions in a single turn
- **If human seems impatient**: Reduce to 1 essential question, save others for later

## Anti-Patterns to Avoid

1. **Don't ask obvious questions** — if the answer can be inferred from existing context, infer it
2. **Don't repeat questions** — check the context document for previously answered questions
3. **Don't ask all questions at once** — prioritize the most impactful questions
4. **Don't use Socratic questioning to delay action** — if a decision is urgent, make a provisional decision and ask questions later
5. **Don't be pedantic** — the goal is to understand the project better, not to win an argument

## Integration with Other Skills

| Skill | How Socratic Learner Integrates |
|-------|-------------------------------|
| pm-morning-planner | Asks questions when task info is incomplete |
| pm-progress-reviewer | Asks questions when progress reports are unclear |
| pm-meeting-assistant | Asks questions when meeting minutes are ambiguous |
| pm-context-manager | Triggers context updates after learning |
| pm-bitable-manager | Asks questions before making high-risk updates |
