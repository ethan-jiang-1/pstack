# `interrogate`

> 源码：[`skills/interrogate/SKILL.md`](../../skills/interrogate/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

一个配置模型一个只读 reviewer，同一份 diff、同一段 intent、同一套 rubric，各自独立找问题；父 agent 以 lead reviewer 的身份把发现分进 Act on / Consider / Noted / Dismissed 四个桶，产出一个综合 verdict。它不自动改代码。

## 它解决什么

单模型的评审会和写代码的模型共享同一个盲点：它对代码的理解方式，正是产生这份代码的方式。让作者自评则多一层动机性偏差。

`interrogate` 用模型多样性换独立性，再用一个有完整对话上下文的 lead 来过滤。它同时解决另一个反方向的问题：对抗评审容易把版面填满，把 nitpick 说成风险。四桶和 `Dismissed` 一节就是为此存在的。

## 核心机制

### 1. Scope 与 Intent

先确定评审对象：用户指了文件或 diff 就用它；在 feature branch 上跑 `git diff main...HEAD`；用户提到最近的工作就收集相关文件。然后把它和 reviewer 需要的上下文文件一起打包。

派 reviewer 之前，用一段话写清 intent，来源是用户消息、commit message、PR 描述和代码本身。不确定就先去问用户。reviewer 的模板明确要求"不要质疑 intent，假设目标正确，挑战执行"，所以 intent 错的时候，这套流程帮不上忙。

### 2. 按面板派 reviewer

一条消息里全部派出，按设置文件 `interrogate reviewers` 行的条目数来，标 Reviewer A、B 等。每个 reviewer 是 `generalPurpose`、`readonly: true`，模型取配置值或默认表；`auto` / `inherit-parent` 时省略 `model`，跟随父会话。

prompt 由 `references/reviewer-prompt.md` 填充：intent 段落、diff 或文件内容、`references/rubric.md`、`references/code-quality-review.md`。输出要求是带 severity（`critical` / `warning` / `nit`）、位置、证据、可选建议的结构化发现。找不到问题就写 "no findings"，空评审是合法结果。

### 3. 两套 lens，作用不同

`rubric.md` 是六个可选用视角：正确性、根因与症状、结构完整性、验证、复杂度预算、安全。源码要求"不要强行套用不相关的视角，一个简单 bug fix 不需要架构完整性段落"。

`code-quality-review.md` 是一条恒定的质量底线，语气明显更硬：找"code judo"式重构、把接近 1k 行的文件当成强信号、把 spaghetti 式新增分支当设计问题、要求逻辑回到 canonical layer。它的 **Approval Bar** 说"不要因为行为看起来正确就 approve"，并把若干情况列为 presumptive blocker。

### 4. 综合与 lead 判断

结果回来后先做机械合并：描述同一问题的发现并成一条，记下是哪些模型提出的。两个以上模型独立提出的是最高信号；单个模型的发现要读，但降权；互相矛盾的发现记为上下文。

然后进入 lead 判断。`references/lead-judgment.md` 的前提是：reviewer 只看到代码的一个切片和一段 intent，不知道什么已经试过、有什么外部约束、哪些是临时脚手架、下一个 PR 要做什么。由此派生出过滤原则：nitpick gravity、hypothetical 与 actual 的区别、premature abstraction、"I would have done it differently" 不是 finding、以及识别 reviewer 缺上下文的信号。

反过来，源码也给了"reviewer 是对的"的信号：多个模型独立同指、给出具体执行路径而不是假设、暴露了你心智模型的缺口、或者你读完想"确实如此"。安全与正确性问题即使只有一个模型提出，也要格外谨慎地对待。

### 5. 输出形状

固定七节：Intent、Reviewers（每个模型一行，带发现数）、Act On、Consider、Noted、Dismissed、Agreement Map。每个发现带提出它的模型和一句理由。`Dismissed` 不是凑数，而是信任机制：把否决的发现和理由摆出来，用户才能覆盖你的判断。

lead-judgment 还给了一个校准：Act On 超过 5 条，通常说明过滤不够狠。

## 设计意图

- **对抗信号来自模型多样性，不来自人设。** 源码直说这一点。人设换的是提示词，模型的训练数据和失败模式没变，错误的相关性也不会变。
- **intent 先冻结。** 评审一旦可以质疑目标，就会滑向"如果是我会做另一个功能"。
- **reviewer 是传感器，lead 是判决。** 两者信息量不对称是设计出来的：reviewer 看得窄但独立，lead 看得全但有立场。
- **把发现变成决定。** 四桶强迫每条发现落到一个可执行状态，而不是交一份清单。
- **不自动应用。** 评审产出判断，改动是另一个决定；否则评审意见会绕过实现和验证的门。

## 依赖关系

- **上游**：`architect` 在 Phase C 可选地对合成草图跑 `interrogate`，作为实现前的对抗压力；`opening-a-pr` 要求开 PR 的 subagent 跑 `interrogate`；`feature` 在设计有争议时要求先 interrogate；`poteto-mode` 的路由把"有争议的设计"指向它。
- **下游**：Act On 的条目进入一个修复轮；按 `poteto-mode` 的规则，修复用带完整 brief 的 fresh subagent，而不是续在 reviewer 上。
- **共享**：设置文件的 `interrogate reviewers` 行；`poteto-mode` 的 **Subagents** 规则（档位、后台、fresh）；四个 reference 文件；[`model-diversity`](../patterns/model-diversity.md) 与 [`subagent-topology`](../patterns/subagent-topology.md) 两个模式页描述的通用约束。
- **对照**：`arena` 产生候选并合成，`swarm` 做覆盖或竞速，`interrogate` 只判断一个已经存在的改动。

## 使用边界

- **它不改代码。** 源码开头就写 "Do NOT auto-apply changes"。
- **它不评审目标。** intent 不清时先问用户；让 reviewer 猜意图，只会得到一堆基于错误前提的发现。
- **它不是投票。** 多数票会放大多个模型共有的偏差；共识只是加权信号，单个模型的安全发现仍要认真读。
- **没有子代理工具的 harness 会退化。** 源码要求逐个自己跑 reviewer，此时独立性只剩顺序，不再有模型独立。
- **它适合 diff，不适合抽象计划。** reviewer prompt 的结构就是围绕"code under review"设计的；没有代码时更容易收到理论化风险。

## 值得注意

- **共享 prompt 削弱了独立性。** 所有 reviewer 拿同一个模板、同一份 rubric、同一段 intent；"多样性"只落在模型权重上，模板层面的共同偏差无法靠多模型消除。
- **两套 lens 的优先级没有写死。** code-quality 的 Approval Bar 把 1k 行、spaghetti、缺 abstraction 当 presumptive blocker，而 lead-judgment 要求 dismiss "I would have done it differently"。同一个发现可以同时落进两边，谁赢取决于 lead 的判断。
- **Act On 不超过 5 条是启发式。** 它防止 verdict 变成清单，也可能把第 6 个真实问题压进 Consider。
- **Dismissed 依赖 lead 的上下文，而上下文本身可能错。** 源码把"reviewer 没理解上下文"当作 dismiss 理由，但没有要求 lead 证明自己的上下文是对的。

## 关键洞察

1. **独立性是评审的货币。** 不同模型的错误相关性低，人设不改变这一点。
2. **先冻结 intent，评审才有边界。** 目标是给定的，执行才是被挑战的对象。
3. **分桶把发现变成决定。** 没有桶的评审只是信息，有桶的评审才是判断。
4. **Dismissed 与 Act On 一样是交付物。** 说清否决理由，用户才能覆盖你。
5. **共识加权、分歧记录。** 两者都是证据，投票会把它们都浪费掉。
