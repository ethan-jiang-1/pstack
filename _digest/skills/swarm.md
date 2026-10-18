# `swarm`

> 源码：[`skills/swarm/SKILL.md`](../../skills/swarm/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

把 N 个 worker 一次派出去，可以各自覆盖一个切片，也可以拿同一个 brief 竞速；父 agent 等它们全部结束，按事先声明的收敛规则聚合，返回一份带结果表、一行一条证据的问题、以及明确缺口的报告。

## 它解决什么

有些工作不是"想不出方案"，而是"覆盖面太大"：每个 package 跑一遍检查、十个场景各验证一次、同一个问题让几个 worker 独立试。串行做会拖长周期，而且父 agent 的上下文会被每一份原始输出占满。

`swarm` 把这类工作变成一次扇出加一次收敛。它和 `arena` 的区别在目的：arena 要的是从多个候选里合成一个更好的产物，swarm 要的是把一片区域扫完，或者用多个独立尝试竞速。

## 核心机制

### 1. Frame 先定完成条件和收敛规则

开始前写清 done predicate，以及 swarm 必须返回的产物或报告。然后选形状：按切片分区、N 个 worker 在同一份 brief 上竞速，或两者混合。

竞速和混合形状必须在派出之前声明收敛规则：`first pass`、`rank all`、`best-of`。这三者对应三种完全不同的读法，事后选一个会让结果偏向当时看起来最好的那份。

N 是 worker 总数，不是云的并发上限。worker 模型取设置文件 `swarm workers` 行，默认 `grok-4.7-xhigh-fast`；`auto` / `inherit-parent` 时省略 `model`。竞速里每一臂的模型要提前点名。

### 2. brief 是产品

每个 worker 要有自己的可写输出位置。验证或测量 commit 的 brief 必须点名精确的 SHA；测量 brief 还要点名方法（样本数、一个样本是什么、顺序），worker 在结果里两者都记。

派出时一条消息、`subagent_type: generalPurpose`、`environment: "cloud"`、后台运行。只有 worker 需要访问用户电脑上的东西时才用 `environment: "local"`。从非默认的已推送分支开始时传 `cloud_base_branch`。

每份 brief 必须自包含：目标、范围、确切的切片或竞速臂、怎么验证、报什么。报告用 `PASS`、`ISSUES`、`BLOCKED` 三态并附证据。能证明缺陷的 worker 报 `ISSUES`，并列出它**能证明的每一个**问题，不只第一个。

### 3. Aggregate 按证据收货

读终态结果。没记录 brief 要求的 SHA 和方法的结果直接丢掉，并重派该 worker 一次；第二次仍缺就记为一个 gap。gap 不算 pass。

覆盖形状下，每个必需切片都要有结果；竞速形状下，套用事先声明的那条规则。不要把 worker 的原始 dump 贴回来，聚合是判断，不是转发。最终保留一张紧凑结果表、一行一条有证据的问题、以及明确的 gap 和 dropout。

### 4. Report 只有一份

在对话里返回一份合并报告：结果表、问题一行、缺口或掉线、以及用到时的竞速规则。

## 设计意图

- **报告形状先定。** brief 就是契约；worker 不能反问你，含糊的 brief 会安静地失败。
- **收敛规则前置。** `first pass` 和 `best-of` 对同一批结果的解释完全不同；先声明，才能防止事后挑一个自己喜欢的结果。
- **SHA 与方法进 brief。** 这是把"我测过了"变成可复查材料的两个字段；缺了就无法判断结果对应哪一版、按什么口径。
- **三态而不是成败二分。** `BLOCKED` 既不是失败也不是通过，必须单独记录，否则会被当成通过。
- **列全可证明的问题。** 一个 worker 停在第一个缺陷上，会隐藏它已经看到的其余缺陷。
- **掉线可接受，隐瞒不可接受。** N-1 是合法降级，但必须写进报告。

## 依赖关系

- **上游**：`poteto-mode` 的并行风扇路由（覆盖矩阵、竞速、gauntlet、探索分区）指向 swarm；`multi-phase-plan` 要求每个 PR 的 live 验证跑十条 lane，每 lane 一个场景、一张截图、一个 pass 谓词，按 swarm 的形状扇出；`autopilot-full` 在 code-ready head 上跑一次完整 swarm 验证。用户也可以直接 `/swarm`。
- **下游**：父 agent 把 lane 结果和截图路径写进 PR 的验证块或 merge-ready 报告；`multi-phase-plan` 的 live block 就是按这个形状消费结果的。
- **共享**：设置文件 `swarm workers` 行；`poteto-mode` 的 **Subagents** 规则；Cursor 的 cloud agent 参数；principle 词汇 `prove-it-works`、`separate-before-serializing-shared-state`。
- **对照**：`arena` 同样并行，但要选 base 并 graft；`orchestrate` 是长期 program，有队列、frontier 与 in-flight 上限；`interrogate` 是对一个改动的多模型评审。

## 使用边界

- **不做设计合成。** 要比较形状并合成一个更好的，用 `arena`。
- **不是单点任务的默认选择。** 一个 worker 能做的事，扇出只是更贵。
- **worker 之间不共享状态。** 需要跨切片协调时，先按 seam 分区，而不是让 worker 互相通信；共享写入会互相破坏。
- **云端隔离是 Cursor 专有的。** 别的 harness 里 worker 本地跑，源码只要求给每个 worker 自己的 worktree 或输出路径；隔离假设和云端已经不是一回事。
- **测量 brief 的方法要求有成本。** 没写方法的结果会被丢弃并重派一次，这是显式的时间成本。

## 值得注意

- **竞速各臂的方法不必相同。** 源码要求测量 brief 点名方法，但没有要求所有臂用同一个方法；`rank all` 的比较基础因此可能不成立。
- **`rank all` 只规定要声明，没规定怎么排。** 谁排序、用哪些指标、平局怎么办，都留给执行者。
- **"列全问题"与"报告紧凑"有张力。** worker 被要求列出每个能证明的问题，聚合时又要求一行一条、不贴 dump；压缩发生在父 agent 侧，靠后的发现容易丢证据。
- **有 gap 时整体算不算完成，源码没有回答。** 它只说 gap 不算 pass，没说报告能否在存在 gap 时交付。

## 关键洞察

1. **扇出买的是覆盖和周期，不是判断。** 判断仍由父 agent 在做。
2. **brief 的质量决定上限。** worker 不会追问，缺的字段会变成安静的失败。
3. **先声明收敛规则，再看结果。** 否则竞速退化成挑一个顺眼的。
4. **证据字段要短而硬。** SHA、方法、截图路径；散文不能当证据。
5. **缺口是一等公民。** 掉线、gap、`BLOCKED` 都要出现在报告里，否则覆盖面是假的。
