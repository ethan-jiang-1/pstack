# 设计与计划：先定形状，再定每个 PR 的证据

> 本章 3 个实战单元 · [手册目录](./README.md) · [上一章：定义](./define.md)

## 这一章解决什么

第 1 章把请求变成了 prompt、playbook 和 brief。这一章处理下一个问题：在写实现之前，代码里该有什么形状，以及这些形状怎样变成一组带证据的 PR。

pstack 没有一份要先写好的架构文档。它的设计动作集中在几处：

1. 先给数据一个形状，按 `principle-model-the-domain` 选它的 organizing structure；
2. 用 `architect` 的 Phase B 草图把 public surface 和 `not implemented` 函数体画出来，用 design-red-flags 筛候选；设计空间开放时用 `arena`；
3. 用 `feature` step 3 的 throughput checkpoint 决定谁先做、哪些能并行；
4. 跨多个 PR 时，用 `multi-phase-plan` 写 plan 文件，一个 PR 是一个变更，带自己的证据。

三个单元对应三种卡点：

| 单元 | 卡点 | 带走 |
|---|---|---|
| 2.1 | 不知道代码里该有什么结构，分支越加越多 | 一个命名的 data shape 和它的 organizing structure |
| 2.2 | 接口一说就锁死，测试只能进内部 | `architect` Phase B 的 public surface 与 `not implemented` 草图 |
| 2.3 | 工作跨多个 PR，顺序和证据都靠记 | `feature` step 3 的 throughput checkpoint 与 per-PR plan 文件 |

产物之间的关系：

```text
prompt 的五项（1.1）
  → data shape（2.1）
  → architect Phase B 草图（2.2）
  → feature step 3 throughput checkpoint（2.3）
  → multi-phase-plan 的 plan 文件：一个 PR 一个变更，一个 PR 一节（2.3）
  → 派 worker 时的 brief（第 3 章）与每个 PR 的 verdict（第 4 章）
```

---

## 2.1 先给数据一个形状

### 工作情境

"按勾选导出"听起来像一个小功能。实现里先出现一个 `isFiltered` boolean；下一次需求加"只导出我负责的订单"，于是多一个 `onlyMine`；再下一次加"排除已归档"，两个 boolean 开始需要保持同步。导出页、导出接口和测试各有一套判断，改一处要跟着改三处。

这不是实现写得差，是写第一行逻辑之前没有给数据一个形状。

pstack 在 `poteto-mode` 的 Non-negotiables 里把这条放在"任何代码"之前：写任何代码，先命名 data shape，再按 `principle-model-the-domain` 选它的 organizing structure。（`skills/poteto-mode/SKILL.md#Non-negotiables`）

### 带走什么

一个命名的 data shape，和它的 organizing structure。`principle-model-the-domain` 给了一组可选结构，按代码真正要防止的事来选：

| 结构 | 替代什么 |
|---|---|
| state machine | 散落的 boolean、phase、lifecycle check |
| typed model | 松散参数、在多处重复的形状假设 |
| map / registry / lookup table / discriminated union | 跨文件铺开的分支 |
| reducer / command/event model | 随手改状态 |
| 围绕一份 domain 知识组织的 module | load、validate、transform、save 这种按执行顺序切的 module |
| 一个小的 module boundary | 重复的行为、owner 或 invariant |
| queue / cache / index / graph / tree / normalized collection | 访问方式要求它的场合 |

结构不必来自这张表。原则的原话是：当没有一个合适时，先想清楚代码绝不能允许什么、数据会被怎样读，再找恰好编码这两件事的结构。（`skills/principle-model-the-domain/SKILL.md`）

### 做法

**第一步，在写逻辑之前说出 data shape。** `feature` step 4 要求给 subagent 的 scope 里点名 data shape 和它的 organizing structure，并且要在 delegate 写逻辑之前选好。临时任务也一样，Non-negotiables 把这一步放在任何代码之前。

**第二步，先问代码绝不能允许什么。** 空选择不能被解释成全量；`missing` 不能包含没被请求的 id；同一份数据不能有两个 writer。答案决定结构，不决定命名。

**第三步，再问数据怎样被读。** 按 id 批量取、按状态扫、按时间窗口聚合，各自对应不同的集合与索引。访问方式是结构的一半。

