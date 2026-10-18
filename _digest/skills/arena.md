# `arena`

> 源码：[`skills/arena/SKILL.md`](../../skills/arena/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

同一个 brief 并行跑 N 个候选，让一个跨模型裁判按事先写好的 rubric 逐项打分，父 agent 自己读完全部候选再选 base，然后把落选者最强的部分手工移植进去。产物是一个合成物，外加一份说清取舍的 synthesis note。

## 它解决什么

非平凡产物的第一次尝试会锁定形状。形状一旦定下，评审只能挑毛病，不能给出替代形状；于是"这个设计对不对"退化成"这个设计有没有明显 bug"。

arena 让形状本身成为可比较的对象：多个模型从同一个任务出发，各自走出一条完整路线，再由父 agent 和一个跨模型裁判对照，选一条继续。它适合设计草图、代码 bakeoff，以及任何"一次尝试会锁死后续结构"的活。它不负责覆盖，也不负责挑错。

## 核心机制

### 1. Frame 把 prompt 变成契约

所有候选拿同一个 prompt，所以 prompt 是唯一的共同前提。Frame 做四件事：写清产物是什么；从任务里推导 3-6 条可评分的 rubric；按设置文件的 `arena runners` 行选模型；给每个候选独立输出位置（能用 git worktree 就用，否则 `/tmp/arena-<slug>/candidate-<n>/`）。

rubric 只给 Phase D 的 picker 和 Phase C 的裁判看，候选只看任务。多设计方向可以多派 runner；工作如果是生成受限而不是判断受限，同一个模型跑 N 次也可以。

### 2. Fan out 一次派完

一条消息、`run_in_background: true`，每个候选拿到任务、共享 grounding 的路径、自己的输出路径，以及"产出产物 + 一段 rationale，说明考虑过哪些替代方案、否决了什么"。候选没产出就按 N-1 继续，并记录掉线。

### 3. Cross-judge 是裁判，不是决策者

从 `arena cross-judge pool` 里选一个模型，优先与父模型不同家族，派一个 readonly 的裁判。它看到 rubric 和按路径标签给出的候选，逐条打分，推荐 base 并附理由。它与父 agent 在 Phase D 的阅读并行，但不能在候选还在写的时候派出。

### 4. Pick 由父 agent 自己做

父 agent 必须从头到尾读完每个候选，按 rubric 逐条打分，不能凭整体感觉。然后与裁判对照：一致就确认；不一致时源码的判断是"其中一方有偏差，或者 rubric 本身有歧义"，要求读完两边理由再决定。

选择标准是可扩展性：未来维护者能在不破坏不变量的前提下继续改的那一版。打平时偏向更干净的边界或更小的 API。

### 5. Graft 是手工活

落选者每份通常只值得搬一两样东西。搬的时候手工折进 base，不做机械粘贴，目的是让结果仍然在一个心智模型下连贯。

候选收敛到同一形状时，直接采用共识、不做 graft；分歧过大时，源码判定 Phase A 欠规格，要求重框重跑，而不是取平均。

### 6. Verify 与 synthesis note

合成产物按 `prove-it-works` 的方式验证。验证发现 arena 漏掉的问题时，要么回 Phase A 重框（说明 rubric 或任务本身错了），要么回 Phase E 补 graft（说明有候选看到了但没搬进来）。

Outputs 要求一份 synthesis note：base 及理由、裁判结论、每处 graft 的来源候选、否决了什么及为什么、收敛与掉线情况、验证结果。

## 设计意图

- **形状要能竞争。** 评审是对既有文本的判断，候选是对问题的独立解；前者产生不了后者。
- **同 prompt 是控制变量。** 差异才能归因于模型，而不是归因于任务描述。
- **rubric 与生成分离。** 源码只规定谁看得到 rubric；效果是候选不会针对评分标准写作，picker 拿到的是一把独立的尺子。
- **手工 graft 保连贯。** 自动合并会把两个心智模型拼在一起；手工搬一两样，合成物还能被一个人讲清楚。
- **分歧要重框，不要平均。** 平均是委员会设计，会同时失去两个候选各自的理由。

## 依赖关系

- **上游**：`architect` 的 Phase B 直接调用 arena，传入设计草图任务和 Phase A 的 grounding，并用 `architect runners` 替换 `arena runners`、以 `references/runner-prompt.md` 作为 runner 的 prompt。`figure-it-out` 在 one-way-door 设计决定上跑 `architect`，因此间接走这条路。`poteto-mode` 的路由也说设计或代码 bakeoff 用 arena。
- **下游**：`architect` 拿 arena 的 synthesis decision 填自己 rationale 的 "Synthesis decision" 一节，再在 Phase D 按草图实现。
- **共享**：设置文件的 `arena runners`、`arena cross-judge pool` 两行；`poteto-mode` 的 **Subagents** 规则（档位、后台、fresh）；principle 词汇 `separate-before-serializing-shared-state`、`redesign-from-first-principles`、`prove-it-works`、`laziness-protocol`。
- **对照**：`swarm` 也是并行 fan-out，但它要覆盖或竞速，收敛规则是事先声明的 `first pass` / `rank all` / `best-of`，没有 base 选择与 graft。`interrogate` 评审已存在的改动，不产生候选。

## 使用边界

- **不要用它做覆盖。** "检查每个 package"这类扇出是 `swarm`；用 arena 会为每一份跑一轮裁判，成本翻倍而没有合成收益。
- **不要用它替代评审。** 已有改动要挑错用 `interrogate`；arena 的前提是还没有一个要评审的产物。
- **形状已定的机械活不要开 arena。** `figure-it-out` 明说对已定设计再开 arena 是 over-engineering。
- **候选必须能独立产出完整产物。** 只能串行共享状态的任务，隔离成本高于收益。

## 值得注意

- **裁判与父 agent 信息不对称。** 裁判只看候选文本和 rubric，父 agent 有对话上下文。Phase D 把分歧解释成"一方有偏差或 rubric 有歧义"，但还有第三种可能：两边看到的信息不同。源码没有给这种情形的处理规则。
- **N 没有上限。** 源码只说覆盖多个设计方向时可以多派；每个候选一个输出位置，再加一个裁判，成本线性增长，源码没有给数量上限。
- **graft 的"一两样"是经验值。** 没有可检查的标准区分"值得搬的洞察"和"顺手也搬过来的实现细节"。
- **面板配置带着 harness 补丁。** `composer-*` 被归进 `grok-*` 家族、被拒后跑 `grok-4.7-xhigh-fast`，这类规则说明默认 slug 在真实 harness 里会漂移，多模型假设需要持续维护。

## 关键洞察

1. **生成与判断是两种能力，分开配置。** 便宜模型并行生成，强模型做裁判，父 agent 保留最终判断。
2. **同 prompt 是控制变量。** 候选之间唯一的差异应当来自模型，而不是任务描述。
3. **合成不是合并。** 选一个 base 再手工移植，比把几份产物拼在一起更容易保持连贯。
4. **分歧是规格问题。** 候选差异过大时，先怀疑 Frame 没说清，而不是怪候选。
5. **判断权不外包。** 裁判给建议，父 agent 必须自己读完所有候选。
