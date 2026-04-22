# AI Agent 协作项目管理系统

> 基于 AI Agent + 飞书多维表格 + 飞书群组 IM 的三人团队协作系统

## 项目简介

本系统解决三人开发团队在推进两个 MVP 项目时的协作痛点：信息孤岛、协调缺失、流程缺失。通过 Person A 的合并 Agent（PM+Dev）自动规划/分配/跟踪任务，Person B/C 的开发者 Agent 自动上报进度/报告阻塞/追踪代码活动，飞书多维表格作为 Single Source of Truth，实现结构化、自动化的项目协作。

## 系统架构

```
飞书多维表格 (Single Source of Truth)
        │
        ▼
 Person A 的 Agent (PM+Dev合并) ──── 飞书群组 IM ──── Person B/C 的 Dev Agent
  规划/分配/审核 + 开发/上报/阻塞        ↕                    上报/接收/阻塞
```

**核心设计**：Person A 的 Agent 同时承担 PM 和开发者双重角色，既能操作多维表格管理项目，又能追踪代码活动和上报进度。阻塞时可以同时发群组通知 + 直接更新 bitable，合并确认提升效率。

## 快速开始

### 1. 飞书侧准备

1. 安装飞书 CLI：`npm install -g @larksuite/cli`
2. 可选：安装飞书 CLI Skill 扩展：`npx skills add larksuite/cli -y -g`
3. 创建飞书自建应用，获取 App ID 和 App Secret
4. 认证飞书 CLI：`lark auth login --type=app --app-id=<APP_ID> --app-secret=<APP_SECRET>`
5. 创建多维表格应用"项目协作管理"，按 `lead-agent/skills/bitable-manager/references/bitable-schema.md` 定义字段
6. 创建飞书群组，将三人 + 机器人拉入
7. 记录 `APP_TOKEN`、`TABLE_ID`、`CHAT_ID`

### 2. Person A 的 Agent 安装（PM+Dev合并版）

将以下 prompt 粘贴到你的 Agent 对话中（支持 OpenClaw / Claude Code / Hermes 等任何 AI Agent）：

```
请阅读以下文档并执行其中的安装指令：

https://raw.githubusercontent.com/beihuan/Collab-system/main/lead-agent/skills/setup-guide/SKILL.md
```

### 3. Person B/C 的开发者 Agent 安装

将以下 prompt 粘贴到每个开发者的 Agent 对话中：

```
请阅读以下文档并执行其中的安装指令：

https://raw.githubusercontent.com/beihuan/Collab-system/main/dev-agent/skills/dev-setup-guide/SKILL.md
```

## 项目结构

```
Collab-system/
├── README.md
├── AI-Agent协作项目管理系统-设计方案.docx
│
├── lead-agent/                                  ← Person A 的合并 Agent (PM+Dev)
│   ├── config/
│   │   ├── agent-config.json                     ← Cron/Heartbeat/Skill 注册（Agent无关格式）
│   │   └── HEARTBEAT.md                         ← 心跳检查项（PM+Dev双角色）
│   └── skills/
│       ├── collab-system/                       ← 🧠 总控（双重角色定义）
│       ├── setup-guide/                         ← 🚀 安装引导
│       ├── bitable-manager/                     ← 🔧 多维表格CRUD
│       ├── morning-planner/                     ← 📋 晨间任务规划
│       ├── progress-reviewer/                   ← 📋 晚间进度审核
│       ├── meeting-assistant/                   ← 📋 会议辅助
│       ├── context-manager/                     ← 📚 项目上下文管理
│       ├── socratic-learner/                    ← 📚 苏格拉底式追问
│       ├── progress-reporter/                   ← 📋 进度上报（Dev角色）
│       ├── blocker-notifier/                    ← 📋 阻塞通知（Dev+PM合并处理）
│       └── code-activity-tracker/               ← 🔧 代码活动追踪
│
├── dev-agent/                                   ← Person B/C 的开发者 Agent
│   ├── config/
│   │   ├── agent-config.json
│   │   └── HEARTBEAT.md
│   └── skills/
│       ├── dev-collab-system/                   ← 🧠 总控
│       ├── dev-setup-guide/                     ← 🚀 安装引导
│       ├── dev-progress-reporter/               ← 📋 进度上报
│       ├── dev-task-receiver/                   ← 📋 任务接收
│       ├── dev-blocker-notifier/                ← 📋 阻塞通知
│       └── dev-code-activity-tracker/           ← 🔧 代码活动追踪
│
└── shared/                                      ← 共享模板
    └── templates/
        ├── task-assignment-template.md
        ├── progress-report-template.md
        ├── meeting-agenda-template.md
        └── blocker-report-template.md
```

