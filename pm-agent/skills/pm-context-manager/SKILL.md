---
name: pm-context-manager
description: Manages PM Agent's understanding of the project by reading code repositories, existing documents, and maintaining a persistent project context document. Use when initializing PM Agent for the first time, when user asks to "update project context", "review project understanding", "read codebase", or when the Socratic learning skill identifies context gaps that require deeper investigation.
---

# PM Context Manager

## Overview

This skill is responsible for building and maintaining the PM Agent's understanding of the projects. It reads code repositories and existing documents, maintains a persistent context document, and ensures the PM Agent always has up-to-date project knowledge for making informed decisions.

## Prerequisites

- Access to project code repositories (configured in `references/repo-config.md`)
- Feishu CLI authenticated (for reading existing Feishu documents)
- Read `references/project-context.md` for current understanding

## Workflow

### Initial Context Building (First-Time Setup)

When the PM Agent is first initialized, follow this sequence:

#### Step 1: Read Code Repositories

```bash
# For each project repository
cd {REPO_PATH}

# Understand project structure
ls -la
cat README.md 2>/dev/null || echo "No README found"
cat package.json 2>/dev/null || cat requirements.txt 2>/dev/null || cat go.mod 2>/dev/null || echo "No dependency file found"
cat .env.example 2>/dev/null || echo "No env example found"

# Understand architecture
ls -la src/ 2>/dev/null || ls -la app/ 2>/dev/null || ls -la lib/ 2>/dev/null
cat docs/architecture.md 2>/dev/null || echo "No architecture doc found"
cat docs/api.md 2>/dev/null || echo "No API doc found"

# Understand current development status
git log --oneline -20
git branch -a
```

#### Step 2: Read Existing Documents

```bash
# List and read Feishu documents if available
lark-cli doc list --folder-token {FOLDER_TOKEN} --limit 20

# Read specific documents
lark-cli doc get --doc-id {DOC_ID}
```

#### Step 3: Generate Initial Understanding

Based on the code and document analysis, generate an initial project context:

```markdown
# 项目上下文 — 初始理解

## 项目A: {PROJECT_NAME}

### 我的理解
[基于代码和文档的分析]

### 技术架构
[从代码结构推断]

### 当前开发状态
[从git log和代码分析]

### 不确定的地方
1. ...
2. ...

## 项目B: {PROJECT_NAME}
[同上]
```

#### Step 4: Socratic Verification

Present the initial understanding to the human and ask deep questions:

```
我已阅读了项目代码库和文档，以下是我的初步理解：

[展示理解]

我有以下深层问题需要确认：

1. 我注意到项目A使用了 {TECHNOLOGY}，但代码中有两套实现（{PATH_A} 和 {PATH_B}）。
   → 这两套实现的关系是什么？是在做技术迁移吗？
   → 当前应该以哪套为准？对任务分配有什么影响？

2. 项目B的 {MODULE} 似乎没有测试代码。
   → 这是有意为之（MVP阶段先不写测试）还是遗漏？
   → 对代码质量和后续维护有什么考虑？

3. 两个项目之间似乎有共享的 {COMPONENT}。
   → 这个共享组件的维护责任属于谁？
   → 修改时如何确保不破坏另一个项目？
```

### Ongoing Context Maintenance

#### Trigger: When Socratic Learning Identifies Gaps

When `pm-socratic-learner` identifies a context gap, this skill is invoked to:

1. Read the relevant code/docs to fill the gap
2. Update `references/project-context.md`
3. Present the updated understanding for human verification

#### Trigger: When Major Code Changes Occur

When the progress reviewer detects significant code changes (new modules, architecture changes):

1. Read the changed code areas
2. Update the context document
3. Flag any implications for task planning

#### Trigger: When Meeting Decisions Affect Architecture

After processing meeting minutes that contain architectural decisions:

1. Update the context document
2. Verify the changes are consistent with existing understanding
3. Flag any contradictions

## Context Document Structure

The project context document (`references/project-context.md`) should contain:

```markdown
# 项目上下文

> 最后更新: {DATE}
> 更新原因: {REASON}

## 项目A: {NAME}

### 项目概述
[1-2段描述项目的目标和价值]

### 技术架构
- 前端: {TECH_STACK}
- 后端: {TECH_STACK}
- 数据库: {TECH_STACK}
- 部署: {DEPLOYMENT_INFO}

### 核心模块
1. {MODULE_1}: {DESCRIPTION}, 位于 {PATH}
2. {MODULE_2}: {DESCRIPTION}, 位于 {PATH}

### 关键决策记录
| 日期 | 决策 | 原因 | 影响 |
|------|------|------|------|
| {DATE} | {DECISION} | {REASON} | {IMPACT} |

### 已知约束
- {CONSTRAINT_1}
- {CONSTRAINT_2}

### 当前焦点
- {CURRENT_FOCUS}

---

## 项目B: {NAME}
[同上结构]

---

## 跨项目信息

### 共享组件
- {SHARED_COMPONENT}: {DESCRIPTION}, 维护者: {PERSON}

### 资源分配原则
- {PRINCIPLE_1}
- {PRINCIPLE_2}

### 优先级冲突解决规则
- {RULE_1}
- {RULE_2}
```

## Context Update Rules

1. **Never delete information** — only add or mark as outdated
2. **Always record the reason** for each update
3. **Always record the date** of each update
4. **When understanding changes**, keep both old and new understanding with explanation
5. **When human corrects**, record the correction in "关键决策记录"

## Error Handling

| Scenario | Action |
|----------|--------|
| Repository not accessible | Ask human for the correct path or access |
| No README or docs found | Rely more on code analysis and human Q&A |
| Conflicting information | Ask human to clarify, record both versions |
| Context document becomes too large | Archive older sections, keep recent context prominent |
