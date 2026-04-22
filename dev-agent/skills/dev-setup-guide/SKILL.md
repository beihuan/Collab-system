---
name: dev-setup-guide
description: "开发者Agent一次性安装引导。当用户要求安装开发者Agent协作系统、初始化开发者Agent、设置协作环境时使用。此Skill是一段可执行prompt，读取后自动从GitHub仓库克隆所有开发者Agent Skills到本地、安装和配置飞书CLI及其Skills、配置飞书群组连接信息和个人身份、设置定时任务和心跳检查。只需执行一次，安装完成后此Skill不再需要。"
---

# 开发者 Agent 安装引导

> ⚠️ 这是一段可执行指令。请严格按照以下步骤顺序执行，每一步完成后才进入下一步。遇到任何错误立即停止并向人类报告。

## 你即将做什么

你正在安装一个**AI Agent协作项目管理系统**的开发者 Agent 部分。安装完成后，你将成为开发者的双重助手：
1. **开发者的个人助手**：继续帮助开发者写代码、调试、查阅文档
2. **协作系统的节点**：每天接收任务指派、上报工作进度、报告阻塞、追踪代码活动

你将与 Lead Agent 通过飞书群组协作，Lead Agent 负责规划和分配任务，你负责执行和反馈。

**重要**：开发者 Agent 不直接操作飞书多维表格。所有表格更新由 Lead Agent 负责。你只通过飞书群组与 Lead Agent 通信。

## 第一步：确认前置条件

在开始安装前，请逐项确认以下条件是否满足。**任何一项不满足都必须停止安装并告知人类**：

```
前置条件检查清单：

[ ] 1. Node.js (npm) 已安装
     → 验证命令: node --version && npm --version

[ ] 2. Git 已安装
     → 验证命令: git --version

[ ] 3. 项目代码仓库已在本地克隆
     → 验证命令: 在人类提供的仓库路径下执行 git status

[ ] 4. 人类已准备好以下配置信息：
     - CHAT_ID (协作群组ID)
     - APP_ID (飞书机器人App ID)
     - APP_SECRET (飞书机器人App Secret)
     - 开发者姓名
     - 开发者角色（全栈开发/后端开发）
     - Git 作者名和邮箱
     - GitHub 用户名（如有）
     - 各项目仓库的本地路径
```

> 注意：开发者 Agent 不需要 APP_TOKEN 和 TABLE_ID，多维表格由 Lead Agent 管理。

向人类展示检查结果，确认全部通过后继续。

## 第二步：安装飞书 CLI

飞书 CLI 是与飞书平台交互的核心工具。按以下步骤安装：

### 2a. 安装 CLI 工具

```bash
# 安装飞书 CLI
npm install -g @larksuite/cli

# 验证安装
lark-cli --version
```

### 2b. 安装飞书 CLI 的 Agent Skills

飞书 CLI 提供了 22 个面向 AI Agent 的 Skills，安装后 Agent 可以直接操作飞书的各个业务域：

```bash
# 安装飞书 CLI Skills（全局安装）
npx skills add larksuite/cli -y -g
```

### 2c. 配置飞书应用凭证

```bash
# 交互式配置应用凭证（需要人类在终端输入 App ID 和 App Secret）
lark-cli config init --new
```

> 如果此命令输出授权 URL，请将 URL 发送给人类，让其在浏览器中完成操作。

### 2d. 登录飞书

```bash
# 使用应用身份登录（推荐自动选择常用权限范围）
lark-cli auth login --recommend
```

> 同上：如果输出授权 URL，请将 URL 发送给人类在浏览器中完成授权。

### 2e. 验证飞书连接

```bash
# 验证认证状态
lark-cli auth status

# 验证群组可通信
lark-cli chat +message-list --chat-id <CHAT_ID> --limit 1
```

如果以上命令都成功，说明飞书 CLI 配置完成。

## 第三步：从 GitHub 克隆并安装 Skills

### 3a. 克隆仓库

