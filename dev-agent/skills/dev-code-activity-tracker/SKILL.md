---
name: dev-code-activity-tracker
description: Tracks developer's code activity (git commits, PRs, code reviews) and associates them with Bitable tasks. Use when generating progress reports, when user asks to "check my code activity", "what did I commit today", "show my PRs", or when the heartbeat checks for activity on in-progress tasks. Provides data for progress reports and blocker detection.
---

# Dev Code Activity Tracker

## Overview

This skill monitors a developer's code activity and associates it with tasks in the Bitable. It provides the data foundation for progress reports and blocker detection, enabling the collaboration system to have visibility into actual development work beyond self-reported progress.

## Prerequisites

- Git installed and configured
- Access to project repositories (configured in `references/repo-config.md`)
- Feishu CLI authenticated (for reading Bitable task info)
- Read `references/identity.md` for your name and git author info

## Workflow

### Collecting Code Activity

#### Git Commits

```bash
# Today's commits across all repos
for repo in {REPO_PATHS}; do
  echo "=== $(basename $repo) ==="
  cd "$repo"
  git log --author="{GIT_AUTHOR}" --since="today" --format="%h %s" 2>/dev/null
done
```

#### Pull Requests

```bash
# Open PRs
for repo in {REPO_PATHS}; do
  cd "$repo"
  gh pr list --author="{GITHUB_USER}" --state open --json number,title,createdAt 2>/dev/null
done

# Recently merged PRs
for repo in {REPO_PATHS}; do
  cd "$repo"
  gh pr list --author="{GITHUB_USER}" --state merged --json number,title,mergedAt 2>/dev/null
done
```

#### Code Changes

```bash
# Lines changed today
for repo in {REPO_PATHS}; do
  cd "$repo"
  git log --author="{GIT_AUTHOR}" --since="today" --numstat --format="" 2>/dev/null | \
    awk '{added+=$1; deleted+=$2} END {print "Added:", added, "Deleted:", deleted}'
done
```

### Associating Commits with Tasks

The skill uses multiple strategies to associate git activity with Bitable tasks:

#### Strategy 1: Branch Name Matching

```bash
# Branch names like: feature/TASK-0001-login-api, fix/TASK-0015-payment-callback
git branch --show-current
# Extract task ID with regex: TASK-\d{4}
```

#### Strategy 2: Commit Message Matching

```bash
# Commit messages like: "feat(TASK-0001): implement JWT validation"
git log --author="{GIT_AUTHOR}" --since="today" --format="%h %s"
# Extract task ID with regex: TASK-\d{4}
```

#### Strategy 3: Human Confirmation

If no task ID is found in branch name or commit message:

```
我检测到以下代码活动无法自动关联到任务：

📝 提交: abc1234 "优化数据库查询性能"
📂 变更: src/db/queries.ts (+12/-5)

这个提交关联哪个任务？
1. [TASK-0002] 数据库迁移
2. [TASK-0010] 性能优化
3. 计划外工作（不需要关联任务）
4. 其他任务ID: ______
```

### Generating Activity Summary

```markdown
🔧 代码活动摘要 — {PERSON_NAME} — {DATE}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 提交记录 (5个提交)

| 提交 | 消息 | 关联任务 |
|------|------|---------|
| abc1234 | feat: 实现JWT验证 | TASK-0001 |
| def5678 | test: 添加登录API单测 | TASK-0001 |
| ghi9012 | fix: 修复token过期处理 | TASK-0001 |
| jkl3456 | docs: 更新API文档 | TASK-0001 |
| mno7890 | refactor: 优化查询性能 | TASK-0010 |

## Pull Requests

| PR | 标题 | 状态 | 关联任务 |
|----|------|------|---------|
| #42 | feat: 用户登录API | 待审核 | TASK-0001 |
| #40 | fix: 支付回调bug | 已合并 | TASK-0015 |

## 代码变更统计

| 项目 | 新增行 | 删除行 | 变更文件数 |
|------|--------|--------|-----------|
| 项目A | +213 | -17 | 8 |
| 项目B | +0 | -0 | 0 |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Integration with Other Skills

| Skill | How Code Activity Data Is Used |
|-------|-------------------------------|
| dev-progress-reporter | Auto-fills git activity section of progress report |
| dev-blocker-notifier | Detects lack of code activity as potential blocker signal |
| pm-progress-reviewer | PM Agent uses code activity to validate reported progress |

### Heartbeat Integration

Every 30 minutes, check for activity on in-progress tasks:

```bash
# For each in-progress task assigned to this person
for task in {IN_PROGRESS_TASKS}; do
  # Check if there are recent commits related to this task
  recent_commits=$(git log --author="{GIT_AUTHOR}" --since="4 hours ago" --grep="$task" --format="%h" 2>/dev/null)
  if [ -z "$recent_commits" ]; then
    echo "⚠️ No recent activity on $task"
  fi
done
```

## Configuration

### references/repo-config.md

```markdown
# 代码仓库配置

## 项目A
- 路径: /home/user/project-a
- Git作者: Your Name <your@email.com>
- GitHub用户: your-github-username

## 项目B
- 路径: /home/user/project-b
- Git作者: Your Name <your@email.com>
- GitHub用户: your-github-username
```

## Error Handling

| Scenario | Action |
|----------|--------|
| Git repo not found | Skip that repo, report to human |
| gh CLI not available | Skip PR tracking, rely on git log only |
| No commits today | Report "no code activity" — this is valid for non-coding days |
| Task ID not found in Bitable | Flag as potentially new task or incorrect ID |
