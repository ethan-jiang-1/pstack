# mode 与 playbook

## 一句话

`poteto-mode` 同时是路由器和执行者：它读请求，匹配 23 个 playbook 之一，把步骤逐字复制进 todo，再按步骤调用其他 skill 和 subagent。流程由 playbook 决定，不由模型的临场发挥决定。

## 一次运行

1. **读请求。** 识别任务类型、要交付的产物、以及用户已经说出的验收方式（"repro first"、"land the stack"、"pixel exact"）。
2. **匹配 playbook。** 从 23 个里选一个。匹配依据是请求里的信号，不是关键词表。
3. **复制步骤。** 打开 `playbooks/<title>.md`，把步骤逐字放进 todo list 的最前面，然后才写任务特有的 todo。
4. **执行。** 每个步骤可能调用解释类 skill、设计类 skill、验证类 skill，或派 subagent。
5. **收口。** 需要开 PR 的 playbook 以 `Opening a PR` 结束；回复按 mode 的 **Writing the reply** 与 `unslop` 写。

### 为什么步骤要逐字复制

- **防转述漂移。** 模型概括步骤时会丢条件。"reproduce it yourself" 被概括成 "reproduce it"，责任主体就没了。
- **让跳过可见。** 不做的步骤不能删，要留 `skip: <reason>`。同理，不适用的维度写 `n/a: <reason>`。这让"没做"和"忘了"可区分。
- **让进度可审计。** todo list 是最便宜的状态记录；用户扫一眼就知道现在在第几步。

## 23 个 playbook

完整索引见 [playbook 索引](../playbooks/README.md) 与 `SKILL.md#Playbooks`。下表覆盖按请求信号选择的主要入口。另有三个按名字就能选的：`Authoring a skill`（写或改 `SKILL.md`）、`Pause safely`（显式暂停）、`Worktree cleanup`（回收磁盘）。`Opening a PR` 是共享收尾，不由请求直接选。

| 你看到的信号 | 先进 | 不要进 |
|---|---|---|
| "how does X work"，只要一个带引用的答案 | Investigation | Feature；不要顺手改代码 |
| 有可复现的缺陷 | Bug fix | Feature；修 bug 的步骤要求先复现 |
| 有测得的变慢 | Perf issue | Hillclimb；一次修复用 Perf issue |
| 一个指标要持续逼近目标 | Hillclimb | Perf issue；hillclimb 是循环 |
| 只有 trace / heap snapshot，没有活体 | Trace forensics | Runtime forensics |
| 有活体运行时症状（leak、idle-CPU、glitch） | Runtime forensics | Trace forensics；它不负责修 |
| 新行为，且能说清数据形状 | Feature | Prototype；形状未定时先做探针 |
| 行为不变，只改结构 | Refactoring | Feature |
| 一个设计选择需要证据才能定 | Prototype | Feature；用完就丢 |
| 两套实现要像素一致 | Visual parity | Visual parity 是专门流程，不是普通 review |
| 一个 skill 或 prompt 改动是否真的改变行为 | Eval | 凭感觉合并 |
| PR 要推到可合并 | Babysit | Shipping；babysit 不负责合并 |
| 栈已绿，要落地 | Shipping | Babysit；shipping 要求独立验证 |
| 一个任务跑到可检查的谓词 | Autonomous run | Orchestrate；一个 agent 能做完就不算 program |
| 一个项目跑多天、几十上百个 subagent | Orchestrate | Autonomous run |
| 一批独立 PR 全自动落地 | Autopilot-full | Shipping；autopilot 是队列形态 |
| 一批改动要形成一条待人工落地的栈 | Autopilot-stack | Autopilot-full；stack 不自动合并 |
| 接管别人未完成的工作 | Session pickup | Recall；recall 只重建上下文 |
| 下游工作跨阶段或跨 PR | Multi-phase plan | Feature；先定每个 phase 的边界 |