**第四步，选一个结构，不要叠三个。** 状态机加 reducer 加 registry 通常说明还没想清。优先 boring code：当前形状已经清楚、局部、不打算长，就不要为了模式加抽象。

**第五步，警惕抽象本身。** 一个只增加间接层、不删除分支、不删除重复规则、不消除非法状态、不降低 lifecycle 风险的抽象，不值得引入。`principle-model-the-domain` 把"新功能只是给现有 if/else 链多加一个分支""第二个 boolean 必须和第一个保持同步""按 phase 命名的 module 重复同一套规则"列为跳过这一步的信号。（`skills/principle-model-the-domain/SKILL.md`）

**第六步，把形状写进给 delegate 的 scope。** 文件路径、named data shape、organizing structure、success criteria，四样一起给。没有形状，形状就由第一版实现决定。（`skills/poteto-mode/playbooks/feature.md` step 4）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 从代码和需求提出 data shape 与结构候选 | 确认真实的 domain 规则 |
| 指出会逼迫形状变坏的分支与状态 | 做只能由 operator 做的产品与偏好决定（见 1.3） |
| 用 Prototype 验证两个结构中的一个 | 接受抽象带来的维护成本 |

### 质量门

- 写第一行逻辑之前，data shape 有名字。
- 结构来自 `principle-model-the-domain` 的一种，或写清了"没有现成结构适配"以及代码绝不能允许什么。
- 新功能没有只给现有 if/else 链加一个分支。
- 没有第二个必须和第一个保持同步的 boolean。
- 没有按执行顺序命名、重复同一套 domain 规则的 module。
- 没有只增加间接层的抽象。
- 给 delegate 的 scope 里有文件路径、data shape、结构、success criteria。

### 进阶

同一个 data shape 可能有多个合法实现形状：错误处理、抽象层、测试结构，都会分歧。`feature` step 4 说，当实现本身允许多个合法形状时，不要派一个 worker 直接写，改用 `arena`，让多个 runner 把备选方案做出来，再由 cross-judge 守住选择。这条进入 2.2。

---

## 2.2 用 architect 的 Phase B 把接口画出来

### 工作情境

data shape 定了，但接口还没定。你面前有几个形状选择：

- 把导出写进页面组件，测试只能点按钮；
- 抽一个 `ExportService`，接口里全是内部步骤；
- 为了测试加一个 `getSelectedRowsForTest()`；
- 引入 repository、factory、adapter，但只有一个实现。

这些选择不会立刻报错。它们会在三个月后变成"改一处、牵五处"和"测试很慢、没人愿意跑"。

### 带走什么

`architect` 的 Phase B 产出：至少两个结构上不同的候选草图，一个综合后的设计包，以及一份 rationale。草图包括类型、函数签名、类形状、module boundary，函数体是 `not implemented` 加伪代码。对比候选的标准是 interface depth：用更小、更简单的 public surface 藏住更多复杂度。（`skills/architect/SKILL.md` Phase B）

### 做法

**第一步，先 Ground，不要从文件清单开始。** Phase A 用 `how` 跑一遍新代码会碰到的每个子系统；说出一个文件名不算 ground。设计会重新定义 ownership 或分层时，再跑 `why`，让原有形状的理由变成约束而不是猜测。只有真正 greenfield、没有周边系统时才能跳过 Phase A。（`skills/architect/SKILL.md` Phase A）

**第二步，Phase B 跑 `arena`。** 把 design-sketch 任务和 Phase A 的 grounding 产物交给 `arena`，每个 runner 用 `references/runner-prompt.md`；runner 从 `architect runners` 这一行取，而不是 `arena runners`。（`skills/architect/SKILL.md` Phase B）

**第三步，设计两次。** 至少两个结构上不同的候选，即使第一个看起来已经够。候选之间是整体形状的差别，不是同一个形状里的点修。这条是 `exhaust-the-design-space` 的具体做法。

**第四步，每个候选都过 design-red-flags。** 筛的是：

| red flag | 它筛掉什么 |
|---|---|
| shallow module | public surface 很大、藏住的复杂度很少 |
| information leakage | 同一个内部决定泄漏到多个 module |
| temporal decomposition | 按执行顺序而不是按知识所有权切 module |
| pass-through method | 同形状转发参数、只加一层的方法 |
| split ownership | 多个 module 写同一份 state 或各存一份 |
| two ways to do one task | 同一件事有两条路，后来者会复制先看到的那条 |
| importable internals | 外部能 import module 内部，最短路径会把它变成接口 |
| hand-synced list | 多处维护同一份列表，加一项要改每一处 |

