# team-arch Skill Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a multi-agent Claude Code skill (`team-arch`) that analyzes codebases through dual-analyzer consensus and outputs structured architecture documents with Mermaid diagrams.

**Architecture:** Single SKILL.md file following the established patterns from team-dev and team-review skills. Uses TeamCreate/TeamDelete, TaskCreate/TaskUpdate, SendMessage, and Agent tools. Four roles: scanner, analyzer-1, analyzer-2, writer.

**Tech Stack:** Claude Code skills (Markdown with YAML frontmatter), Mermaid diagrams

---

### Task 1: Create skill directory

**Files:**
- Create: `team-arch/SKILL.md` (empty placeholder)

**Step 1: Create directory and empty file**

```bash
mkdir -p /home/ketor/.claude/skills/team-arch
touch /home/ketor/.claude/skills/team-arch/SKILL.md
```

**Step 2: Verify directory structure matches siblings**

```bash
ls -la /home/ketor/.claude/skills/
```

Expected: `team-arch/`, `team-dev/`, `team-review/` all present as sibling directories.

**Step 3: Commit**

```bash
cd /home/ketor/.claude/skills
git init  # if not already a git repo
git add team-arch/SKILL.md
git commit -m "feat: scaffold team-arch skill directory"
```

---

### Task 2: Write YAML frontmatter and parameter parsing

**Files:**
- Modify: `team-arch/SKILL.md`

**Step 1: Write the frontmatter and parameter section**

Write the YAML frontmatter and parameter parsing table to `team-arch/SKILL.md`. Must follow these exact patterns from team-dev:
- `name: team-arch`
- `description:` in Chinese, starts with 启动, lists roles, includes usage syntax
- `disable-model-invocation: true`
- `argument-hint:` with all flags
- Parameter parsing table with 标准模式/自主模式 rows
- Auto-mode decision rules specific to architecture analysis (analyzer disagreements, large project handling)
- TeamCreate naming format: `team-arch-{YYYYMMDD-HHmmss}`

Reference patterns from `team-dev/SKILL.md:1-23` and `team-review/SKILL.md:1-23`.

Content for this section:

```markdown
---
name: team-arch
description: 启动一个架构分析团队（scanner/analyzer×2/writer），通过双分析师独立分析和共识合并，对代码仓库进行全面架构分析，输出结构化架构文档和 Mermaid 可视化图表。使用方式：/team-arch [--auto] [--depth=shallow|standard|deep] 项目路径或描述
disable-model-invocation: true
argument-hint: [--auto] [--depth=shallow|standard|deep] 项目路径或描述
---

**参数解析**：从 `$ARGUMENTS` 中检测以下标志：
- `--auto`：自主模式（减少用户确认）
- `--depth=shallow|standard|deep`：分析深度（默认 `standard`）

| 模式 | 用户确认范围 | 条件节点处理 |
|------|-------------|-------------|
| **标准模式**（默认） | 项目概况确认 + 分歧仲裁 + 最终文档确认 | 正常询问用户 |
| **自主模式**（`--auto`） | 仅最终文档确认 | 自动决策 + 收尾汇总 |

自主模式下条件节点自动决策规则：
- **分析深度不确定** → team lead 根据项目规模自行判断，在最终文档中说明
- **两位 analyzer 分歧** → writer 合并时标注分歧，team lead 综合论证后裁决，收尾时汇总
- **分歧超过 50%** → **不可跳过，必须暂停问用户**（熔断机制）
- **项目过大无法完整分析** → scanner 识别核心模块，analyzer 聚焦核心模块

分析深度说明：

| 深度 | 分析范围 |
|------|---------|
| `shallow` | 仅项目结构和依赖概览，跳过深入架构模式分析 |
| `standard` | 结构 + 依赖 + 核心模块架构模式 + 数据流 |
| `deep` | 全量分析，额外包括性能热点、安全边界、技术债务、演进建议 |

使用 TeamCreate 创建 team（名称格式 `team-arch-{YYYYMMDD-HHmmss}`），你作为 team lead 按以下流程协调。
```

