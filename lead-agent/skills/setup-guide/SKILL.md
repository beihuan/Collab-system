---
name: setup-guide
description: "Agent一次性安装引导。当用户要求安装协作系统、初始化Agent、设置协作环境时使用。此Skill是一段可执行prompt，读取后自动从GitHub仓库克隆所有Skills到本地、配置飞书连接信息和个人身份、设置Cron Jobs和Heartbeat。只需执行一次，安装完成后此Skill不再需要。"
---

# Agent 安装引导（Person A — PM+Dev合并版）

> ⚠️ 这是一段可执行指令。请严格按照以下步骤顺序执行，每一步完成后才进入下一步。遇到任何错误立即停止并向人类报告。

## 你即将做什么

你正在安装一个**AI Agent协作项目管理系统**，你同时承担 PM 和开发者双重角色。安装完成后，你将：
- **作为PM**：每天早晨为三位团队成员规划并分配任务，晚间审核进度并更新飞书多维表格
- **作为开发者**：帮助 Person A 做全栈开发，追踪代码活动，上报工作进度，报告阻塞

## 第一步：确认前置条件

在开始安装前，请逐项确认以下条件是否满足。**任何一项不满足都必须停止安装并告知人类**：

```
前置条件检查清单：

[ ] 1. OpenClaw 已安装并正常运行
     → 验证命令: openclaw --version

[ ] 2. 飞书CLI (lark-cli) 已安装
     → 验证命令: lark-cli --version

[ ] 3. 飞书CLI已通过机器人身份认证
     → 验证命令: lark-cli auth status
     → 如果未认证，请人类执行: lark-cli auth login --type=app --app-id=<APP_ID> --app-secret=<APP_SECRET>

[ ] 4. 飞书多维表格已创建，且字段结构符合要求
     → 验证命令: lark-cli bitable field list --app-token=<APP_TOKEN> --table-id=<TABLE_ID>
     → 字段要求见: https://raw.githubusercontent.com/beihuan/Collab-system/main/lead-agent/skills/bitable-manager/references/bitable-schema.md

[ ] 5. 飞书协作群组已创建，机器人已加入
     → 验证命令: lark-cli chat list

[ ] 6. 项目代码仓库已在本地克隆
     → 验证命令: 在人类提供的仓库路径下执行 git status

[ ] 7. 人类已准备好以下配置信息：
     - APP_TOKEN (多维表格应用token)
     - TABLE_ID (任务看板表ID)
     - CHAT_ID (协作群组ID)
     - APP_ID (飞书机器人App ID)
     - APP_SECRET (飞书机器人App Secret)
     - Person B 和 Person C 的飞书用户ID
     - Git 作者名和邮箱
     - GitHub 用户名（如有）
     - 各项目仓库的本地路径
```

向人类展示检查结果，确认全部通过后继续。

## 第二步：确定本地 Skills 安装路径

OpenClaw 的 Skills 可以安装在两个位置：
- **全局 Skills**: `~/.openclaw/skills/`
- **工作空间 Skills**: `~/.openclaw/workspace/skills/`（如果当前有工作空间）

请执行以下命令确定安装路径：

```bash
# 检查工作空间是否存在
ls ~/.openclaw/workspace/skills/ 2>/dev/null && echo "workspace skills exists" || echo "no workspace skills"

# 检查全局skills目录
ls ~/.openclaw/skills/ 2>/dev/null && echo "global skills exists" || echo "no global skills"
```

**决策规则**：
- 如果有工作空间 → 安装到工作空间 Skills 目录
- 如果没有工作空间 → 安装到全局 Skills 目录

将确定的路径记为 `{SKILLS_DIR}`。

## 第三步：克隆所有 Skills

从 GitHub 仓库克隆 lead-agent 的所有 Skills：

```bash
# 创建临时目录
TMPDIR=$(mktemp -d)
cd "$TMPDIR"

# 克隆仓库
git clone https://github.com/beihuan/Collab-system.git

# 复制所有 Skills 到本地
cp -r Collab-system/lead-agent/skills/* "{SKILLS_DIR}/"

# 清理临时目录
rm -rf "$TMPDIR"
```

