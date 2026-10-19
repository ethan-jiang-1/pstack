# Playbook 索引

23 个 playbook 是 `poteto-mode` 的流程库。每个文件是一份步骤协议；多数以"你拥有什么"开头、以 Reply 结尾，少数（如 worktree cleanup、opening a PR）只给操作规则或共享收尾。

> 不要从这里直接"挑一个跑"。playbook 由 `poteto-mode` 匹配；这一页是给已经知道要做什么、但想确认边界的人看的。

## 选择顺序

```text
只读问题，交付一个带引用的答案        → Investigation
有可复现的缺陷                        → Bug fix
有测得的变慢                          → Perf issue
一个指标要持续逼近目标                → Hillclimb
只有活体症状，没有捕获                → Runtime forensics
只有捕获的 profile / trace / heap     → Trace forensics
新行为或行为变化                      → Feature
行为不变，只改结构                    → Refactoring
一个设计选择需要先看见才能定          → Prototype
两套实现要像素一致                    → Visual parity
改 skill / prompt，要测行为变化       → Eval / Authoring a skill
PR 要推到可合并                       → Babysit
栈已绿，要落地                        → Shipping
一个任务跑到可检查的谓词              → Autonomous run
一个项目跑多天、几十上百个 subagent   → Orchestrate
一批独立 PR 全自动落地                → Autopilot-full
一批改动要形成待人工落地的栈          → Autopilot-stack
接管别人未完成的工作                  → Session pickup
要干净地暂停                          → Pause safely
跨阶段或跨 PR                         → Multi-phase plan
回收磁盘                              → Worktree cleanup
没有更窄的 playbook 可用              → figure-it-out（skill，不是 playbook）
需要开 PR 的工作的收尾            → Opening a PR
```

## 理解与诊断

### [Investigation](../../skills/poteto-mode/playbooks/investigation.md)

只读。经 `how` 产出带文件定位的解释，动机问题再加 `why`；如果是二选一，给带 trade-off 表的推荐。不写代码、不开 PR。throughput checkpoint 固定写 `n/a, read-only investigation`。

### [Bug fix](../../skills/poteto-mode/playbooks/bug-fix.md)

先在该表面上自己复现，再二分排除假设，用运行时证据确认机制，然后规划修复、验证、把失败复现排在修复提交之前。回复要逐字粘贴失败到通过的输出。

### [Perf issue](../../skills/poteto-mode/playbooks/perf-issue.md)

一次性的性能修复。先抓基线 trace 并用 `benchmark-checklist` 检查每个数字；按"不做 → 不重复做 → 少做 → 晚做 → 趁人不注意时做 → 并发做 → 更便宜地做"的顺序找最便宜的优化。不先跑就宣称性能上限。

### [Hillclimb](../../skills/poteto-mode/playbooks/hillclimb.md)

持续改进一个指标。一次只改一个假设，一次只测一次，保留或回退；决策日志一行一次尝试。停止谓词同时要求目标和最少尝试次数，防止早赢收工。与 Perf issue 的区别是循环，不是单次修复。

### [Runtime forensics](../../skills/poteto-mode/playbooks/runtime-forensics.md)

诊断活体症状：通过控制技能抓 CPU profile、heap snapshot 或 CDP trace，把大 artifact 交给 subagent 归约，用 CDP 注入 instrumentation 证明机制，最后映射回源码。交付诊断，不交付修复。

### [Trace forensics](../../skills/poteto-mode/playbooks/trace-forensics.md)

诊断已经捕获好的 artifact。先把 cpuprofile、trace、spindump 或 heap snapshot 转成可查询的形状（例如 sqlite），再查热点与 retainer chain，最后按 artifact 自带符号映射到源码。没有符号就明说。

## 构建

### [Feature](../../skills/poteto-mode/playbooks/feature.md)

新行为或行为变化。先 `how` 再 `architect`，写四维 throughput checkpoint，委派实现，在真实表面验证，重排成小的有序提交。设计有争议时先 `interrogate`。

### [Refactoring](../../skills/poteto-mode/playbooks/refactoring.md)

行为不变，只改结构。先钉住行为契约（characterization test、snapshot 或等价 harness），再命名缺失的结构与目标形状，先减后加，小步移动，每一步保持 pin 绿。不引入兼容 shim。

### [Prototype](../../skills/poteto-mode/playbooks/prototype.md)

一次性的探针，用来做一个设计或行为决定。速度优先，代码质量不重要；变化放在隔离的 scratch 目录；视觉决定截图比较，行为决定直接观察。产出是决定，不是可交付代码。

### [Visual parity](../../skills/poteto-mode/playbooks/visual-parity.md)

像素级等价。先建基线截图，再逐组件迁移，用 image diff 判定，零差异才算过。禁止改基线或改 harness 让 diff 通过。

## 验证与交付

### [Eval](../../skills/poteto-mode/playbooks/eval.md)

测一个 skill、结构或 prompt 改动是否真的改变 agent 行为。核心是盲法：候选看到的目录、文件名和 prompt 都不能出现 eval/judge/rubric 等元信息；从 transcript 判断它真的读了什么，不看自报。

### [Babysit](../../skills/poteto-mode/playbooks/babysit.md)

