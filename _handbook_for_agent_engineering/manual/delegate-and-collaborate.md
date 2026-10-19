# 委派：Subagent 与 brief

> 本章 4 个单元 · [手册目录](./README.md) · [上一章：设计与计划](./design-and-plan.md) · [下一章：验证与 verdict](./verify.md)

## 这一章解决什么

委派失控通常不是模型能力问题，而是输入问题：brief 缺字段，subagent 类型选错，或者父 agent 不看 diff 就转述"做完了"。pstack 把这些问题拆成四块：Subagents 的默认值、brief 的字段、fresh 与 resume 的边界，以及 `orchestrate` 的多天 program。

| 单元 | 卡点 | 带走 |
|---|---|---|
| 3.1 | 不知道 subagent 默认该怎样跑 | `poteto-mode#Subagents` 的默认值 |
| 3.2 | brief 写完，worker 还是做偏 | brief 的九个字段 |
| 3.3 | 该新开还是续用？角色模型怎么选？ | fresh / resume 判断与 role line |
| 3.4 | 多个 agent 跑多天 program | `orchestrate` 的 roles、one writer、单一 stacker |

---

## 3.1 Subagents 的默认值

### 工作情境

你让一个 subagent 改导出器。它改了，但没跑测试，也没告诉你改了哪些文件。回来看 diff，发现它顺手重构了页面。

pstack 的 Subagents 一节先把默认值定死：后台运行、agent 模式、给文件指针、按角色指定模型、父 agent 自己复核。源码：`skills/poteto-mode/SKILL.md#Subagents`。

### 做法

- playbook 步骤里的任何 subagent 默认用 `poteto-agent`，不只是写代码。`how`、`why`、`interrogate`、`reflect`、`swarm` 自带 `subagent_type`，不被覆盖。
- 默认后台运行（`run_in_background: true`），agent 模式；`readonly` 会剥掉 MCP，除非 skill 明确要只读，否则不要加。
- 给文件指针，不把整段上下文内联。
- 每个角色显式指定模型；难度决定 effort，参考 `setup-pstack` 的 role lines。
- 父 agent 拥有每个 subagent 的产出：自己看 diff、自己跑检查、自己写总结，不原样转述。

### 质量门

- brief 里有明确的验收与验证命令。
- subagent 类型正确；没有把自带 `subagent_type` 的 skill 覆盖成 `poteto-agent`。
- 父 agent 看过了 diff 和验证输出。

### 进阶

后台不是默认就安全。`readonly: true` 会剥掉 MCP；需要查 ticket、chat、trace 时，用 agent 模式并在 prompt 里限制可写范围。

---

## 3.2 brief 的九个字段

### 工作情境

你写了一句"把勾选导出做完"，worker 做出来的东西和你想的不一样。它不能反问你，只能按自己的默认值补。

brief 有九个字段，缺一个就不能 spawn；缺字段是拒绝条件，不是少一个约束。`STANDING` 把 `preferences.md` 逐字贴进去。源码：`skills/poteto-mode/playbooks/orchestrate.md#The brief`。

### 带走什么

```text
GOAL: 一句话，结果，一个没有聊天记录的人也能执行。
SCOPE: 这个 unit 可以写的路径；不可以写的路径；独占的 worktree 或分支。
CONTEXT: 文件和 PR 的指针；依赖的上游报告逐字粘贴，因为 worker 看不到兄弟。
ACCEPTANCE: 可检查的判据，一行一条。
VERIFY: 确切的命令或 control skill 路径，加上已知的坑。
TIMEBOX: 运行时间上限；到点返回部分发现并停下，不要跑过头。
FORBIDDEN: 不 retarget 或 merge PR、不 rebase、不 force-push、不修 scope 外的代码，加 unit 专属的禁令。
REPORT: status、branch、head SHA、PR、verdict、实际跑了什么、偏离、建议的后续。
STANDING: <preferences.md 原文，逐字贴入>
```

### 做法

