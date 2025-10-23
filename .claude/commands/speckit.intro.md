---
description: Generate a comprehensive project overview by analyzing specifications, actual implementation, codebase architecture, and progress metrics.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Generate a comprehensive, executive-level overview of the entire project by analyzing:

**Specification Layer:**
- Project constitution and principles
- All feature specifications (.specify/specs/)
- All implementation plans
- All task lists and their completion status

**Implementation Layer:**
- Actual codebase structure and architecture
- Implemented files and directories
- Code statistics (file counts, lines of code)
- Technology detection from actual files (package.json, requirements.txt, etc.)
- Configuration files and build scripts

**Alignment Analysis:**
- Spec-to-implementation mapping
- Planned vs. actual technology stack
- Task completion vs. actual code delivery
- Missing implementations
- Over-implemented features (code without specs)

This command provides a "birds-eye view" of the project, helping stakeholders understand:
- What the project is building (from specs)
- What has been built (from actual code)
- How far along each feature is (task completion + code presence)
- What technologies are actually being used (detected from files)
- Gaps between planning and reality

## Operating Constraints

**STRICTLY READ-ONLY for analysis**: Do not modify project files. Only create the output report file.

## Output Behavior

**Default behavior**: The report is **automatically saved to a file** in `.specify/intro/` directory.

**File naming convention**:
```
.specify/intro/{YYYY-MM-DD}_{HH-MM-SS}_{BRANCH_NAME}.md
```

**Examples**:
- `2025-10-23_14-30-45_main.md`
- `2025-10-23_09-15-20_001-user-auth.md`
- `2025-10-23_16-42-10_feature-dashboard.md`

**File naming logic**:
1. Get current date and time
2. Get current Git branch name (or "main" if not in Git repo)
3. Sanitize branch name (replace `/` with `-`, remove special characters)
4. Combine: `{date}_{time}_{branch}.md`

**Directory structure**:
```
.specify/
├── intro/                    ← Report output directory
│   ├── 2025-10-23_14-30-45_main.md
│   ├── 2025-10-23_09-15-20_001-user-auth.md
│   └── 2025-10-24_10-20-30_002-dashboard.md
├── specs/                    ← Feature specifications
├── scripts/                  ← Shell scripts
└── templates/                ← Templates
```

## Execution Steps

### 1. Initialize Analysis Context and Determine Output Path

**Step 1a: Get repository root and current branch**

```bash
source .specify/scripts/bash/common.sh
REPO_ROOT=$(get_repo_root)
SPECS_DIR="$REPO_ROOT/.specify/specs"
CURRENT_BRANCH=$(get_current_branch)  # Uses common.sh function
```

**Step 1b: Generate output file path**

1. Get current date and time:
   ```bash
   DATE=$(date +%Y-%m-%d)
   TIME=$(date +%H-%M-%S)
   ```

2. Sanitize branch name:
   - Replace `/` with `-`
   - Remove or replace special characters (`*`, `?`, `:`, etc.)
   - Example: `feature/user-auth` → `feature-user-auth`

3. Construct file path:
   ```bash
   INTRO_DIR="$REPO_ROOT/.specify/intro"
   OUTPUT_FILE="$INTRO_DIR/${DATE}_${TIME}_${CURRENT_BRANCH}.md"
   ```

4. Create intro directory if it doesn't exist:
   ```bash
   mkdir -p "$INTRO_DIR"
   ```

**Step 1c: Store output path for later use**

Remember this path to write the report at the end of the analysis.

For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

### 2. Load Project Foundation

**Constitution**: Read `.specify/memory/constitution.md` (if it exists)
- Extract project principles
- Extract quality requirements
- Extract key constraints

**Agent Context**: Read any existing agent context files (`CLAUDE.md`, `GEMINI.md`, etc.) for technology summary

### 3. Analyze Project Architecture

**Scan the entire project directory structure** (excluding common ignore patterns):

```bash
# Use find or ls to discover project structure
find "$REPO_ROOT" -type f -not -path "*/.git/*" -not -path "*/node_modules/*" \
  -not -path "*/.specify/*" -not -path "*/__pycache__/*" -not -path "*/venv/*" \
  -not -path "*/.venv/*" -not -path "*/dist/*" -not -path "*/build/*"
```

**Detect project type and technology stack** from actual files:

| File Pattern | Technology Detected |
|--------------|-------------------|
| `package.json` | Node.js/npm project |
| `requirements.txt`, `pyproject.toml`, `setup.py` | Python project |
| `Gemfile` | Ruby project |
| `pom.xml`, `build.gradle` | Java project |
| `*.csproj`, `*.sln` | .NET/C# project |
| `go.mod` | Go project |
| `Cargo.toml` | Rust project |
| `composer.json` | PHP project |
| `tsconfig.json` | TypeScript |
| `.eslintrc.*`, `.prettierrc` | JavaScript/TypeScript tooling |
| `Dockerfile`, `docker-compose.yml` | Docker containerization |
| `.github/workflows/` | GitHub Actions CI/CD |