需要安装的 Skills 共 11 个：
- `collab-system` — 总控
- `setup-guide` — 安装引导（本Skill）
- `bitable-manager` — 多维表格管理
- `morning-planner` — 晨间任务规划
- `progress-reviewer` — 晚间进度审核
- `meeting-assistant` — 会议辅助
- `context-manager` — 项目上下文管理
- `socratic-learner` — 苏格拉底式追问
- `progress-reporter` — 进度上报（Dev角色）
- `blocker-notifier` — 阻塞通知（Dev角色）
- `code-activity-tracker` — 代码活动追踪（Dev角色）

验证安装：

```bash
ls "{SKILLS_DIR}/" | sort
```

确认以上 11 个目录都存在。

## 第四步：配置飞书连接信息

向人类获取配置信息，然后写入各 Skill 的 references 文件。

### 4a. 写入 bitable-manager 的飞书配置

编辑 `{SKILLS_DIR}/bitable-manager/references/feishu-config.md`：

```markdown
# 飞书配置

## 多维表格配置
- APP_TOKEN: {人类提供的APP_TOKEN}
- TABLE_ID: {人类提供的TABLE_ID}

## 群组配置
- CHAT_ID: {人类提供的CHAT_ID}

## 机器人配置
- APP_ID: {人类提供的APP_ID}
- APP_SECRET: {人类提供的APP_SECRET}

## 人员映射
- Person A (你/PM+Dev): {人类提供的Person A飞书ID}
- Person B (全栈开发): {人类提供的Person B飞书ID}
- Person C (后端开发): {人类提供的Person C飞书ID}
```

### 4b. 写入 morning-planner 的飞书配置

将同样的配置复制到 `{SKILLS_DIR}/morning-planner/references/feishu-config.md`。

### 4c. 写入 progress-reporter 的飞书配置

编辑 `{SKILLS_DIR}/progress-reporter/references/feishu-config.md`：

```markdown
# 飞书配置

## 群组配置
- CHAT_ID: {人类提供的CHAT_ID}

## 机器人配置
- APP_ID: {人类提供的APP_ID}
- APP_SECRET: {人类提供的APP_SECRET}

## 人员映射
- Person A (你/PM+Dev): {人类提供的Person A飞书ID}
- Person B (全栈开发): {人类提供的Person B飞书ID}
- Person C (后端开发): {人类提供的Person C飞书ID}
```

> 注意：progress-reporter 只需要群组通信权限，不需要多维表格权限。

### 4d. 写入 progress-reporter 的身份配置

编辑 `{SKILLS_DIR}/progress-reporter/references/identity.md`：

```markdown
# 身份配置

## 个人信息
- 姓名: Person A
- 角色: 产品经理 + 全栈开发
- 飞书用户ID: {人类提供的ID}

## Git 配置
- Git作者名: {人类提供的名字}
- Git邮箱: {人类提供的邮箱}
- GitHub用户名: {人类提供的用户名}

## 项目参与
- 项目A: 参与开发 + 产品决策 + 项目管理
- 项目B: 参与开发 + 产品决策 + 项目管理

## 仓库路径
- 项目A: {人类提供的路径}
- 项目B: {人类提供的路径}
```

### 4e. 写入 code-activity-tracker 的仓库配置

编辑 `{SKILLS_DIR}/code-activity-tracker/references/repo-config.md`：

```markdown
# 代码仓库配置

## 项目A
- 路径: {人类提供的路径}
- Git作者: {人类提供的名字} <{人类提供的邮箱}>
- GitHub用户: {人类提供的用户名}

## 项目B
- 路径: {人类提供的路径}
- Git作者: {人类提供的名字} <{人类提供的邮箱}>
- GitHub用户: {人类提供的用户名}
```

### 4f. 写入 context-manager 的仓库配置

编辑 `{SKILLS_DIR}/context-manager/references/repo-config.md`：