## 没有窄 playbook 时

有更窄的 playbook 能覆盖就用它。没有，且任务是一次跨切面的大改、或用户会离开后复查的工作，交给 `figure-it-out`：它为这一件事设计一份一次性 playbook。持续多天、多个 PR、几十上百个 subagent 的 standing program 交给 `Orchestrate`。区别是：`figure-it-out` 设计一次 run，`Orchestrate` 跑一个 program；`Autonomous run` 是把一个任务跑到谓词。这几条边界在正文里比名字清楚，选错时先读描述。

## gate 写在步骤里

playbook 的约束不是结尾一句"请谨慎"，而是步骤内的可检查条件：

- **Bug fix step 1**：在 matching surface 上用 control skill 自己复现；只有 control surface 确实够不到目标时才请用户，且要说明具体原因。
- **Bug fix step 2**：存活的机制有运行时证据；继续二分，不猜。
- **Shipping step 1**：每个 PR 由没写这份代码的 verifier 独立验证，`PASS` 或 `PASS+NOTES` 才算数；绿 CI 和 bot approval 都不是 verdict。
- **Shipping step 2**：只落地从最低未合并 PR 向上的 contiguous verified run；遇到第一个没有 `PASS` 或 `PASS+NOTES` 的就停。
- **Babysit step 1**：先声明 `drive`、`background`、`threads-only` 或 `check`，再轮询；未声明默认 `drive`，但小 PR 或 docs-only 默认 `check`。
- **Feature step 3**：写 throughput checkpoint（阻塞步骤、可并行工作流、共享可变状态、最小安全拆解），不适用也要留 `n/a`。
- **Benchmark**：报告任何性能数字前走 `benchmark-checklist` 的七问。

这些 gate 只在具体转移处生效，不是把 agent 变成每步都要请示的工作流引擎。

## 共享收尾

`Opening a PR` 自称在每个其他 playbook 的结尾被调用。它统一了 worktree、提交顺序、PR 形状、forge 选择、栈的规则和"开 PR 不等于开始 babysit"。

源码内部有一条张力：Investigation 和 Pause safely 明确说不开 PR；Runtime forensics 与 Trace forensics 只做诊断、未经要求不改代码；但 `opening-a-pr` 的描述写的是 "every other playbook"。合理读法是：它是需要交付改动时的默认收尾，不是无条件收尾。这让"最后一个步骤"只有一个事实来源。

## 边界

- **23 个 playbook 本身是发现成本。** mode 不做交互式澄清，它直接选。选错的代价在长流程里会被放大。
- **描述承担了路由。** playbook 的标题和描述是唯一的匹配信号，措辞变化会影响选择；这里没有单独的 router skill。
- **`poteto-mode` 是重依赖入口。** 它要求大多数 skill 同时在场；部分安装时，步骤会走到不存在的 skill。
- **"逐字复制"依赖执行者愿意照做。** 没有机械检查保证 todo 与 playbook 一致；这是纪律，不是编译器。

## 关键洞察

1. 路由器同时是执行器。选完流程后不换手，减少了"选完就忘"的损耗。
2. 步骤是状态。把步骤搬进 todo，比让模型记住一段长提示更可靠。
3. 兜底不是分类，而是三个真实出口：更窄的 playbook、`figure-it-out`、`Orchestrate`。
4. gate 要贴着转移写。抽象成"要谨慎"没有用；写成"在哪个表面、看到什么、才允许下一步"才有用。

## 源码

- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Playbooks、Non-negotiables）
- [`skills/poteto-mode/playbooks/`](../../skills/poteto-mode/playbooks/)
- [`playbooks/opening-a-pr.md`](../../skills/poteto-mode/playbooks/opening-a-pr.md)
- [`playbooks/shipping.md`](../../skills/poteto-mode/playbooks/shipping.md)、[`babysit.md`](../../skills/poteto-mode/playbooks/babysit.md)