## Skill 清单

### Person A 的合并 Agent Skills（11个）

| Skill | 类型 | 说明 |
|-------|------|------|
| `collab-system` | 🧠 总控 | 双重角色定义、每日工作流、Skill调度、通信协议 |
| `setup-guide` | 🚀 安装 | 一次性安装引导，自动克隆Skills并配置Cron/Heartbeat |
| `bitable-manager` | 🔧 工具 | 多维表格CRUD封装，含分级确认机制 |
| `morning-planner` | 📋 PM流程 | 晨间任务规划，B/C发群组，A直接展示 |
| `progress-reviewer` | 📋 PM流程 | 晚间进度审核，分析偏差并更新表格 |
| `meeting-assistant` | 📋 PM流程 | 会前摘要生成 + 会后纪要处理 |
| `context-manager` | 📚 知识 | 项目上下文维护，读取代码库和文档 |
| `socratic-learner` | 📚 知识 | 苏格拉底式追问，持续深化项目理解 |
| `progress-reporter` | 📋 Dev流程 | 生成Person A的进度报告，记忆优先+Git辅助验证 |
| `blocker-notifier` | 📋 Dev+PM | 报告阻塞+同时更新bitable，合并确认 |
| `code-activity-tracker` | 🔧 工具 | 追踪git提交和PR状态，关联任务ID |

### Person B/C 的开发者 Agent Skills（6个）

| Skill | 类型 | 说明 |
|-------|------|------|
| `dev-collab-system` | 🧠 总控 | 开发者角色定义、协作模式、通信协议 |
| `dev-setup-guide` | 🚀 安装 | 一次性安装引导 |
| `dev-progress-reporter` | 📋 流程 | 生成结构化进度报告，记忆优先+Git辅助验证 |
| `dev-task-receiver` | 📋 流程 | 接收PM Agent的任务指派 |
| `dev-blocker-notifier` | 📋 流程 | 检测阻塞并通知PM Agent |
| `dev-code-activity-tracker` | 🔧 工具 | 追踪git提交和PR状态 |

## 每日工作流

```
08:00  你(PM角色) → 晨间规划 → B/C任务发群组 + A任务终端展示
09:30  你(PM角色) → 生成会议议程 → 发送飞书群组
10:00  人类会议 → 讨论对齐
会后    人类提供会议纪要 → 你(PM角色)处理 → 更新任务和上下文
       你(Dev角色) → 正常开发工作...
18:00  你(Dev角色) → 记忆/对话提取工作+Git辅助验证 → 生成报告 → 确认 → 发群组
19:00  你(PM角色) → 审核所有报告(含自己的) → 分析偏差 → 更新bitable
```

## 双角色合并的优势

| 场景 | 分离Agent | 合并Agent |
|------|----------|----------|
| 遇到阻塞 | Dev Agent发群组 → PM Agent读消息 → 更新bitable | 一次确认：发群组 + 更新bitable |
| 查看任务 | Dev Agent从群组读取 | 直接在终端展示（你生成的） |
| 进度审核 | PM Agent从群组读所有人的报告 | 直接使用18:00生成的数据 |
| 项目理解 | PM Agent和Dev Agent各自理解 | 统一理解，开发中发现的信息直接用于PM决策 |

## 人类确认分级

| 级别 | 操作类型 | 确认方式 |
|------|---------|---------|
| L0-自动 | 只读操作 | 无需确认 |
| L2-确认 | 中风险写入 | 终端 y/n 确认 |
| L3-审批 | 高风险写入 | 终端确认 + 二次确认 |

## 迭代路线

- **Phase 1（第1周）**：最小可用 — 基础CRUD + 任务指派 + 进度上报
- **Phase 2（第2-3周）**：自动化 — 进度审核 + 会议辅助 + 阻塞检测 + 代码追踪
- **Phase 3（第4-6周）**：智能化 — 上下文管理 + 深层追问 + 风险预警
- **Phase 4（后续）**：高级特性 — 跨项目冲突 + 周报 + CI/CD集成

## 仓库地址

https://github.com/beihuan/Collab-system
