---
name: pm-setup-guide
description: "PM Agent一次性安装引导。当用户要求安装PM Agent协作系统、初始化PM Agent、设置协作环境时使用。此Skill是一段可执行prompt，读取后自动从GitHub仓库克隆所有PM Agent Skills到本地、配置飞书连接信息、设置Cron Jobs和Heartbeat。只需执行一次，安装完成后此Skill不再需要。"
---

# PM Agent 安装引导

> ⚠️ 这是一段可执行指令。请严格按照以下步骤顺序执行，每一步完成后才进入下一步。遇到任何错误立即停止并向人类报告。

## 你即将做什么

你正在安装一个**AI Agent协作项目管理系统**的 PM Agent 部分。安装完成后，你将成为团队的产品经理兼项目经理，负责：
- 每天早晨为三位团队成员规划并分配任务
- 每天晚间审核进度报告并更新飞书多维表格
- 生成会议摘要、处理会议纪要
- 通过苏格拉底式追问持续深化对项目的理解

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
     → 字段要求见: https://raw.githubusercontent.com/beihuan/Collab-system/main/pm-agent/skills/pm-bitable-manager/references/bitable-schema.md

[ ] 5. 飞书协作群组已创建，机器人已加入
     → 验证命令: lark-cli chat list

[ ] 6. 人类已准备好以下配置信息：
     - APP_TOKEN (多维表格应用token)
     - TABLE_ID (任务看板表ID)
     - CHAT_ID (协作群组ID)
     - APP_ID (飞书机器人App ID)
     - APP_SECRET (飞书机器人App Secret)
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

**安装路径选择规则**：
- 如果当前有 OpenClaw 工作空间 → 安装到工作空间 Skills 目录
- 如果没有工作空间 → 安装到全局 Skills 目录 `~/.openclaw/skills/`

将确定的安装路径记录为变量 `SKILLS_DIR`，后续步骤中使用。

## 第三步：从 GitHub 克隆 Skills

执行以下命令，将仓库克隆到临时目录，然后复制所有 PM Agent Skills：

```bash
# 克隆仓库到临时目录
TMP_DIR=$(mktemp -d)
git clone https://github.com/beihuan/Collab-system.git "$TMP_DIR/Collab-system"

# 复制所有 PM Agent Skills 到本地 Skills 目录
# 包括: pm-collab-system, pm-bitable-manager, pm-morning-planner,
#       pm-progress-reviewer, pm-meeting-assistant, pm-context-manager,
#       pm-socratic-learner
cp -r "$TMP_DIR/Collab-system/pm-agent/skills/"* "$SKILLS_DIR/"

# 验证复制结果
echo "已安装的 PM Agent Skills:"
ls -1 "$SKILLS_DIR/" | grep "^pm-"

# 清理临时目录
rm -rf "$TMP_DIR"
```

**验证安装**：确认以下7个 Skill 目录都已存在：
- [ ] `pm-collab-system/` — 总控 Skill
- [ ] `pm-bitable-manager/` — 多维表格管理
- [ ] `pm-morning-planner/` — 晨间规划
- [ ] `pm-progress-reviewer/` — 进度审核
- [ ] `pm-meeting-assistant/` — 会议辅助
- [ ] `pm-context-manager/` — 上下文管理
- [ ] `pm-socratic-learner/` — 苏格拉底追问

每个目录下必须有 `SKILL.md` 文件。如果任何 Skill 缺失，重新执行复制命令。

## 第四步：配置飞书连接信息

需要将飞书配置信息写入每个 Skill 的 `references/feishu-config.md` 文件。

**请向人类索取以下信息**（如果第一步已确认则直接使用）：

```
请提供以下飞书配置信息：
1. APP_TOKEN (多维表格应用token):
2. TABLE_ID (任务看板表ID):
3. CHAT_ID (协作群组ID):
4. APP_ID (飞书机器人App ID):
5. APP_SECRET (飞书机器人App Secret):
```

获取信息后，找到所有包含 `feishu-config.md` 的文件并填入配置：

