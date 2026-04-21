---
name: dev-setup-guide
description: "开发者Agent一次性安装引导。当用户要求安装开发者Agent协作系统、初始化开发者Agent、设置协作环境时使用。此Skill是一段可执行prompt，读取后自动从GitHub仓库克隆所有开发者Agent Skills到本地、配置飞书连接信息和个人身份、设置Cron Jobs和Heartbeat。只需执行一次，安装完成后此Skill不再需要。"
---

# 开发者 Agent 安装引导

> ⚠️ 这是一段可执行指令。请严格按照以下步骤顺序执行，每一步完成后才进入下一步。遇到任何错误立即停止并向人类报告。

## 你即将做什么

你正在安装一个**AI Agent协作项目管理系统**的开发者 Agent 部分。安装完成后，你将成为开发者的双重助手：
1. **开发者的个人助手**：继续帮助开发者写代码、调试、查阅文档
2. **协作系统的节点**：每天接收任务指派、上报工作进度、报告阻塞、追踪代码活动

你将与 PM Agent 通过飞书群组协作，PM Agent 负责规划和分配任务，你负责执行和反馈。

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

[ ] 4. 飞书协作群组已创建，机器人已加入
     → 验证命令: lark-cli chat list

[ ] 5. 项目代码仓库已在本地克隆
     → 验证命令: 在人类提供的仓库路径下执行 git status

[ ] 6. 人类已准备好以下配置信息：
     - APP_TOKEN (多维表格应用token)
     - TABLE_ID (任务看板表ID)
     - CHAT_ID (协作群组ID)
     - 开发者姓名
     - 开发者角色（全栈开发/后端开发）
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

**安装路径选择规则**：
- 如果当前有 OpenClaw 工作空间 → 安装到工作空间 Skills 目录
- 如果没有工作空间 → 安装到全局 Skills 目录 `~/.openclaw/skills/`

将确定的安装路径记录为变量 `SKILLS_DIR`，后续步骤中使用。

## 第三步：从 GitHub 克隆 Skills

执行以下命令，将仓库克隆到临时目录，然后复制所有开发者 Agent Skills：

```bash
# 克隆仓库到临时目录
TMP_DIR=$(mktemp -d)
git clone https://github.com/beihuan/Collab-system.git "$TMP_DIR/Collab-system"

# 复制所有开发者 Agent Skills 到本地 Skills 目录
# 包括: dev-collab-system, dev-progress-reporter, dev-task-receiver,
#       dev-blocker-notifier, dev-code-activity-tracker
cp -r "$TMP_DIR/Collab-system/dev-agent/skills/"* "$SKILLS_DIR/"

# 验证复制结果
echo "已安装的开发者 Agent Skills:"
ls -1 "$SKILLS_DIR/" | grep "^dev-"

# 清理临时目录
rm -rf "$TMP_DIR"
```

**验证安装**：确认以下5个 Skill 目录都已存在：
- [ ] `dev-collab-system/` — 总控 Skill
- [ ] `dev-progress-reporter/` — 进度上报
- [ ] `dev-task-receiver/` — 任务接收
- [ ] `dev-blocker-notifier/` — 阻塞通知
- [ ] `dev-code-activity-tracker/` — 代码活动追踪

每个目录下必须有 `SKILL.md` 文件。如果任何 Skill 缺失，重新执行复制命令。

## 第四步：配置个人身份信息

开发者 Agent 需要知道"我是谁的助手"。请向人类索取以下信息：

```
请提供你的个人信息：
1. 你的姓名（在团队中使用的名称）:
2. 你的角色（全栈开发 / 后端开发）:
3. 你的 Git 作者名（git config user.name 的值）:
4. 你的 Git 邮箱（git config user.email 的值）:
5. 你的 GitHub 用户名（如有）:
6. 项目A的本地仓库路径:
7. 项目B的本地仓库路径:
```

获取信息后，编辑以下配置文件：