**Read configuration files** to extract actual dependencies:
- `package.json` → dependencies, devDependencies, scripts
- `requirements.txt` → Python packages
- `pyproject.toml` → Python project metadata
- `Dockerfile` → Base images, runtime commands

**Analyze directory structure** to identify common patterns:

```text
Common patterns to detect:
- Backend: /backend, /server, /api, /src/server
- Frontend: /frontend, /client, /web, /src/client, /ui
- Tests: /tests, /test, /__tests__, /spec
- Docs: /docs, /documentation
- Config: /config, /conf
- Scripts: /scripts, /tools
- Database: /migrations, /db, /prisma, /alembic
- Infrastructure: /infra, /terraform, /k8s, /kubernetes
```

**Calculate project statistics:**
- Total files (by type: .js, .ts, .py, .cs, etc.)
- Total directories
- Lines of code (use `wc -l` or similar)
- Configuration files count
- Test files count

### 4. Discover All Features (Specification Layer)

Scan the `.specify/specs/` directory to find all feature directories matching the pattern `###-*`.

For each feature directory, attempt to read:
- `spec.md` - Feature specification
- `plan.md` - Implementation plan
- `tasks.md` - Task breakdown
- `research.md` - Research notes (optional)
- `data-model.md` - Data models (optional)
- `contracts/` - API contracts (optional)

### 5. Analyze Each Feature (Specification Layer)

For each discovered feature, extract:

**From spec.md:**
- Feature name and branch number
- Status (Draft, In Progress, Complete)
- User stories count
- Functional requirements count
- Success criteria

**From plan.md:**
- Technology stack (Language/Version, Framework, Database, etc.)
- Phase completion status
- Architecture approach

**From tasks.md:**
- Total task count
- Completed tasks (marked with `[X]` or `[x]`)
- In-progress tasks
- Pending tasks
- Parallel tasks count

**Status determination logic:**
- Draft: Only spec.md exists
- Planning: spec.md + plan.md exist, but no tasks.md
- Ready for Implementation: All docs exist, tasks.md has 0% completion
- In Progress: tasks.md exists with 1-99% completion
- Complete: tasks.md exists with 100% completion

### 6. Perform Alignment Analysis

**Compare specifications with actual implementation:**

For each feature, determine if it has been implemented by:

1. **Searching for feature-related files** in the codebase:
   - Extract file paths mentioned in `tasks.md`
   - Check if those files exist in the project
   - Example: If tasks.md mentions "src/auth/login.py", verify it exists

2. **Code presence indicators:**
   - ✅ Fully Implemented: All mentioned files exist + tasks 100% complete
   - 🟡 Partially Implemented: Some files exist + tasks 1-99% complete
   - ⚠️ Code without Spec: Files exist but no corresponding tasks
   - ❌ Missing Implementation: Tasks marked complete but files don't exist
   - ⏳ Not Started: No files exist + tasks 0% complete or no tasks.md

3. **Technology stack alignment:**
   - Compare planned tech stack (from plan.md) with detected tech stack (from actual files)
   - Highlight discrepancies:
     - "Spec says Python 3.11, but found Python 3.9 in runtime"
     - "Spec says PostgreSQL, but found MySQL in docker-compose.yml"
     - "Spec mentions React, but no package.json or .jsx/.tsx files found"

4. **Calculate implementation rate:**
   ```
   Implementation Rate = (Files Found / Files Mentioned in Tasks) × 100%
   ```

### 7. Generate Comprehensive Bilingual Project Overview Report

Output a comprehensive **bilingual (Chinese & English)** Markdown report with the following structure:

**IMPORTANT**: Every section heading, description, and content must be provided in BOTH Chinese and English. Use the format "Chinese / English" for section headings and provide full bilingual content for all text.