```bash
# 查找所有需要配置的文件
find "$SKILLS_DIR" -name "feishu-config.md" -path "*/pm-*"
```

对每个找到的文件，将占位符 `<请填入...>` 替换为实际值。可以使用 sed 命令批量替换，或逐个文件编辑。

**同时配置 pm-context-manager 的仓库路径**：

编辑 `$SKILLS_DIR/pm-context-manager/references/repo-config.md`，填入：
- 项目A的名称和本地仓库路径
- 项目B的名称和本地仓库路径

**同时配置 pm-morning-planner 的项目上下文**：

编辑 `$SKILLS_DIR/pm-morning-planner/references/project-context.md`，填入初始项目信息（如果人类已提供的话）。

## 第五步：验证飞书连接

执行以下命令验证飞书CLI可以正常操作多维表格和群组：

```bash
# 验证多维表格可读
lark-cli bitable record list --app-token=<APP_TOKEN> --table-id=<TABLE_ID> --limit 1

# 验证群组可访问
lark-cli chat message list --chat-id=<CHAT_ID> --limit 1
```

如果任一命令失败，请人类检查：
1. 飞书CLI认证是否有效
2. 机器人是否有对应多维表格和群组的访问权限
3. APP_TOKEN / TABLE_ID / CHAT_ID 是否正确

## 第六步：配置 Cron Jobs

PM Agent 需要三个定时任务。请根据你的 OpenClaw 版本选择配置方式：

### 方式A：通过对话创建（推荐）

直接告诉 OpenClaw 以下三句话，让它自动创建 Cron Job：

```
帮我创建一个定时任务：工作日每天早上8点，执行 pm-morning-planner Skill，读取多维表格并生成今日任务指派

帮我创建一个定时任务：工作日每天早上9:30，执行 pm-meeting-assistant Skill，生成每日对齐会议的进度摘要

帮我创建一个定时任务：工作日每天晚上7点，执行 pm-progress-reviewer Skill，审核所有进度报告并更新多维表格
```

### 方式B：编辑配置文件

如果 OpenClaw 使用 `openclaw.json` 配置文件，请在 `cron.jobs` 数组中添加：

```json
{
  "cron": {
    "jobs": [
      {
        "name": "morning-planning",
        "schedule": {
          "kind": "cron",
          "expr": "0 8 * * 1-5",
          "tz": "Asia/Shanghai"
        },
        "payload": {
          "kind": "agentTurn",
          "message": "执行每日早晨任务规划：先加载 pm-collab-system 总控Skill了解全局上下文，然后执行 pm-morning-planner Skill，读取多维表格当前状态，为每个团队成员生成今日任务指派文档，经人类确认后发送到飞书群组"
        },
        "sessionTarget": "isolated"
      },
      {
        "name": "pre-meeting-summary",
        "schedule": {
          "kind": "cron",
          "expr": "30 9 * * 1-5",
          "tz": "Asia/Shanghai"
        },
        "payload": {
          "kind": "agentTurn",
          "message": "执行会前摘要生成：先加载 pm-collab-system 总控Skill了解全局上下文，然后执行 pm-meeting-assistant Skill 的会前阶段，读取多维表格当前状态，生成每日对齐会议的进度摘要和议程，发送到飞书群组"
        },
        "sessionTarget": "isolated"
      },
      {
        "name": "evening-review",
        "schedule": {
          "kind": "cron",
          "expr": "0 19 * * 1-5",
          "tz": "Asia/Shanghai"
        },
        "payload": {
          "kind": "agentTurn",
          "message": "执行每日晚间进度审核：先加载 pm-collab-system 总控Skill了解全局上下文，然后执行 pm-progress-reviewer Skill，收集飞书群组中所有进度报告，分析进度偏差，提出多维表格更新建议，经人类确认后执行更新"
        },
        "sessionTarget": "isolated"
      }
    ]
  }
}
```

**验证 Cron 配置**：

```bash
openclaw cron list
```

