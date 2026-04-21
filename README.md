# AI Agent 协作项目管理系统

> 基于 OpenClaw + 飞书多维表格 + 飞书群组 IM 的三人团队协作系统

## 项目简介

本系统解决三人开发团队在推进两个 MVP 项目时的协作痛点：信息孤岛、协调缺失、流程缺失。通过 PM Agent 自动规划/分配/跟踪任务，开发者 Agent 自动上报进度/报告阻塞/追踪代码活动，飞书多维表格作为 Single Source of Truth，实现结构化、自动化的项目协作。

## 系统架构

```
飞书多维表格 (Single Source of Truth)
        │
        ▼
   PM Agent ────────── 飞书群组 IM ────────── 开发者 Agent ×3
 (规划/分配/审核)      (任务指派/进度报告)      (上报/接收/阻塞告警)
```

## 快速开始

### 1. 飞书侧准备

1. 安装飞书 CLI：`npm install -g lark-cli`
2. 创建飞书自建应用，获取 App ID 和 App Secret
3. 认证飞书 CLI：`lark-cli auth login --type=app --app-id=<APP_ID> --app-secret=<APP_SECRET>`
4. 创建多维表格应用"项目协作管理"，按 `pm-agent/skills/pm-bitable-manager/references/bitable-schema.md` 定义字段
5. 创建飞书群组，将三人 + 机器人拉入
6. 记录 `APP_TOKEN`、`TABLE_ID`、`CHAT_ID`

### 2. PM Agent 安装

将以下 prompt 粘贴到你的 OpenClaw 对话中：

```
请阅读以下文档并执行其中的安装指令：

https://raw.githubusercontent.com/beihuan/Collab-system/main/pm-agent/skills/pm-setup-guide/SKILL.md
```

### 3. 开发者 Agent 安装

将以下 prompt 粘贴到每个开发者的 OpenClaw 对话中：

```
请阅读以下文档并执行其中的安装指令：

https://raw.githubusercontent.com/beihuan/Collab-system/main/dev-agent/skills/dev-setup-guide/SKILL.md
```

## 项目结构

```
Collab-system/
├── README.md                                    ← 你正在看的文件
├── AI-Agent协作项目管理系统-设计方案.docx          ← 完整设计文档
│
├── pm-agent/                                    ← PM Agent 完整配置
│   ├── skills/
│   │   ├── pm-collab-system/                    ← 🧠 总控 Skill（协作系统宪法）
│   │   ├── pm-setup-guide/                      ← 🚀 安装引导 Skill
│   │   ├── pm-bitable-manager/                  ← 多维表格 CRUD
│   │   ├── pm-morning-planner/                  ← 晨间任务规划
│   │   ├── pm-progress-reviewer/                ← 进度审核
│   │   ├── pm-meeting-assistant/                ← 会议辅助
│   │   ├── pm-context-manager/                  ← 项目上下文管理
│   │   └── pm-socratic-learner/                 ← 苏格拉底式追问
│   └── config/
│       ├── openclaw.json                        ← Cron/Heartbeat/Skill 注册
│       └── HEARTBEAT.md                         ← 心跳检查项
│
├── dev-agent/                                   ← 开发者 Agent 完整配置
│   ├── skills/
│   │   ├── dev-collab-system/                   ← 🧠 总控 Skill（协作系统宪法）
│   │   ├── dev-setup-guide/                     ← 🚀 安装引导 Skill
│   │   ├── dev-progress-reporter/               ← 进度上报
│   │   ├── dev-task-receiver/                   ← 任务接收
│   │   ├── dev-blocker-notifier/                ← 阻塞通知
│   │   └── dev-code-activity-tracker/           ← 代码活动追踪
│   └── config/
│       ├── openclaw.json                        ← Cron/Heartbeat/Skill 注册
│       └── HEARTBEAT.md                         ← 心跳检查项
│
└── shared/                                      ← 共享模板
    └── templates/
        ├── task-assignment-template.md           ← 任务指派模板
        ├── progress-report-template.md           ← 进度报告模板
        ├── meeting-agenda-template.md            ← 会议议程模板
        └── blocker-report-template.md            ← 阻塞报告模板
```

## Skill 体系

### PM Agent Skills（7个）

| Skill | 类型 | 说明 |
|-------|------|------|
| `pm-collab-system` | 🧠 总控 | 协作系统宪法，定义角色/流程/协议，所有协作操作前必读 |
| `pm-setup-guide` | 🚀 安装 | 一次性安装引导，自动克隆Skills并配置Cron/Heartbeat |
| `pm-bitable-manager` | 🔧 工具 | 多维表格CRUD封装，含分级确认机制 |
| `pm-morning-planner` | 📋 流程 | 晨间任务规划，生成每人任务指派文档 |
| `pm-progress-reviewer` | 📋 流程 | 晚间进度审核，分析偏差并更新表格 |
| `pm-meeting-assistant` | 📋 流程 | 会前摘要生成 + 会后纪要处理 |
| `pm-context-manager` | 📚 知识 | 项目上下文维护，读取代码库和文档 |
| `pm-socratic-learner` | 📚 知识 | 苏格拉底式追问，持续深化项目理解 |

### 开发者 Agent Skills（5个）

| Skill | 类型 | 说明 |
|-------|------|------|
| `dev-collab-system` | 🧠 总控 | 协作系统宪法，定义角色/流程/协议 |
| `dev-setup-guide` | 🚀 安装 | 一次性安装引导，自动克隆Skills并配置Cron/Heartbeat |
| `dev-progress-reporter` | 📋 流程 | 生成结构化进度报告，关联git活动 |
| `dev-task-receiver` | 📋 流程 | 接收并展示PM Agent的任务指派 |
| `dev-blocker-notifier` | 📋 流程 | 检测阻塞并主动通知PM Agent |
| `dev-code-activity-tracker` | 🔧 工具 | 追踪git提交和PR状态，关联任务ID |

## 每日工作流

```
08:00  PM Agent → 晨间规划 → 生成任务指派 → 人类确认 → 发送飞书群组
09:00  开发者Agent → 读取任务指派 → 展示给人类
09:30  PM Agent → 生成会议议程 → 发送飞书群组
10:00  人类会议 → 讨论对齐
会后    人类提供会议纪要 → PM Agent处理 → 更新任务和上下文
18:00  开发者Agent → 收集进度+代码活动 → 生成报告 → 人类确认 → 发送飞书群组
19:00  PM Agent → 审核所有报告 → 分析偏差 → 提出表格更新 → 人类确认 → 执行
```

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