```markdown
# 🌱 项目概览与进度报告 / Project Overview & Progress Report

**生成时间 / Generated**: [Current Date]
**代码仓库 / Repository**: [Repo Root Path]
**项目类型 / Project Type**: [Detected from files: Node.js / Python / .NET / etc.]
**分支 / Branch**: [Current Branch Name]

---

## 📋 执行摘要 / Executive Summary

[Provide 1-2 paragraph summary in BOTH Chinese and English, explaining what this project is building, synthesized from constitution and feature specs]

**中文摘要：**
[Chinese summary paragraph]

**English Summary:**
[English summary paragraph]

### 项目原则 / Project Principles

**原则状态 / Principles Status**: [Constitution status]

[List top 3-5 principles from constitution.md in both languages, if available]

**中文原则：**
- [Principle 1 in Chinese]
- [Principle 2 in Chinese]

**English Principles:**
- [Principle 1 in English]
- [Principle 2 in English]

---

## 🏗️ 项目架构 / Project Architecture

### 目录结构 / Directory Structure

```text
[Root Directory]
├── [backend/]           [X files, Y lines of code]
│   ├── [src/]
│   ├── [tests/]
│   └── [config/]
├── [frontend/]          [X files, Y lines of code]
│   ├── [src/]
│   ├── [public/]
│   └── [tests/]
├── [docs/]
├── [.specify/]          [Spec-kit specifications]
└── [README.md]
```

### 技术栈（从项目检测）/ Technology Stack (Detected from Project)

**编程语言 / Languages**:
- [Language] ([X] 个文件 / files, [Y] 行代码 / lines of code)
- [Language] ([X] 个文件 / files, [Y] 行代码 / lines of code)

**框架与库 / Frameworks & Libraries** (from package.json / requirements.txt / etc.):
- [Framework Name] [Version]
- [Library Name] [Version]

**基础设施 / Infrastructure**:
- [Docker] (Dockerfile present)
- [CI/CD] (GitHub Actions workflows found)
- [Database] (detected from config files)

### 项目统计 / Project Statistics

- **总文件数 / Total Files**: [N] ([breakdown by extension])
- **总代码行数 / Total Lines of Code**: [N] (excluding comments/blank lines)
- **配置文件 / Configuration Files**: [N]
- **测试文件 / Test Files**: [N] ([X]% 测试覆盖率 / test coverage ratio)
- **文档文件 / Documentation Files**: [N]

---

## 🏛️ 核心架构（适用于 Spec Kit 项目）/ Core Architecture (For Spec Kit Projects)

**Note**: This section is automatically included when analyzing a Spec Kit-based project.

### 架构概述 / Architecture Overview

**中文描述：**
Spec Kit 采用"规格驱动开发"方法论，通过三层架构实现 AI 辅助开发：命令定义层（AI 理解）、脚本执行层（自动化）、模板生成层（文档标准化）。

**English Description:**
Spec Kit employs a "Specification-Driven Development" methodology with a three-tier architecture for AI-assisted development: Command Definition Layer (AI understanding), Script Execution Layer (automation), and Template Generation Layer (documentation standardization).

### 三层架构 / Three-Tier Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│  AI 编码助手层 / AI Coding Assistant Layer                    │
│  (Claude Code, Cursor, Windsurf, Cline, etc.)              │
└────────────────────┬────────────────────────────────────────┘
                     │ 读取 / Reads
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  命令定义层 / Command Definition Layer                        │
│  (.claude/commands/*.md - YAML front matter + Markdown)    │
│  - /speckit.specify: 创建功能规格 / Create feature spec      │
│  - /speckit.plan: 生成实现计划 / Generate implementation    │
│  - /speckit.tasks: 生成任务列表 / Generate task breakdown   │
│  - /speckit.intro: 生成项目概览 / Generate project overview │
└────────────────────┬────────────────────────────────────────┘
                     │ 调用 / Invokes
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  脚本执行层 / Script Execution Layer                          │
│  (.specify/scripts/bash/*.sh)                              │
│  - 文件系统操作 / File system operations                     │
│  - Git 分支管理 / Git branch management                     │
│  - JSON 输出 / JSON output for AI parsing                  │
└────────────────────┬────────────────────────────────────────┘
                     │ 使用 / Uses
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  模板生成层 / Template Generation Layer                       │
│  (templates/*.md - 占位符模板 / Placeholder templates)       │
│  - spec.md: 功能规格模板 / Feature spec template             │
│  - plan.md: 实现计划模板 / Implementation plan template      │
│  - tasks.md: 任务列表模板 / Task list template              │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件说明 / Core Component Description

#### 1. 命令定义文件 / Command Definition Files

**位置 / Location**: `.claude/commands/speckit.*.md`

**结构 / Structure**:
```markdown
---
description: [Command description]
---

## User Input
$ARGUMENTS

## Goal
[What this command accomplishes]

## Execution Steps
[Detailed AI instructions]
```

**职责 / Responsibilities**:
- 为 AI 提供结构化指令 / Provide structured instructions for AI
- 定义工作流程步骤 / Define workflow steps
- 指定输出格式 / Specify output format
- 处理边缘情况 / Handle edge cases

#### 2. Shell 脚本 / Shell Scripts

**位置 / Location**: `.specify/scripts/bash/common.sh`

**关键函数 / Key Functions**:
- `get_repo_root()`: 获取仓库根路径 / Get repository root path
- `get_current_branch()`: 获取当前分支名 / Get current branch name
- `sanitize_branch_name()`: 清理分支名特殊字符 / Sanitize branch name
- `find_feature_by_branch()`: 按分支查找功能 / Find feature by branch

**JSON 通信协议 / JSON Communication Protocol**:
```bash
# Shell 脚本输出 JSON，AI 解析
# Shell scripts output JSON, AI parses
echo "{"
echo "  \"status\": \"success\","
echo "  \"featureName\": \"$FEATURE_NAME\","
echo "  \"branch\": \"$BRANCH_NAME\""
echo "}"
```

#### 3. 模板系统 / Template System

**占位符格式 / Placeholder Format**:
- `[功能名称]` / `[FEATURE_NAME]`
- `[当前日期]` / `[CURRENT_DATE]`
- `[分支名称]` / `[BRANCH_NAME]`
- `[用户输入]` / `[USER_INPUT]`

**模板类型 / Template Types**:
- **spec.md**: 功能规格 (用户故事 + 需求 / User stories + Requirements)
- **plan.md**: 实现计划 (技术栈 + 架构 / Tech stack + Architecture)
- **tasks.md**: 任务分解 (可执行步骤 / Executable steps)
- **constitution.md**: 项目治理原则 / Project governance principles

---

## 🔄 工作流程图 / Workflow Diagrams

### 完整开发流程 / Complete Development Flow

```text
[用户请求 / User Request]
        │
        ▼
