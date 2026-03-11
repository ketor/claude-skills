---
name: team
description: 智能路由——根据任务描述自动选择最合适的 team-* skill 并调用。无需记忆 16 个 skill 名称，只需 /team 你的任务。使用方式：/team [--auto] [--lang=zh|en] 任务描述
disable-model-invocation: true
argument-hint: [--auto] [--lang=zh|en] 任务描述
---

**参数解析**：从 `$ARGUMENTS` 中检测以下标志：
- `--auto`：传递给目标 skill 的自主模式标志
- `--lang=zh|en`：传递给目标 skill 的输出语言标志

解析后保留完整的任务描述（含未识别的参数，如 `--focus`、`--depth` 等，一并传递给目标 skill）。

---

## 路由决策流程

```
用户输入 → 提取任务关键词和意图
         → 匹配决策矩阵（按优先级）
         → 单一匹配 → 确认并调用
         → 多个候选 → AskUserQuestion 让用户选择
         → 无匹配   → AskUserQuestion 让用户指定
```

---

## 决策矩阵

按**任务意图**匹配，优先匹配排在前面的规则。当多条规则同时匹配时，选择最具体的那条。

### 开发与修复

| 意图信号 | 目标 Skill | 传递参数 |
|---------|-----------|---------|
| "开发""实现""新增功能""添加""创建功能""做一个" | `/team-dev` | `[--auto] [--lang]` |
| "重构""迁移""拆分模块""合并模块""重命名""提取接口" | `/team-refactor` | `[--auto] [--scope=module\|package\|system] [--lang]` |
| "修复 bug""debug""定位问题""排查""为什么报错""崩溃" | `/team-debug` | `[--auto] [--lang]` |

### 审查与分析

| 意图信号 | 目标 Skill | 传递参数 |
|---------|-----------|---------|
| "review 代码""代码审查""代码质量""review PR" | `/team-review` | `[--auto] [--focus] [--lang]` |
| "分析架构""架构评估""模块关系""依赖分析" | `/team-arch` | `[--auto] [--depth] [--focus] [--lang]` |
| "性能优化""性能分析""慢""延迟高""OOM""内存泄漏" | `/team-perf` | `[--auto] [--focus] [--lang]` |
| "安全审计""安全扫描""漏洞""安全检查""渗透" | `/team-security` | `[--auto] [--scope] [--lang]` |

### 设计与方案

| 意图信号 | 目标 Skill | 传递参数 |
|---------|-----------|---------|
| "写 RFC""技术方案""设计文档""技术设计""写方案" | `/team-rfc` | `[--auto] [--type] [--lang]` |
| "评审方案""评审设计""review RFC""评审文档""评估可行性" | `/team-design-review` | `[--auto] [--lang]` |
| "设计 API""API 评审""接口设计""定义接口" | `/team-api-design` | `[--auto] [--style] [--lang]` |

### 运维与发布

| 意图信号 | 目标 Skill | 传递参数 |
|---------|-----------|---------|
| "线上故障""生产事故""告警""服务不可用""紧急" | `/team-incident` | `[--auto] [--severity] [--lang]` |
| "复盘""事后分析""postmortem""故障总结""经验教训" | `/team-postmortem` | `[--auto] [--lang]` |
| "发布""上线""release""版本""changelog" | `/team-release` | `[--auto] [--type] [--from] [--lang]` |

### 调研与文档

| 意图信号 | 目标 Skill | 传递参数 |
|---------|-----------|---------|
| "调研""研究""对比""技术选型""了解""分析趋势" | `/team-research` | `[--auto] [--depth] [--lang]` |
| "入职文档""知识库""上手指南""项目文档""新人" | `/team-onboard` | `[--auto] [--target] [--lang]` |
| "成本优化""成本分析""降本""资源利用率""GPU 利用率" | `/team-cost` | `[--auto] [--scope] [--lang]` |

---

## 路由执行步骤

### 步骤 1：意图识别

分析 `$ARGUMENTS` 中的任务描述，提取：
1. **动作动词**：开发/修复/审查/设计/发布/调研/优化/...
2. **对象名词**：代码/架构/API/安全/性能/成本/...
3. **紧急程度**：是否包含"紧急""线上""生产"等关键词
4. **是否指定了目标 skill**：如果用户已经写了 `team-xxx`，直接转发

