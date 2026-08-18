# Agent Skill 设计指南

Skill 是供 AI Agent 按需加载的模块化能力包。它向 Agent 提供通用模型知识之外的工作流程、领域规则、工具集成或可复用资源，使 Agent 能以稳定过程完成某一类任务。

## 推荐架构

```text
skill-name/
├── SKILL.md                 必需：触发元数据与核心执行流程
├── agents/
│   └── openai.yaml          可选：Skill 列表或界面的展示信息
├── scripts/                 可选：可执行的确定性工具
├── references/              可选：按需读取的详细规则与知识
└── assets/                  可选：用于最终产物的模板和资源
```

目录名应与 Skill 名称一致，使用小写字母、数字和连字符，例如 `report-audit`、`pptx-export`。名称宜简短、以动作或任务命名，并避免超过 64 个字符。

只创建真正需要的目录。Skill 不应包含面向人的附属文档，例如 `README.md`、安装指南、变更日志或速查表；这些文件会增加搜索和维护成本，却不直接帮助 Agent 执行任务。

### 资源文件职责

| 目录 | 适合放置的内容 | 加入条件 |
|---|---|---|
| `scripts/` | Python、Shell、JavaScript 等可执行程序 | 同一段操作会反复编写，或操作易出错且必须稳定执行 |
| `references/` | 领域知识、接口文档、Schema、复杂规则、变体流程 | 资料只在特定任务分支中需要被 Agent 阅读 |
| `assets/` | 模板、图片、字体、样例文件、代码骨架 | 最终产物需要直接复制、嵌入或修改这些资源 |
| `agents/openai.yaml` | 展示名称、简介、默认提示词等界面信息 | 所属运行时支持 Skill 列表、卡片或快捷提示时 |

脚本应有明确输入、输出和失败行为，并在加入前实际运行验证。引用资料应通过 `SKILL.md` 按条件加载，而不是与主流程重复复制。资产不用于承载规则说明；规则应放在 `SKILL.md` 或 `references/`。

## `SKILL.md` 内容格式

`SKILL.md` 是唯一必需文件，由 YAML Frontmatter 和 Markdown 正文构成。

```markdown
---
name: report-audit
description: Audit reports for required sections and factual consistency. Use when the user asks to review, verify, or quality-check a report.
---

# Report Audit

## Workflow

1. Read the report and identify its scope.
   Completion criterion: list the required sections and factual claims.

2. Compare every factual claim with the supplied source material.
   Completion criterion: mark every claim as supported, unsupported, or unverifiable.

3. Produce findings with evidence.
   Completion criterion: every finding cites its report location and supporting source.

## Conditional references

- When the report contains financial metrics, read [references/financial-metrics.md](references/financial-metrics.md).
- When the report contains regulated language, read [references/compliance.md](references/compliance.md).
```

Frontmatter 的基本字段如下：

| 字段 | 要求 | 作用 |
|---|---|---|
| `name` | 必需；与目录名一致 | 标识 Skill |
| `description` | 必需 | 描述能力和触发条件，是自动发现的主要入口 |
| `disable-model-invocation` | 仅在手动调用模式使用 | 标识该 Skill 不参与模型自动发现 |
| 平台专用字段 | 仅在运行时要求时使用 | 提供界面或分发元数据 |

正文使用 Markdown，并以祈使句或动词原形表达指令，例如“读取”“比较”“生成”“验证”。正文定义 Skill 触发之后的执行过程；它不能决定自己是否被加载。

## 设计目标

Skill 的目标是让 Agent 重复采用正确过程，而不是强制生成完全相同的输出。设计时应明确以下内容：

- 任务范围：Skill 要解决的具体问题，以及不负责的相邻问题。
- 触发条件：哪些用户表达、文件格式、工具需求或工作状态应使用它。
- 工作流程：Agent 应按什么顺序做什么。
- 分支条件：哪些情况需要读取额外资料、运行脚本或切换工具。
- 完成标准：怎样确认每一步和整个任务确实完成。
- 可复用资源：哪些规则、代码和资产值得固化。

## 自动调用与手动调用

调用方式标识在 `SKILL.md` 顶部的 YAML Frontmatter 中。Skill 的调用方式决定发现能力和上下文成本。