把 PR 或栈推到可合并。先声明 `drive` / `background` / `threads-only` / `check`，只处理最低未合并的 frontier；顺序是冲突、评审线程、CI；CI 失败先分类再重试；bugbot 的评论按证据逐条处置。不负责合并。

### [Shipping](../../skills/poteto-mode/playbooks/shipping.md)

栈已绿之后落地。每个 PR 由没写代码的 verifier 独立跑真实表面并给 verdict；只落地从最低未合并 PR 开始连续通过的那一段；用 patch-id 检查 verdict 是否仍覆盖当前 head；自底向上一次合一个，落地过程中不再动栈。

### [Opening a PR](../../skills/poteto-mode/playbooks/opening-a-pr.md)

需要开 PR 的 playbook 的共享收尾。worktree、提交节奏、Conventional Commits 标题、PR 正文六节（Why / What changed / Scope（总是保留）/ Tradeoffs / Blast Radius / Verification；其余没有内容的节可省略）、forge 解析、栈的规则，以及"开 PR 不等于开始 babysit"。

## 自主运行

### [Autonomous run](../../skills/poteto-mode/playbooks/autonomous-run.md)

一个任务跑到一个可检查的谓词。先写退出条件，再选唤醒机制（事件 watcher 或定时 heartbeat），每轮做最小改动、验证、决定保留或丢弃；中途发现的问题自己处理，但不放松谓词。

### [Orchestrate](../../skills/poteto-mode/playbooks/orchestrate.md)

一个持续数天、几十上百个 subagent 的 program。coordinator 写 brief、排空队列、守 frontier、做判断，不写代码；sub-coordinator 按 track 分；验证按单元成本决定：便宜的由 worker 跑、coordinator 抽查收据，昂贵、判断密集或高爆炸半径的才派独立 verifier，并用不同模型家族。brief 是唯一产品。

### [Autopilot-full](../../skills/poteto-mode/playbooks/autopilot-full.md)

一批独立 PR 全自动落地。每个 PR 一个 owner，从构建到合并；root 用没写代码的 swarm 独立验证后再放行。只有队列内的项被授权，operator 明确保留的项停在 merge-ready。

### [Autopilot-stack](../../skills/poteto-mode/playbooks/autopilot-stack.md)

一批改动形成一条线性、base 链接的 PR 栈，交给 operator 评审和落地。owner 只报告 STACK-READY；拓扑只有一个写者（root）；落地开始后不再动链。

### [Session pickup](../../skills/poteto-mode/playbooks/session-pickup.md)

接管别人的在途工作。先定位并读原轨迹（transcript、cloud URL 或已推送分支），重建操作状态，diff 已完成与待办，然后路由到对应 playbook。不要重做已经完成的工作。

## 维护

### [Pause safely](../../skills/poteto-mode/playbooks/pause-safely.md)

干净地暂停。在原子步骤边界停下，取消 nested subagent，不启动新工作，不推送（除非之前已经推过）；把未提交改动做成一个 `wip:` 提交；在上下文之外写恢复说明。只用于明确暂停，不用于"继续"。

### [Worktree cleanup](../../skills/poteto-mode/playbooks/worktree-cleanup.md)

回收磁盘：worktree 与陈旧的 iOS 模拟器。先审计 worktree（大小、年龄、合并状态、未提交工作、PR 状态、最近会话），把删除建议当参考而不是许可；被固定的会话、正在使用的 worktree、未提交工作都要停下来确认。这是唯一没有代码评审兜底的删除流程。

## 元

### [Authoring or modifying a skill](../../skills/poteto-mode/playbooks/authoring-a-skill.md)

写或改 `SKILL.md`。走 skill-authoring 工具，校验 frontmatter、引用文件和跨 skill 链接，结构性的改动要有测试用例。拿不准就删。

### [Multi-phase or multi-PR plan](../../skills/poteto-mode/playbooks/multi-phase-plan.md)

工作跨阶段或跨堆叠 PR。先写每个 phase 的边界、产物与验证、PR 的切法与顺序；计划本身是交付物，执行要等 operator 明确 go，并走计划里指定的执行 playbook。

## 共同结构

多数 playbook 是同一个骨架：

```text
### <Title>

**You own <something>. <一句边界>.**

1. 步骤（带可检查条件与失败出口）
...
N. Run **Opening a PR**.   ← 只读或不再交付改动的流程以 Reply 结束，不执行这一步

**Reply:** 这次交付必须包含的文本形状。
```

遵守这个骨架的 playbook，`You own` 定义责任，步骤定义顺序，`Reply` 定义交付面。三者缺一，就会退化成一份提示词建议。

## 值得注意

- **有重叠。** `Babysit` 与 `Shipping`、`Autonomous run` 与 `Orchestrate`、`Autopilot-full` 与 `Autopilot-stack` 都需要读描述才能分对。
- **`figure-it-out` 不是 playbook。** 它是一个 skill，用来在没有合适 playbook 时设计一份一次性流程。
- **playbook 依赖 mode 的上下文。** 单独读一份 playbook 能理解步骤，但"逐字复制进 todo"、"按角色选模型"、"回复按 unslop"这些要求写在 `poteto-mode` 里。
- **不是所有 playbook 都开 PR。** 是否开由步骤决定；Investigation、Runtime forensics、Trace forensics、Pause safely 明确不开。