┌───────────────────┐
│ /speckit.specify  │  创建功能规格 / Create feature spec
│ 生成 spec.md      │  → 生成分支 001-feature-name
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ /speckit.clarify  │  澄清需求 (可选 / Optional)
│ 更新 spec.md      │  → 通过问答完善规格
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ /speckit.plan     │  生成实现计划 / Generate implementation plan
│ 生成 plan.md      │  → 技术栈 + 架构设计
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ /speckit.tasks    │  生成任务列表 / Generate task list
│ 生成 tasks.md     │  → 可执行任务分解
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ /speckit.analyze  │  一致性检查 (可选 / Optional)
│ 检查工件一致性     │  → 质量门检查
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│/speckit.implement │  执行实现 / Execute implementation
│ 按 tasks.md 编码  │  → 逐任务实现
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ /speckit.intro    │  生成项目概览 / Generate project overview
│ 生成进度报告       │  → 全局视图报告
└───────────────────┘
```

### 数据流图 / Data Flow Diagram

```text
用户输入 / User Input
    │
    ▼
┌─────────────────────────────────────────┐
│  命令定义 .md 文件 / Command .md File    │
│  - 解析 $ARGUMENTS                       │
│  - 提供 AI 指令                          │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  AI 编码助手 / AI Coding Assistant       │
│  - 理解指令                              │
│  - 调用 Bash 工具                        │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Shell 脚本 / Shell Scripts              │
│  - 文件系统操作                          │
│  - Git 操作                              │
│  - 输出 JSON                             │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  JSON 响应 / JSON Response               │
│  { status, data, errors }                │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  AI 解析 JSON / AI Parses JSON           │
│  - 提取数据                              │
│  - 填充模板                              │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  生成文件 / Generated Files              │
│  - .specify/specs/###-name/spec.md      │
│  - .specify/specs/###-name/plan.md      │
│  - .specify/specs/###-name/tasks.md     │
└─────────────────────────────────────────┘
```

### 组件交互序列 / Component Interaction Sequence

```text
用户 / User     AI助手 / AI     命令文件 / Command     脚本 / Scripts     文件系统 / FS
   │              │                  │                    │                  │
   │─── /cmd ────▶│                  │                    │                  │
   │              │                  │                    │                  │
   │              │──── 读取 / Read ─▶│                    │                  │
   │              │◀──── 指令 / Inst ─┤                    │                  │
   │              │                  │                    │                  │
   │              │───────── 执行 Bash / Execute Bash ────▶│                  │
   │              │                  │                    │                  │
   │              │                  │                    │── 操作 / Ops ───▶│
   │              │                  │                    │◀── 结果 / Result ─┤
   │              │                  │                    │                  │
   │              │◀──────── JSON 响应 / JSON Response ────┤                  │
   │              │                  │                    │                  │
   │              │──────────── Write 工具 / Write Tool ───────────────────▶│
   │              │                  │                    │                  │
   │◀── 完成 ──────┤                  │                    │                  │
```

---

## 🔑 关键技术点 / Key Technical Points

### 1. JSON 通信协议 / JSON Communication Protocol

**中文说明：**
Shell 脚本输出标准 JSON 格式，AI 解析后填充模板。这确保跨平台兼容性和结构化数据传输。

**English Explanation:**
Shell scripts output standard JSON format, which AI parses to fill templates. This ensures cross-platform compatibility and structured data transfer.

**示例 / Example**:
```bash
# Bash 输出 / Bash outputs
echo '{"status":"success","branch":"001-auth","feature":"User Authentication"}'

# AI 解析 / AI parses
branch_name = json.status["branch"]  # "001-auth"
```

### 2. 占位符替换系统 / Placeholder Replacement System

**占位符语法 / Placeholder Syntax**: `[PLACEHOLDER_NAME]`

**替换过程 / Replacement Process**:
1. AI 读取模板文件 / AI reads template file
2. 识别所有 `[...]` 占位符 / Identifies all `[...]` placeholders
3. 用实际数据替换 / Replaces with actual data
4. 写入最终文件 / Writes final file

**常见占位符 / Common Placeholders**:
- `[功能名称]` → 实际功能名 / Actual feature name
- `[当前日期]` → 2025-10-23
- `[分支名称]` → 001-feature-name
- `[用户描述]` → 用户输入的自然语言描述

### 3. 分支命名策略 / Branch Naming Strategy

**格式 / Format**: `###-feature-name`
- `###`: 三位数字编号 (001, 002, 003...)
- `feature-name`: 小写连字符分隔

