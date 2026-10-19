# 教程地图

教程从一条 prompt 开始，走一遍 pstack 的实际流程：prompt 五项 → mode/playbook → todo → data shape → `architect` → 切片 → `tdd` → brief → verdict → `interrogate` → `shipping` → `correct` / `reflect`。每页只处理一个机制，并给源码位置。

> 状态：严格重写完成（2026-10-19）。只使用 pstack 的 skill、playbook、字段、角色与状态；没有 pstack 依据的概念不写。结构见 [PLAN.md](../PLAN.md)，术语见 [CONTEXT.md](../CONTEXT.md)。

## 一条连续的路径

```text
00 把 prompt 写成 pstack 能接住的样子
  → 01 用 mode 还是单 skill
  → 02 把 playbook 步骤放进 todo
  → 03 先命名 data shape
  → 04 用 architect 画 public surface
  → 05 把工作切小
  → 06 用 tdd 先看见红
  → 07 用 brief 派 subagent
  → 08 用 verdict 证明
  → 09 用 interrogate 评审
  → 10 把验证过的一段栈落地
  → 11 用 correct 与 reflect 改环境
  → 教程之后
```

## 每页的五个区块

| 区块 | 问题 |
|---|---|
| 眼前困惑 | 这一页要解决的具体卡点 |
| 现在动作 | 这一页只让读者做的一件事 |
| 可见成果 | 十分钟后能看见、复述或留下的东西 |
| 休息点 | 为什么现在可以停 |
| 下一段 | 下一页处理哪个机制 |

## 台阶清单

| 台阶 | 主题 | 可见成果 | 状态 |
|---|---|---|---|
| [`00` 从这里开始](./00-start-here.md) | prompt 五项 | 一条能进 mode 的 prompt | 已重写 |
| [`01` 用 mode 还是单 skill](./01-define.md) | 入口选择与 AskQuestion 分类 | 一个入口决定 | 已重写 |
| [`02` 把 playbook 步骤放进 todo](./02-give-words-a-home.md) | playbook 匹配与 `skip` | 一个带步骤的 todo | 已重写 |
| [`03` 先命名 data shape](./03-make-a-promise-observable.md) | 类型与可观察结果 | 输入 / 输出 / 失败出口 | 已重写 |
| [`04` 用 architect 画 public surface](./04-design-a-testable-shape.md) | Phase B 草图 | 一个 `not implemented` 草图 | 已重写 |
| [`05` 把工作切小](./05-cut-work-into-delegable-units.md) | throughput checkpoint / plan | 四条 checkpoint 或一个 plan | 已重写 |
| [`06` 用 tdd 先看见红](./06-build-a-feedback-loop.md) | red-green 与 skip | 一条先失败后通过的检查 | 已重写 |
| [`07` 用 brief 派 subagent](./07-delegate-and-keep-control.md) | brief 八字段 | 一份可独立执行的 brief | 已重写 |
| [`08` 用 verdict 证明](./08-prove-the-promise.md) | verdict / patch-id | 非作者 `PASS` 或 `PASS+NOTES` | 已重写 |
| [`09` 用 interrogate 评审](./09-review-for-merge.md) | 多模型评审与四桶 | 一个 lead verdict | 已重写 |
| [`10` 把验证过的一段栈落地](./10-ship-observe-and-loop.md) | shipping / babysit | 一段 contiguous verified run | 已重写 |
| [`11` 用 correct 与 reflect 改环境](./11-improve-the-environment.md) | 结构约束与 skill 修改 | 一条会失败的检查或一次 reflect 结论 | 已重写 |
| [`教程之后`](./after-tutorial.md) | 按卡点查手册 | 六章入口 | 已重写 |

## 与手册的关系

教程给一条连续路径；手册按机制给完整字段、例外和源码位置。卡住时，从 [手册目录](../manual/README.md) 的卡点表进入。