```markdown
# 代码仓库配置

## 项目A
- 名称: {人类提供的项目A名称}
- 仓库路径: {人类提供的路径}
- 主要分支: main
- 技术栈: {人类提供}

## 项目B
- 名称: {人类提供的项目B名称}
- 仓库路径: {人类提供的路径}
- 主要分支: main
- 技术栈: {人类提供}
```

## 第五步：配置 Cron Jobs

使用 OpenClaw 的对话式创建或配置文件方式，设置以下 4 个 Cron Jobs：

### 方式A：对话式创建

依次告诉 OpenClaw：

```
1. "创建一个每天工作日08:00的定时任务，名称为morning-planning，执行：读取多维表格当前状态，为每个团队成员生成今日任务指派文档，经人类确认后发送到飞书群组，同时在终端展示Person A自己的今日任务"

2. "创建一个每天工作日09:30的定时任务，名称为pre-meeting-summary，执行：生成每日对齐会议的进度摘要和议程，发送到飞书群组"

3. "创建一个每天工作日18:00的定时任务，名称为evening-progress-report，执行：收集今日代码活动和人工输入，生成Person A的结构化进度报告，经人类确认后发送到飞书群组"

4. "创建一个每天工作日19:00的定时任务，名称为evening-review，执行：收集所有进度报告，分析进度偏差，提出多维表格更新建议，经人类确认后执行更新"
```

### 方式B：编辑 openclaw.json

如果使用配置文件方式，在 `openclaw.json` 中添加 cron 配置：

```json
{
  "cron": {
    "jobs": [
      {
        "name": "morning-planning",
        "schedule": { "kind": "cron", "expr": "0 8 * * 1-5", "tz": "Asia/Shanghai" },
        "payload": { "kind": "agentTurn", "message": "执行每日早晨任务规划：读取多维表格当前状态，为每个团队成员生成今日任务指派文档，经人类确认后发送到飞书群组，同时在终端展示Person A自己的今日任务" },
        "sessionTarget": "main"
      },
      {
        "name": "pre-meeting-summary",
        "schedule": { "kind": "cron", "expr": "30 9 * * 1-5", "tz": "Asia/Shanghai" },
        "payload": { "kind": "agentTurn", "message": "生成每日对齐会议的进度摘要和议程，发送到飞书群组" },
        "sessionTarget": "main"
      },
      {
        "name": "evening-progress-report",
        "schedule": { "kind": "cron", "expr": "0 18 * * 1-5", "tz": "Asia/Shanghai" },
        "payload": { "kind": "agentTurn", "message": "收集今日代码活动和人工输入，生成Person A的结构化进度报告，经人类确认后发送到飞书群组" },
        "sessionTarget": "main"
      },
      {
        "name": "evening-review",
        "schedule": { "kind": "cron", "expr": "0 19 * * 1-5", "tz": "Asia/Shanghai" },
        "payload": { "kind": "agentTurn", "message": "收集所有进度报告，分析进度偏差，提出多维表格更新建议，经人类确认后执行更新" },
        "sessionTarget": "main"
      }
    ]
  }
}
```

验证：

```bash
openclaw cron list
```

确认 4 个任务都已注册。

## 第六步：配置 Heartbeat

在工作空间根目录创建 `HEARTBEAT.md`：

```markdown
# Heartbeat

> 每30分钟执行一次检查

## PM角色检查项

- 检查飞书群组是否有新的阻塞报告（来自Person B或C），如有则分析影响范围并通知人类
- 检查是否有人在19:30前未提交进度报告，如有则发送提醒
- 检查多维表格中是否有逾期任务，如有则通知人类
- 检查是否有新的会议纪要需要处理
- 检查项目上下文文档是否需要更新

## Dev角色检查项

- 检查进行中的任务是否有代码活动（4小时无提交则询问是否遇到阻塞）
- 检查是否有PR需要处理（审核/合并）
- 检查分配给自己的任务是否有截止日期临近的

## 综合判断

- 如果有阻塞报告 → PM角色优先处理（评估影响+更新bitable）
- 如果无异常 → HEARTBEAT_OK
```