筛的时候假设下一个贡献者是 agent：它只看打开的文件，复制最近的例子，走最短能编译的路径。优先选那个"从一个文件里看起来对的改动，对整个仓库也对"的设计。（`skills/architect/references/design-red-flags.md`）

**第五步，比较 interface depth。** 优先选把更多复杂度藏在更小 public surface 后面的设计。深 module 不等于深调用链：深调用链把理解散到多层，深 module 把能力集中在一个接口后面。调用者要协调多个方法才能完成一件事、public options 暴露内部阶段、学接口不能省掉学实现，都是 shallow module 的信号。

**第六步，写调用者的用法，再反推类型。** `architect` 的 Outputs 要求先写调用者怎么用，再从用法推出类型草图；小改动一个文件，大改动 module map 加类型定义。`not implemented` 函数体加伪代码，此时改接口还很便宜。（`skills/architect/SKILL.md` Outputs）

**第七步，接住 arena 的综合结果。** `arena` 返回一个综合后的设计包；它的 synthesis decision 填进 rationale 的 "Synthesis decision" 一节。候选收敛到一个形状，是强一致，记下来直接走；候选严重分歧，说明 Phase A 没定够，重新 frame 再跑，而不是把分歧平均掉。（`skills/arena/SKILL.md` Phase E）

**第八步，实现时把草图当契约。** Phase D 把 `not implemented` 换成代码，把伪代码换成逻辑。偏离草图是信号，要浮出来，不要默默吸收；一个函数需要草图没预料到的参数时，先问是草图错了、需求漏了，还是实现越界。（`skills/architect/SKILL.md` Phase D）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 跑 Phase A、起草候选、过 red flags | 在做产品与偏好取舍时给方向 |
| 写 `not implemented` 草图与 rationale | 在明确要求 checkpoint 时签收 |
| 跑 `arena` 并综合候选 | 判断抽象是否值得它的成本 |
| 在实现偏离时把偏离浮出来 | 决定要不要重新设计 |

Phase C 的默认是没有 checkpoint：综合完直接进入实现；只有当 invoker 明确要求（"/architect with checkpoint"、"实现前停下来给我看"）时才停。人对形状提出异议时，把异议当成 Phase A 的证据，重新 Ground 并重跑 Phase B，而不是在实现里折中。（`skills/architect/SKILL.md` Phase C）

### 质量门

- 至少两个结构上不同的候选，不是同一个形状的点修。
- 每个候选都过了 design-red-flags。
- 选中的设计有更小的 public surface，而不是更多层。
- 类型草图从调用者的用法推出。
- 函数体是 `not implemented` 加伪代码，不是一半实现。
- rationale 里有 synthesis decision。
- 实现阶段的偏离被浮出来，没有被默默吸收。

### 进阶

设计空间真的开放时，`arena` 是默认动作；设计有争议时，`interrogate` 在落地前做多模型对抗评审；实现本身允许多个合法形状时，`feature` step 4 也把实现交给 `arena`。（`skills/poteto-mode/SKILL.md#Non-negotiables`、`skills/poteto-mode/playbooks/feature.md`）

Phase E 是丢掉草图的时机：同一种 workaround 反复出现在不相关代码里、多个不相关的边界情况都要特判、类型要靠 `any` 或 cast 才能编译、"这里需要一把锁"的反射、调用者必须先懂抽象的内部规则才能用。信号是模式，不是单次。出现模式时，回到 `how`，按 `redesign-from-first-principles` 重新设计，先 `subtract-before-you-add`，再回 Phase B 重跑 `arena`。（`skills/architect/SKILL.md` Phase E）

---

## 2.3 用 throughput checkpoint 和 plan 文件排开工作

### 工作情境

形状定了，工作还是一整块："把勾选导出做完"。

直接交给一个 worker，它会自己决定顺序；顺序不一定错，但你没有同意过。想并行，又不知道哪些能同时做；想交接，又说不清一个 unit 到哪里算完成。跨多个 PR 时更糟：哪个 PR 带哪份证据、谁先合、哪一段能一起落地，全靠记忆。

