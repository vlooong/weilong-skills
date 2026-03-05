---
name: thesis-advisor-review
description: 学位论文逐章逐节导师式严谨审核与反复打磨。用于逐节审核、提出可执行修改建议、与作者反复讨论直至通过后再进入下一节。Triggers on "论文审核", "导师审核", "逐节修改", "毕业论文修改", "thesis review", "advisor review".
allowed-tools: Read, Write, AskUserQuestion, Task, Glob, Grep, Bash
---

# Thesis Advisor Review

面向**硕士学位论文**的“导师角色”审核技能：以**按节推进**为默认节奏，对每一节进行结构、论证、实验/方法严谨性与表达质量的细致审阅；在你确认“本节已通过”之前，会持续迭代打磨。

## Architecture Overview

```
Phase 0: Specification Study (Must read specs/ before using)
        ↓
┌──────────────────────────────────────────────┐
│ Orchestrator (State-driven, section-by-section)│
└───────────────┬──────────────────────────────┘
                │
     ┌──────────┼───────────┬───────────┬────────────┐
     ↓          ↓           ↓           ↓            ↓
  Init      Load Doc   Select Next   Review Section  Gate & Advance
(action)   (action)     (action)       (action)       (action)
                │
                ↓
         Iterate revisions until PASS
```

## Key Design Principles

1. **按节门控**：每一节必须达到“通过”门槛才进入下一节；不通过则继续迭代。
2. **导师批注式输出**：以“问题 → 原因 → 修改建议 →（可选）示例改写”为主，直接可执行。
3. **证据链闭合**：任何主张必须对应方法、实验或引用；不接受“空泛结论”。
4. **最小不必要输出**：对话中只输出当前节的审核与下一步请求；历史与状态写入工作目录。

---

## Mandatory Prerequisites

> **Do NOT skip**：在开始使用前，必须完整阅读以下规范文件；否则 orchestration 和门控会失效或输出不一致。

### Specification Documents (Required Reading)

| Document | Purpose | When |
|----------|---------|------|
| [specs/thesis-review-requirements.md](specs/thesis-review-requirements.md) | 领域要求：硕士学位论文审核维度、硬性门槛、输入约定 | **Must read before execution** |
| [specs/quality-standards.md](specs/quality-standards.md) | 质量标准：通过/不通过判据、问题分级、修改优先级 | Must read before execution |
| [specs/output-style.md](specs/output-style.md) | 输出风格：导师批注式模板与措辞约束 | Must read before first review |
| [specs/state-schema.md](specs/state-schema.md) | 状态机字段：章节/小节进度、迭代轮次、通过记录 | Must read before orchestrator edits |

### Template Files (Must read before generation)

| Document | Purpose |
|----------|---------|
| [templates/section-review-template.md](templates/section-review-template.md) | 单节审核输出外壳（问题-原因-建议-改写） |
| [templates/gate-decision-template.md](templates/gate-decision-template.md) | 通过门控输出外壳（Pass/Revise/Block） |

---

## Execution Flow

### Phase 0: Specification Study
→ **Refer to**: [specs/thesis-review-requirements.md](specs/thesis-review-requirements.md), [specs/quality-standards.md](specs/quality-standards.md)

- 阅读规范、确认输入形态（单个 Markdown）与推进粒度（按节）。

### Orchestrator Loop (Autonomous)

1. **Init**：建立工作目录与 state.json
   → **Refer to**: [specs/state-schema.md](specs/state-schema.md)
2. **Load Document**：读取 Markdown，解析章节/小节结构，生成 section 索引
   → **Refer to**: [specs/thesis-review-requirements.md](specs/thesis-review-requirements.md)
3. **Select Next Section**：根据 state 选择下一个待审核小节
   → **Refer to**: [specs/state-schema.md](specs/state-schema.md)
4. **Review Section**：输出导师批注（问题/原因/建议/改写）并提出本轮你需要提供的材料
   → **Refer to**: [specs/output-style.md](specs/output-style.md), [templates/section-review-template.md](templates/section-review-template.md)