## 第七步：验证飞书连接

执行以下命令验证飞书CLI可以正常访问：

```bash
# 验证多维表格可读写
lark-cli bitable record list --app-token=<APP_TOKEN> --table-id=<TABLE_ID> --limit 1

# 验证群组可通信
lark-cli chat message list --chat-id=<CHAT_ID> --limit 1

# 验证可以发送消息（发送一条测试消息，人类确认后可删除）
lark-cli chat message send --chat-id=<CHAT_ID> \
  --msg-type text \
  --content "🔧 Person A 的协作系统Agent安装测试 — 请忽略此消息"
```

如果以上命令都成功，说明飞书连接正常。

## 第八步：初始化项目上下文

安装完成后，你需要初始化对项目的理解。请向人类确认：

```
协作系统安装完成！现在需要初始化项目上下文。

请告诉我：
1. 两个项目的代码仓库分别在本地的什么路径？
2. 你是否希望我现在就阅读代码库来了解项目？
3. 还是你想先口头告诉我项目的基本情况？
```

根据人类的回答：
- 如果允许阅读代码库 → 执行 context-manager Skill 的初始化流程
- 如果口头描述 → 记录到 project-context.md，然后通过苏格拉底式追问补充细节

## 第九步：端到端验证

请人类配合进行一次完整流程验证：

```
安装验证清单：

[ ] 1. 手动触发 morning-planner，验证能否读取多维表格并生成任务指派
[ ] 2. 验证 Person B/C 的任务指派能发送到飞书群组
[ ] 3. 验证 Person A 的任务能在终端直接展示
[ ] 4. 手动触发 progress-reporter，验证能否收集git活动并生成报告
[ ] 5. 验证报告经确认后能发送到飞书群组
[ ] 6. 手动触发 progress-reviewer，验证能分析偏差并更新bitable
[ ] 7. 检查 Cron Jobs: openclaw cron list
[ ] 8. 检查 Heartbeat: 等待下一次心跳或手动触发
```

## 安装完成

如果所有验证通过，向人类展示：

```
✅ 协作系统安装完成！（PM+Dev合并版）

已安装 Skills: 11个
  PM角色:
  - collab-system (总控)
  - bitable-manager (多维表格)
  - morning-planner (晨间规划)
  - progress-reviewer (进度审核)
  - meeting-assistant (会议辅助)
  - context-manager (上下文管理)
  - socratic-learner (苏格拉底追问)
  Dev角色:
  - progress-reporter (进度上报)
  - blocker-notifier (阻塞通知)
  - code-activity-tracker (代码活动追踪)
  通用:
  - setup-guide (安装引导)

Cron Jobs: 4个
  - 08:00 晨间规划（PM角色）
  - 09:30 会前摘要（PM角色）
  - 18:00 进度上报（Dev角色）
  - 19:00 进度审核（PM角色）

Heartbeat: 每30分钟（PM+Dev双角色检查）

下一步：
1. 确保飞书多维表格中已有初始任务数据
2. 确保Person B和C也已安装开发者Agent
3. 明天早上8点，第一个Cron将自动触发晨间规划
```

## 故障排除

| 问题 | 解决方案 |
|------|---------|
| git clone 失败 | 检查网络连接，或让人类手动下载仓库ZIP并解压 |
| Skills 未被 OpenClaw 识别 | 检查 SKILL.md 文件名大小写是否正确，重启 OpenClaw |
| 飞书CLI认证失败 | 重新执行 `lark-cli auth login` |
| 多维表格操作403 | 在飞书开放平台给机器人应用添加多维表格权限 |
| 群组消息发送失败 | 检查机器人是否在群组中，检查 CHAT_ID |
| git log 无结果 | 检查仓库路径是否正确，检查 git config user.name |
| Cron未触发 | 检查 `openclaw cron list` 确认任务状态，检查时区设置 |
| Heartbeat未执行 | 检查 HEARTBEAT.md 是否在工作空间根目录 |