**Step 2: Verify frontmatter is valid**

Read the file back and verify:
- YAML frontmatter has exactly `name`, `description`, `disable-model-invocation`, `argument-hint`
- Description is under 1024 characters total frontmatter
- Chinese text consistent with team-dev/team-review style

**Step 3: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add frontmatter and parameter parsing"
```

---

### Task 3: Write flow overview and role definitions

**Files:**
- Modify: `team-arch/SKILL.md` (append after parameter section)

**Step 1: Write the flow overview and roles**

Append the flow overview ASCII diagram and role definition table. Follow the exact formatting patterns from `team-dev/SKILL.md:27-51`.

Content to append:

```markdown
## 流程概览

​```
阶段零  项目探测 → scanner 扫描结构 + 技术栈 → 输出项目概况
         ↓
阶段一  并行分析 → scanner 深入依赖分析 + analyzer-1 独立分析 + analyzer-2 独立分析
         ↓
阶段二  共识合并 → writer 对比两份分析 → 标注一致/分歧 → 生成初稿
         ↓
阶段三  分歧仲裁 → team lead 对分歧点让两位 analyzer 各自论证 → 仲裁（或问用户）
         ↓
阶段四  文档生成 → writer 生成最终 Markdown + Mermaid 图表 → 用户确认
         ↓
阶段五  收尾 → 保存文档 + 清理团队
​```

## 角色定义

| 角色 | 职责 |
|------|------|
| scanner | 扫描项目结构（目录布局、入口文件、构建系统）、识别技术栈、解析依赖关系（包管理器/import graph）、识别配置文件。**只做结构层面分析，不深入业务逻辑。** |
| analyzer-1 | 深入阅读核心模块代码，分析架构模式、模块间通信方式、数据流、错误处理策略。**独立分析，不与 analyzer-2 交流。** |
| analyzer-2 | 同 analyzer-1 的职责，独立执行相同分析。**独立分析，不与 analyzer-1 交流。** |
| writer | 对比两位 analyzer 的分析结果，标注一致和分歧之处，合并 scanner 的结构数据，生成 Mermaid 图表，撰写最终架构文档。**不直接阅读代码，只基于分析报告工作。** |

---
```

**Step 2: Verify role table formatting**

Read the file and verify the table renders correctly with consistent column widths and bold markers.

**Step 3: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add flow overview and role definitions"
```

---

### Task 4: Write Phase 0 (Project Detection)

**Files:**
- Modify: `team-arch/SKILL.md` (append)

**Step 1: Write Phase 0**

Content to append:

```markdown
## 阶段零：项目探测

### 步骤 1：启动 scanner

Team lead 启动 scanner，指示其快速扫描：
- 目录结构（顶层目录布局、最大深度 3 层）
- 入口文件（main 文件、index 文件、启动脚本）
- README 和文档目录
- 构建配置（Makefile、CMakeLists.txt、build.gradle、package.json、Cargo.toml 等）
- 包管理器和依赖文件
- CI/CD 配置（.github/workflows、.gitlab-ci.yml 等）
- 代码规范配置（lint、format、editorconfig）

Scanner 输出项目概况报告，包含：
- 项目名称和简述（来自 README）
- 编程语言及占比估算
- 框架和主要库
- 构建工具和包管理器
- 项目规模估算（文件数、预估代码行数）
- 识别到的核心模块/目录

### 步骤 2：评估分析深度

Team lead 根据 scanner 报告评估：
- 如果用户指定了 `--depth`，使用指定值
- 如果未指定：
  - 小型项目（<1 万行）→ 建议 `standard`
  - 中型项目（1-10 万行）→ 建议 `standard`
  - 大型项目（>10 万行）→ 建议 `shallow`，`deep` 时需警告耗时较长

**标准模式**：向用户展示项目概况 + 深度建议，AskUserQuestion 确认
**自主模式**：team lead 自行决定，收尾汇总时说明

---
```

**Step 2: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add phase 0 project detection"
```

---

### Task 5: Write Phase 1 (Parallel Analysis)

**Files:**
- Modify: `team-arch/SKILL.md` (append)

**Step 1: Write Phase 1**

Content to append:

```markdown
## 阶段一：并行分析

