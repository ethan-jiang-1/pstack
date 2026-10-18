# `how`

> 源码：[`skills/how/SKILL.md`](../../skills/how/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

把一个"X 是怎么工作的"问题，变成一份资深工程师 onboarding 级别的架构说明：简单的直接解释，复杂的先派 2-4 个只读 explorer 分角度探索，再由一个 explainer 合成。全程不写代码。

## 它解决什么

改代码之前，人需要一个能装在脑子里的模型：有哪些组件、控制流怎样走、边界在哪里、哪里反直觉。读源码当然能得到这些，但很容易陷入逐行注释，或者只看到自己熟悉的那一块。

`how` 把"探索"和"解释"分成两个角色：explorer 负责找全并记录证据，explainer 负责组织成可以讲的模型。它也覆盖"这个东西该放在哪一层"这类归属问题。

## 核心机制

### 1. 先判复杂度

- **简单**（单模块、小工具、"函数 X 怎么工作"）：不派 explorer，一个 explainer 边探索边解释。
- **复杂**（跨文件/服务的子系统、横切特性、完整架构概览）：2-4 个 explorer，每人一个角度，同一个消息里全部派出。
- 拿不准时走简单路径。

这条规则防止小题大做：一个小问题不值得四个 subagent。

### 2. Explorer 的输出形状

每个 explorer 拿到 `references/explorer-prompt.md`，按固定小节回报：

- **Components Found**：类型、服务、类、抽象，每个带文件路径和一句职责。
- **Flow**：逐步执行流，谁调用谁、数据怎样流动。
- **Files Read**：读过哪些文件，让 explainer 能回溯。
- **Boundaries**：与外部连接的输入输出。
- **Non-Obvious Things**：看起来应该怎样、实际怎样的地方。
- **Open Questions**：没能追踪或理解的部分，诚实标注。

`Files Read` 和 `Open Questions` 是关键：一个让结论可追溯，一个防止把猜测写成机制。

### 3. 一个 explainer 合成

复杂路径下，等所有 explorer 返回后，再派一个 explainer，把全部发现填进 `references/explainer-prompt.md`。输出小节是：

- Overview
- Key Concepts
- How It Works
- Where Things Live
- Gotchas

不适用的小节可以删。父 agent 只做轻微编辑（补上下文、理清句子），不做大幅改写。

### 4. 角色与模型

- explorer 用 `how explorer` 角色（默认 `grok-4.7-xhigh-fast`）。
- explainer 用 `how explainer` 角色（默认 `claude-opus-5-5-xhigh`）。
- 两者都是 `readonly: true`，读代码但不改代码。

只读是一个设计约束：探索阶段一旦开始改东西，"解释"就会变成"边改边解释"，结论不再中立。

## 设计意图

- **搜索与综合分离。** 让快模型并行找，让强模型组织。两者需要的注意力不同。
- **输出形状先定。** 先规定 explorer 回报什么，再让它探索；否则每个 explorer 会返回不同粒度的东西，explainer 无法合成。
- **只读保护结论。** 不修改代码，才能保证解释描述的是当前状态。
- **诚实标注缺口。** `Open Questions` 把"没查到"变成可见的边界，而不是让 explainer 用推测填满。

## 依赖关系

- **上游**：`poteto-mode` 的 Investigation、Feature、Bug fix、Refactoring 都会先调用 `how`；`teach` 把 `how` 与 `why` 组合起来。
- **下游**：`blast-radius` 在 `how` 的解释之上追跨模块影响；`architect` 在理解形状之后画目标形状。
- **共享**：模型设置文件（两个角色）、`poteto-mode` 的 Subagents 规则（模型档位、后台、fresh）。
- **对照**：`why` 回答动机，`how` 回答机制。两者可以并行，但不要互相替代。

## 使用边界

- **不要用它回答"为什么"**。动机、历史权衡、回归原因属于 `why`。
- **不要用它做评审**。它解释现状，不判断改动是否值得合入；评审用 `interrogate` 或 mode 的 review 路径。
- **不要用它替代动手验证**。解释可能漏掉运行时行为；需要证据时用 `runtime-forensics` 或 control skill。
- **小问题不要开 explorer**。源码里明确说"拿不准走简单路径"。

## 值得注意

- **explainer 是单点。** 所有 explorer 的发现都要经过一个模型组织；explorer 的偏差可能被继承。
- **父 agent 不能大幅改写。** 这让输出更接近 explainer 的判断，但也意味着父 agent 很难纠正一个整体性的误解。
- **依赖模型设置。** 没有设置文件时退回默认 slug；在模型目录不同的 harness 里可能连续降级。
- **只读会剥掉 MCP。** 如果问题需要 issue tracker 或可观测性数据，`how` 本身拿不到；这是 `why` 的领域。

## 关键洞察

1. **并行探索，单点综合。** 这是"广度换速度、综合换一致性"的典型分工。
2. **输出模板就是接口。** 先定义回报格式，才能让多个 explorer 的发现可合并。
3. **只读让解释保持中立。** 一旦允许修改，"怎么工作"就会滑向"我改成什么样"。
4. **诚实标注 open questions。** 一个说明边界的解释，比一个假装完整的解释更有用。