**自动递增 / Auto-increment**:
```bash
# 查找最大编号 / Find max number
MAX_NUM=$(ls -d .specify/specs/[0-9][0-9][0-9]-* 2>/dev/null |
          sed 's/.*\/\([0-9][0-9][0-9]\)-.*/\1/' |
          sort -n | tail -1)
NEXT_NUM=$(printf "%03d" $((MAX_NUM + 1)))
```

### 4. 非 Git 仓库支持 / Non-Git Repository Support

**回退策略 / Fallback Strategy**:
```bash
if git rev-parse --git-dir > /dev/null 2>&1; then
    BRANCH=$(git branch --show-current)
else
    BRANCH="main"  # 默认分支 / Default branch
fi
```

### 5. 跨平台兼容性 / Cross-Platform Compatibility

**支持的 AI 助手 / Supported AI Assistants** (15+):
- Claude Code, Cursor, Windsurf, Cline
- Aider, Continue, Zed AI, Copilot Edits
- GitHub Copilot Workspace, Mentat
- PearAI, Melty, Void, Pieces
- v0.dev

**通用模板 / Universal Templates**:
- `.claude/commands/` → Claude Code 专用
- `templates/commands/` → 其他 AI 助手通用模板

### 6. 文件命名约定 / File Naming Conventions

**时间戳优先 / Timestamp-First** (便于排序 / For easy sorting):
```
.specify/intro/{YYYY-MM-DD}_{HH-MM-SS}_{BRANCH}.md
示例 / Example: 2025-10-23_14-30-45_main.md
```

**功能目录 / Feature Directory**:
```
.specify/specs/{###-feature-name}/
├── spec.md       # 功能规格 / Feature specification
├── plan.md       # 实现计划 / Implementation plan
├── tasks.md      # 任务列表 / Task breakdown
└── research.md   # 研究笔记 / Research notes (optional)
```

### 7. 双语支持机制 / Bilingual Support Mechanism

**格式规范 / Format Convention**:
- **标题 / Headings**: "中文 / English"
- **段落内容 / Paragraph Content**: 分段提供中英文

**示例 / Example**:
```markdown
## 📋 执行摘要 / Executive Summary

**中文摘要：**
此项目实现用户认证系统...

**English Summary:**
This project implements a user authentication system...
```

### 8. 质量门检查 / Quality Gate Checks

**/speckit.analyze 命令 / Command**:
- 规格完整性 / Spec completeness
- 计划-任务一致性 / Plan-task consistency
- 技术栈验证 / Technology stack validation
- 依赖关系检查 / Dependency validation

### 9. 渐进式披露 / Progressive Disclosure

**参数处理 / Argument Handling**:
```bash
case "$1" in
    "summary"|"brief")
        # 输出简化版 / Output condensed version
        ;;
    [0-9][0-9][0-9])
        # 输出特定功能详情 / Output specific feature details
        ;;
    *)
        # 输出完整报告 / Output full report
        ;;
esac
```

### 10. 元项目概念 / Meta-Project Concept

**Spec Kit 自身 / Spec Kit Itself**:
- 是一个工具包 / Is a toolkit
- 不是使用方法论的项目 / Not a project using the methodology
- `.specify/specs/` 为空是正常的 / Empty `.specify/specs/` is normal
- 提供给其他项目使用 / Provides tools for other projects

---

## 🎯 功能概览 / Features Overview

| # | 功能 / Feature | 规格状态 / Spec Status | 任务进度 / Task Progress | 实现状态 / Implementation | 对齐情况 / Alignment |
|---|---------|-------------|---------------|----------------|-----------|
| 001 | [Name] | 🟡 进行中 / In Progress | [█████░░░░░] 50% | 🟡 部分完成 / Partial | ✅ 已对齐 / Aligned |
| 002 | [Name] | 🔵 就绪 / Ready | [░░░░░░░░░░] 0% | ⏳ 未开始 / Not Started | ✅ 已对齐 / Aligned |
| 003 | [Name] | 🟢 完成 / Complete | [██████████] 100% | ✅ 完成 / Complete | ⚠️ 技术不匹配 / Tech Mismatch |

**图例 / Legend:**
- **规格状态 / Spec Status**: 🟢 完成 / Complete | 🟡 进行中 / In Progress | 🔵 就绪 / Ready | 🟣 规划中 / Planning | ⚪ 草稿 / Draft
- **实现状态 / Implementation**: ✅ 完成 / Complete | 🟡 部分 / Partial | ⏳ 未开始 / Not Started | ❌ 缺失 / Missing | ⚠️ 无规格代码 / Code w/o Spec
- **对齐情况 / Alignment**: ✅ 已对齐 / Aligned | ⚠️ 差异 / Discrepancy | ❌ 不匹配 / Mismatch

