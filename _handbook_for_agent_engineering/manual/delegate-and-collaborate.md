# 委派：Subagent 与 brief

> 本章 4 个单元 · [手册目录](./README.md) · [上一章：设计与计划](./design-and-plan.md) · [下一章：验证与 verdict](./verify.md)

## 这一章解决什么

委派失控通常不是模型能力问题，而是输入问题：brief 缺字段，subagent 类型选错，或者父 agent 不看 diff 就转述"做完了"。pstack 把这些问题拆成三块：Subagents 的默认值、brief 的字段、fresh 与 resume 的边界。

| 单元 | 卡点 | 带走 |
|---|---|---|
| 3.1 | 不知道 subagent 默认该怎样跑 | `poteto-mode#Subagents` 的默认值 |
| 3.2 | brief 写完，worker 还是做偏 | 八字段 brief |
| 3.3 | 该新开还是续用？角色模型怎么选？ | fresh / resume 判断与 role line |

---

## 3.1 Subagents 的默认值

### 工作情境

你让一个 subagent 改导出器。它改了，但没跑测试，也没告诉你改了哪些文件。回来看 diff，发现它顺手重构了页面。

pstack 的 Subagents 一节先把默认值定死：后台运行、agent 模式、给文件指针、按角色指定模型、父 agent 自己复核。源码：`skills/poteto-mode/SKILL.md#Subagents`。

### 做法

- code-writing 的委派默认用 `poteto-agent`。`how`、`why`、`interrogate`、`reflect`、`swarm` 自带 `subagent_type`，不被覆盖。
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

## 3.2 brief 的八个字段

### 工作情境

你写了一句"把勾选导出做完"，worker 做出来的东西和你想的不一样。它不能反问你，只能按自己的默认值补。

`orchestrate` 的 brief 有八个字段，缺一个就少一个约束。源码：`skills/poteto-mode/playbooks/orchestrate.md#The brief`。

### 带走什么

```text
GOAL: 要改变什么，为谁。
SCOPE: 哪些文件、模块、层；不做什么。
CONTEXT: 权威输入在哪；指路径，不内联。
ACCEPTANCE: 什么算完成，能通过或失败。
VERIFY: 跑什么命令、看什么表面、贴什么输出。
TIMEBOX: 一个 session，还是更长。
FORBIDDEN: 不许动什么；不许加什么依赖。
REPORT: 回来时必须给什么。
```

### 做法

1. 先写 GOAL 和 ACCEPTANCE。写不出 ACCEPTANCE，说明工作还没切开。
2. SCOPE 和 FORBIDDEN 分开写。SCOPE 说"做哪里"，FORBIDDEN 说"哪里不能碰"。
3. CONTEXT 只给指针：spec、issue、文件路径、上一个 agent 的报告。不要把整段聊天粘进去。
4. VERIFY 写成可执行命令或可观察表面。`prove-it-works` 认的是真实产物，不是"我跑过了"。
5. REPORT 规定回来时的形状：改了哪些文件、跑了什么、结果、未解风险。

源码里有一句判断标准：A field you cannot fill is a unit you have not scoped yet。

### 质量门

- 八个字段都有值；没有"待定"。
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
- 角色行从设置文件读：`judgment and prose`、`hardest tasks`、`how explorer`、`how explainer`、`why investigators`、`why synthesizer`、`reflect tooling`、`reflect judgment, divergent, synthesizer`、`arena runners`、`arena cross-judge pool`、`swarm workers`、`architect runners`、`interrogate reviewers`。
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

- **Coordinator（本地）**：只写 brief、排空 inbox、维护 human report、做判断；绝不写代码。冲突合并、chain rebase、代码改动都是 task。
- **Sub-coordinator**：只在 program 超过一个 coordinator 的 drain 能力时加；每条 track 一个，持久、本地。它能自己 drain 的 track 不需要中间层；嵌套最多 3 层。
- **Worker / verifier**：默认 cloud，除非任务需要本机（`control-ui` / `control-cli`、本地 transcript、模拟器、只在本机的 auth）。一个 worktree 或 branch 一个写者；verifier 与 worker 不同模型家族。
- **brief 是产品**：每个 spawn 都带完整八字段；填不出的字段说明 unit 还没切好。
- **状态**：`units.tsv` 一行一个 unit；读写走 `scripts/orch/orch.ts`，一条命令进、一行出。
- **协作**：`preferences.md` 记团队偏好；单一 stacker 处理文本冲突；coordinator 不做代码冲突。

### 质量门

- Coordinator 没有直接改代码。
- 每个 worktree / branch 只有一个写者。
- 每个 unit 有 brief、verdict、PR 和 head SHA。
- 嵌套只在确有必要时出现，且不超过 3 层。

---

## 本章与 pstack 的边界

pstack 用 `orchestrate` 的 roles、`preferences.md`、one writer per worktree 和单一 stacker 处理协作；文本冲突由 stacker 处理。不要给这些机制另起名字。