| 类型 | 适用场景 | 配置方式 |
|---|---|---|
| 自动调用 | Agent 应在识别到特定任务时自行使用，或其他 Skill 需要引用它 | 保留面向模型的 `description`，写明能力与独立触发分支 |
| 手动调用 | 只应由用户明确选择，且不需要被其他 Skill 自动发现 | 使用平台规定的禁用自动调用字段；`description` 改为面向用户的一行摘要 |

自动调用会增加始终存在的元数据上下文，但降低用户记忆 Skill 名称的成本。手动调用没有自动发现成本，却要求用户记住它的存在。

自动调用 Skill 不写 `disable-model-invocation`：

```yaml
---
name: diagnosing-bugs
description: Diagnose hard bugs and performance regressions. Use when the user reports failures, errors, regressions, or slow behavior.
---
```

手动调用 Skill 写入 `disable-model-invocation: true`：

```yaml
---
name: ask-matt
description: Ask which skill or flow fits your situation. A router over the skills in this repo.
disable-model-invocation: true
---
```

以https://github.com/mattpocock/skills 的Skill 为例，Frontmatter 中添加 `disable-model-invocation: true` 的手动调用 Skill 有：

```text
ask-matt
grill-me
grill-with-docs
handoff
implement
improve-codebase-architecture
teach
to-questionnaire
to-spec
to-tickets
wait-what
```

未添加该字段、可被模型自动发现的 Skill 有：

```te
codebase-design
code-review
diagnosing-bugs
domain-modeling
find-skills
grilling
prototype
tdd
writing-for-agents
```

需要 Agent 或其他 Skill 自主发现的能力采用自动调用，只应由用户主动选择的流程采用手动调用。

当手动调用 Skill 数量较多时，可创建一个手动调用的路由 Skill。路由 Skill 只负责说明不同 Skill 的适用场景，帮助用户选择；它不应承担各个子流程的实现。

## `description` 的写法

`description` 是顶层上下文指针。它在 Skill 正文加载前决定 Agent 是否能发现 Skill，因此应同时回答两个问题：

1. Skill 提供什么能力。
2. 哪些互不重复的场景应触发它。

推荐形式：

```yaml
description: Diagnose hard bugs and performance regressions. Use when the user reports failures, errors, regressions, or slow behavior.
```

设计原则：

- 使用任务领域中稳定、常见的关键词。
- 每个独立分支保留一个触发描述；同义词不重复罗列。
- 先写最具辨识度的动作或领域词。
- 不重复正文已有的细节。
- 不把“何时使用”写在正文的 `When to use` 小节中；正文只有在 Skill 已触发后才会被读取。

## 渐进式披露

Skill 的信息应按“何时需要”分为三层：

```text
Frontmatter 的 name 与 description
→ Skill 被发现后加载的 SKILL.md 正文
→ 特定分支才读取的 references/、scripts/ 和 assets/
```

这种设计的重点不只是减少 Token，而是让当前工作流保持清晰。所有分支都需要的步骤放在 `SKILL.md`；只有部分分支需要的资料放到 `references/`，并从正文以明确条件链接。

```markdown
- When processing a scanned PDF, read [references/ocr.md](references/ocr.md).
- When preserving Word tracked changes, read [references/redlining.md](references/redlining.md).
```

引用本身必须说明“是什么”和“何时读取”。只写“参见某文档”会使 Agent 难以判断是否需要加载它。

保持引用扁平：`SKILL.md` 直接链接引用文档，避免多层跳转。较长的引用文档应在开头提供目录；当资料很大时，可在 `SKILL.md` 给出精确的搜索词或章节名。

## Skill 之间的协作

一个 Skill 可以要求 Agent 在条件满足时使用另一个自动调用 Skill：

```markdown
- When the task requires creating or editing raster images, use the image-generation Skill.
- When the task requires tracked changes in a `.docx` file, use the document-editing Skill.
```

这不是传统程序中的函数调用。实际行为是：当前 Skill 提供条件，目标 Skill 的 `description` 提供发现入口，Agent 读取目标 Skill 后按其流程工作。

手动调用 Skill 不能被其他 Skill 自动触发。多个手动调用 Skill 共同需要的资料，应放在 Skill 系统外的普通共享文档中，再由各自的 `SKILL.md` 按条件引用。

## 正文的工作流结构

对多步骤任务，推荐按执行顺序组织正文：