### 步骤 3：启动 analyzer-1、analyzer-2 和 scanner 深入分析

三者并行启动，全程保持存活直到收尾。

**Scanner 深入依赖分析**：
- 解析 import/include/require 语句，构建模块间引用关系
- 解析包依赖（直接依赖 vs 间接依赖）
- 识别循环依赖
- 统计各模块的入度/出度（被依赖数/依赖数）

**Analyzer-1 和 Analyzer-2 各自独立分析**（team lead 必须确保两者不互相看到对方的报告）：

每位 analyzer 阅读核心模块代码，输出结构化分析报告，包含：

1. **架构模式识别**：分层架构 / 微服务 / Monolith / 事件驱动 / CQRS / 管道-过滤器 / 其他。提供判断依据。
2. **模块划分与职责**：列出主要模块，每个模块的职责、对外接口、内部结构。
3. **数据流**：核心数据如何在模块间流转，入口到出口的主要路径。
4. **状态管理**：全局状态、共享状态、状态持久化方式。
5. **关键抽象和接口设计**：核心接口/抽象类/trait/protocol，设计意图和使用方式。
6. **错误处理策略**：错误如何传播、是否统一处理、异常 vs 错误码。

`deep` 模式额外分析：
7. **性能热点**：可能的性能瓶颈、资源密集操作、缓存策略。
8. **安全边界**：输入验证、认证/授权边界、敏感数据处理。
9. **技术债务**：过时依赖、TODO/FIXME/HACK 注释、已弃用 API 使用。

### 步骤 4：收集报告

三者完成后各自向 team lead 发送报告。Team lead 确认收到全部 3 份报告后，进入阶段二。

---
```

**Step 2: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add phase 1 parallel analysis"
```

---

### Task 6: Write Phase 2 (Consensus Merge)

**Files:**
- Modify: `team-arch/SKILL.md` (append)

**Step 1: Write Phase 2**

Content to append:

```markdown
## 阶段二：共识合并

### 步骤 5：启动 writer 进行对比合并

Team lead 启动 writer，将以下内容传递给 writer：
- Scanner 的项目概况报告和依赖分析报告
- Analyzer-1 的分析报告（标记为"分析师 A"）
- Analyzer-2 的分析报告（标记为"分析师 B"）

**重要**：传递时不透露 analyzer 编号，仅用"分析师 A"和"分析师 B"标记，避免暗示优先级。

### 步骤 6：Writer 对比分析

Writer 逐项对比两份分析报告，按以下规则处理：

| 对比结果 | 处理方式 |
|---------|---------|
| **一致结论** | 直接采纳，标记为"共识" |
| **互补发现**（A 发现了 B 没注意的点，或反之） | 合并，标记为"互补" |
| **措辞/粒度差异**（本质相同，表述不同） | 合并最佳表述，标记为"共识" |
| **分歧/矛盾**（对同一模块/模式有不同判断） | 标注为"待仲裁"，记录双方观点 |

Writer 输出：
1. **合并初稿**：整合所有共识和互补发现的架构文档草稿
2. **共识度评估**：一致比例（共识+互补 / 总发现数 × 100%）
3. **分歧清单**：每个分歧点的双方观点对比

### 步骤 7：检查熔断条件

如果共识度 < 50%（分歧占比超过一半）：
- **必须暂停**，team lead 向用户报告情况
- 可能原因：项目太大导致两位 analyzer 分析了不同部分、需求描述不够明确
- 建议：调整分析范围或缩小 depth

共识度 ≥ 50%：继续下一阶段。

---
```

**Step 2: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add phase 2 consensus merge"
```

---

### Task 7: Write Phase 3 (Dispute Arbitration)

**Files:**
- Modify: `team-arch/SKILL.md` (append)

**Step 1: Write Phase 3**

Content to append:

```markdown
## 阶段三：分歧仲裁

### 步骤 8：判断是否需要仲裁

