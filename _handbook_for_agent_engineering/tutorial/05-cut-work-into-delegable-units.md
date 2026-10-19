# 把工作切成可委派的单元

> 教程 05 / 12 · 上一页：[04 设计一个可测试的形状](./04-design-a-testable-shape.md) · 下一页：[06 用 tdd 先看见红](./06-build-a-feedback-loop.md)

## 眼前困惑

形状和接口都有了，工作还是一整块："把勾选导出做完"。

如果整块丢给一个 agent，它会自己决定顺序。顺序不一定错，但你没有同意过；出问题时也说不清是哪一步偏了。

`feature` 的做法是先写 throughput checkpoint，再决定谁写代码。源码：`skills/poteto-mode/playbooks/feature.md` 第 3 步。

## 现在动作

### 1. 把 throughput checkpoint 写成四条 todo

`feature` 第 3 步要求把四个维度写成 todo。某个维度确实不适用时，条目留着，写 `n/a: <reason>`，不要删掉。以勾选导出为例：

```text
- [ ] 阻塞先行：先定 OrderExporter.export 与 ExportResult 的形状
- [ ] 独立工作流：全量导出的回归检查与勾选导出不共享文件，可以并行
- [ ] 共享可变状态：接口只在阻塞步骤里定一次，其余部分只消费
- [ ] 最小安全切分：这是一个 feature，交给一个 owner，阻塞步骤之后由它自己 fan-out
```

四个维度各自在问什么：

- **Blocking first steps。** 门在 fan-out 之前跑。这一轮的门是 `export` 的签名和 `ExportResult`（`rows`、`missing`、`file`）。它没定之前，页面把勾选状态放哪、`missing` 怎么显示，都只能猜。
- **Independent workstreams。** 不共享文件、服务或层的活可以并行，共享写入的地方串行。P1 写 `exporter/` 和页面接线，P3 只写 `exporter/regression.test.ts`，两者不相交；P2 也要改 `export-page/`，所以它排在 P1 之后，不并行。
- **Shared mutable state。** 默认先把目标拆开（**principle-separate-before-serializing-shared-state**）：不同部分用不同文件、不同 branch 或 worktree。只有真实不变量才串行，别用锁把并行改成排队。
- **Smallest safe decomposition。** 如果一个 worker 最合适，写出为什么。`feature` 自己给了一条：代码耦合的活（一个 feature、一次迁移）交给一个 owner，把 checkpoint 内联进去；父层的 fan-out 只留给产出独立产物的切片，例如审计、跨子系统调查、并行的设计实验。

源码：`skills/poteto-mode/playbooks/feature.md` 第 3 步，以及该文件末尾关于 code-coupled work 的一段。

### 2. 一个 PR 是一个 section，自带证据

一处变更跨多个 PR 时，用 `multi-phase-plan` 写计划。它的骨架把每个 PR 写成一段，字段是 **Depends on / Files / Build / You see / Verify, unit / Verify, live / Verify, perf / Merge**。规则是一句话："One PR is one change with its own evidence." 源码：`skills/poteto-mode/playbooks/multi-phase-plan.md` 第 4 步与骨架。

| PR | Depends on | Files | You see | Verify（unit / live / perf） |
|---|---|---|---|---|
| P1 | none | `exporter/export.ts`、`exporter/export.test.ts`、`export-page/ExportButton.tsx` | 勾选 3 行，下载文件 3 行 | unit：`pnpm test exporter`；live：`control-ui` 勾选 3 行下载 |
| P2 | P1 | `export-page/` | 无勾选时按钮禁用并提示；删 1 行后结果标注该 id | unit：组件测试；live：`control-ui` 走这两种状态 |
| P3 | none | `exporter/regression.test.ts` | 全量导出行为不变 | unit + live：trunk 与 head 各跑一次全量导出 |

**You see** 必须是能看见的结果，不是"改完了"。框只在证据存在时才勾：一个文件、一行日志、一张截图、一次测试运行或一个 SHA。源码：骨架的 How to read 段。

### 3. 从 Depends on 读出当前 ready work

P1 和 P3 没有未完成的前置，是当前 **ready work**；P2 等 P1。ready work 不是"最重要的那一段"，而是"现在就能开始的那一段"。

`orchestrate` 在每轮排空后重新计算 ready work，并在依赖图里写明：依赖的工作等父 PR 合并后再开始，执行 playbook 允许 stacking 时也可以基于父分支。源码：`skills/poteto-mode/playbooks/orchestrate.md` 的 Steps 与 Spawn owners。

ready work 不是 frontier。frontier 只指 PR 栈的合并前沿，`babysit` 每轮只处理最低的未合并 PR。源码：`skills/poteto-mode/playbooks/babysit.md` 第 2 步、`orchestrate.md#Stack safety`。

### 4. 先定谁写，再定谁验

`feature` 第 4 步把"把代码写作派给 subagent"写成强制项：不能用一行 `skip: <reason>` 绕过；一个被禁止派 subagent 的 owner 就自己拥有 diff，保持同样的评审分离。第 5 步要求在匹配的表面上验证，inconclusive 或 wrong surface 都不算通过。

所以切分的目的是让每一段有一个 owner、一个能验的表面，不是让更多人同时写。派出去时把 checkpoint 这一段作为输入，07 会把它写成 brief。

源码：`skills/poteto-mode/playbooks/feature.md` 第 4、5 步。

### 5. 计划写完就交回，不开始执行

`multi-phase-plan` 把计划本身当成交付物：

- 只改一两个文件、方法显然时，跳过计划并说明（第 1 步）；
- 写之前先把没定的问题用 prototype 跑掉，留下 branch、SHA 和截图（第 2 步）；
- 跑 `node pstack/skills/poteto-mode/scripts/check-plan.mjs <plan.md>`，修掉它打印的每一行（第 6 步）；
- 把计划路径和脚本输出交回，然后停；执行等 operator 明确说 go，由计划里写明的执行 playbook 接手（第 7 步）。

源码：`skills/poteto-mode/playbooks/multi-phase-plan.md`。

## 可见成果

todo 里的 checkpoint，加一份按 PR 分段的计划：

```text
throughput checkpoint
- [x] 阻塞先行：OrderExporter.export 与 ExportResult 先定
- [x] 独立工作流：三段各写各的文件
- [x] 共享可变状态：接口只定一次，每段一个 branch
- [x] 最小安全切分：一个 feature 一个 owner，阻塞步骤后 owner 自己 fan-out

当前 ready work：P1、P3；P2 等 P1。
执行 playbook：计划里写明（autopilot-full、autopilot-stack 或 orchestrate）。
```

## 休息点

可以停在这里。计划交回之后，pstack 的下一步不是你继续写代码，而是 operator 说 go，执行 playbook 接手。如果某一段写不出 **You see** 或 **Verify**，先回到 checkpoint 第四条重切，不要带着一个填不满的 section 进入执行。

## 下一段

下一页 [06 用 tdd 先看见红](./06-build-a-feedback-loop.md)，会拿当前 ready work 里的 P1，按 `tdd` 先让一条检查因为正确的原因失败，再写最小实现。

想先看完整方法，查 [手册：设计与计划](../manual/design-and-plan.md)；想回到教程起点，见 [教程地图](./README.md)。
