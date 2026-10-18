# `architect`

> 源码：[`skills/architect/SKILL.md`](../../skills/architect/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

在写代码之前，先把类型、签名和模块边界画成带 `not implemented` 的草图，用 `arena` 在至少两个结构上不同的候选之间综合出目标形状，然后按这张草图填实现；填不动就把草图扔掉重画。

## 它解决什么

直接开始写，会锁定第一个能编译的形状。类型和边界一旦被调用者使用，返工成本随调用点增长；更常见的情况是形状本身不对，但每个局部补丁都还能编译，于是错误被一层层吸收进实现。

`architect` 把"决定形状"和"填充实现"分成两个阶段，并强制在综合前看到至少两个结构上不同的候选。原因很直接：第一个看起来够用的设计会锚定后续所有判断，包括评审者的判断。

## 核心机制

### 1. 五个阶段，开工前先建 todolist

Ground、Sketch、Agree、Implement、Scrap，每个阶段一条 todo。顺序是固定的，Scrap 是回到 Sketch 的循环，不是失败退出。

### 2. Phase A：Ground

对每个会被碰到的系统建立真实心智模型，跑 `how`。"说出一个文件名"不算 grounding，要产出 `how` 规定的那种架构解释。如果这次设计会重定义所有权或分层，再跑 `why`，把现有形状的理由变成设计约束，而不是让它停留在猜测。

只有真正 greenfield、没有周边系统要集成时，才允许跳过 Phase A。

### 3. Phase B：Sketch

用 design-sketch 任务加 Phase A 的 grounding 产物跑 `arena`，把 `references/runner-prompt.md` 作为每个 runner 的 prompt 传进去。runner 取自 pstack 设置文件的 `architect runners` 行（替代 `arena runners` 行）；文件或该行缺失时回退到 `claude-opus-5-5-xhigh` 和 `grok-4.7-xhigh-fast`。alias 与被拒条目的处理沿用 `arena` 的 Phase A 规则。

硬要求是至少两个结构上不同的候选，即使第一个看起来已经够用。整形状的替代，不是同一形状内的点修。源码把这称为 `exhaust-the-design-space` 的具体化。

综合前用 [`references/design-red-flags.md`](../../skills/architect/references/design-red-flags.md) 逐条筛候选。筛选时假设下一个贡献者是 agent：它只看自己打开的文件、抄最近的例子、走能编译的最短路径。所以要优先选那种"从单个文件看是对的改动，对整仓也是对的"设计。

可行候选按 interface depth 比较：用更小、更简单的公开面藏住更多复杂度。富接口也可以是好设计，前提是它把能力集中起来、让调用链保持短，而不是把复杂度摊到多层。

`arena` 返回一个综合后的 design package，它的 synthesis decision 填进 rationale 的 "Synthesis decision" 一节。

### 4. Phase C：Agree，默认关

默认直接进入实现，不设人工检查点。只有 invoker 明确要求（"/architect with checkpoint"、"stop and show me before implementing" 之类）才展示综合设计并等待签字。

综合结果无论哪种模式都可以作为独立 commit 落地，即 `foundational-thinking` 的 scaffold first 模式；填实现期间计划内、有范围的破坏是允许的，按 `outcome-oriented-execution`。想在实现前对设计施压，就跑 `interrogate`。人如果反对这个形状（无论是在检查点还是事后），把它当作 Phase A 的证据，重新 grounding 并重跑 Phase B，再写更多代码。

### 5. Phase D：Implement

把 `not implemented` 的 body 换成代码，把伪代码换成逻辑。草图是契约，偏离草图是要浮出来的信号，不是可以默默吸收的摩擦。如果一个函数需要草图没预料到的参数，就问三个问题：草图错了、需求漏了、还是实现越界了。

### 6. Phase E：Scrap

实现持续产生草图吸收不了的摩擦时，扔掉草图，不在错设计上打补丁（`redesign-from-first-principles`、`fix-root-causes`）。信号是模式而不是单个实例：不同代码里反复出现同形状的 workaround；多个无关 edge case 都需要特判；类型需要逃生舱（`any`、cast、实际总会被设置的 optional 字段）才能编译；草图说状态不共享，却出现"我们需要一把锁"的反射；调用者必须知道抽象的内部规则才能用；同一形状的 Phase D 偏离出现两次以上。

判断力仍然重要：几个 edge case 不否定一个架构，数据复杂也不等于设计复杂。真要重画时，先重跑 `how` 看已经建成的东西，把新约束当作 day-one 假设重新设计，先减后加（新草图应该先比旧的小），再回 Phase B 重跑 `arena`。

### 7. 产出

先写调用者的 usage，再由它推导类型草图。小改动一个文件，放新类型和签名；大改动是模块图加类型定义。rationale 按 [`references/rationale-template.md`](../../skills/architect/references/rationale-template.md) 随行，包含 usage 草图与 synthesis decision。

## 设计意图

- **形状先于填充。** `not implemented` 加伪代码给"还没想清楚"一个明确状态，让评审对象从代码变成契约。
- **强制两个候选对抗锚定。** 也把"设计空间"从口号变成可检查的产物。
- **按 interface depth 而不是实现简单度选型。** 维护者（很可能是 agent）读的是接口，不是实现。
- **red flags 的读者模型是 agent。** shallow module、information leakage、temporal decomposition、pass-through method、split ownership、two ways to do one task、importable internals、hand-synced list，八条几乎都是"agent 会怎样抄错"的清单。
- **Scrap 是一等阶段且带阈值。** 给"扔掉重来"一个可触发的条件，抵消沉没成本；pattern 门槛防止一遇摩擦就重写。
- **usage first。** 调用者的体验是规格，类型服务于它；这也让 rationale 可评审。

## 依赖关系

- **上游**：`how` 负责 Ground，`why` 在重定义所有权或分层时补理由，`arena` 负责候选生成、跨模型裁判与综合，设置文件的 `architect runners` 行，`poteto-mode` 的 Subagents 规则。
- **下游**：实现阶段；`interrogate` 可对综合草图施压；`poteto-mode` 的 `feature`、`bug-fix`、`refactoring`、`investigation`、`perf-issue`、`prototype` playbook 在非平凡形状处调用它。
- **共享**：`references/design-red-flags.md`、`references/runner-prompt.md`、`references/rationale-template.md`；`exhaust-the-design-space`、`foundational-thinking`、`outcome-oriented-execution`、`redesign-from-first-principles`、`fix-root-causes`、`subtract-before-you-add`、`boundary-discipline` 等 principle skill。
- **对照**：`how` 解释现有形状，`architect` 画目标形状。

## 使用边界

- **不适用于一行修复或形状已经确定的工作。** 成本远高于收益。
- **greenfield 可以跳过 Ground，但 Sketch 仍要两个候选。**
- **没有 `arena` 或多模型能力时会退化。** "至少两个结构上不同的候选"这条硬要求落空，源码没有给单模型替代流程。
- **默认没有人工检查点。** 人通常在实现已经开始后才看到形状，反对的代价是重跑 Phase A 和 B。
- **它不保证实现正确。** 草图是契约，正确性仍要另外验证。

## 值得注意

- **成本前置且高。** `how`、`arena` 的多 runner 候选生成、综合、rationale，可能还有 `why` 和 `interrogate`，全部发生在第一行实现代码之前。
- **"至少两个"只是下限，而且由配置决定。** `architect runners` 是一个列表，一个条目一个 subagent，列表长度就是候选数（`setup-pstack` 对 panel 角色的规则）；源码没有给上限或停止规则，`arena` 自己的收敛与发散处理（收敛就发共识形状，发散就重框）在 `architect` 这一层也没有被重述。
- **checkpoint 默认关闭与"人拥有形状决定权"有张力。** 源码用"反对即 Phase A 证据"来补偿，但补偿发生在返工之后，而不是之前。
- **`architect runners` 替代 `arena runners` 是一个细粒度耦合。** 设置文件只写了 `arena runners` 时，`architect` 用自己那两个默认 slug，而不是复用 `arena` 的配置；两处配置需要分别维护。
- **Scrap 的触发门槛靠判断。** 源码给了六条 tells 和一句"几个 edge case 不否定架构"，但"持续产生摩擦"到什么程度算持续，没有量化。

## 关键洞察

1. **设计的第一个产物是 usage，不是类型。** 类型从调用者的体验推导出来。
2. **强制多个结构不同的候选是对抗锚定的机制，不是流程装饰。**
3. **选型标准是 interface depth。** 公开面越小、藏得越多越好。
4. **为"下一个 agent 会抄最近的例子"而设计**，是当前环境下最现实的架构约束。
5. **重画需要触发条件。** pattern 而非单例；先减后加，新草图先比旧的小。
