# 系统总览

## 一句话

pstack 的主入口是 `poteto-mode`。它把请求匹配到 23 个 playbook 之一，把该 playbook 的步骤逐字复制进 todo，再按步骤调用 skill、派 subagent。模型从设置文件里的 role line 开始，每个 spawn 前再按任务调整；完成的依据是 `verdict`、`benchmark-checklist` 和 `show-me-your-work` 的 decision trail。以下计数取自 plugin `0.15.25`（2026-10-19）：54 个 skill（其中 24 个 `principle-*`）和 23 个 playbook。

## 一次运行

### 0. 配置模型（可选）

`setup-pstack` 探测当前 harness 能传的模型 slug，问一个 budget，把每个 role 的模型写进设置文件。skill 派 subagent 时先读这份设置；role line 缺失时退回 skill 内默认值。Cursor 写 `~/.cursor/rules/pstack-models.mdc`，其他 harness 写 `~/.agents/pstack-models.md`。见 [role line 与模型设置](./model-panel.md)。

### 1. 用 mode 进入

`poteto-mode` 读请求，匹配一个 playbook，打开 `playbooks/<title>.md`，把步骤逐字复制进 todo。不做的步骤不删，留一行 `skip: <reason>`。匹配信号与 23 个入口见 [mode 与 playbook](./mode-and-playbooks.md)。

### 2. 在步骤里调用 skill

playbook 只规定顺序和交接点。例如 Bug fix 的步骤点名 `how`、`why`、`architect`、`tdd`，每一步的具体协议在各自的 `SKILL.md`。skill 默认不会自己触发：54 个里有 53 个写了 `disable-model-invocation: true`，只有 `setup-pstack` 例外。见 [调用模型](./invocation-model.md)。

### 3. 派 subagent

playbook 步骤里的委派默认用 `poteto-agent`；`how`、`why`、`interrogate`、`reflect`、`swarm` 各自规定 `subagent_type`，按 skill 写的类型派。父 agent 保留 brief、todo 和最终判断，子 agent 的报告要父 agent 自己复核，不能原样转述。

### 4. 验证与收口

"测试绿了"不是完成。`prove-it-works` 要求在真实产物上验证；`benchmark-checklist` 在报告任何性能数字前逐项检查；`create-verification-skill` 把"像用户一样驱动应用"固化成项目内的 `verify-*` control skill。需要开 PR 的 playbook 以 `Opening a PR` 收尾。`shipping` 只落地从最低未合并 PR 开始连续的 verified run，每个 PR 要有一个没写这份代码的 verifier 给出 `PASS` 或 `PASS+NOTES`。

## 其他入口

| 起点 | 入口 | 产物 |
|---|---|---|
| 不知道用哪个 skill，或 pstack 怎么配 | `poteto-help` | 回答 + 一段可直接发送的 prompt，不替用户开工 |
| 只想知道代码怎样运行 | `how` | 带文件定位的机制说明 |
| 想知道为什么这样设计 | `why` | 带引用的决策与权衡阅读 |
| 要把一套工作讲给一个人听懂 | `teach` | 组合 `how` 与 `why` 的一段说明 |
| 接手别人未完成的工作 | `poteto-mode` → Session pickup | 接管在途工作 + 当前状态 |
| 重建自己最近的工作上下文 | `recall` | 当前状态简报 + 下一步 |
| 离开一阵后要补课 | `ketchup` | 平实语言的变更摘要 |
| 任务本身没说清 | `sharpen` | 专家路径、真实目标、约束该松还是紧 |
| 环境反复出同一种错 | `correct` / `reflect` | 结构性检查 / 写回 skill 的改进 |
| 想把个人习惯变成 skill | `automate-me` | 一个定制的 `-mode` skill |

这些入口里，`how` 和 `why` 会被 mode 的步骤点名；`ketchup` 出现在 `Autopilot-full` 与 `Autopilot-stack` 的 Reply 里；Session pickup 是 playbook；其余是独立入口，不需要先跑 mode。

## 人和 agent 的分工

目标、产品取舍和 `Always pause` 的授权留在人手里；可逆工作由 agent 直接做并报告。operator 点名的 gate 即使在 full grant 下也要等。提问分类、full grant、product/preference call、operator-only call 与 `Always pause` 见 [Autonomy 与 gate](./autonomy-and-gates.md)。

## 容易误读的地方

1. **它几乎不自动触发。** 54 个 skill 里 53 个写了 `disable-model-invocation: true`。模型不会自己挑 skill；要么人输入命令，要么 mode 在 playbook 步骤里点名。
2. **23 个 playbook 不是一条单链。** mode 按请求选一个；`figure-it-out` 与 `Orchestrate` 是没有窄 playbook 时的两个出口。
3. **playbook 不是 prompt 模板。** 步骤要逐字复制，跳过的步骤要写原因；playbook 带 gate、失败出口和 `Reply:`。
4. **多模型不是所有流程的前提。** `arena` 和 `interrogate` 的收益来自不同模型；`swarm` 默认让每个 worker 用 `swarm workers` 的模型，模型竞速是可选。
5. **文档是交接接口。** PR 正文、回复、`show-me-your-work` 的 decision trail 都要能被下一位（人或 agent）复核。

## 关键洞察

1. 主流程只有一个入口，路由和执行不换手。
2. 步骤放进 todo，是最便宜的状态记录。
3. role line 让模型选择可以一处修改；skill 内默认值是缺行时的退路。
4. 能停下来的依据是可检查的 verdict、gate 和 decision trail，不是 agent 的自述。

## 系统页

其它系统页见 [系统页索引](./system-map.md)。

## 源码

- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Non-negotiables、Autonomy、Subagents、Playbooks）
- [`skills/poteto-mode/playbooks/`](../../skills/poteto-mode/playbooks/)
- [`skills/setup-pstack/SKILL.md`](../../skills/setup-pstack/SKILL.md)
- [`skills/show-me-your-work/SKILL.md`](../../skills/show-me-your-work/SKILL.md)
- [`plugin.json`](../../plugin.json)