```markdown
# <Skill 名称>

## Preconditions

确认输入、权限、环境和不可变约束。

## Workflow

1. 收集并检查输入。
   Completion criterion: 所有必需输入已存在且格式可读取。

2. 选择处理分支。
   Completion criterion: 已记录选择依据，并加载该分支需要的引用资料。

3. 执行处理。
   Completion criterion: 已生成目标产物或获得可复现的失败结果。

4. 验证并交付。
   Completion criterion: 输出通过目标格式和任务要求的检查。

## Conditional resources

- 当 <条件> 成立时，读取 <引用文件>。
- 当 <条件> 成立时，运行 <脚本>。
```

每个步骤都需要完成标准。高质量完成标准同时具备：

- 可检查：能明确判断完成或未完成。
- 覆盖充分：不会只完成局部。
- 与任务直接相关：验证真实行为或产物，而非“已阅读”“已理解”等主观状态。

例如，“修复问题”不是完成标准；“已运行能复现原始症状的自动化用例，修复前失败、修复后通过”才是完成标准。

## 自由度与约束强度

根据任务风险选择指令的具体程度：

| 自由度 | 使用条件 | 推荐形式 |
|---|---|---|
| 高 | 多种方案都合理，结果依赖上下文判断 | 原则、启发式和决策标准 |
| 中 | 有推荐模式，但允许参数或实现变化 | 伪代码、参数化脚本、分支规则 |
| 低 | 操作脆弱、易出错或必须严格一致 | 明确命令、固定顺序、严格校验门禁 |

不要为了“完整”而把高自由度任务写成僵化步骤，也不要让高风险操作只依赖模糊建议。

## 内容质量原则

### 单一事实来源

每条规则只在一个权威位置维护。`SKILL.md` 保存核心流程，细节规则保存在对应引用文档，环境中的 `package.json`、配置文件、命令帮助和目录结构保持其自身权威。

Skill 应记录环境无法直接表达的信息，例如设计原因、隐含边界、历史兼容性约束和容易遗漏的风险；不应复制一条可即时查询的命令或配置字段。

### 术语和正向表达

为关键行为选择稳定、简短的术语，并在全文一致使用。例如用“紧凑反馈循环”代表快速、确定、聚焦的诊断命令。

优先描述目标行为，而不是反复列举禁止行为。只有不可妥协的安全或正确性边界才使用否定式约束，并同时说明应采用的正向替代行为。

### 精简与相关性

每一行都应改变 Agent 的行为。删除以下内容：

- 模型默认已经会做的通用建议。
- 与当前 Skill 无关的背景介绍。
- 已由配置文件或脚本自描述的信息。
- 仅属于少数分支却占据正文的细节。
- 与其他文件重复的规则。

当 `SKILL.md` 接近 500 行或核心流程难以一眼读清时，应按分支或顺序将细节拆分到引用文档。拆分的目的不是机械控制行数，而是保护主流程的可读性。

## 创建与迭代流程

1. 收集具体使用示例，明确用户会如何表达需求。
2. 划定任务边界，确定自动调用或手动调用方式。
3. 识别可复用内容，分别决定是否需要脚本、引用资料和输出资源。
4. 使用初始化工具创建符合规范的目录和 `SKILL.md` 骨架。
5. 编写 Frontmatter、核心工作流、分支条件和完成标准。
6. 实现并实际运行新增脚本。
7. 运行校验工具，检查 Frontmatter、命名和目录结构。
8. 用真实提示词前向测试，观察触发、分支、输出和失败行为。
9. 根据测试结果删除无效说明、补足缺失边界，并再次验证。

复杂 Skill 的前向测试应使用尽量独立的上下文和真实任务材料，避免向测试 Agent 泄露预期答案或待验证结论。测试目标是验证 Skill 是否能泛化，而不是验证 Agent 是否能复述设计者的意图。

## 发布前检查清单

- [ ] 名称、目录名和 Frontmatter 一致，且使用小写连字符格式。
- [ ] `description` 同时描述能力和独立触发条件。
- [ ] 自动调用与手动调用方式符合实际使用频率。
- [ ] `SKILL.md` 仅包含每个分支都需要的核心流程。
- [ ] 条件引用明确说明读取条件和目标文件。
- [ ] 每个关键步骤都具有可检查、覆盖充分的完成标准。
- [ ] 规则没有在多个文件中重复维护。
- [ ] 资源目录只包含直接支持任务的文件。
- [ ] 新增脚本已实际运行并验证输入、输出和失败行为。
- [ ] Skill 已通过结构校验，并至少使用真实任务进行过一次测试。