```bash
# 克隆协作系统仓库到临时目录
TMPDIR=$(mktemp -d)
cd "$TMPDIR"
git clone https://github.com/beihuan/Collab-system.git
```

### 3b. 安装 Skills 到本地

**重要**：不同 Agent 平台的 Skills 安装目录不同。请根据你所在平台确定安装路径：

| Agent 平台 | Skills 安装路径 | 确认方法 |
|-----------|---------------|---------|
| OpenClaw | `~/.openclaw/skills/` 或 `~/.openclaw/workspace/skills/` | `ls ~/.openclaw/skills/ 2>/dev/null` |
| Claude Code | `~/.claude/skills/` | `ls ~/.claude/skills/ 2>/dev/null` |
| Hermes | `~/.hermes/skills/` | `ls ~/.hermes/skills/ 2>/dev/null` |
| 其他 | 参考平台文档或询问人类 | — |

**决策规则**：
1. 优先使用工作空间级 Skills 目录（如果存在）
2. 否则使用全局 Skills 目录
3. 如果不确定，询问人类确认

将确定的路径记为 `{SKILLS_DIR}`。

### 3c. 复制开发者 Agent Skills

```bash
# 复制所有开发者 Agent Skills
cp -r "$TMPDIR/Collab-system/dev-agent/skills/"* "{SKILLS_DIR}/"
```

需要安装的 Skills 共 6 个：
- `dev-collab-system` — 总控
- `dev-setup-guide` — 安装引导（本Skill）
- `dev-progress-reporter` — 进度上报
- `dev-task-receiver` — 任务接收
- `dev-blocker-notifier` — 阻塞通知
- `dev-code-activity-tracker` — 代码活动追踪

### 3d. 验证安装

```bash
ls "{SKILLS_DIR}/" | sort
```

确认以上 6 个目录都存在，每个目录下都有 `SKILL.md` 文件。

### 3e. 清理临时目录

```bash
rm -rf "$TMPDIR"
```

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

编辑 `{SKILLS_DIR}/dev-progress-reporter/references/identity.md`，填入：
- 姓名
- 角色
- 飞书用户ID（如已知）
- Git 作者名和邮箱
- GitHub 用户名
- 项目参与信息
- 仓库路径

### 4b. 配置飞书群组连接

编辑 `{SKILLS_DIR}/dev-progress-reporter/references/feishu-config.md`，填入：
- CHAT_ID（协作群组ID）
- APP_ID（飞书机器人App ID）
- APP_SECRET（飞书机器人App Secret）
- 三位成员的飞书用户ID

> 注意：开发者 Agent 的飞书配置只需要群组信息，不需要多维表格的 APP_TOKEN 和 TABLE_ID。

### 4c. 配置代码仓库（dev-code-activity-tracker）

在 `{SKILLS_DIR}/dev-code-activity-tracker/` 下创建 `references/repo-config.md`，内容：

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

## 第五步：验证飞书群组连接

执行以下命令验证飞书 CLI 可以正常访问群组：

```bash
# 验证群组可访问
lark-cli chat +message-list --chat-id <CHAT_ID> --limit 1
```

如果命令失败，请人类检查：
1. 飞书 CLI 认证是否有效
2. 机器人是否有该群组的访问权限
3. CHAT_ID 是否正确

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

## 第七步：配置定时任务

根据你所在 Agent 平台的定时任务机制，设置以下 2 个任务：

| 时间 | 名称 | 说明 |
|------|------|------|
| 工作日 09:00 | morning-task-check | 检查飞书群组中的今日任务指派并展示给开发者 |
| 工作日 18:00 | evening-progress-report | 收集今日工作，生成进度报告，确认后发群组 |

**各平台配置方式参考**：

- **OpenClaw**: 通过 `openclaw.json` 的 `cron.jobs` 配置或对话式创建
- **Hermes**: 使用 `cronjob` 工具，schedule 格式如 `0 9 * * 1-5`
- **Claude Code**: 参考平台文档