如果分歧清单为空 → 跳过此阶段，直接进入阶段四。

### 步骤 9：逐项仲裁

Team lead 对分歧清单中的每个分歧点：

1. 将分歧描述分别发给 analyzer-1 和 analyzer-2，要求各自提供论证：
   - 你的判断是什么？
   - 依据是哪些代码/文件/模式？
   - 为什么你认为对方的判断不准确？

2. 收到双方论证后：
   - **标准模式**：team lead 向用户展示分歧摘要和双方论证，AskUserQuestion 让用户裁决
   - **自主模式**：team lead 综合双方论证和代码证据自行裁决

3. 将仲裁结果发送给 writer 更新文档

### 步骤 10：更新初稿

Writer 根据仲裁结果更新合并初稿，将所有"待仲裁"项替换为最终结论。在文档附录中保留分歧记录和仲裁理由。

---
```

**Step 2: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add phase 3 dispute arbitration"
```

---

### Task 8: Write Phase 4 (Document Generation) with output format

**Files:**
- Modify: `team-arch/SKILL.md` (append)

**Step 1: Write Phase 4 with full document template**

Content to append:

```markdown
## 阶段四：文档生成

### 步骤 11：Writer 生成最终文档

Writer 基于合并后的分析结果，生成最终架构文档。文档格式：

​```markdown
# [项目名] 架构分析报告

> 生成时间：YYYY-MM-DD | 分析深度：shallow|standard|deep | 共识度：XX%

## 1. 项目概览

| 属性 | 值 |
|------|---|
| 编程语言 | [语言及占比] |
| 框架 | [主要框架] |
| 构建工具 | [构建工具] |
| 包管理器 | [包管理器] |
| 项目规模 | [文件数/预估行数] |
| 入口点 | [主入口文件] |

## 2. 架构总览

### 架构模式
[架构模式描述及判断依据]

### 架构总览图
​```mermaid
graph TB
    subgraph 模块A
        A1[组件1]
        A2[组件2]
    end
    subgraph 模块B
        B1[组件3]
    end
    A1 --> B1
​```

## 3. 模块结构

### 模块列表

| 模块 | 路径 | 职责 | 对外接口 |
|------|------|------|---------|
| [模块名] | [路径] | [职责描述] | [关键接口] |

### 模块关系图
​```mermaid
graph LR
    模块A -->|调用| 模块B
    模块A -->|依赖| 模块C
    模块B -->|事件| 模块D
​```

## 4. 数据流

### 核心数据流
[数据流描述]

### 数据流图
​```mermaid
flowchart LR
    入口 --> 处理层 --> 存储层 --> 输出
​```

## 5. 依赖分析

### 外部依赖
| 依赖 | 版本 | 用途 |
|------|------|------|
| [名称] | [版本] | [用途] |

### 内部模块依赖
[循环依赖警告（如有）]
[高入度/高出度模块说明]

## 6. 关键设计决策
1. **[决策名称]**：[描述] — Trade-off：[权衡分析]
2. ...

## 7. 技术债务与演进建议（仅 deep 模式）

### 技术债务
| 类型 | 位置 | 描述 | 建议优先级 |
|------|------|------|-----------|
| [类型] | [文件:行号] | [描述] | Critical/Major/Minor |

### 演进建议
1. [建议描述]
2. ...

## 附录：分析共识说明

### 共识结论
[两位分析师一致的核心结论列表]

### 分歧点及仲裁结果
| 分歧点 | 分析师 A 观点 | 分析师 B 观点 | 仲裁结果 | 理由 |
|--------|-------------|-------------|---------|------|
| [描述] | [观点] | [观点] | [结论] | [理由] |
​```

**注意**：每张 Mermaid 图不超过 15 个节点。如果模块过多，writer 分层展示（总览图 + 子模块详图）。

### 步骤 12：用户确认

Team lead 向用户展示文档摘要：
- 项目概览（一句话）
- 识别到的架构模式
- 模块数量和核心模块
- 共识度
- 分歧数量及处理结果

AskUserQuestion 确认：
- 接受文档
- 需要补充某些方面的分析
- 需要调整某些结论

**自主模式**：必须经用户确认。

---
```

