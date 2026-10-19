# `why`

> 源码：[`skills/why/SKILL.md`](../../skills/why/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

把"这段代码为什么长这样"变成一个带引用的证据阅读：先建代码锚点，再按有匹配 MCP 的类别各派一个 investigator 并行去找（source control 始终有），最后由一个 synthesizer 按五级置信度合成，并明确写出不知道的部分。

## 它解决什么

代码只说明它做什么，不说明它为什么存在。原因散在 commit、PR、ticket、设计文档、聊天记录、监控面板、错误追踪和数仓里，每一处都不完整、有偏、有时根本不存在。没有方法时，agent 会读一遍代码，给它反推一个干净的理由；或者顺着用户问题里嵌的假设确认下去。两种都产出"听起来很对"的答案，而用户会据此行动。

`why` 的做法是先把证据面铺开成覆盖图，再把"收集"和"推理"分给不同的角色，最后用可检查的置信度语言交付。

## 核心机制

### 1. 先建代码锚点

目标通常是一段代码、一个模式、一个 feature 或一个被命名的决定。目标含混时，先从对话上下文猜一个（打开的文件、最近编辑、刚讨论的内容），简短说明你的理解再继续，让用户能纠正。

在派任何 investigator 之前，先内联建好种子上下文：文件路径与行范围、关键符号、最近几次 commit、从 merge commit 标题里的 `(#1234)` 提取的 PR 号、以及 commit 或 PR 里出现的 ticket ID。命令包括 `git blame -L`、`git log --follow -p`、`git log --oneline -20`、`git log -1 --format=%B`，以及 `gh pr view <number> --json title,body,author,...` 拉 PR 正文和讨论。

锚点同时发给每个 investigator 和 synthesizer。它让所有并行分支讨论同一段代码，而不是各自猜一个目标。

### 2. 发现可用的 MCP，映射到七类证据

七类是：source control、issue/ticket tracker、long-form documents、real-time team chat、infrastructure observability、error tracking、product analytics warehouse。source control 通过 git 和 `gh` 永远可用，其余六类靠 harness 里连着的 MCP。

分类依据是 MCP 名字、server instructions、工具名和资源描述；一个 MCP 能落进两类时按主要证据选，并把模糊情况记进覆盖图。

### 3. 每个类别一个 investigator，同一条消息派出

有对应 MCP 的类别各派一个，全部并行，每个 agent 只拥有一个工具或 MCP——不要求一个 agent 覆盖多个 MCP。prompt 由 `references/investigator-prompt.md` 填充，再加上该类别在 `references/sources/<source>.md` 里的 playbook；如果目标代码看起来是防御性的（null check、retry、timeout、rate limit、feature flag、egress guard、OOM handler），再加 `incident-postmortem.md`。

investigator 的纪律是这份 skill 的核心：只在自己的源里深挖，先撒网再收窄，读全文而不是标题，只在源内跟链接，跨源线索写进 `Additional Leads`，逐字引用并给出 PR 号、ticket ID、URL、commit hash 或 `file:line`，记录搜过什么（查询原文），空搜索也是发现，遇到矛盾两条都记，最后只收集不下结论。

### 4. 跳过要留书面理由

跳过某个类别只有两个合法原因：这个环境没有对应 MCP（记为 gap，不是选择）；或可证明该源不相关（门槛很高，例如目标是构建期脚本、没有运行时代码路径）。理由要写进最终输出的 `Sources Consulted`。

只有确认七类搜索全部冗余时，才允许 inline 回答；源码说这种情况应该很少。

### 5. 一个 synthesizer 合成

synthesizer 拿到全部发现（包括空结果和被跳过的类别）、代码锚点、原始问题、`references/epistemics.md` 和 `references/synthesizer-prompt.md`。它自己权衡证据、合并重叠引用、呈现矛盾、给每条结论定级，并抽查可疑引用。父 agent 只可轻度编辑，不可改写置信度措辞。

### 6. 五级置信度决定措辞和小节

Direct 是有作者明说的文本证据；Supported 是多条间接证据汇聚；Inferred 是合理但无明说的解读；Speculative 是薄证据上的假设；Unknown 是查过但查不到。Direct/Supported 必须带引用，Inferred/Speculative 必须用 hedge，代码不能作为自身意图的证据。

输出结构是 the Question、the Code in Question、What We Found、What We Can Reasonably Infer、Competing Hypotheses、What We Don't Know、Sources Consulted、Confidence Summary，置信度分层不能压平。

### 7. 为改动铺路时多给一份约束集

如果这个 `why` 是在为改这段代码做准备，`Sources Consulted` 之后要把谱系发现转成 Preserve / Change / Avoid / Risk 四类约束，供后续规划使用。

## 设计意图

- **覆盖图优先于最小答案。** "Document the null" 意味着搜过且没有，本身就是要交付的信息。
- **收集与推理分离。** investigator 不为叙事挑选证据，synthesizer 不为结论伪造证据；两边各自被 prompt 约束。
- **置信度分层就是产品。** 用户会拿这份阅读去做决定，把猜测写成结论比承认不知道伤害更大。
- **反谄媚。** 用户问题里的假设只是一个候选，必须独立核对。
- **反合理化。** 不因为代码"看起来合理"就倒推作者意图；不把一致的模式当成有意为之，也不把没有证据当成没有顾虑。

## 依赖关系

- **上游**：`poteto-mode` 的 `investigation` 与 `bug-fix` playbook 在需要动机时调用；`teach` 把它和 `how` 组合成一段解释。
- **下游**：`recall` 复用它的 source investigator 与 per-source playbook，把问题从"为什么这样建"改成"当前状态、试过什么、用户还在报什么"；`blast-radius` 用它的 Step 2 拉 PR 和 commit；`architect` 在重定义所有权或分层时用它把现有形状的理由变成约束。
- **共享**：MCP 服务器、`gh`、pstack 设置文件里的 `why investigators` 与 `why synthesizer` 行、`poteto-mode` 的 Subagents 规则（推理档位、快慢、模型）。
- **对照**：`how` 回答机制，`why` 回答动机。两者可并行，但不能互相替代。

## 使用边界

- **没有 MCP 时覆盖会塌缩。** 只剩 source control 一类，答案的广度与证据组合会完全不同；`Sources Consulted` 会诚实记录，但记录不等于补齐。
- **它慢。** 默认是完整并行调查加一个 synthesizer；`teach` 因此建议默认收窄 `why`。把问题问小是最有效的加速手段。
- **它不产出改动，也不替用户裁决。** 证据冲突时两边都摆出来，由用户判断。
- **纯运行时行为用 `how`。** "这个函数现在做什么"不是它的问题。
- **历史可能真的没有留下原因。** 合并方式、外部约束、口头决定都会造成空洞；Unknown 是合法结果。

## 值得注意

- **这里故意用 `readonly: false`。** 源码明确说 readonly/Ask 模式会剥掉 MCP，从而关掉 MCP-backed investigator 和 synthesizer 的引用抽查。`how` 走的是相反的 `readonly: true`；源码只给了这个设置，没有解释原因。代价是 investigator 的"不应该写任何东西"只由 prompt 约束，没有工具层强制。
- **同一问题在不同环境里得到不同深度的答案。** 接不接 Linear、Slack、Datadog 决定覆盖图长什么样；skill 记录差异，但不消除差异。
- **成本高且默认全量。** 最多 7 个 investigator 加 1 个 synthesizer，synthesizer 的抽查还需要 MCP 可用。这是"完整性优先于延迟"的明确选择。
- **设置文件缺失时回退到两个具体 slug**（`grok-4.7-xhigh-fast`、`claude-opus-5-5-xhigh`）；模型目录不同的 harness 里可能连续降级，源码要求降级时说出来。

## 关键洞察

1. **覆盖图优先于最小答案。** 空结果要写下来，否则读者无法判断是"没有原因"还是"没找"。
2. **收集者与推理者分开。** 让收集者只负责准确，让合成者只负责分层，比让一个模型同时做两件事更抗叙事偏差。
3. **置信度分层让"不知道"可以交付。** 它把诚实的缺口变成可引用的结论。
4. **代码是机制，不是动机。** 这条反合理化规则适用于任何从产物倒推原因的场合。
5. **用户的假设是候选，不是结论。** 反谄媚要写进流程，而不是靠自觉。
