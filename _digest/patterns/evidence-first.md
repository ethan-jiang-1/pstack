# 验证与 verdict

pstack 不把"做完了"当一句话。它把完成拆成几种可检查的东西：`tdd` 的 red-green、`prove-it-works` 的真实产物、`benchmark-checklist` 的七个问题、`shipping` 的 verdict 与 patch-id、`show-me-your-work` 的 decision trail。测试绿了、编译过了、CI 通过，都不自动等于兑现。

## 源码里的五个装置

### 1. `tdd`

用户明确要求 TDD、失败测试或回归测试，或修 bug 且测试路径便宜时，先写一个会失败的回归测试，再改实现。测试要断言用户可观察的行为，不镜像实现。测试路径不清楚、昂贵、集成很重时，跳过新测试，用最接近的验证。源码：`skills/tdd/SKILL.md`。

### 2. `prove-it-works`

在真实产物上验证：运行功能、读实际值、看 diff。不用代理指标。"能编译"不是证据；"测试绿了"只在测试覆盖了该行为时才是证据。源码：`skills/principle-prove-it-works/SKILL.md`。

### 3. `benchmark-checklist`

报告任何性能数字前回答七个问题：什么限制了它、调优空间、上限、错误、可重复性、相关性，以及工作是否真的发生了。用户只要 ballpark 时，至少回答第 4、7 问，并说明只有一次运行；在两个选项之间做选择时，一次运行永远不够。源码：`skills/benchmark-checklist/SKILL.md`。

### 4. `shipping` 的 verdict

每个 PR 由一个没写这份代码的 verifier 给出 `PASS`、`PASS+NOTES` 或 `FAIL`。只有 `PASS` 或 `PASS+NOTES` 才算通过；绿 CI 和 bot approval 不是 verdict。落地只取从最低未合并 PR 开始连续通过的那一段。rebase 会改 SHA，旧 verdict 是否仍覆盖当前 head，用 `git patch-id` 判断。源码：`skills/poteto-mode/playbooks/shipping.md`。

### 5. `show-me-your-work`

长任务、自主运行或用户离开后要复查的工作，用 `show-me-your-work` 留 decision trail：默认 `decisions.tsv`，一行一个决定（ts、phase、decision、why、evidence、result），只追加。交付前用本次运行的 transcript 审计，再让一个不同模型家族的 subagent 复查。源码：`skills/show-me-your-work/SKILL.md`。

## 什么算完成

| 你要证明的 | pstack 要求 | 源码 |
|---|---|---|
| 这条 bug 修好了 | 真实表面复现通过；测试路径便宜时先红后绿 | `bug fix` step 4–5、`tdd` |
| 这个功能真的能用 | 在真实表面跑用户路径，读实际结果 | `prove-it-works`、`create-verification-skill` |
| 这个数字可信 | 七问；基线、限制、重复运行；选择时不能只跑一次 | `benchmark-checklist` |
| 这个 PR 可以落地 | 非作者 verifier 的 `PASS` / `PASS+NOTES`；patch-id 覆盖当前 head | `shipping` |
| 这次长任务可审计 | decision trail + transcript 审计 + 跨模型复查 | `show-me-your-work` |

## 边界

- **证据有范围。** 一次通过只证明跑过的那条路径；`create-verification-skill` 的 feature map 记录覆盖到哪些 feature，没覆盖的要写出来。
- **verdict 需要另一个 agent。** `shipping` 的每个 PR 都要一个非作者 verifier，成本随栈长度增长。
- **decision trail 可能自报。** 所以有 transcript 审计和跨模型复查；单靠 TSV 不能证明 TSV。
- **pstack 到 merge 为止。** 部署、发布后监控、回滚不在 pstack 里；部署只是 `Always pause` 的一个不可逆动作。源码：`skills/poteto-mode/SKILL.md#Autonomy`。

## 关键洞察

1. **证据要落在表面上，不是测试数量上。** 在错误表面上跑一百个测试，仍然不能证明用户路径。
2. **独立性比强度更重要。** 一个没写代码的 verifier 的 `PASS`，比作者自己跑十遍更有信息量。
3. **trail 的价值在于可审计。** 它让人能沿着指针检查，而不是相信总结。