1. 先写 GOAL 和 ACCEPTANCE。写不出 ACCEPTANCE，说明工作还没切开。
2. SCOPE 和 FORBIDDEN 分开写。SCOPE 说"做哪里"，FORBIDDEN 说"哪里不能碰"。
3. CONTEXT 用文件和 PR 指针；依赖的上游报告逐字粘贴，因为 worker 看不到兄弟。不要把整段聊天粘进去。
4. VERIFY 写成可执行命令或可观察表面。`prove-it-works` 认的是真实产物，不是"我跑过了"。
5. REPORT 规定回来时的形状：status、branch、head SHA、PR、verdict、实际跑了什么、偏离、建议的后续。

源码里有一句判断标准：A field you cannot fill is a unit you have not scoped yet。

### 质量门

- 九个字段按 unit 大小收放；一条命令的 unit 可压成一段，仍点名 goal、scope、verify、report；填不出的字段拒绝 spawn。
- ACCEPTANCE 能通过或失败。
- VERIFY 是命令或真实表面，不是"检查一下"。
- FORBIDDEN 至少挡住一种扩大解释。

---

## 3.3 fresh、resume 与角色

### 工作情境

一个 subagent 做到一半，你要它继续。是 resume，还是新开一个？

pstack 的默认是 fresh：新任务带 consolidated scope（原始 brief、后续指令、上一个 agent 的报告和分支）。只有新工作确实依赖旧 agent 的状态（本地 checkout、未提交改动、还在跑的进程）时才 resume。源码：`skills/poteto-mode/SKILL.md#Subagents`。

### 做法

- fresh 时，把上一个 agent 的报告和分支指针放进 CONTEXT，不要把它的整段对话塞进去。
- resume 只在状态搬不走时用；resume 链会安静地丢指令，源码明确警告过。
- 角色行从设置文件读，例如 `feature, refactoring`、`bug-fix`、`perf-issue`、`hillclimb`、`judgment and prose`、`hardest tasks`、`how explorer`、`how explainer`、`why investigators`、`why synthesizer`、`reflect tooling`、`reflect judgment, divergent, synthesizer`、`arena runners`、`arena cross-judge pool`、`swarm workers`、`architect runners`、`interrogate reviewers`。
- 角色行缺失时，退回 skill 内默认值；不要把默认值硬编码进 brief。

### 质量门

- 新任务默认 fresh；resume 有明确的状态理由。
- 每个 subagent 有角色行或 skill 默认值。
- brief 里没有"继续上次"这种无法执行的指令。

---

## 3.4 用 `orchestrate` 跑多天 program

### 工作情境

工作超出一个 agent 的一个 session：几十个 stacked PR、几十到几百个 subagent、operator 一天看两次。这时用 `orchestrate`。源码：`skills/poteto-mode/playbooks/orchestrate.md`。

### 做法

- **Coordinator（本地）**：写 brief、排空 inbox、维护 human report、做判断；绝不写代码。冲突合并、chain rebase、代码改动都是 task。
- **Sub-coordinator**：只在 program 超过一个 coordinator 的 drain 能力时加；每条 track 一个，持久、本地。它能自己 drain 的 track 不需要中间层；嵌套最多 3 层。
- **Worker / verifier**：默认 cloud，除非任务需要本机（`control-ui` / `control-cli`、本地 transcript、模拟器、只在本机的 auth）。一个 worktree 或 branch 一个写者；verifier 与 worker 不同模型家族。
- **brief 是产品**：每个 spawn 都带按 unit 大小收放的 brief；一条命令的 unit 可以压成一段，但仍点名 goal、scope、verify 和 report。填不出的字段说明 unit 还没切好。
- **状态**：`units.tsv` 一行一个 unit；读写走 `scripts/orch/orch.ts`，一条命令进、一行出。
- **协作**：`preferences.md` 记团队偏好；单一 stacker 只能改拓扑（chain rebase、force-push、base retarget、merge）；合并冲突和代码改动都是 task。

### 质量门

- Coordinator 没有直接改代码。
- 每个 worktree / branch 只有一个写者。
- 每个 unit 有 brief、verdict、PR 和 head SHA。
- 嵌套只在确有必要时出现，且不超过 3 层。

---

## 本章与 pstack 的边界

pstack 用 `orchestrate` 的 roles、`preferences.md`、one writer per worktree 和单一 stacker 处理协作；stacker 只改拓扑，合并冲突和代码改动都是 task。不要给这些机制另起名字。