---

## 📊 总体统计 / Overall Statistics

### 功能统计 / Feature Statistics
- **总功能数 / Total Features**: [N]
- **已完成 / Completed**: [N] 🟢
- **进行中 / In Progress**: [N] 🟡
- **准备实现 / Ready to Implement**: [N] 🔵
- **规划中 / In Planning**: [N] 🟣
- **草稿 / Draft**: [N] ⚪

### 任务统计 / Task Statistics
- **总任务数 / Total Tasks**: [N]
- **已完成 / Completed**: [N] ([X]%)
- **进行中 / In Progress**: [N]
- **待处理 / Pending**: [N]
- **并行任务 / Parallel Tasks**: [N]

### 覆盖度指标 / Coverage Metrics
- **用户故事 / User Stories**: [N] 个跨所有功能 / total across all features
- **功能需求 / Functional Requirements**: [N] 个跨所有功能 / total across all features
- **成功标准 / Success Criteria**: [N] 个跨所有功能 / total across all features

---

## 🔄 规格与实现对齐分析 / Spec-to-Implementation Alignment

### 对齐摘要 / Alignment Summary

- **✅ 完全对齐的功能 / Fully Aligned Features**: [N] 个功能 / features
  - 规划技术栈与实际技术栈匹配 / Planned tech matches actual tech
  - 所有指定文件都存在 / All specified files exist
  - 任务完成度与代码交付匹配 / Tasks completion matches code delivery

- **⚠️ 部分对齐 / Partial Alignment**: [N] 个功能 / features
  - 规格与实现之间存在一些差异 / Some discrepancies between spec and implementation
  - 详见下文 / See details below

- **❌ 未对齐的功能 / Misaligned Features**: [N] 个功能 / features
  - 规格与实际之间存在显著差距 / Significant gaps between spec and reality
  - 需要关注 / Requires attention

### 技术栈对比 / Technology Stack Comparison

| 组件 / Component | 规划中（来自 plan.md）/ Specified (from plan.md) | 实际检测到 / Actual (detected) | 状态 / Status |
|-----------|-------------------------|-------------------|--------|
| 语言 / Language | Python 3.11 | Python 3.9 | ⚠️ 版本不匹配 / Version mismatch |
| 框架 / Framework | FastAPI | FastAPI 0.104.1 | ✅ 匹配 / Match |
| 数据库 / Database | PostgreSQL | PostgreSQL 15 | ✅ 匹配 / Match |
| 前端 / Frontend | React 18 | 未找到 / Not found | ❌ 缺失 / Missing |

### 实现差距 / Implementation Gaps

