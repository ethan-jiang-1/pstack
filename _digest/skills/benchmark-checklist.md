# `benchmark-checklist`

> 源码：[`skills/benchmark-checklist/SKILL.md`](../../skills/benchmark-checklist/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

在报告或依据一个性能数字之前，先用七问把它钉死：什么限制了它、两边是否都调优过、有没有越过物理上限、有没有报错、能不能复现、对用户有没有意义、以及被计时的工作是否真的发生了。

## 它解决什么

性能数字是最容易自欺的一类证据。一个看起来变快的数字，可能来自更冷的缓存、更快的机器、没算进去的错误、被 JIT 丢掉的代码，或者一次根本没有执行的工作。这类错误通常不是造假，而是测量脚本和结论之间少了一次对账。

`benchmark-checklist` 就是那次对账。它不教怎么优化，也不判断改动值不值得合入；它只回答"这个数字能不能作为证据"。

## 核心机制

### 1. 先把要 ship 的那句话写下来

七问测的是这句话，不是代码。例如 "export is 30% faster at p50 on the 60k-row dataset"。写不出这句话，说明这次测量还只有动作，没有 claim。

同时读 measurement script：它计什么、不计什么。再用 `uptime` 看 load average、`nproc` 看核数。机器忙就找出是什么在跑；停不掉就交错跑两边，让它们看到同样的噪声，并在报告里说明。源码的总要求是：每一问都用一次真实运行的证据回答，不是对代码的猜测。

### 2. 七问

1. **Why not double?** 命名 limiter。profiler 会拖慢被测工作，所以 profile 放在不报告的那次运行里：CPU per process（`top`、`pidstat`）、runtime profiler（`node --cpu-prof`、`py-spy`、`perf`）、I/O wait、syscall 计数（`strace -c`，Linux），再把热点映射回源码。也要看 load generator：它先饱和，你测的就是它。改动没动数字时，limiter 解释原因，所以要先找它，再判定改动无用。
2. **Was it tuned?** 两边都按生产的方式跑：release build、生产 flags 和 env、batching 和事务设置、连接池、缓存冷暖、同样的版本和数据。一边跑默认值，比的就是配置，不是实现。如果 limiter 本身是一个设置（每行一次 commit、debug build、缺索引），那一边就是没调优；调完再测，调不了就不要从那次运行里选赢家。用户是在选采用什么时，把 claim 缩窄成"今天 ship 的代码"也不解决问题，因为他采用选项，不是今天的设置。
3. **Did it break limits?** 做算术。bytes/s 对磁盘和网络带宽；ops/s 乘每次操作成本对核数；省下的时间对被改部分占的时间。去掉占 10% 的部分，最多快约 11%。超过上限的结果一定测了别的东西：cache、no-op 或 bug。
4. **Did it error?** 数失败和非成功响应，并检查输出正确，不只是存在。rejection 往往快，timeout 和 retry 慢。脚本不数错误，就加上计数。
5. **Does it reproduce?** 每边至少 5 次，交替跑（A、B、A、B），让 warmup、lazy init、cache 和漂移不偏向任何一边。报中位数和范围。差距小于 run-to-run variation 就是没有可测差异；接近时用 rank-sum test 或 harness 自己的统计。
6. **Does it matter?** 任何 micro 结果旁边，测用户实际等待的端到端路径，用真实的数据规模和并发，并把 micro 结果报成整体的份额。占请求 1% 的 helper，再快也只能让请求快 1%。
7. **Did it even happen?** 确认工作在计时区内真的发生：请求到了 server、行写进去了、字节读出来了、代码用了结果。lazy 代码（没人迭代的 generator、没人 await 的 promise、JIT 能丢掉的结果）和 timeout 会为没发生的工作产出数字。

### 3. ballpark 的例外很窄

用户要一个快速 ballpark 时，一次运行够用。仍然要查 Q4 和 Q7，并说明只有一次；其余各问可以跳过，除非那一次看起来不对。

选型永远不是 ballpark。在两个选项之间做选择，一次运行不够，因为选择会把一次性的噪声固化成一个长期决定。

### 4. 报告形状

先给 verdict：faster、slower、no measurable difference 或 inconclusive。数字带单位、运行次数、范围、limiter，例如 "p50 41 ms → 33 ms, median of 7 runs per side, range 32 to 35 ms after, bound by JSON parsing on one core"。

三种情况必须报 inconclusive：声称有差异但说不出 limiter、有一边没调优、查不了 Q4 和 Q7。同时点名 gap。

PR body 只放一个主数字，这是 `Opening a PR` playbook 的规定；runs、range 和 limiter 证据放链接的 artifact 或 notes 文件。

## 设计意图

- **先写 claim，再测。** 句子是规格，七问是对这句话的逐条攻击面；没有句子，七问就退化成泛泛的检查。
- **limiter 是枢纽。** 说不出 limiter 的数字既不能解释，也不能外推；Q1 同时解释了"为什么改了没动"。
- **算术比实验便宜。** Q3 不用跑就能否证一个数字，是最便宜的谎言检测器。
- **把噪声写进报告。** 交替运行加中位数加范围，让读者看到不确定性，而不是用平均值把它藏掉。
- **Q7 单独设一问。** 它针对的不是"测错了值"，而是"测了别的东西"：lazy code、timeout、没走到的路径。
- **报告只留一个数字。** 证据放链接里，避免 PR body 变成表格，同时保留可复查的入口。

## 依赖关系

- **上游**：`Perf issue` playbook 在从 baseline 规划之前要用它 vet baseline，之后每个数字也要 vet；`Hillclimb` playbook 在冻结 harness 之前要用它 vet harness。任何报告 before/after 的工作、回归声明、库或配置选型，都在它的触发范围内。
- **下游**：报告、PR body、以及 `Hillclimb` 的每次 keep-or-revert。冻结后的 hillclimb harness 会打印 error count 和 work count，于是这两问每次自动被检查。
- **共享**：`uptime`、`nproc`、`strace -c`（Linux）、`node --cpu-prof`、`py-spy`、`perf` 这些测量工具；`Opening a PR` 的主数字规则；principle 层的 `explain-the-number` 是同一族判断（源码没有点名它）。
- **对照**：`prove-it-works` 管"在真实产物上验证行为"，`benchmark-checklist` 管"数字本身的测量合法性"；`show-me-your-work` 的 result 列可以记下 verdict，这里定义 verdict 需要什么证据。

## 使用边界

- **它不是优化方法。** 它只 vet 测量，不告诉你该改哪里。
- **它不覆盖非性能数字。** 正确性、覆盖率、通过率不适用这七问。
- **ballpark 例外有条件。** 一次运行、必查 Q4 与 Q7、必须说明只有一次；选型不适用。
- **交错不可行时，证据强度会下降。** 源码只给了"交错并说明"这一条，没有给替代方案；两边无法交错运行时，报告仍然要交代这个缺口。
- **说不出 limiter 就是 inconclusive。** 这不是建议，是报告的硬条件。

## 值得注意

- **七问是人工清单，没有脚本强制。** 唯一的结构化装置是 `Hillclimb` 冻结 harness 打印 error 和 work count，其余各问靠执行者自律。
- **"一个主数字"与 Q5 的范围要求有张力。** 范围放在链接的 artifact 里，只读 PR body 的读者看不到噪声；这是用可读性换可复查性。
- **Q2 会让一些选型问题无法回答。** 生产配置无法在测量环境复现时，源码的选择是"不要从那次运行选赢家"，但没有说接下来该怎么办。
- **Q3 的 11% 是上界，不是预测。** 它只能用来否证"这个改动能带来 X%"，不能反过来承诺收益。

## 关键洞察

1. **先写 claim，再测。** 测量的对象是那句准备发出去的话。
2. **命名 limiter 是报告的最低门槛。** 没有 limiter 的数字没有解释力。
3. **先用算术否证，再考虑加运行次数。** 上限检查比多跑几次便宜得多。
4. **噪声属于报告。** 中位数和范围不是补充材料，是 verdict 的一部分。
5. **"工作真的发生了"是最容易漏的一问。** lazy 代码和 timeout 会产出看起来很正常的数字。
