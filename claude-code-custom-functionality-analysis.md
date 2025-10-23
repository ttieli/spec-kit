# Claude Code 自定义功能实现分析

**项目**: Spec Kit
**分析日期**: 2025-10-23
**分析目标**: 理解如何在 Claude Code 中实现自定义斜杠命令（Slash Commands）

## 目录

1. [概述](#概述)
2. [核心架构](#核心架构)
3. [命令实现详解](#命令实现详解)
   - [/speckit.specify 命令](#speckit-specify-命令)
   - [/speckit.plan 命令](#speckit-plan-命令)
4. [文件类型说明](#文件类型说明)
5. [工作流程图](#工作流程图)
6. [关键技术点](#关键技术点)

---

## 概述

Spec Kit 项目通过 Claude Code 的自定义命令功能，实现了一套完整的规范驱动开发（Spec-Driven Development）工作流。核心机制是：

1. **命令定义文件** (`.md`)：定义命令的行为、参数和执行逻辑
2. **Shell 脚本** (`.sh`)：执行实际的文件操作和环境配置
3. **模板文件** (`.md`)：生成标准化文档的骨架

### 文件组织结构

```text
spec-kit/
├── templates/                    # 模板目录
│   ├── commands/                 # 命令定义文件
│   │   ├── specify.md           # /speckit.specify 命令定义
│   │   ├── plan.md              # /speckit.plan 命令定义
│   │   ├── tasks.md             # /speckit.tasks 命令定义
│   │   ├── implement.md         # /speckit.implement 命令定义
│   │   └── ...
│   ├── spec-template.md         # 规范文档模板
│   ├── plan-template.md         # 实现计划模板
│   ├── tasks-template.md        # 任务列表模板
│   └── agent-file-template.md   # Agent 上下文文件模板
├── scripts/
│   └── bash/                     # Bash 脚本
│       ├── create-new-feature.sh      # 创建新功能分支和规范
│       ├── setup-plan.sh              # 设置实现计划
│       ├── common.sh                  # 公共函数库
│       ├── update-agent-context.sh    # 更新 AI agent 上下文
│       └── check-prerequisites.sh     # 检查前置条件
└── memory/
    └── constitution.md           # 项目原则和约束
```

---

## 核心架构

### 1. 命令定义文件 (Command Definition Files)

位置：`templates/commands/*.md`

这些文件定义了 Claude Code 识别的斜杠命令。每个文件包含：

#### Front Matter (YAML 头部)

```yaml
---
description: "命令的简短描述"
scripts:
  sh: "scripts/bash/script-name.sh --json"        # Bash 脚本路径
  ps: "scripts/powershell/script-name.ps1 -Json"  # PowerShell 脚本路径
agent_scripts:                                     # (可选) Agent 上下文更新脚本
  sh: "scripts/bash/update-agent-context.sh __AGENT__"
  ps: "scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__"
---
```

#### 命令执行指令 (Markdown Body)

包含给 AI 的详细指令，告诉它：
- 如何解析用户输入
- 调用哪些脚本
- 如何处理脚本输出
- 生成哪些文件
- 使用哪些模板
- 验证规则

### 2. Shell 脚本 (Shell Scripts)

位置：`scripts/bash/*.sh`

脚本负责：
- 创建目录和文件
- 切换 Git 分支
- 解析命令行参数
- 输出 JSON 格式的结果供 AI 解析
- 环境变量管理

### 3. 模板文件 (Template Files)

位置：`templates/*.md`

提供标准化文档结构，包含：
- 占位符（如 `[FEATURE NAME]`, `[DATE]`）
- 章节标题和格式
- 注释说明
- 示例内容

---

## 命令实现详解

### `/speckit.specify` 命令

**作用**: 从自然语言描述创建功能规范文档

#### 1. 命令定义文件

**文件**: `templates/commands/specify.md`

```yaml
---
description: Create or update the feature specification from a natural language feature description.
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---
```

**关键指令**:

```markdown
## Outline

1. **Generate a concise short name** (2-4 words) for the branch
2. Run the script `{SCRIPT}` from repo root **with the short-name argument**
3. Load `templates/spec-template.md` to understand required sections
4. Follow this execution flow:
   - Parse user description from Input
   - Extract key concepts
   - Fill User Scenarios & Testing section
   - Generate Functional Requirements
   - Define Success Criteria
   - Identify Key Entities
5. Write the specification to SPEC_FILE
6. **Specification Quality Validation**: Validate against quality criteria
7. Report completion with branch name, spec file path
```

**执行流程**:

1. AI 从用户输入中提取关键词，生成简短名称（如 "user-auth"）
2. 调用 `create-new-feature.sh --short-name "user-auth" "用户描述..."`
3. 脚本返回 JSON：`{"BRANCH_NAME":"001-user-auth","SPEC_FILE":"/path/to/spec.md",...}`
4. AI 读取 `spec-template.md`
5. AI 根据用户描述填充模板
6. AI 运行质量检查，生成检查清单
7. AI 将填充后的内容写入 `SPEC_FILE`

#### 2. Shell 脚本

**文件**: `scripts/bash/create-new-feature.sh`

**核心功能**:

```bash
# 1. 解析参数
--json              # 输出 JSON 格式
--short-name <name> # 自定义短名称
<feature_description> # 功能描述

# 2. 确定仓库根目录
find_repo_root() {
    # 查找 .git 或 .specify 目录
}

# 3. 计算功能编号
HIGHEST=0
for dir in "$SPECS_DIR"/*; do
    number=$(echo "$dirname" | grep -o '^[0-9]\+')
    if [ "$number" -gt "$HIGHEST" ]; then HIGHEST=$number; fi
done
NEXT=$((HIGHEST + 1))
FEATURE_NUM=$(printf "%03d" "$NEXT")

# 4. 生成分支名称
generate_branch_name() {
    # 过滤停用词（the, a, is, to 等）
    # 保留有意义的词（长度 >= 3 或大写缩写）
    # 取前 3-4 个词，用连字符连接
}

BRANCH_NAME="${FEATURE_NUM}-${BRANCH_SUFFIX}"

# 5. 创建分支和目录
git checkout -b "$BRANCH_NAME"
mkdir -p "$FEATURE_DIR"
cp "$TEMPLATE" "$SPEC_FILE"

# 6. 输出 JSON 结果
printf '{"BRANCH_NAME":"%s","SPEC_FILE":"%s","FEATURE_NUM":"%s"}\n' \
    "$BRANCH_NAME" "$SPEC_FILE" "$FEATURE_NUM"
```

**关键技术**:

1. **智能分支命名**:
   ```bash
   # 输入: "I want to add user authentication system"
   # 输出: "001-user-authentication-system"

   # 过滤: "I", "want", "to", "add" (停用词)
   # 保留: "user", "authentication", "system"
   ```

2. **长度限制**:
   ```bash
   # GitHub 强制 244 字节限制
   MAX_BRANCH_LENGTH=244
   if [ ${#BRANCH_NAME} -gt $MAX_BRANCH_LENGTH ]; then
       # 截断后缀，保留完整的编号
   fi
   ```

3. **非 Git 仓库支持**:
   ```bash
   if [ "$HAS_GIT" = true ]; then
       git checkout -b "$BRANCH_NAME"
   else
       echo "[specify] Warning: Git repository not detected"
   fi
   ```

#### 3. 模板文件

**文件**: `templates/spec-template.md`

```markdown
# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]
**Status**: Draft

## User Scenarios & Testing *(mandatory)*

### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value]
**Independent Test**: [How to test independently]

**Acceptance Scenarios**:
1. **Given** [initial state], **When** [action], **Then** [expected outcome]

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST [specific capability]
- **FR-002**: System MUST [specific capability]

### Key Entities *(include if feature involves data)*

- **[Entity 1]**: [What it represents, key attributes]

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: [Measurable metric, e.g., "Users can complete account creation in under 2 minutes"]
```

**占位符替换**:

AI 在填充时将：
- `[FEATURE NAME]` → "User Authentication System"
- `[###-feature-name]` → "001-user-auth"
- `[DATE]` → "2025-10-23"
- `[NEEDS CLARIFICATION: ...]` → 标记需要澄清的点

---

### `/speckit.plan` 命令

**作用**: 基于规范创建技术实现计划

#### 1. 命令定义文件

**文件**: `templates/commands/plan.md`

```yaml
---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---
```

**关键指令**:

```markdown
## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON
2. **Load context**: Read FEATURE_SPEC and `/memory/constitution.md`
3. **Execute plan workflow**:
   - Fill Technical Context
   - Fill Constitution Check section
   - Evaluate gates
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
4. **Stop and report**: Command ends after Phase 1 planning

## Phases

### Phase 0: Outline & Research
1. Extract unknowns from Technical Context
2. Generate and dispatch research agents
3. Consolidate findings in `research.md`

### Phase 1: Design & Contracts
1. Extract entities from feature spec → `data-model.md`
2. Generate API contracts from functional requirements
3. Agent context update: Run `{AGENT_SCRIPT}`
```

**执行流程**:

1. 调用 `setup-plan.sh --json`
2. 脚本返回路径信息
3. AI 读取 `spec.md` 和 `constitution.md`
4. AI 基于技术栈（用户提供）填充 `plan.md`
5. AI 研究不明确的技术决策 → `research.md`
6. AI 提取数据模型 → `data-model.md`
7. AI 生成 API 合约 → `contracts/*.json`
8. AI 调用 `update-agent-context.sh` 更新上下文文件

#### 2. Shell 脚本

**文件**: `scripts/bash/setup-plan.sh`

```bash
#!/usr/bin/env bash
set -e

# 加载公共函数
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "$SCRIPT_DIR/common.sh"

# 获取所有路径
eval $(get_feature_paths)

# 检查是否在功能分支上
check_feature_branch "$CURRENT_BRANCH" "$HAS_GIT" || exit 1

# 确保目录存在
mkdir -p "$FEATURE_DIR"

# 复制计划模板
TEMPLATE="$REPO_ROOT/.specify/templates/plan-template.md"
if [[ -f "$TEMPLATE" ]]; then
    cp "$TEMPLATE" "$IMPL_PLAN"
else
    touch "$IMPL_PLAN"
fi

# 输出 JSON 结果
if $JSON_MODE; then
    printf '{"FEATURE_SPEC":"%s","IMPL_PLAN":"%s","SPECS_DIR":"%s","BRANCH":"%s"}\n' \
        "$FEATURE_SPEC" "$IMPL_PLAN" "$FEATURE_DIR" "$CURRENT_BRANCH"
fi
```

**文件**: `scripts/bash/common.sh`

这是一个公共函数库，提供：

```bash
# 1. 获取仓库根目录（支持非 Git 仓库）
get_repo_root() {
    if git rev-parse --show-toplevel >/dev/null 2>&1; then
        git rev-parse --show-toplevel
    else
        # 回退到脚本位置
        local script_dir="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
        (cd "$script_dir/../../.." && pwd)
    fi
}

# 2. 获取当前分支/功能名称
get_current_branch() {
    # 优先使用 SPECIFY_FEATURE 环境变量
    if [[ -n "${SPECIFY_FEATURE:-}" ]]; then
        echo "$SPECIFY_FEATURE"
        return
    fi

    # 其次使用 git
    if git rev-parse --abbrev-ref HEAD >/dev/null 2>&1; then
        git rev-parse --abbrev-ref HEAD
        return
    fi

    # 最后查找最新的功能目录
    # ...
}

# 3. 检查是否有 Git
has_git() {
    git rev-parse --show-toplevel >/dev/null 2>&1
}

# 4. 验证功能分支名称
check_feature_branch() {
    local branch="$1"
    local has_git_repo="$2"

    if [[ "$has_git_repo" != "true" ]]; then
        echo "[specify] Warning: Git repository not detected" >&2
        return 0
    fi

    if [[ ! "$branch" =~ ^[0-9]{3}- ]]; then
        echo "ERROR: Not on a feature branch. Current branch: $branch" >&2
        return 1
    fi
}

# 5. 根据数字前缀查找功能目录
find_feature_dir_by_prefix() {
    # 支持多个分支使用同一个 spec（如 004-fix-bug, 004-add-feature）
    local prefix="${BASH_REMATCH[1]}"  # 提取 "004"
    # 在 specs/ 中查找以此前缀开头的目录
}

# 6. 获取所有功能相关路径
get_feature_paths() {
    local repo_root=$(get_repo_root)
    local current_branch=$(get_current_branch)
    local feature_dir=$(find_feature_dir_by_prefix "$repo_root" "$current_branch")

    cat <<EOF
REPO_ROOT='$repo_root'
CURRENT_BRANCH='$current_branch'
HAS_GIT='$has_git_repo'
FEATURE_DIR='$feature_dir'
FEATURE_SPEC='$feature_dir/spec.md'
IMPL_PLAN='$feature_dir/plan.md'
TASKS='$feature_dir/tasks.md'
RESEARCH='$feature_dir/research.md'
DATA_MODEL='$feature_dir/data-model.md'
QUICKSTART='$feature_dir/quickstart.md'
CONTRACTS_DIR='$feature_dir/contracts'
EOF
}
```

**使用方式**:

```bash
# 在其他脚本中
source "$SCRIPT_DIR/common.sh"
eval $(get_feature_paths)

# 现在可以使用这些变量
echo "Feature spec: $FEATURE_SPEC"
echo "Implementation plan: $IMPL_PLAN"
```

#### 3. Agent 上下文更新

**文件**: `scripts/bash/update-agent-context.sh`

**作用**: 自动更新 AI agent 的上下文文件（如 `CLAUDE.md`, `GEMINI.md` 等）

```bash
# 支持的 Agent 类型
claude     → CLAUDE.md
gemini     → GEMINI.md
copilot    → .github/copilot-instructions.md
cursor     → .cursor/rules/specify-rules.mdc
qwen       → QWEN.md
windsurf   → .windsurf/rules/specify-rules.md
# ... 等

# 主要功能
update_agent_file() {
    # 1. 从 plan.md 提取技术信息
    NEW_LANG=$(extract_plan_field "Language/Version" "$plan_file")
    NEW_FRAMEWORK=$(extract_plan_field "Primary Dependencies" "$plan_file")
    NEW_DB=$(extract_plan_field "Storage" "$plan_file")

    # 2. 如果文件不存在，从模板创建
    if [[ ! -f "$target_file" ]]; then
        create_new_agent_file()  # 复制模板，替换占位符
    else
        # 3. 更新现有文件
        update_existing_agent_file()  # 添加新技术到 Active Technologies
    fi
}

# 更新逻辑
update_existing_agent_file() {
    # 在 "## Active Technologies" 部分添加新条目
    # - Python 3.11 + FastAPI (001-user-auth)
    # - PostgreSQL (001-user-auth)

    # 在 "## Recent Changes" 部分添加新条目（保留最近 3 个）
    # - 001-user-auth: Added Python 3.11 + FastAPI

    # 更新时间戳
    # **Last updated**: 2025-10-23
}
```

**模板文件**: `templates/agent-file-template.md`

```markdown
# [PROJECT NAME] Development Guidelines

Auto-generated from all feature plans. Last updated: [DATE]

## Active Technologies

[EXTRACTED FROM ALL PLAN.MD FILES]

## Project Structure

```text
[ACTUAL STRUCTURE FROM PLANS]
```

## Commands

[ONLY COMMANDS FOR ACTIVE TECHNOLOGIES]

## Code Style

[LANGUAGE-SPECIFIC, ONLY FOR LANGUAGES IN USE]

## Recent Changes

[LAST 3 FEATURES AND WHAT THEY ADDED]

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
```

**占位符替换示例**:

```markdown
# MyApp Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-23

## Active Technologies

- Python 3.11 + FastAPI (001-user-auth)
- PostgreSQL (001-user-auth)
- React 18 (002-dashboard)

## Project Structure

```text
backend/
frontend/
tests/
```

## Commands

cd backend && pytest && ruff check .
cd frontend && npm test && npm run lint

## Recent Changes

- 002-dashboard: Added React 18
- 001-user-auth: Added Python 3.11 + FastAPI
```

#### 4. 模板文件

**文件**: `templates/plan-template.md`

```markdown
# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE]

## Summary

[Extract from feature spec: primary requirement + technical approach]

## Technical Context

**Language/Version**: [e.g., Python 3.11 or NEEDS CLARIFICATION]
**Primary Dependencies**: [e.g., FastAPI or NEEDS CLARIFICATION]
**Storage**: [if applicable, e.g., PostgreSQL or N/A]
**Testing**: [e.g., pytest or NEEDS CLARIFICATION]
**Target Platform**: [e.g., Linux server or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research.*

[Gates determined based on constitution file]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```
```

---

## 文件类型说明

### 1. 命令定义文件 (`templates/commands/*.md`)

| 文件 | 命令 | 调用脚本 | 生成文件 |
|------|------|---------|---------|
| `specify.md` | `/speckit.specify` | `create-new-feature.sh` | `spec.md`, 检查清单 |
| `plan.md` | `/speckit.plan` | `setup-plan.sh`, `update-agent-context.sh` | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` |
| `tasks.md` | `/speckit.tasks` | `check-prerequisites.sh` | `tasks.md` |
| `implement.md` | `/speckit.implement` | `check-prerequisites.sh` | 实际代码文件 |
| `clarify.md` | `/speckit.clarify` | - | 更新 `spec.md` 的 Clarifications 部分 |
| `analyze.md` | `/speckit.analyze` | - | 分析报告 |
| `checklist.md` | `/speckit.checklist` | - | 质量检查清单 |
| `constitution.md` | `/speckit.constitution` | - | `memory/constitution.md` |

### 2. Shell 脚本 (`scripts/bash/*.sh`)

| 脚本 | 作用 | 输入 | 输出 |
|------|------|------|------|
| `create-new-feature.sh` | 创建新功能分支和规范文件 | `--short-name`, 功能描述 | JSON: BRANCH_NAME, SPEC_FILE, FEATURE_NUM |
| `setup-plan.sh` | 初始化实现计划 | `--json` | JSON: FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH |
| `common.sh` | 公共函数库 | - | 函数: get_repo_root, get_current_branch, get_feature_paths 等 |
| `update-agent-context.sh` | 更新 AI agent 上下文 | agent_type (claude/gemini/...) | 更新 CLAUDE.md 等文件 |
| `check-prerequisites.sh` | 检查前置条件 | `--require-tasks`, `--include-tasks` | JSON: FEATURE_DIR, AVAILABLE_DOCS |

### 3. 模板文件 (`templates/*.md`)

| 模板 | 用途 | 占位符示例 |
|------|------|-----------|
| `spec-template.md` | 功能规范结构 | `[FEATURE NAME]`, `[###-feature-name]`, `[DATE]` |
| `plan-template.md` | 实现计划结构 | `[FEATURE]`, `[DATE]`, `[NEEDS CLARIFICATION]` |
| `tasks-template.md` | 任务列表结构 | `[FEATURE NAME]`, `[ID]`, `[Story]` |
| `agent-file-template.md` | Agent 上下文文件 | `[PROJECT NAME]`, `[DATE]`, `[EXTRACTED FROM ALL PLAN.MD FILES]` |
| `checklist-template.md` | 检查清单结构 | `[FEATURE NAME]`, `[DATE]` |

### 4. 生成的文档 (`specs/###-feature-name/`)

| 文件 | 生成命令 | 作用 |
|------|---------|------|
| `spec.md` | `/speckit.specify` | 功能规范文档 |
| `plan.md` | `/speckit.plan` | 技术实现计划 |
| `research.md` | `/speckit.plan` (Phase 0) | 技术研究和决策 |
| `data-model.md` | `/speckit.plan` (Phase 1) | 数据模型定义 |
| `quickstart.md` | `/speckit.plan` (Phase 1) | 快速开始指南 |
| `contracts/*.json` | `/speckit.plan` (Phase 1) | API 合约（OpenAPI 等） |
| `tasks.md` | `/speckit.tasks` | 任务分解列表 |
| `checklists/*.md` | `/speckit.checklist` | 质量检查清单 |

---

## 工作流程图

### 完整开发流程

```text
┌─────────────────────────────────────────────────────────────────┐
│                    1. /speckit.constitution                      │
│                                                                  │
│  创建项目原则和开发指南                                           │
│  ├─ 输入: 项目约束、编码标准、质量要求                           │
│  └─ 输出: memory/constitution.md                                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     2. /speckit.specify                          │
│                                                                  │
│  从自然语言创建功能规范                                          │
│  ├─ 输入: 功能描述（用户需求）                                   │
│  ├─ 调用: create-new-feature.sh --short-name "..." "描述"       │
│  ├─ 生成: specs/001-feature-name/spec.md                        │
│  ├─ 生成: specs/001-feature-name/checklists/requirements.md     │
│  └─ 创建: Git 分支 001-feature-name                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    3. /speckit.clarify (可选)                    │
│                                                                  │
│  澄清规范中的不明确点                                            │
│  ├─ 查找: [NEEDS CLARIFICATION: ...] 标记                       │
│  ├─ 询问: 结构化问题（最多 3 个）                               │
│  └─ 更新: spec.md 中的 Clarifications 部分                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      4. /speckit.plan                            │
│                                                                  │
│  创建技术实现计划                                                │
│  ├─ 输入: 技术栈选择（Python, React, PostgreSQL 等）            │
│  ├─ 调用: setup-plan.sh --json                                  │
│  ├─ 读取: spec.md, constitution.md                              │
│  ├─ Phase 0: 技术研究                                           │
│  │   └─ 生成: research.md                                       │
│  ├─ Phase 1: 设计与合约                                         │
│  │   ├─ 生成: data-model.md                                     │
│  │   ├─ 生成: contracts/*.json (OpenAPI)                        │
│  │   ├─ 生成: quickstart.md                                     │
│  │   └─ 调用: update-agent-context.sh claude                    │
│  │       └─ 更新: CLAUDE.md (或其他 agent 文件)                 │
│  └─ 输出: plan.md                                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   5. /speckit.analyze (可选)                     │
│                                                                  │
│  交叉检查一致性和覆盖度                                          │
│  ├─ 验证: spec.md ↔ plan.md ↔ tasks.md 一致性                  │
│  └─ 报告: 缺失的需求、不一致的设计                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     6. /speckit.tasks                            │
│                                                                  │
│  生成可执行任务列表                                              │
│  ├─ 调用: check-prerequisites.sh --json                         │
│  ├─ 读取: spec.md (用户故事), plan.md, data-model.md, contracts/│
│  ├─ 按用户故事组织任务:                                         │
│  │   ├─ Phase 1: Setup (共享基础设施)                          │
│  │   ├─ Phase 2: Foundational (阻塞性前置任务)                 │
│  │   ├─ Phase 3+: User Story 1, 2, 3... (按优先级)             │
│  │   └─ Final Phase: Polish & Cross-Cutting Concerns            │
│  ├─ 标记并行任务: [P]                                           │
│  ├─ 标记故事归属: [US1], [US2]...                              │
│  └─ 输出: tasks.md                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                  7. /speckit.checklist (可选)                    │
│                                                                  │
│  生成自定义质量检查清单                                          │
│  └─ 输出: checklists/*.md (如 ux.md, security.md, test.md)      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   8. /speckit.implement                          │
│                                                                  │
│  执行实现                                                        │
│  ├─ 调用: check-prerequisites.sh --json --require-tasks         │
│  ├─ 检查: checklists/ 完成状态                                  │
│  ├─ 读取: tasks.md, plan.md, data-model.md, contracts/          │
│  ├─ 创建: .gitignore, .dockerignore 等（根据技术栈）            │
│  ├─ 按阶段执行:                                                 │
│  │   ├─ Phase 1: Setup                                          │
│  │   ├─ Phase 2: Foundational (必须完成才能继续)               │
│  │   ├─ Phase 3+: 用户故事（按优先级或并行）                   │
│  │   └─ Final Phase: Polish                                     │
│  ├─ 遵循 TDD: 测试 → 模型 → 服务 → 端点                        │
│  ├─ 更新: tasks.md (标记 [X] 已完成任务)                        │
│  └─ 输出: 实际代码文件                                          │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流图

```text
┌───────────────┐
│  用户输入     │
│  (自然语言)   │
└───────┬───────┘
        │
        ↓
┌─────────────────────────────────────────┐
│  命令定义文件 (templates/commands/*.md)  │
│  ┌─────────────────────────────────────┐│
│  │ YAML Front Matter                   ││
│  │ - description                       ││
│  │ - scripts: sh/ps                    ││
│  │ - agent_scripts (可选)              ││
│  └─────────────────────────────────────┘│
│  ┌─────────────────────────────────────┐│
│  │ Markdown Body (AI 指令)             ││
│  │ - 解析逻辑                          ││
│  │ - 调用流程                          ││
│  │ - 验证规则                          ││
│  └─────────────────────────────────────┘│
└─────────────┬───────────────────────────┘
              │
              ↓
┌──────────────────────────────────────────┐
│  Shell 脚本 (scripts/bash/*.sh)          │
│  ┌────────────────────────────────────┐ │
│  │ create-new-feature.sh              │ │
│  │ - 生成分支名称                     │ │
│  │ - 创建目录结构                     │ │
│  │ - 输出 JSON 路径                   │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ setup-plan.sh                      │ │
│  │ - 复制 plan 模板                   │ │
│  │ - 输出 JSON 路径                   │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ common.sh                          │ │
│  │ - get_repo_root()                  │ │
│  │ - get_current_branch()             │ │
│  │ - get_feature_paths()              │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ update-agent-context.sh            │ │
│  │ - 解析 plan.md                     │ │
│  │ - 更新 Agent 上下文文件            │ │
│  └────────────────────────────────────┘ │
└──────────────┬───────────────────────────┘
               │
               ↓ (JSON 输出)
┌──────────────────────────────────────────┐
│  AI 处理                                 │
│  1. 解析 JSON 获取路径                   │
│  2. 读取模板文件                         │
│  3. 读取上下文（spec, constitution）     │
│  4. 生成内容（填充模板）                 │
│  5. 写入目标文件                         │
│  6. 调用后续脚本（如 agent update）      │
└──────────────┬───────────────────────────┘
               │
               ↓
┌──────────────────────────────────────────┐
│  模板文件 (templates/*.md)               │
│  ┌────────────────────────────────────┐ │
│  │ spec-template.md                   │ │
│  │ - 占位符: [FEATURE NAME]           │ │
│  │ - 章节结构                         │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ plan-template.md                   │ │
│  │ - 占位符: [NEEDS CLARIFICATION]    │ │
│  │ - 技术上下文                       │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ tasks-template.md                  │ │
│  │ - 任务格式: [ID] [P?] [Story]      │ │
│  │ - 阶段结构                         │ │
│  └────────────────────────────────────┘ │
└──────────────┬───────────────────────────┘
               │ (AI 填充)
               ↓
┌──────────────────────────────────────────┐
│  生成的文档 (specs/###-feature-name/)    │
│  ├─ spec.md                              │
│  ├─ plan.md                              │
│  ├─ research.md                          │
│  ├─ data-model.md                        │
│  ├─ quickstart.md                        │
│  ├─ contracts/*.json                     │
│  ├─ tasks.md                             │
│  └─ checklists/*.md                      │
└──────────────┬───────────────────────────┘
               │
               ↓
┌──────────────────────────────────────────┐
│  Agent 上下文文件 (根目录)               │
│  ├─ CLAUDE.md                            │
│  ├─ GEMINI.md                            │
│  ├─ .github/copilot-instructions.md      │
│  ├─ .cursor/rules/specify-rules.mdc      │
│  └─ ...                                  │
│                                          │
│  内容:                                   │
│  - Active Technologies                   │
│  - Project Structure                     │
│  - Recent Changes                        │
└──────────────────────────────────────────┘
```

---

## 关键技术点

### 1. JSON 通信协议

**为什么使用 JSON?**

Claude Code 可以轻松解析 JSON 输出，获取路径和配置信息。

**示例**:

```bash
# Shell 脚本输出
printf '{"BRANCH_NAME":"%s","SPEC_FILE":"%s"}\n' "$BRANCH_NAME" "$SPEC_FILE"

# AI 在命令定义中的指令
# 2. Run the script and parse its JSON output for BRANCH_NAME and SPEC_FILE
```

**优势**:
- 结构化数据，易于解析
- 避免字符串拼接错误
- 支持复杂数据结构

### 2. 占位符系统

**模板中的占位符**:

```markdown
# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]
**Status**: Draft
```

**AI 替换逻辑**:

```text
[FEATURE NAME] → 从用户输入提取的标题
[###-feature-name] → 从 JSON 获取的分支名称
[DATE] → 当前日期
[NEEDS CLARIFICATION: ...] → 需要用户澄清的具体问题
```

**高级占位符**（用于 agent 模板）:

```markdown
[EXTRACTED FROM ALL PLAN.MD FILES]
→ AI 遍历所有 plan.md，提取技术栈，生成列表

[LAST 3 FEATURES AND WHAT THEY ADDED]
→ AI 查找最近的 3 个功能分支，提取变更摘要
```

### 3. 分支命名策略

**格式**: `###-descriptive-name`

**生成逻辑**:

```bash
# 1. 计算编号
NEXT=$((HIGHEST + 1))
FEATURE_NUM=$(printf "%03d" "$NEXT")  # 001, 002, 003...

# 2. 生成描述性名称
generate_branch_name() {
    # a. 转小写
    # b. 过滤停用词 (the, a, is, to, for...)
    # c. 保留有意义的词（长度 >= 3）
    # d. 保留大写缩写（API, JWT, OAuth）
    # e. 取前 3-4 个词
    # f. 用连字符连接
}

# 3. 组合
BRANCH_NAME="${FEATURE_NUM}-${BRANCH_SUFFIX}"
# 结果: 001-user-authentication-system
```

**长度限制**:

```bash
# GitHub 限制: 244 字节
if [ ${#BRANCH_NAME} -gt 244 ]; then
    # 截断，保留编号
fi
```

### 4. 非 Git 仓库支持

**挑战**: 有些项目可能不使用 Git

**解决方案**:

```bash
# 1. 检测 Git
has_git() {
    git rev-parse --show-toplevel >/dev/null 2>&1
}

# 2. 分支名称回退
get_current_branch() {
    # 优先级 1: SPECIFY_FEATURE 环境变量
    if [[ -n "${SPECIFY_FEATURE:-}" ]]; then
        echo "$SPECIFY_FEATURE"
        return
    fi

    # 优先级 2: Git 分支
    if git rev-parse --abbrev-ref HEAD >/dev/null 2>&1; then
        git rev-parse --abbrev-ref HEAD
        return
    fi

    # 优先级 3: 查找最新功能目录
    # 扫描 specs/ 目录，找到最大编号
}

# 3. 跳过 Git 操作
if [ "$HAS_GIT" = true ]; then
    git checkout -b "$BRANCH_NAME"
else
    echo "[specify] Warning: Git repository not detected"
fi
```

**使用环境变量**:

```bash
# 用户手动设置
export SPECIFY_FEATURE="001-my-feature"

# 脚本自动设置（在 create-new-feature.sh 中）
export SPECIFY_FEATURE="$BRANCH_NAME"
```

### 5. 模板继承和覆盖

**Agent 模板的手动添加保护**:

```markdown
<!-- MANUAL ADDITIONS START -->
用户手动添加的内容会被保留
<!-- MANUAL ADDITIONS END -->
```

**更新逻辑**:

```bash
update_existing_agent_file() {
    # 1. 扫描 "## Active Technologies" 部分
    # 2. 添加新技术（如果不存在）
    # 3. 不删除现有技术
    # 4. 保留 <!-- MANUAL ADDITIONS --> 之间的内容
}
```

### 6. 阶段性执行（Phase-based Execution）

**为什么分阶段?**

- 允许用户在每个阶段检查和调整
- 避免一次性生成大量文件
- 支持迭代式开发

**示例**（`/speckit.plan`）:

```text
Phase 0: Research
  - 解决 NEEDS CLARIFICATION
  - 技术决策研究
  - 输出: research.md

Phase 1: Design & Contracts
  - 数据模型设计
  - API 合约生成
  - 快速开始指南
  - 输出: data-model.md, contracts/, quickstart.md

停止并报告
(AI 不会自动执行 Phase 2，需要用户调用 /speckit.tasks)
```

### 7. 任务依赖和并行化

**任务格式**:

```text
- [ ] T001 创建项目结构
- [ ] T002 [P] 配置 linting 工具
- [ ] T003 [P] [US1] 创建 User 模型
```

**标记含义**:

- `T###`: 任务 ID（执行顺序）
- `[P]`: 可并行执行（不同文件，无依赖）
- `[US#]`: 归属的用户故事

**执行规则**:

```text
Phase 2: Foundational (必须完成)
  - [ ] T004 设置数据库架构
  - [ ] T005 [P] 实现认证框架
  ↓ 必须全部完成才能继续

Phase 3: User Story 1
  - [ ] T010 [P] [US1] 合约测试
  - [ ] T011 [P] [US1] 集成测试
  ↓ 可以并行
  - [ ] T012 [P] [US1] User 模型
  - [ ] T013 [P] [US1] Profile 模型
  ↓ 可以并行
  - [ ] T014 [US1] UserService (依赖 T012, T013)
```

### 8. Constitution Check (项目约束检查)

**作用**: 确保实现符合项目原则

**流程**:

```text
1. /speckit.constitution
   创建: memory/constitution.md
   内容: 项目原则、复杂度限制、编码标准

2. /speckit.plan
   读取: constitution.md
   检查: 是否违反原则

   示例检查:
   - 最多 3 个子项目 (如果有 4 个，需要说明)
   - 避免过度工程 (Repository 模式是否必要?)
   - 性能要求 (是否满足 <200ms p95?)

3. Complexity Tracking 表格
   | Violation | Why Needed | Simpler Alternative Rejected |
   |-----------|-----------|------------------------------|
   | 4th project | 需要独立的管理面板 | 单体应用无法满足权限隔离 |
```

### 9. 质量检查清单（Checklists）

**生成时机**:

```text
/speckit.specify → checklists/requirements.md
/speckit.checklist → checklists/ux.md, security.md, test.md...
```

**检查逻辑**（在 `/speckit.implement` 中）:

```bash
# 1. 扫描 checklists/ 目录
for file in checklists/*.md; do
    # 2. 统计
    total=$(grep -c '^- \[' "$file")
    completed=$(grep -c '^- \[[Xx]\]' "$file")
    incomplete=$(grep -c '^- \[ \]' "$file")

    # 3. 判断状态
    if [ $incomplete -eq 0 ]; then
        status="✓ PASS"
    else
        status="✗ FAIL"
    fi
done

# 4. 如果有未完成的检查项
if [ $incomplete -gt 0 ]; then
    echo "Some checklists are incomplete. Proceed anyway? (yes/no)"
    # 等待用户确认
fi
```

### 10. 多 Agent 支持

**支持的 AI Agent**:

| Agent | 上下文文件 |
|-------|-----------|
| Claude Code | `CLAUDE.md` |
| Gemini CLI | `GEMINI.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursor/rules/specify-rules.mdc` |
| Qwen Code | `QWEN.md` |
| Windsurf | `.windsurf/rules/specify-rules.md` |
| Kilo Code | `.kilocode/rules/specify-rules.md` |
| Auggie CLI | `.augment/rules/specify-rules.md` |
| Roo Code | `.roo/rules/specify-rules.md` |
| CodeBuddy CLI | `CODEBUDDY.md` |
| Amp | `AGENTS.md` |
| Amazon Q Developer CLI | `AGENTS.md` |

**更新策略**:

```bash
# 更新所有存在的 agent 文件
update_all_existing_agents() {
    if [[ -f "$CLAUDE_FILE" ]]; then
        update_agent_file "$CLAUDE_FILE" "Claude Code"
    fi
    if [[ -f "$GEMINI_FILE" ]]; then
        update_agent_file "$GEMINI_FILE" "Gemini CLI"
    fi
    # ...

    # 如果没有任何 agent 文件，创建默认的 Claude 文件
    if [[ "$found_agent" == false ]]; then
        update_agent_file "$CLAUDE_FILE" "Claude Code"
    fi
}

# 更新特定 agent
./update-agent-context.sh claude
./update-agent-context.sh gemini
```

---

## 总结

### 核心设计原则

1. **分离关注点**:
   - 命令定义（`.md`）→ AI 理解的指令
   - Shell 脚本（`.sh`）→ 系统操作
   - 模板文件（`.md`）→ 文档结构

2. **JSON 通信**:
   - Shell 脚本输出 JSON
   - AI 解析 JSON 获取路径
   - 避免字符串解析错误

3. **占位符系统**:
   - 模板中使用 `[PLACEHOLDER]`
   - AI 根据上下文替换
   - 支持动态内容生成

4. **阶段性执行**:
   - 每个命令完成特定阶段
   - 用户可以在阶段间检查和调整
   - 支持迭代式开发

5. **灵活性**:
   - 支持 Git 和非 Git 仓库
   - 支持多种 AI Agent
   - 支持多种项目类型（单体、Web、移动）

### 实现自定义命令的步骤

如果你想为 Claude Code 创建自己的自定义命令：

1. **创建命令定义文件**:
   ```markdown
   ---
   description: "Your command description"
   scripts:
     sh: "scripts/bash/your-script.sh --json"
   ---

   ## Outline

   1. Run `{SCRIPT}` and parse JSON output
   2. Load template from `templates/your-template.md`
   3. Fill in the template based on user input
   4. Write to output file
   ```

2. **编写 Shell 脚本**:
   ```bash
   #!/usr/bin/env bash
   set -e

   # 解析参数
   # 执行操作
   # 输出 JSON

   if $JSON_MODE; then
       printf '{"OUTPUT_FILE":"%s"}\n' "$OUTPUT_FILE"
   fi
   ```

3. **创建模板文件**:
   ```markdown
   # [TITLE]

   **Created**: [DATE]

   ## Section 1

   [CONTENT]
   ```

4. **测试命令**:
   ```bash
   # 在 Claude Code 中
   /your-command Your input here
   ```

### 最佳实践

1. **清晰的命令定义**: 在 `.md` 文件中提供详细的步骤和验证规则
2. **健壮的脚本**: 处理边缘情况，提供清晰的错误消息
3. **结构化输出**: 使用 JSON 传递数据给 AI
4. **模块化设计**: 使用 `common.sh` 共享函数
5. **版本控制**: 在模板中包含版本和日期信息
6. **质量检查**: 在命令定义中包含验证步骤
7. **用户友好**: 提供清晰的输出和进度信息

---

## 参考资源

- **项目文档**: `/home/user/spec-kit/README.md`
- **命令定义**: `/home/user/spec-kit/templates/commands/`
- **Shell 脚本**: `/home/user/spec-kit/scripts/bash/`
- **模板文件**: `/home/user/spec-kit/templates/`
- **Claude Code 文档**: https://docs.claude.com/en/docs/claude-code/

---

**分析完成日期**: 2025-10-23
**分析者**: Claude (Sonnet 4.5)