**缺失的实现 / Missing Implementations** (任务标记完成但文件不存在 / Tasks marked complete but files don't exist):
- Feature 001: `src/auth/oauth.py` (任务 T015 标记为 [X] / Task T015 marked [X])
- Feature 002: `frontend/components/Dashboard.tsx` (任务 T023 标记为 [X] / Task T023 marked [X])

**孤立代码 / Orphaned Code** (文件存在但无对应规格 / Files exist without corresponding specs):
- `src/utils/legacy_processor.py` (1,234 行 / lines)
- `scripts/data_migration.sh` (156 行 / lines)

**建议操作 / Recommended Actions**:
1. 为孤立代码创建规格或标记删除 / Create specs for orphaned code or mark for removal
2. 验证已完成任务或更新任务状态 / Verify completed tasks or update task status
3. 使实际技术栈版本与规格对齐 / Align actual tech stack versions with specifications

---

## 🛠️ 技术栈摘要 / Technology Stack Summary

### 规划的技术栈（来自 plan.md 文件）/ Specified Stack (from plan.md files)
- Python 3.11 + FastAPI (001-user-auth)
- React 18 + TypeScript (002-dashboard)
- PostgreSQL (001-user-auth, 003-reporting)
- Redis Cache (002-dashboard)

### 实际技术栈（从项目检测）/ Actual Stack (detected from project)
- Python 3.9 (from runtime)
- FastAPI 0.104.1 (from requirements.txt)
- PostgreSQL 15 (from docker-compose.yml)
- 未检测到前端框架 / No frontend framework detected

### 差异 / Discrepancies
⚠️ Python 版本 / Python version: 规格要求 3.11，项目使用 3.9 / Spec says 3.11, project uses 3.9
⚠️ React 缺失 / React missing: 规格提到 React 18，但未找到 package.json 或 .jsx/.tsx 文件 / Spec mentions React 18, but no package.json or .jsx/.tsx files found

---

## 🚀 详细功能分解 / Detailed Feature Breakdown

### 功能 001 / Feature 001: [Feature Name]

**分支 / Branch**: `001-feature-name`
**规格状态 / Spec Status**: 🟡 进行中 / In Progress (45% 任务完成 / tasks complete)
**实现状态 / Implementation Status**: 🟡 部分实现 / Partially Implemented (12/20 文件存在 / files exist)

**概述 / Overview**: [Provide 1-2 sentence summary in both Chinese and English from spec.md]

**规格指标 / Specification Metrics**:
- 用户故事 / User Stories: [N]
- 功能需求 / Functional Requirements: [N]
- 成功标准 / Success Criteria: [N]

**规划的技术栈 / Planned Technology Stack** (来自 / from plan.md):
- 语言 / Language: Python 3.11
- 框架 / Framework: FastAPI
- 数据库 / Database: PostgreSQL

**实际实现 / Actual Implementation** (检测到 / detected):
- 已创建文件 / Files Created: 12/20 (60%)
- 代码行数 / Lines of Code: [N] 行 / lines
- 测试覆盖率 / Test Coverage: [X]% (如果可检测 / if detectable)
- 技术匹配 / Technology Match: ✅ 已对齐 / Aligned / ⚠️ 部分 / Partial / ❌ 不匹配 / Mismatch

**任务进度 / Tasks Progress**:
- ✅ 已完成 / Completed: [N]/[Total] 个任务 / tasks
- 🔄 进行中 / In Progress: [N] 个任务 / tasks
- ⏳ 待处理 / Pending: [N] 个任务 / tasks

**实现细节 / Implementation Details**:
- ✅ 已实现 / Implemented: `src/auth/login.py`, `src/auth/register.py` (按规格 / as specified)
- ⚠️ 部分 / Partial: `src/auth/oauth.py` (文件存在，但不完整 / file exists, but incomplete)
- ❌ 缺失 / Missing: `src/auth/password_reset.py` (任务标记完成，文件未找到 / task marked complete, file not found)

**关键里程碑 / Key Milestones**:
- [从 tasks.md 阶段或验收标准中提取，中英文 / Extracted from tasks.md phases or acceptance criteria, bilingual]

---

[为每个功能重复 / Repeat for each feature]

---

## 📈 进度时间线 / Progress Timeline

[如果 Git 历史可用，显示简单时间线，中英文 / If Git history is available, show a simple timeline, bilingual]

```text
[Date] - 功能 001 已创建 / Feature 001 created (spec.md)
[Date] - 功能 001 已规划 / Feature 001 planned (plan.md)
[Date] - 功能 001 实现已开始 / Feature 001 implementation started (tasks.md)
[Date] - 功能 002 已创建 / Feature 002 created (spec.md)
```

---

## 🎓 建议 / Recommendations

[基于分析，提供可操作的建议，中英文 / Based on the analysis, provide actionable recommendations, bilingual:]

### 立即行动 / Immediate Actions
- [ ] [E.g., "使用 /speckit.plan 完成功能 002 的规划 / Complete planning for Feature 002 using /speckit.plan"]
- [ ] [E.g., "恢复功能 001 的实现 - 还剩 10 个任务 / Resume implementation of Feature 001 - 10 tasks remaining"]

### 质量改进 / Quality Improvements
- [ ] [E.g., "为功能 001 添加缺失的测试覆盖 / Add missing test coverage for Feature 001"]
- [ ] [E.g., "在实现前对功能 003 运行 /speckit.analyze / Run /speckit.analyze on Feature 003 before implementation"]

### 下一步 / Next Steps
1. [优先级 1 行动 / Priority 1 action - bilingual]
2. [优先级 2 行动 / Priority 2 action - bilingual]
3. [优先级 3 行动 / Priority 3 action - bilingual]

---

## 🔧 可用命令 / Available Commands

要处理特定功能，使用这些命令 / To work on a specific feature, use these commands:

- `/speckit.constitution` - 创建或更新项目治理原则 / Create or update project governing principles
- `/speckit.specify` - 创建新功能规格 / Create a new feature specification
- `/speckit.clarify` - 澄清未充分指定的需求 / Clarify underspecified requirements
- `/speckit.plan` - 创建技术实现计划 / Create technical implementation plan
- `/speckit.tasks` - 生成可操作的任务列表 / Generate actionable task list
- `/speckit.analyze` - 检查工件间的一致性 / Check consistency across artifacts
- `/speckit.implement` - 执行实现 / Execute implementation
- `/speckit.checklist` - 生成自定义质量检查清单 / Generate custom quality checklists
- `/speckit.intro` - 生成此概览（刷新）/ Generate this overview (refresh)

---

## 📚 其他资源 / Additional Resources

- **主仓库 / Main Repository**: [Repository URL]
- **文档 / Documentation**: [Documentation URL]
- **问题/支持 / Issues/Support**: [Issues URL]
- **许可证 / License**: [License type]

---

**报告结束 / Report End**

*此报告由 /speckit.intro 生成 / This report was generated by `/speckit.intro` on [Date] at [Time]*
```

### 8. Save Report to File

**Always save the report** to the file path determined in Step 1:

1. Generate the complete Markdown report as a string (following the structure in Step 7)

2. Use the Write tool to save the report:
   ```
   Write the report content to: $OUTPUT_FILE
   ```

3. After saving, output a **concise bilingual summary** to the terminal:
   ```markdown
   # ✅ 项目概览报告已生成 / Project Overview Report Generated

   **报告已保存至 / Report saved to**: `.specify/intro/{date}_{time}_{branch}.md`

   ## 快速摘要 / Quick Summary

   **项目状态 / Project Status**:
   - 总功能数 / Total Features: [N]
   - 总体进度 / Overall Progress: [X]%
   - 实现对齐度 / Implementation Alignment: [Y]%

   **功能分解 / Feature Breakdown**:
   - 🟢 完成 / Complete: [N] 个功能 / features
   - 🟡 进行中 / In Progress: [N] 个功能 / features
   - 🔵 准备实现 / Ready to Implement: [N] 个功能 / features
   - 🟣 规划中 / Planning: [N] 个功能 / features

   **技术栈 / Technology Stack**:
   - [Primary languages/frameworks detected]

   **前 3 项建议 / Top 3 Recommendations**:
   1. [Most critical action - bilingual]
   2. [Second priority action - bilingual]
   3. [Third priority action - bilingual]

   ---

   📄 **查看完整报告 / View full report**: `.specify/intro/{date}_{time}_{branch}.md`
   ```

4. This bilingual summary gives the user immediate visibility in both Chinese and English while the full detailed report is saved for reference.

**Example output to terminal:**
```markdown
# ✅ 项目概览报告已生成 / Project Overview Report Generated

**报告已保存至 / Report saved to**: `.specify/intro/2025-10-23_14-30-45_main.md`

## 快速摘要 / Quick Summary

**项目状态 / Project Status**:
- 总功能数 / Total Features: 3
- 总体进度 / Overall Progress: 45%
- 实现对齐度 / Implementation Alignment: 67%

**功能分解 / Feature Breakdown**:
- 🟢 完成 / Complete: 0 个功能 / features
- 🟡 进行中 / In Progress: 1 个功能 / feature
- 🔵 准备实现 / Ready to Implement: 1 个功能 / feature
- 🟣 规划中 / Planning: 1 个功能 / feature

**技术栈 / Technology Stack**:
- Python 3.9 + FastAPI
- PostgreSQL 15

**前 3 项建议 / Top 3 Recommendations**:
1. 继续实现功能 001 - 还剩 11 个任务 / Continue implementation of Feature 001 - 11 tasks remaining
2. 使用 /speckit.tasks 为功能 002 生成任务 / Generate tasks for Feature 002 using /speckit.tasks
3. 对齐 Python 版本：规格要求 3.11，项目使用 3.9 / Align Python version: Spec says 3.11, project uses 3.9

---

📄 **查看完整报告 / View full report**: `.specify/intro/2025-10-23_14-30-45_main.md`
```

### 9. Handle Edge Cases (Bilingual)

Handle all edge cases with bilingual messages:

- **No features exist**: Report "未找到功能。使用 `/speckit.specify` 创建您的第一个功能。/ No features found. Use `/speckit.specify` to create your first feature."
- **No constitution exists**: Report "未找到项目治理文档。使用 `/speckit.constitution` 建立项目原则。/ No project constitution found. Use `/speckit.constitution` to establish project principles."
- **Empty specs directory**: Provide bilingual guidance on getting started with spec-kit
- **Incomplete features**: Highlight what's missing in both languages and suggest next commands

### 10. Progressive Disclosure (Token Efficiency) - Bilingual

If the user provides arguments like "summary" or "brief", output a condensed bilingual version:

```markdown
# 🌱 项目摘要 / Project Summary

**功能 / Features**: [N] 个总计 / total ([X] 完成 / complete, [Y] 进行中 / in progress, [Z] 已规划 / planned)
**总体进度 / Overall Progress**: [X]%
**主要技术栈 / Primary Tech Stack**: [Most common languages/frameworks]

[Feature list with bilingual status badges only, no detailed breakdown]
```

If the user provides a specific feature number (e.g., "001"), show only that feature's details in bilingual format.

## Output Guidelines (Bilingual Requirements)

- **Use emojis sparingly** for visual hierarchy (section headers only)
- **Provide ALL content in both Chinese and English** - This is the PRIMARY requirement
- **Keep language professional and concise** in both languages
- **Focus on actionable insights**, not just data dumps, in both languages
- **Use progress bars** for visual representation: `[█████░░░░░] 50%`
- **Highlight blockers** if any feature is stuck, using bilingual descriptions
- **Calculate percentages accurately** based on task completion
- **Sort features by number** (001, 002, 003...)
- **Use consistent bilingual format**: "中文 / English" for headings, separate paragraphs for detailed content

## Success Criteria

A successful bilingual report should answer (in both Chinese and English):
1. **这个项目是什么？/ What is this project?** (from constitution/specs)
2. **我们进展如何？/ How far along are we?** (from task completion)
3. **我们使用什么技术？/ What technologies are we using?** (from plans)
4. **下一步该做什么？/ What should we do next?** (recommendations)

## Context

$ARGUMENTS