### 步骤 2：匹配决策

按以下优先级匹配：

1. **精确匹配**：任务描述中直接包含 skill 名称（如"用 team-perf 分析性能"）→ 直接调用
2. **紧急优先**：包含紧急/线上/告警关键词 → 优先匹配 `/team-incident`
3. **意图匹配**：按决策矩阵匹配最具体的规则
4. **组合任务**：如果任务包含多个意图（如"开发完后做 review"），选择第一个意图对应的 skill，在完成后建议下一个

### 步骤 3：确认与调用

**单一匹配**（置信度高）：
- 向用户简要说明匹配结果：`"根据任务描述，我将使用 /team-xxx 来处理。"`
- 用 AskUserQuestion 确认（提供匹配的 skill 和 2 个最近的候选）
- 确认后，使用 **Skill 工具** 调用目标 skill，将完整参数传递

**多个候选**（置信度中等）：
- 列出 2-3 个候选 skill 及匹配理由
- 用 AskUserQuestion 让用户选择
- 确认后调用

**无匹配**（置信度低）：
- 展示全部 16 个 skill 的简表
- 用 AskUserQuestion 让用户选择

### 步骤 4：调用目标 Skill

使用 Skill 工具调用选中的 team-* skill：
- 将 `--auto`（如有）传递
- 将 `--lang`（如有）传递
- 将其余参数和任务描述作为 args 传递
- 如果用户提供了目标 skill 特有的参数（如 `--depth`、`--focus`、`--scope`），一并传递

---

## Skill 速查表

路由无法匹配时展示此表：

| Skill | 一句话描述 | 典型场景 |
|-------|-----------|---------|
| `/team-dev` | 完整研发流程 | "帮我实现用户登录功能" |
| `/team-debug` | 系统化 Bug 诊断 | "这个接口偶尔返回 500" |
| `/team-perf` | 性能剖析优化 | "列表页加载太慢了" |
| `/team-security` | 安全审计 | "检查一下这个项目的安全性" |
| `/team-review` | 代码审查 | "review 一下这个项目的代码质量" |
| `/team-arch` | 架构分析 | "分析一下这个项目的架构" |
| `/team-rfc` | 技术方案撰写 | "写一个缓存方案的 RFC" |
| `/team-design-review` | 方案评审 | "评审一下这个设计文档" |
| `/team-api-design` | API 设计 | "设计用户管理的 API" |
| `/team-incident` | 故障响应 | "线上订单服务挂了" |
| `/team-postmortem` | 复盘分析 | "复盘一下昨天的故障" |
| `/team-release` | 发布管理 | "准备发布 v2.1.0" |
| `/team-refactor` | 重构工程 | "把这个模块拆成微服务" |
| `/team-research` | 技术调研 | "调研 Rust vs Go 的选型" |
| `/team-onboard` | 知识库构建 | "生成项目入职文档" |
| `/team-cost` | 成本优化 | "分析一下 GPU 集群的使用效率" |

---

## 组合任务处理

当任务描述包含多个阶段时，路由建议分步执行：

| 组合模式 | 推荐顺序 |
|---------|---------|
| "开发 + 审查" | `/team-dev` → `/team-review` |
| "设计 + 开发" | `/team-rfc` → `/team-dev` |
| "故障 + 复盘" | `/team-incident` → `/team-postmortem` |
| "审计 + 修复" | `/team-security` → `/team-dev` |
| "架构分析 + 重构" | `/team-arch` → `/team-refactor` |
| "调研 + 方案" | `/team-research` → `/team-rfc` |
| "开发 + 发布" | `/team-dev` → `/team-release` |
| "性能分析 + 优化代码" | `/team-perf`（内含优化实施） |

路由只执行第一个 skill。第一个 skill 完成后，其跨团队衔接建议会自然引导到下一个。

---

## 核心原则

- **零记忆负担**：用户不需要记住 16 个 skill 名称，自然语言描述任务即可
- **保守路由**：不确定时宁可多问一次，不误导到错误的 skill
- **紧急优先**：包含紧急信号的任务优先匹配 `/team-incident`
- **参数透传**：所有用户指定的参数完整传递给目标 skill，不丢失
- **单一职责**：路由只做选择和转发，不做任何实际工作

---

## 需求

$ARGUMENTS