**Step 2: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add phase 4 document generation with template"
```

---

### Task 9: Write Phase 5 (Wrap-up), core principles, and error handling

**Files:**
- Modify: `team-arch/SKILL.md` (append)

**Step 1: Write Phase 5, principles, error handling, and $ARGUMENTS**

Content to append:

```markdown
## 阶段五：收尾

### 步骤 13：保存文档

将最终架构文档保存到项目的 `docs/architecture/` 目录：
- 文件名：`architecture-analysis-YYYY-MM-DD.md`
- 如果目录不存在，创建之

### 步骤 14：最终总结

Team lead 向用户输出：
- 分析了什么（项目名称、范围、深度）
- 核心发现（架构模式、关键模块、主要数据流）
- 共识度和分歧处理情况
- 文档保存位置
- **（自主模式）自动决策汇总**：列出所有自动决策的节点、决策内容和理由

### 步骤 15：清理

关闭所有 teammate，用 TeamDelete 清理 team。

---

## 核心原则

- **独立分析**：两位 analyzer 必须完全独立工作，不互相看到对方结果，确保共识的客观性
- **共识驱动**：通过独立分析后的对比合并确保分析准确性，分歧必须仲裁
- **并行高效**：scanner 和两位 analyzer 并行工作，最大化效率
- **分层展示**：Mermaid 图表分层展示，每张图不超过 15 个节点
- **有限分析**：根据项目规模和 depth 参数控制分析范围，避免过度分析

---

## 错误处理

| 异常情况 | 处理方式 |
|---------|---------|
| 项目无法识别技术栈 | Scanner 输出目录结构和文件类型统计，analyzer 基于代码内容推断 |
| 项目过大无法完整分析 | Scanner 识别核心模块，analyzer 聚焦核心模块分析，文档说明分析范围限制 |
| 两位 analyzer 分析差异极大（共识度 < 50%） | 触发熔断，暂停问用户确认分析方向 |
| Mermaid 图表过于复杂 | Writer 分层展示（总览图 + 子模块详图），每张图不超过 15 个节点 |
| Analyzer 无法理解某模块 | 在报告中标注"未充分分析"，writer 在文档中说明 |
| 项目缺少 README/文档 | Scanner 基于代码结构和构建配置推断项目信息 |

---

## 需求

$ARGUMENTS
```

**Step 2: Verify the complete SKILL.md**

Read the entire file from top to bottom. Verify:
- Frontmatter is valid YAML
- All 6 phases (0-5) are present with correct step numbering (1-15)
- Role table has exactly 4 roles
- Document template has all 7 sections + appendix
- Error handling table covers all edge cases from design doc
- File ends with `$ARGUMENTS`
- Chinese text is consistent throughout

**Step 3: Commit**

```bash
git add team-arch/SKILL.md
git commit -m "feat(team-arch): add phase 5, core principles, error handling"
```

---

### Task 10: Verify skill is discoverable and well-formed

**Files:**
- Read: `team-arch/SKILL.md` (full verification)

**Step 1: Verify YAML frontmatter parses correctly**

Read the file and check:
- `name: team-arch` — no special characters
- `description:` — under 500 characters, starts with 启动
- `disable-model-invocation: true`
- `argument-hint:` — matches usage pattern

**Step 2: Verify structural consistency with team-dev and team-review**

Compare the following patterns between all three skills:
- Frontmatter fields (must match)
- Mode table format (标准模式/自主模式)
- Phase naming convention (阶段X)
- Step numbering (步骤 N)
- Role table format
- Error handling table format
- Ends with `$ARGUMENTS`

**Step 3: Word count check**

```bash
wc -w /home/ketor/.claude/skills/team-arch/SKILL.md
```

Expected: ~800-1200 words (comparable to team-dev at ~900 words and team-review at ~1000 words).

**Step 4: Final commit with all verification passing**

```bash
git add -A
git commit -m "feat: complete team-arch skill for multi-agent codebase architecture analysis"
```
