# 用 `architect` 画 public surface

> 教程 04 / 12 · 上一页：[03 先命名 data shape](./03-make-a-promise-observable.md) · 下一页：[05 把工作切成可委派的单元](./05-cut-work-into-delegable-units.md)

## 眼前困惑

data shape 定了，接下来很自然就是写实现。但代码一跨函数边界，问题就出现：接口该暴露什么、实现该藏什么、不变量放在哪、测试从哪里进入。写出来再改，成本比先画一遍高得多。

pstack 的触发条件是：任何 code crossing a function boundary，先跑 `architect`，并行做设计探索，再实现。源码：`skills/poteto-mode/SKILL.md#Non-negotiables`。

## 现在动作

### 1. 让 `architect` 走 Phase B

`architect` 的 Phase B 是 Sketch：用 design-sketch 任务加 Phase A 的 grounding 产物跑 `arena`，至少两个结构上不同的候选，整形状的替代，不是同一形状内的点修。源码：`skills/architect/SKILL.md#Phase-B-Sketch`。

### 2. 看每个候选的 public surface

public surface 是调用者和测试共同跨越、观察外部行为的位置。它应该是窄的：调用者正确使用模块必须知道的全部事实，包括可表达的业务意图、可观察结果、不变量、顺序约束、错误方式和必要配置。源码：`skills/architect/SKILL.md#Phase-B-Sketch`。

如果测试必须越过 public surface 去读私有状态，说明 surface 太深，或者状态放错了位置。

### 3. 要 `not implemented` 草图，不要实现

`architect` 的输出是带 `not implemented` 函数体和 pseudocode 的草图。它用来固定形状，不替代实现。源码：`skills/architect/SKILL.md#Outputs`。

### 4. 查 design red flags

`skills/architect/references/design-red-flags.md` 列了常见红旗：shallow module、pass-through wrapper、隐藏的共享状态、过早抽象。候选在 Phase B 之后用这份清单过一遍。

### 5. 在实现前做一次对抗压力

如果形状有争议，跑 `interrogate` 对综合草图施压；这是可选的，但它比实现后再返工便宜。源码：`skills/architect/SKILL.md`（Phase C）。

### 6. 选定后，实现仍按 playbook 走

`architect` 只到草图。实现、验证、提交顺序仍由当前 playbook 的步骤决定；不要用草图替代 todo 里的门。

## 可见成果

一个 `architect` 草图：public surface、`not implemented` 函数体、不变量、失败出口，以及一份 rationale。它不叫"模块形状卡"，也不需要另起名字。

## 休息点

可以停在这里。形状已经固定，下一页处理把实现切成可验证的小块。

## 下一段

[05 把工作切小](./05-cut-work-into-delegable-units.md)：用 `feature` step 3 的 throughput checkpoint，把实现拆成可并行、可验证的单元。
