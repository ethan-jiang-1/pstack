# `figure-it-out`

> 源码：[`skills/figure-it-out/SKILL.md`](../../skills/figure-it-out/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

当没有任何更窄的 playbook 匹配时，先设计一份一次性的 playbook：可证伪的完成谓词、按风险排序的原子单元、先于功能建好的验证脚手架、一轮一轮的科学循环，以及一份人离开后还能审计的 decision trail。流程本身就是第一个交付物。

## 它解决什么

大迁移、多部分的大改动、以及人走开一阵再回来看的活，最容易在第一步就歪掉：还没有流程就开始写代码，流程在执行中临时决定。这样做的代价不是慢，而是无法回答"做到哪了、凭什么说做完了、中间为什么这样选"。

`figure-it-out` 把流程设计提到代码之前。它不提供现成 playbook，而是要求为这次任务设计一个，并把 rigor 显式化：哪些地方要 gate、哪些地方要 artifact、哪里可以便宜地试错。

## 核心机制

### 1. 先读原则，再列阶段

开始时的 todolist 第一项不是任务步骤，是读 `poteto-mode` 的 **Principles** 一节；之后才把下面五个 Phase 加成 todo。这个顺序有含义：rigor 的词汇来自 principle 层，先有词汇再设计流程，才说得清每个 gate 是为哪一类判断服务的。

### 2. Frame：先接地，再承诺

在跑之前必须能说出三件事：

- **完成定义**：一个可证伪的谓词，按 `prove-it-works` 的要求写。
- **量化范围**：粗略的单元数和工作量，加上接地过程中冒出来的 blocker。
- **rigor 等级**：偏高一档。one-way door 和高 blast radius 的工作更多，可逆的低风险步骤更少。源码强调 rigor 是 gate 和 artifact，不是"更努力"。

然后把 framing 和 tradeoff 摆出来再承诺长跑。可逆工作直接做（`never-block-on-the-human`），但多小时的长跑值得一个 checkpoint。

### 3. Design：拆单元，排序，先建验证

拆成原子的、可独立落地的单元，按"最未知的风险最先"排序。脚手架和验证在功能之前（`foundational-thinking`）。

验证 harness 要在工作之前建好，并且 baseline 从改动前的状态捕获，这样每个检查读起来都是"旧值 vs 新值"。

对 one-way door 的设计决定跑 `architect`（它内部跑 `arena`）；形状已经具体、纯机械的工作跳过这步。对已经定下来的设计再开一轮 arena 是 over-engineering（`laziness-protocol`）。

决定哪些能并行：只在 seam 上 fan out，每个 worker 自己的 worktree 或分支（`separate-before-serializing-shared-state`），不要过度扇出。

把设计好的 phase list 写下来，那份列表就是人评审的东西。然后才执行：把执行步骤作为具体 todo 插在 Phase C 和 Phase D 之间，按 C 的循环纪律跑，并把 D 的日志一路织进去，每落一步写一行，而不是留到最后补。

### 4. Run the loop：每个单元是一个实验

写 hypothesis，做最小改动，在真实 artifact 上按谓词测量，前进就保留，没前进就 revert。按 `sequence-verifiable-units`，每个单元验证完再开下一个，不要批量检查。

验证靠检查 artifact，不靠自报。某件事通过得太容易时，先怀疑观测方法，再怀疑系统。

委派出去的工作要配一个 judge。worker 钻了 gate 的空子，就 reset 并把契约加硬；gate 本身错了，就单独用一个 change 修 gate，而不是绕开它。

verdict 只有三种：`VERIFIED`、`NOT VERIFIED`、`INCONCLUSIVE`。inconclusive 不是 pass，负面的结果不许藏。

### 5. Keep the audit trail

用 `show-me-your-work` 记录这次运行。figure-it-out 的工作通常足够大，值得把轨迹提交，让 reviewer 在 PR 里读。轨迹加 diff，才是人回来之后能信任这份工作的东西。

### 6. Verify and hand back

在真实产物上按 Phase A 的谓词检查整体，不只检查 harness。把反复出现的纠正编码成 gate、lint 规则、检查或脚本（`encode-lessons-in-structure`）。

回复包含：设计的 playbook、rigor 等级及理由、decision trail 路径、哪些已按谓词验证、哪些还开着。

## 设计意图

- **没有匹配的 playbook 时，流程就是产物。** 先设计再执行，才能让 rigor 成为结构而不是事后叙述。
- **rigor 是 gate 和 artifact。** 这句话排除了"我会更小心"这类无法检查的承诺。
- **风险前置。** 最可能推翻计划的未知最早验证，避免在错误的形状上做完大半。
- **baseline 先于改动。** 没有旧值，"变好了"就没有参照，只能靠记忆。
- **轨迹织进执行。** 事后补写的日志是叙述，不是记录；每落一步写一行，行才对应真实时刻。
- **一个 checkpoint 而不是逐步审批。** 它同时满足"长跑前让人看一眼"和"可逆工作不要停下来问"。

## 依赖关系

- **上游**：`poteto-mode` 在没有更窄 playbook 匹配、或工作很大/跨切面、或用户走开后再看时路由到这里；它明说 `figure-it-out` 设计一次 bespoke run，`orchestrate` 运行长期 program。`refactoring` 把大型结构改动路由过来。
- **下游**：`architect`（one-way door 设计，内部再调 `arena`）；`show-me-your-work`（Phase D 的轨迹）。
- **共享**：todolist 机制；`poteto-mode` 的 **Subagents** 规则；principle 词汇 `prove-it-works`、`foundational-thinking`、`laziness-protocol`、`never-block-on-the-human`、`separate-before-serializing-shared-state`、`sequence-verifiable-units`、`encode-lessons-in-structure`。
- **对照**：`orchestrate` 是项目级 program，有队列、frontier 和协调者；`figure-it-out` 只设计一次运行。`autonomous-run` 是把一个任务驱到谓词，流程相对固定。

## 使用边界

- **有匹配的窄 playbook 时，默认不要用。** 但路由层有一个例外：`poteto-mode` 对大型或跨切面的工作，即使 Feature 这类 playbook 能匹配，也会路由到 `figure-it-out`。skill 自己的触发条件是"没有任何更窄的 playbook 匹配"，两处并不完全一致，实际判断落在 mode 的路由规则上。
- **谓词写不出来时，问题在更前面。** 它要求 Phase A 就能陈述完成定义；做不到说明目标还没定，应该先解决目标，而不是硬走流程。
- **它不是审批流。** 只有一个 checkpoint，不是每一步都等人。需要逐步监督的场景应该换模式。
- **执行纪律靠自律。** 双层待办、每步一行的日志、verdict 三态，都没有工具强制。

## 值得注意

- **"rigor 偏高"与 `laziness-protocol` 直接对冲。** 源码只在一个具体点上划界（对已定设计再开 arena 是 over-engineering），其余地方的 rigor 上限没有量化，靠执行者判断。
- **双层待办没有模板。** Phase B 要求把执行步骤插进 Phase C 与 Phase D 之间，并让 D 的日志穿过它们；这依赖"设计的 phase"与"执行的 step"两层结构同时存在。长跑里它容易退化成一个扁平长列表，phase 边界消失。
- **checkpoint 的阈值是经验值。** "multi-hour run earns one" 没有说更长的运行是否需要更多检查点；一个跨天的运行只设一个停点，风险仍然落在执行者身上。
- **验证能力要自建。** 谓词、harness、baseline 都由这次运行自己产出；流程本身不提供测量工具，成本被低估时 rigor 会退化成形式。

## 关键洞察

1. **没有匹配的 playbook 时，第一个产物是流程，不是代码。**
2. **rigor 要能被检查。** gate、artifact、baseline、三态 verdict 都是结构，不是态度。
3. **风险排序比任务排序重要。** 最未知的部分最先做，计划的寿命才长。
4. **轨迹不能事后补。** 决策发生时写一行，审计才有原始材料。
5. **人离开后能信任，来自三件套：谓词、轨迹、diff。** 缺任何一件，回来都只能靠讲述。