### 带走什么

`feature` step 3 的 throughput checkpoint，写成四个 todo item：

```text
Blocking first steps.        先跑 gate，再 fan-out。
Independent workstreams.     不相交的文件、服务或层可以并行；共享写入串行。
Shared mutable state.        默认拆分目标；只对真正的不变量串行。
Smallest safe decomposition. 如果一个 worker 最好，写明为什么。
```

某个维度确实不适用时（单个文件、没有 fan-out），保留该项并写 `n/a: <reason>`，不要删掉。（`skills/poteto-mode/playbooks/feature.md` step 3）

跨多个 PR 时再加一份 `multi-phase-plan` 的 plan 文件。它的骨架是固定的：

- 固定标题：`## How to read this`、`## Program checklist`、`## <Task as a verb phrase> (<PR id>)`、`## Close the program`，以及四个 appendix（A Prototype evidence、B Alternatives rejected、C Risks、D Links and reading list）；`check-plan.mjs` 缺标题会失败；
- Program checklist：arm the program、spawn owners、每个 PR 的 PR mechanics、verdict 与 merge、live lane 的 boot recipe；
- 每个 PR 一节：`Task as a verb phrase (<PR id>)`、Depends on、Files、Build、You see、Verify, unit / Verify, live / Verify, perf、Review gate、Merge；
- Close the program：每个 box 都勾上并附证据；回复按执行 playbook 的 report 形状；
- Appendix A 到 D：prototype 证据、被拒的备选、风险、链接与阅读清单。

一个 box 是一段工作；每个 box 点名检查它的证据；证据存在才勾。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

### 做法

**第一步，先写 blocking first steps。** gate 在 fan-out 之前跑完。需要先定的类型、先合并的前置 PR、先拿到的权限，都放在这一项。

**第二步，写 independent workstreams。** 不相交的文件、服务或层可以并行；共享写入串行。不要用锁把共享写入掩过去，锁会让并行退化成串行，还引入新的失败模式。

**第三步，写 shared mutable state。** 默认按 `separate-before-serializing-shared-state` 拆开目标；只对真正的不变量串行。两个 unit 写同一个文件、同一个分支、同一个配置项或同一张表，它们就不是真并行。

**第四步，写 smallest safe decomposition。** 如果一个 worker 最好，就写明为什么。切得越细不一定越快：一个 unit 太小会产生大量交接成本，太大则无法独立验证。

**第五步，把 checkpoint 放在 owner 手里。** 代码耦合的工作（一个 feature、一次 migration）交给一个 owner，checkpoint 内联；这个 owner 在 blocking phase 之后自己 fan-out。父级 fan-out 留给产出独立产物的片段（audit、跨子系统调查、并行实验）。换 phase 时重写 checkpoint，用新的 owner，不要用 interrupt 链住旧 owner。（`skills/poteto-mode/playbooks/feature.md`）

**第六步，一两个文件且路径明显时，跳过 plan。** `multi-phase-plan` step 1：直接说这次不需要 plan，然后停。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

**第七步，跨 PR 时先关开放问题。** 每个开放问题先跑 Prototype，保留分支、SHA 和截图，写进 plan 的 Appendix A；只有产品与偏好决定才问 operator，并且给选项。（`skills/poteto-mode/playbooks/multi-phase-plan.md` step 2）

**第八步，把 skeleton 复制成 plan 文件。** 每个 placeholder 都填；每个 PR 一节；在 "How to read this" 里点名执行 playbook（`autopilot-full` / `autopilot-stack`，常驻项目用 `orchestrate`）；一个 PR 是一个变更，带自己的证据。plan 是交付物，写 plan 的阶段不实现。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

**第九步，让每个 PR 自带证据。** 每个 PR 的 Verify 分 unit、live、perf 三块，验证规则写在每一块开头：测试本身不算验证，只有 unit、live、perf 三块都勾上，PR 才算 verified。live 块是强制的，用 `control-ui` / `control-cli` 这类 control skill 在 PR head 上驱动真实表面；perf 块是双向的，trunk 与 head 都要产出同一个 metric。（`skills/poteto-mode/playbooks/multi-phase-plan.md` Verification）

