---
name: setup-guide
description: "Agent一次性安装引导。当用户要求安装协作系统、初始化Agent、设置协作环境时使用。此Skill是一段可执行prompt，读取后自动从GitHub仓库克隆所有Skills到本地、安装和配置飞书CLI及其Skills、配置飞书连接信息和个人身份、设置定时任务和心跳检查。只需执行一次，安装完成后此Skill不再需要。"
---

# Agent 安装引导（Lead Agent — PM+Dev合并版）

> ⚠️ 这是一段可执行指令。请严格按照以下步骤顺序执行，每一步完成后才进入下一步。遇到任何错误立即停止并向人类报告。

## 你即将做什么

你正在安装一个**AI Agent协作项目管理系统**，你同时承担 PM 和开发者双重角色。安装完成后，你将：
- **作为PM**：每天早晨为三位团队成员规划并分配任务，晚间审核进度并更新飞书多维表格
- **作为开发者**：帮助 Person A 做全栈开发，追踪代码活动，上报工作进度，报告阻塞

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

# 验证多维表格可读写
lark-cli base +record-list --app-token <APP_TOKEN> --table-id <TABLE_ID> --limit 1

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

### 3c. 复制 Lead Agent Skills

```bash
# 复制所有 Lead Agent Skills
cp -r "$TMPDIR/Collab-system/lead-agent/skills/"* "{SKILLS_DIR}/"
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

### 3d. 验证安装

```bash
ls "{SKILLS_DIR}/" | sort
```

确认以上 11 个目录都存在，每个目录下都有 `SKILL.md` 文件。

### 3e. 清理临时目录

```bash
rm -rf "$TMPDIR"
```

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

## 第五步：配置定时任务

根据你所在 Agent 平台的定时任务机制，设置以下 4 个任务：

| 时间 | 名称 | 说明 | 角色模式 |
|------|------|------|---------|
| 工作日 08:00 | morning-planning | 读取多维表格，生成任务指派，B/C发群组+A终端展示 | PM |
| 工作日 09:30 | pre-meeting-summary | 生成会议议程，发群组 | PM |
| 工作日 18:00 | evening-progress-report | 收集今日工作，生成进度报告，确认后发群组 | Dev |
| 工作日 19:00 | evening-review | 审核所有报告，分析偏差，更新bitable | PM |

**各平台配置方式参考**：

- **OpenClaw**: 通过 `openclaw.json` 的 `cron.jobs` 配置或对话式创建
- **Hermes**: 使用 `cronjob` 工具，schedule 格式如 `0 8 * * 1-5`
- **Claude Code**: 参考平台文档

如果不确定平台如何配置定时任务，先跳过此步，稍后手动配置。

## 第六步：配置心跳检查

根据你所在 Agent 平台的心跳/定期检查机制，设置以下检查项：

**PM角色检查项**：
- 飞书群组是否有新的阻塞报告，如有则分析影响范围并通知人类
- 是否有人在19:30前未提交进度报告，如有则发送提醒
- 多维表格中是否有逾期任务，如有则通知人类
- 是否有新的会议纪要需要处理
- 项目上下文文档是否需要更新

**Dev角色检查项**：
- 进行中的任务是否有代码活动（4小时无提交则询问是否遇到阻塞）
- 是否有PR需要处理（审核/合并）
- 分配给自己的任务是否有截止日期临近的

**综合判断**：
- 如果有阻塞报告 → PM角色优先处理（评估影响+更新bitable）
- 如果无异常 → 一切正常

## 第七步：验证飞书连接

执行以下命令验证飞书 CLI 可以正常访问：

```bash
# 验证多维表格可读写
lark-cli base +record-list --app-token <APP_TOKEN> --table-id <TABLE_ID> --limit 1

# 验证群组可通信
lark-cli chat +message-list --chat-id <CHAT_ID> --limit 1

# 验证可以发送消息（发送一条测试消息，人类确认后可删除）
lark-cli chat +message-send --chat-id <CHAT_ID> \
  --msg-type text \
  --content "🔧 Lead Agent 安装测试 — 请忽略此消息"
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

[ ] 1. 飞书 CLI 已安装且认证成功: lark-cli auth status
[ ] 2. 飞书 CLI Skills 已安装: npx skills list
[ ] 3. 手动触发 morning-planner，验证能否读取多维表格并生成任务指派
[ ] 4. 验证 Person B/C 的任务指派能发送到飞书群组
[ ] 5. 验证 Person A 的任务能在终端直接展示
[ ] 6. 手动触发 progress-reporter，验证能否收集工作信息并生成报告
[ ] 7. 验证报告经确认后能发送到飞书群组
[ ] 8. 手动触发 progress-reviewer，验证能分析偏差并更新bitable
[ ] 9. 检查定时任务是否正常注册
[ ] 10. 检查心跳检查是否正常
```

## 安装完成

如果所有验证通过，向人类展示：

```
✅ 协作系统安装完成！（Lead Agent — PM+Dev合并版）

飞书 CLI:
  - 版本: (lark-cli --version)
  - 认证: ✅
  - CLI Skills: ✅ (22个飞书Agent Skills)

已安装协作 Skills: 11个
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

定时任务: 4个
  - 08:00 晨间规划（PM角色）
  - 09:30 会前摘要（PM角色）
  - 18:00 进度上报（Dev角色）
  - 19:00 进度审核（PM角色）

心跳检查: 每30分钟（PM+Dev双角色检查）

下一步：
1. 确保飞书多维表格中已有初始任务数据
2. 确保Person B和C也已安装开发者Agent
3. 明天早上8点，第一个定时任务将自动触发晨间规划
```

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
| 多维表格操作403 | 在飞书开放平台给机器人应用添加多维表格权限 |
| 群组消息发送失败 | 检查机器人是否在群组中，检查 CHAT_ID |
| git log 无结果 | 检查仓库路径是否正确，检查 git config user.name |
| 定时任务未触发 | 检查定时任务状态，检查时区设置 |