### 4a. 配置身份信息

编辑 `$SKILLS_DIR/dev-progress-reporter/references/identity.md`，填入：
- 姓名
- 角色
- 飞书用户ID（如已知）
- Git 作者名和邮箱
- GitHub 用户名
- 项目参与信息
- 仓库路径

### 4b. 配置飞书连接

编辑 `$SKILLS_DIR/dev-progress-reporter/references/feishu-config.md`，填入：
- APP_TOKEN
- TABLE_ID
- CHAT_ID
- APP_ID
- APP_SECRET
- 三位成员的飞书用户ID

### 4c. 配置代码仓库（dev-code-activity-tracker）

在 `$SKILLS_DIR/dev-code-activity-tracker/` 下创建 `references/repo-config.md`，内容：

```markdown
# 代码仓库配置

## 项目A
- 路径: {项目A仓库路径}
- Git作者: {GIT_AUTHOR_NAME} <{GIT_AUTHOR_EMAIL}>
- GitHub用户: {GITHUB_USERNAME}

## 项目B
- 路径: {项目B仓库路径}
- Git作者: {GIT_AUTHOR_NAME} <{GIT_AUTHOR_EMAIL}>
- GitHub用户: {GITHUB_USERNAME}
```

## 第五步：验证飞书连接

执行以下命令验证飞书CLI可以正常访问群组和多维表格：

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

## 第六步：验证代码仓库访问

对每个项目仓库执行：

```bash
# 验证仓库可访问
cd {REPO_PATH} && git status

# 验证git用户配置
cd {REPO_PATH} && git config user.name && git config user.email

# 验证最近提交（确认git log可用）
cd {REPO_PATH} && git log --oneline -3
```

如果任一仓库不可访问，请人类确认仓库路径是否正确。

## 第七步：配置 Cron Jobs

开发者 Agent 需要两个定时任务。请根据你的 OpenClaw 版本选择配置方式：

### 方式A：通过对话创建（推荐）

直接告诉 OpenClaw 以下两句话，让它自动创建 Cron Job：

```
帮我创建一个定时任务：工作日每天早上9点，执行 dev-task-receiver Skill，检查飞书群组中的今日任务指派并展示给我

帮我创建一个定时任务：工作日每天晚上6点，执行 dev-progress-reporter Skill，收集今日代码活动和工作进展，生成结构化进度报告，经我确认后发送到飞书群组
```

### 方式B：编辑配置文件

如果 OpenClaw 使用 `openclaw.json` 配置文件，请在 `cron.jobs` 数组中添加：

```json
{
  "cron": {
    "jobs": [
      {
        "name": "morning-task-check",
        "schedule": {
          "kind": "cron",
          "expr": "0 9 * * 1-5",
          "tz": "Asia/Shanghai"
        },
        "payload": {
          "kind": "agentTurn",
          "message": "执行每日早晨任务检查：先加载 dev-collab-system 总控Skill了解全局上下文，然后执行 dev-task-receiver Skill，检查飞书群组中PM Agent发送的今日任务指派，读取并展示给开发者"
        },
        "sessionTarget": "isolated"
      },
      {
        "name": "evening-progress-report",
        "schedule": {
          "kind": "cron",
          "expr": "0 18 * * 1-5",
          "tz": "Asia/Shanghai"
        },
        "payload": {
          "kind": "agentTurn",
          "message": "执行每日晚间进度上报：先加载 dev-collab-system 总控Skill了解全局上下文，然后执行 dev-progress-reporter Skill，收集今日代码活动（git提交、PR状态）和开发者手动输入，生成结构化进度报告，经开发者确认后发送到飞书群组"
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

确认两个任务都已注册且状态为 active。

## 第八步：配置 Heartbeat

开发者 Agent 的 Heartbeat 每30分钟执行一次，检查新任务指派、代码活动、PR状态等。

### 方式A：创建 HEARTBEAT.md

在 OpenClaw 工作空间根目录创建 `HEARTBEAT.md`，内容如下：

```markdown
# 开发者 Agent Heartbeat

