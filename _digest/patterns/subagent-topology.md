# Subagent 与 brief

`poteto-mode` 的 Subagents 一节定基本约束，`orchestrate` 的 brief 定输入，两者合起来才是一次委派。父 agent 保留 playbook、todo 和最终判断；subagent 是输入输出封闭的执行单元。

## `poteto-mode` 的 Subagents

- playbook 步骤里派的 subagent 用 `subagent_type: "poteto-agent"`。`how`、`why`、`interrogate`、`reflect`、`swarm` 自带 `subagent_type`，按 skill 写的来，不覆盖成 `poteto-agent`。
- 每次 `Task` 调用的默认：`run_in_background: true`、`readonly: false`（`readonly` 会剥掉 MCP）、给文件指针而不是内联上下文、每个角色显式指定模型。
- 角色行缺失时用 skill 默认值；`inherit-parent` 或 `auto` 时省略 `model`。难度决定档位：最难的工作交给 `hardest tasks` 的模型，例行的机械改动交给快的 code 模型。
- 默认 fresh subagent：新任务带 consolidated scope，即原始 brief、之后每一条指令、上一个 agent 的报告和分支。只有新工作确实依赖它持有的状态（本地 checkout、未提交改动、还在跑的进程）时才 resume。interrupt 过的 resume 会静默丢指令。
- 父 agent 拥有 subagent 的工作：自己看 diff，自己写总结，不原样转述。

源码：[`skills/poteto-mode/SKILL.md#subagents`](../../skills/poteto-mode/SKILL.md#subagents)。

## `orchestrate` 的 brief

九个字段：GOAL / SCOPE / CONTEXT / ACCEPTANCE / VERIFY / TIMEBOX / FORBIDDEN / REPORT / STANDING。STANDING 把 `preferences.md` 逐字贴进去。

- GOAL 一句话，一个没有 chat 访问权限的陌生人也能执行。
- SCOPE 写可写的路径、不可写的路径、独占的 worktree 或 branch。
- CONTEXT 给文件与 PR 指针；依赖的上游报告要全文贴进来，因为 worker 看不到 sibling。
- ACCEPTANCE 一条一行，可检查。VERIFY 给准确命令或 control skill 路径，加已知 gotcha。TIMEBOX 到期返回部分结论并停。FORBIDDEN 写禁止动作。REPORT 要 status、branch、head SHA、PR、verdict、实际跑了什么、偏差、后续建议。
- 填不出的字段说明这个 unit 还没 scoped，是 refuse-to-spawn 条件。
- brief 要按 unit 大小伸缩。一条命令的 unit 可以压成一段，但仍要含 goal、scope、verify 命令和 report 形状。
- 永远不要 resume-chain 一条 brief。用 fresh subagent 加 consolidated scope。

源码：[`skills/poteto-mode/playbooks/orchestrate.md#the-brief`](../../skills/poteto-mode/playbooks/orchestrate.md#the-brief)。

## 其他 skill 的 subagent 形状

- `swarm`：brief 各自独立，写 goal、scope、slice 或 race arm、怎么验证、report 什么。报告用 `PASS` / `ISSUES` / `BLOCKED` 加证据。结果没记录 brief 要求的 SHA 与方法就丢掉并重派一次，第二次仍缺记 gap；gap 不算 pass。源码：[`skills/swarm/SKILL.md`](../../skills/swarm/SKILL.md)。
- `arena`：每个候选有自己的输出位置，因为候选会写文件；跨候选共享写入靠先分开。源码：[`skills/arena/SKILL.md`](../../skills/arena/SKILL.md)。
- `maintain-verification-skill`：每个 feature 一个只读 subagent 读源码，children 不驱动应用也不编辑文件；返回 feature summary、source entry points、likely drift、one recipe。源码：[`skills/maintain-verification-skill/SKILL.md`](../../skills/maintain-verification-skill/SKILL.md)。
- `orchestrate`：coordinator → sub-coordinator → worker / verifier，嵌套最多 3 层；in-flight 子 agent 约 10 个，滚动窗口，不做阻塞批次；每个 worktree 一个写者；unit 的 verifier 换一个模型家族。源码：[`skills/poteto-mode/playbooks/orchestrate.md#roles-and-placement`](../../skills/poteto-mode/playbooks/orchestrate.md#roles-and-placement)。

## 代价

- 成本。每个 subagent 是一次完整生成，N 个 worker 是 N 份 token。
- brief 质量决定上限。`orchestrate` 说 brief 就是产品；含糊的 brief 会安静地失败，因为 worker 不能反问你。
- 共享写入会互相破坏。同一 branch、同一文件、同一 key 上的并发要先分开，不能靠锁。
- 多层会重复付上下文成本。每嵌套一层都要重读一遍上下文。

## 关键洞察

1. subagent 的可靠性来自 brief，不来自模型。
2. fresh 是一种正确性策略，避免指令在 resume 链里悄悄丢失。
3. 父 agent 的复核不可省，否则多 agent 只是把同一个盲点复制了多份。