5. **Gate & Advance**：你确认已修改并提交“本节修订稿”后，进行门控判定：PASS / REVISE / BLOCK
   → **Refer to**: [specs/quality-standards.md](specs/quality-standards.md), [templates/gate-decision-template.md](templates/gate-decision-template.md)

## Directory Setup

```javascript
const timestamp = new Date().toISOString().slice(0,19).replace(/[-:T]/g, '');
const workDir = `.workflow/.scratchpad/thesis-advisor-review-${timestamp}`;

Bash(`mkdir -p "${workDir}"`);
Bash(`mkdir -p "${workDir}/sections" "${workDir}/reviews" "${workDir}/revisions"`);
```

## Output Structure

```
.workflow/.scratchpad/thesis-advisor-review-YYYYMMDDhhmmss/
├── state.json                 # 审核状态机
├── input/                     # 输入（复制/索引信息）
├── sections/                  # 解析得到的章节/小节切片（md）
├── reviews/                   # 每轮审核意见（md/json）
├── revisions/                 # 你提交的修订稿记录（按节）
└── final/                     # 可选：阶段性通过后的整合稿
```

## Reference Documents by Phase

### Phase 0: Specification Study
Documents to refer to when preparing and running the skill.

| Document | Purpose | When to Use |
|----------|---------|-------------|
| [specs/thesis-review-requirements.md](specs/thesis-review-requirements.md) | 审核维度与硕士论文硬性门槛 | 在开始审核前对齐“什么算通过” |
| [specs/quality-standards.md](specs/quality-standards.md) | 通过判据与问题分级 | 在争议点上裁决“必须改/建议改/可不改” |
| [specs/state-schema.md](specs/state-schema.md) | 状态字段含义 | 当推进/回退逻辑出错时校对状态机 |

### Phase 1: Init & Document Loading
Documents to refer to when initializing and parsing the Markdown structure.

| Document | Purpose | When to Use |
|----------|---------|-------------|
| [phases/actions/action-init.md](phases/actions/action-init.md) | 初始化 state 与工作目录 | 首次启动或需要重建状态时 |
| [phases/actions/action-load-markdown.md](phases/actions/action-load-markdown.md) | 解析 Markdown 的章节/小节索引 | 更换论文版本或目录结构变化时 |

### Phase 2: Section Review Loop
Documents to refer to during each section review iteration.

| Document | Purpose | When to Use |
|----------|---------|-------------|
| [templates/section-review-template.md](templates/section-review-template.md) | 单节批注输出外壳 | 每次输出导师批注时保持一致格式 |
| [specs/output-style.md](specs/output-style.md) | 语气与格式约束 | 防止输出变成泛泛而谈或过度列表化 |
| [phases/actions/action-review-section.md](phases/actions/action-review-section.md) | 单节审核动作 | 执行逐节审阅与生成待改清单 |

### Phase 3: Gate & Advance
Documents to refer to when deciding pass/revise/block.

| Document | Purpose | When to Use |
|----------|---------|-------------|
| [templates/gate-decision-template.md](templates/gate-decision-template.md) | 门控判定输出外壳 | 给出 PASS/REVISE/BLOCK 结论时 |
| [phases/actions/action-gate-and-advance.md](phases/actions/action-gate-and-advance.md) | 门控与推进动作 | 你提交修订稿后进行判定与推进 |

### Debugging & Troubleshooting

| Issue | Solution Document |
|-------|-------------------|
| 解析不出小节/标题层级异常 | [phases/actions/action-load-markdown.md](phases/actions/action-load-markdown.md) + [specs/thesis-review-requirements.md](specs/thesis-review-requirements.md) |
| 审核输出风格跑偏/过于空泛 | [specs/output-style.md](specs/output-style.md) |
| “通过/不通过”争议 | [specs/quality-standards.md](specs/quality-standards.md) |
| 推进到错误的小节 | [specs/state-schema.md](specs/state-schema.md) |

### Reference & Background

| Document | Purpose | Notes |
|----------|---------|-------|
| [C:/Users/CQU/.claude/skills/_shared/SKILL-DESIGN-SPEC.md](../_shared/SKILL-DESIGN-SPEC.md) | Skill 结构与规范来源 | 仅在扩展/重构 skill 时参考 |