## 每次检查

- [ ] 飞书群组是否有PM Agent发送的新任务指派？如有，提醒开发者查看
- [ ] 进行中的任务最近4小时是否有代码提交？如无，询问开发者是否遇到阻塞
- [ ] 是否有PR需要处理（审核/合并）？如有，提醒开发者
- [ ] 是否有截止日期在今天或明天的任务？如有，提醒开发者注意优先级

## 触发条件

- 有新任务指派 → 提醒开发者查看
- 4小时无代码活动且任务进行中 → 询问是否阻塞
- 有PR待处理 → 提醒开发者
- 任务即将到期 → 提醒优先级
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

## 第九步：端到端验证

请人类配合进行一次完整流程验证：

```
安装验证清单：

[ ] 1. 手动触发 dev-task-receiver，验证能否从飞书群组读取消息
[ ] 2. 手动触发 dev-progress-reporter，验证能否收集git活动并生成报告
[ ] 3. 验证报告经确认后能否发送到飞书群组
[ ] 4. 检查 Cron Jobs 是否正常注册: openclaw cron list
[ ] 5. 检查 Heartbeat 是否正常: 等待下一次心跳或手动触发
[ ] 6. 验证 dev-code-activity-tracker 能否读取项目仓库的git log
```

## 安装完成

如果所有验证通过，向人类展示：

```
✅ 开发者 Agent 安装完成！

已安装 Skills: 5个
  - dev-collab-system (总控)
  - dev-progress-reporter (进度上报)
  - dev-task-receiver (任务接收)
  - dev-blocker-notifier (阻塞通知)
  - dev-code-activity-tracker (代码活动追踪)

Cron Jobs: 2个
  - 09:00 任务接收
  - 18:00 进度上报

Heartbeat: 每30分钟

身份配置:
  - 开发者: {姓名}
  - 角色: {角色}
  - 仓库: {项目A路径}, {项目B路径}

下一步：
1. 确保PM Agent已安装并运行
2. 确保飞书多维表格中已有分配给你的任务
3. 明天早上9点，第一个Cron将自动检查任务指派
4. 工作中如遇阻塞，随时告诉我，我会帮你通知PM Agent
```

## 日常使用提示

安装完成后，你在日常工作中可以这样与 Agent 交互：

| 场景 | 你说的话 | Agent 的行为 |
|------|---------|-------------|
| 查看今日任务 | "今天有什么任务？" | 触发 dev-task-receiver |
| 提交进度报告 | "帮我写今天的进度报告" | 触发 dev-progress-reporter |
| 报告阻塞 | "我被 [TASK-XXXX] 阻塞了" | 触发 dev-blocker-notifier |
| 查看代码活动 | "我今天提交了什么？" | 触发 dev-code-activity-tracker |
| 了解协作系统 | "协作系统是怎么工作的？" | 加载 dev-collab-system 总控 |

## 故障排除

| 问题 | 解决方案 |
|------|---------|
| git clone 失败 | 检查网络连接，或让人类手动下载仓库ZIP并解压 |
| Skills 未被 OpenClaw 识别 | 检查 SKILL.md 文件名大小写是否正确，重启 OpenClaw |
| 飞书CLI认证失败 | 重新执行 `lark-cli auth login` |
| 多维表格操作403 | 在飞书开放平台给机器人应用添加多维表格权限 |
| git log 无结果 | 检查仓库路径是否正确，检查 git config user.name 是否匹配 |
| Cron未触发 | 检查 `openclaw cron list` 确认任务状态，检查时区设置 |
| Heartbeat未执行 | 检查 HEARTBEAT.md 是否在工作空间根目录 |
| 进度报告发送失败 | 检查飞书群组权限，确认机器人已在群组中 |