如果不确定平台如何配置定时任务，先跳过此步，稍后手动配置。

## 第八步：配置心跳检查

根据你所在 Agent 平台的心跳/定期检查机制，设置以下检查项：

- 飞书群组是否有 Lead Agent 发送的新任务指派？如有，提醒开发者查看
- 进行中的任务最近4小时是否有代码提交？如无，询问开发者是否遇到阻塞
- 是否有PR需要处理（审核/合并）？如有，提醒开发者
- 是否有截止日期在今天或明天的任务？如有，提醒开发者注意优先级

**触发条件**：
- 有新任务指派 → 提醒开发者查看
- 4小时无代码活动且任务进行中 → 询问是否阻塞
- 有PR待处理 → 提醒开发者
- 任务即将到期 → 提醒优先级
- 其他情况 → 一切正常

## 第九步：端到端验证

请人类配合进行一次完整流程验证：

```
安装验证清单：

[ ] 1. 飞书 CLI 已安装且认证成功: lark-cli auth status
[ ] 2. 飞书 CLI Skills 已安装: npx skills list
[ ] 3. 手动触发 dev-task-receiver，验证能否从飞书群组读取消息
[ ] 4. 手动触发 dev-progress-reporter，验证能否收集工作信息并生成报告
[ ] 5. 验证报告经确认后能否发送到飞书群组
[ ] 6. 检查定时任务是否正常注册
[ ] 7. 检查心跳检查是否正常
[ ] 8. 验证 dev-code-activity-tracker 能否读取项目仓库的 git log
```

## 安装完成

如果所有验证通过，向人类展示：

```
✅ 开发者 Agent 安装完成！

飞书 CLI:
  - 版本: (lark-cli --version)
  - 认证: ✅
  - CLI Skills: ✅ (22个飞书Agent Skills)

已安装协作 Skills: 6个
  - dev-collab-system (总控)
  - dev-progress-reporter (进度上报)
  - dev-task-receiver (任务接收)
  - dev-blocker-notifier (阻塞通知)
  - dev-code-activity-tracker (代码活动追踪)
  - dev-setup-guide (安装引导)

定时任务: 2个
  - 09:00 任务接收
  - 18:00 进度上报

心跳检查: 每30分钟

身份配置:
  - 开发者: {姓名}
  - 角色: {角色}
  - 仓库: {项目A路径}, {项目B路径}

飞书通信:
  - 群组ID: {CHAT_ID}
  - 不操作多维表格（由 Lead Agent 管理）

下一步：
1. 确保 Lead Agent 已安装并运行
2. 明天早上9点，第一个定时任务将自动检查任务指派
3. 工作中如遇阻塞，随时告诉我，我会帮你通知 Lead Agent
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
| npm install 失败 | 检查 Node.js 版本，尝试 `npm cache clean --force` 后重试 |
| lark-cli 命令未找到 | 确认 npm 全局安装路径在 PATH 中：`npm config get prefix` |
| 飞书 CLI Skills 安装失败 | 确认 npx 可用，尝试手动执行 `npx skills add larksuite/cli -y -g` |
| lark-cli config init 失败 | 检查是否已有配置文件冲突：`ls ~/.config/lark-cli/` |
| lark-cli auth login 失败 | 确认 App ID 和 App Secret 正确，检查飞书开放平台应用状态 |
| git clone 失败 | 检查网络连接，或让人类手动下载仓库ZIP并解压 |
| Skills 未被 Agent 识别 | 检查 SKILL.md 文件名大小写，检查安装路径是否正确，重启 Agent |
| 飞书群组消息发送失败 | 检查机器人是否在群组中，检查 CHAT_ID 是否正确 |
| git log 无结果 | 检查仓库路径是否正确，检查 git config user.name 是否匹配 |
| 定时任务未触发 | 检查定时任务状态，检查时区设置 |
| 进度报告发送失败 | 检查飞书群组权限，确认机器人已在群组中 |