**第十步，写 review gate 和 merge。** 改交互的 PR 由 operator 在 chat 里看截图和视频后 review，停在 merge-ready；不改交互的 PR 写 `**Review gate.** None. <PR id> is not review-gated.`，下面不写 box。merge 规则按执行 playbook；新 head 需要新 verdict，除开按 `shipping` 的 patch-id 规则仍有效的结果。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

**第十一步，跑检查脚本再交回。** 用 `skills/poteto-mode/scripts/check-plan.mjs` 检查 plan，修掉它打印的每一行；把 plan 路径和脚本输出贴回来，然后停。执行等 operator 的 explicit go，按 plan 里点名的执行 playbook 跑。（`skills/poteto-mode/playbooks/multi-phase-plan.md` step 6–7）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 写四个 checkpoint item，标 `n/a` | 给 explicit go |
| 找共享写入、排依赖、拆 owner | 做 review gate 的 review |
| 写 plan 文件并跑检查 | 做产品与偏好决定（见 1.3） |
| 按执行 playbook 派 owner 和 verifier | 接受 plan 的范围与顺序 |

### 质量门

- 四个 checkpoint item 都在，不适用的写了 `n/a: <reason>`。
- 代码耦合工作有一个 owner，checkpoint 在它手里。
- 没有两个 unit 写同一份共享状态；共享写入要么拆开，要么显式串行。
- 一两个文件的明显改动没有 plan。
- 跨 PR 的 plan 一个 PR 一节，一个 PR 一个变更，带自己的证据。
- 每个 PR 的 unit、live、perf 三块都有勾选证据。
- plan 检查脚本没有输出。
- 执行前有 operator 的 explicit go。

### 进阶

plan 文件按执行 playbook 落地：`autopilot-full` 是独立 PR 队列、一个 owner 管到 merge；`autopilot-stack` 是一条线性 reviewed 栈、由 operator 落地；常驻项目走 `orchestrate`。选哪个写在 plan 的 "How to read this" 里，不靠记忆。（`skills/poteto-mode/playbooks/multi-phase-plan.md` step 4）

`feature` step 3 的 checkpoint 在单次运行内有效；`multi-phase-plan` 的 plan 文件跨 phase 有效。两者不要互相替代：checkpoint 决定这一轮谁先做，plan 决定哪个 PR 带哪份证据。

---

## 三个单元怎样接起来

```text
2.1 data shape（principle-model-the-domain）
  → 2.2 architect Phase B：public surface、not implemented、design-red-flags
  → 2.3 feature step 3 throughput checkpoint
  → 2.3 multi-phase-plan 的 plan 文件：一个 PR 一个变更，一个 PR 一节
  → 第 3 章派 worker 时的 brief
  → 第 4 章每个 PR 的 verdict
```

data shape 是接口的输入，接口是 checkpoint 的输入，checkpoint 是 plan 里每个 PR 的输入。三者不一致时，先回到上游修，不要在下游打补丁。

## 常见失败

| 失败 | 表现 | 修正 |
|---|---|---|
| 先写逻辑再命名 data shape | 分支越加越多，两个 boolean 要同步 | 回到 2.1，按 `principle-model-the-domain` 选结构 |
| 只做一个候选 | 第一版形状被锁死 | 回到 2.2，Phase B 至少两个结构上不同的候选 |
| 接口暴露内部步骤 | 调用者被迫按内部步骤调用 | 用 design-red-flags 的 shallow module 与 information leakage 筛 |
| 测试专用接口 | 为了测试暴露内部状态 | 改 public surface，不要加测试后门 |
| 一个 PR 塞多个变更 | 证据和变更对不上 | 回到 2.3，一个 PR 一个变更 |
| 只有 unit 证据 | 测试绿被当成 verified | 按 `multi-phase-plan` 的 Verification，unit、live、perf 都勾 |
| plan 写完直接开跑 | 执行顺序没经 operator 同意 | 停在 operator 的 explicit go |

## 继续阅读

- [委派与协作：让执行者独立，让责任可见](./delegate-and-collaborate.md)
- [教程 03：让一个承诺变成可观察行为](../tutorial/03-make-a-promise-observable.md)
- [教程 04：设计一个可测试的形状](../tutorial/04-design-a-testable-shape.md)
- [教程 05：把工作切成可委派的单元](../tutorial/05-cut-work-into-delegable-units.md)
- [术语表：principle、playbook、brief、verdict](../CONTEXT.md)
