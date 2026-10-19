# 验证与 verdict

> 本章 4 个单元 · [手册目录](./README.md) · [上一章：Subagent 与 brief](./delegate-and-collaborate.md) · [下一章：评审与落地](./ship-run-and-evolve.md)

## 这一章解决什么

pstack 不把"做完了"当一句话。它要求四种不同的证据：`tdd` 的 red-green、`prove-it-works` 的真实产物、`benchmark-checklist` 的七问、`shipping` 的 verdict 与 patch-id。四种证据各自证明不同的东西，不能互相替代。

| 单元 | 卡点 | 带走 |
|---|---|---|
| 4.1 | 改完只靠肉眼，失败信号不稳定 | 一条先红后绿的检查 |
| 4.2 | 测试绿了，但真实用户路径没跑过 | 真实表面证据 |
| 4.3 | 性能数字没有方法 | 七问 + 基线 |
| 4.4 | 作者自证、绿 CI 被当成通过 | 非作者 verdict 与 patch-id |

---

## 4.1 `tdd`

### 工作情境

bug 修完，测试绿了。但测试是从实现反推出来的，实现错的时候它也一起错。

`tdd` 的规则是：用户明确要求 TDD，或测试路径便宜、清晰时，先写一个会失败的回归测试，再改实现。源码：`skills/tdd/SKILL.md`。

### 做法

1. 理解 bug：intended behavior、current behavior、受影响路径、最小可观察复现。
2. 选最窄的可执行检查：优先用这条 codepath 已有的单元、组件、集成或回归测试。
3. 写会失败的测试：断言 intended behavior，不镜像实现。
4. 在修之前运行它：确认它因为正确的原因失败。
5. 改实现：最小的 production change，保持附近契约。
6. 再跑一次：确认它现在通过。

### 不适用时

测试需要大面积 harness、脆弱的 mock、慢的端到端、production-only 状态、模糊复现步骤或大量无关 fixture 时，跳过新测试，改用最接近的可用验证。源码：`skills/tdd/SKILL.md#If a Failing Test Is Impractical`。

跳过不是免责。没有测试时，仍要有一条可重复的真实表面检查，并写清为什么测试路径不成立。

### 质量门

- 如果建了测试，它在实现之前真的红过，且失败原因正确。
- 测试断言行为，不读私有状态。
- 不适用时，写明了原因和替代检查。

---

## 4.2 `prove-it-works`

### 工作情境

subagent 说"已验证"。你问它跑了什么，它贴了一段测试输出。测试只覆盖了一条分支，用户路径没跑。

`prove-it-works` 的规则是：验证真实产物，不用代理指标。源码：`skills/principle-prove-it-works/SKILL.md`。

### 做法

- 能运行功能就运行功能，读实际值、看实际 diff、贴实际输出。
- "能编译"不是证据；"测试绿了"只在测试覆盖了该行为时才是证据。
- 一次通过只证明跑过的那条路径；写出这次证据覆盖到哪里、不覆盖什么。
- 关键行为至少有一次真实表面证据；有 `create-verification-skill` 生成的项目本地 skill 时，用它。

### 质量门

- 证据指向真实产物，不是"我认为"。
- 覆盖范围写出来了。
- 下一位能按证据复跑。

---

## 4.3 `benchmark-checklist`

### 工作情境

你改了一个查询，报告"快了 30%"。有人问：基线怎么测的？有没有调优？样本多少？你答不上来。

`benchmark-checklist` 要求报告任何性能数字前回答七个问题。源码：`skills/benchmark-checklist/SKILL.md`。

### 七问

1. **Why not double?** 先找 limiter，映射到源码；别把 load generator 当成被测系统。
2. **Was it tuned?** 两边按生产配置跑；一边用默认值，比的是配置不是实现。
3. **Did it break limits?** 算带宽、CPU、时间上限；超过上限说明测的不是你以为的工作。
4. **Did it error?** 数失败和非成功响应；拒绝通常很快，超时和重试很慢。
5. **Does it reproduce?** 每边至少 5 次，交替 A/B；报 median 和 range；差距小于波动就是没有差异。
6. **Does it matter?** 同时测端到端路径，把 micro 结果写成整体占比。
7. **Did it even happen?** 确认工作真的发生在计时区间里；lazy code 和 timeout 会给出"没发生"的数字。

用户只要 ballpark 时，至少回答第 4、7 问，并说明只有一次运行。在两个选项之间做选择时，一次运行永远不够。

### 质量门

- 七问都有真实运行的证据，或明确标注了 ballpark。
- 基线、限制、样本数、波动都写出来。
- 没有把"一次运行"写成"稳定提升"。

---

## 4.4 verdict、patch-id 与 contiguous verified run

### 工作情境

PR 绿了，bot approve 了，作者说"我测过了"。这三样都不是 verdict。

`shipping` 要求一个没写这份代码的 verifier，在真实表面跑用户路径，给出 `PASS`、`PASS+NOTES` 或 `FAIL`。源码：`skills/poteto-mode/playbooks/shipping.md` step 1。

### 做法

1. 每个 PR 一个 verifier；verifier 不能是作者。
2. verifier 跑 parent against head，像用户一样操作。
3. 只有 `PASS` 或 `PASS+NOTES` 算通过；`FAIL` 是停止点。
4. 从最低未合并 PR 往上，取连续有 `PASS` 或 `PASS+NOTES` 的一段；第一个没有的 PR 就是停止点。
5. rebase 后用 `git patch-id` 比较 verdict 时的 SHA 和当前 head；补丁不同就重新验证。
6. 自底向上，一次合一个；落地过程中不改栈的拓扑。

### 质量门

- verdict 来自非作者。
- 只有 `PASS` / `PASS+NOTES` 进入落地段。
- patch-id 覆盖当前 head。
- 停止点之上的 PR 没有被先合。

---

## 本章与 pstack 的边界

pstack 没有额外的验证层名字；它就是 `tdd`、`prove-it-works`、`benchmark-checklist`、`shipping` 的 verdict。直接用这些 skill 名和字段名。