确认三个任务都已注册且状态为 active。

## 第七步：配置 Heartbeat

PM Agent 的 Heartbeat 每30分钟执行一次，检查群组新消息、逾期任务、阻塞告警等。

### 方式A：创建 HEARTBEAT.md

在 OpenClaw 工作空间根目录创建 `HEARTBEAT.md`，内容如下：

```markdown
# PM Agent Heartbeat

## 每次检查

- [ ] 飞书群组是否有新的阻塞报告？如有，分析影响范围并通知人类
- [ ] 是否有人在19:30前未提交进度报告？如有，发送提醒
- [ ] 多维表格中是否有逾期任务（due_date < today 且 status != 已完成）？如有，通知人类
- [ ] 是否有新的会议纪要需要处理？如有，触发 pm-meeting-assistant 会后处理
- [ ] 项目上下文文档是否需要更新？

## 触发条件

- 有新的阻塞报告 → 立即通知人类，分析影响范围
- 有人未提交进度报告且时间 > 19:30 → 发送提醒
- 有逾期任务 → 通知人类
- 有会议纪要 → 触发会后处理流程
- 其他情况 → HEARTBEAT_OK
```

### 方式B：编辑 openclaw.json

如果使用配置文件方式，在 `openclaw.json` 中添加 heartbeat 配置：

```json
{
  "heartbeat": {
    "intervalMs": 1800000
  }
}
```

## 第八步：初始化项目上下文

安装完成后，你需要初始化对项目的理解。请向人类确认：

```
PM Agent 安装完成！现在需要初始化项目上下文。

请告诉我：
1. 两个项目的代码仓库分别在本地的什么路径？
2. 你是否希望我现在就阅读代码库来了解项目？
3. 还是你想先口头告诉我项目的基本情况？
```

根据人类的回答：
- 如果允许阅读代码库 → 执行 `pm-context-manager` Skill 的初始化流程
- 如果口头描述 → 记录到 `project-context.md`，然后通过苏格拉底式追问补充细节

## 第九步：端到端验证

请人类配合进行一次完整流程验证：

```
安装验证清单：

[ ] 1. 手动触发 pm-morning-planner，验证能否读取多维表格并生成任务指派
[ ] 2. 验证任务指派文档能否发送到飞书群组
[ ] 3. 检查 Cron Jobs 是否正常注册: openclaw cron list
[ ] 4. 检查 Heartbeat 是否正常: 等待下一次心跳或手动触发
[ ] 5. 验证飞书群组消息收发正常
```

## 安装完成

如果所有验证通过，向人类展示：

```
✅ PM Agent 安装完成！

已安装 Skills: 7个
  - pm-collab-system (总控)
  - pm-bitable-manager (多维表格)
  - pm-morning-planner (晨间规划)
  - pm-progress-reviewer (进度审核)
  - pm-meeting-assistant (会议辅助)
  - pm-context-manager (上下文管理)
  - pm-socratic-learner (苏格拉底追问)

Cron Jobs: 3个
  - 08:00 晨间规划
  - 09:30 会前摘要
  - 19:00 进度审核

Heartbeat: 每30分钟

下一步：
1. 确保飞书多维表格中已有初始任务数据
2. 确保其他两位团队成员也已安装开发者Agent
3. 明天早上8点，第一个Cron将自动触发晨间规划
```

## 故障排除

| 问题 | 解决方案 |
|------|---------|
| git clone 失败 | 检查网络连接，或让人类手动下载仓库ZIP并解压 |
| Skills 未被 OpenClaw 识别 | 检查 SKILL.md 文件名大小写是否正确，重启 OpenClaw |
| 飞书CLI认证失败 | 重新执行 `lark-cli auth login` |
| 多维表格操作403 | 在飞书开放平台给机器人应用添加多维表格权限 |
| Cron未触发 | 检查 `openclaw cron list` 确认任务状态，检查时区设置 |
| Heartbeat未执行 | 检查 HEARTBEAT.md 是否在工作空间根目录 |
