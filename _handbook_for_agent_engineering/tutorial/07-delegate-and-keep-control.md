# 用 brief 派 subagent

> 教程 07 / 12 · 上一页：[06 用 tdd 先看见红](./06-build-a-feedback-loop.md) · 下一页：[08 用 verdict 证明](./08-prove-the-promise.md)

## 眼前困惑

你把一块工作交给 subagent，回来看见一句"做完了"。diff 里有些你没同意的改动，验证步骤也说不清跑没跑。问题通常不在模型，而在输入：你没有给出一份能让陌生执行者独立完成的 brief。

pstack 的 subagent 默认值是：后台运行、agent 模式（`readonly` 会剥掉 MCP）、给文件指针而不内联、按角色指定模型。源码：`skills/poteto-mode/SKILL.md#Subagents`。

## 现在动作

### 1. 写 brief 的九个字段

brief 是 GOAL / SCOPE / CONTEXT / ACCEPTANCE / VERIFY / TIMEBOX / FORBIDDEN / REPORT / STANDING。每个字段都要能独立读懂：

```text
GOAL: 让勾选导出只导出选中的行。
SCOPE: 可以写 exporter/ 与 export-page/；不改导出格式；独占 branch feat/export-selection。
CONTEXT: 类型在 src/exporter/types.ts；现有测试 dev-notes 里有 fixture。
ACCEPTANCE: 勾选 3 行导出 3 行，表头与全量一致；无勾选时按钮禁用并提示。
VERIFY: pnpm test exporter；再跑一次真实导出，贴行数。
TIMEBOX: 一个 session。
FORBIDDEN: 不 retarget/merge PR、不 rebase、不 force-push、不修 scope 外代码；不改 CSV 列顺序；不加依赖；不动权限。
REPORT: status、branch、head SHA、PR、verdict、实际跑了什么、偏离、建议的后续。
STANDING: <preferences.md 原文，逐字贴入>
```

源码：`skills/poteto-mode/playbooks/orchestrate.md#The brief`。

填不出的字段说明 unit 还没切好，先不要 spawn；含糊的 brief 会安静地失败，因为 worker 不能反问你。源码里有一句很直白：A field you cannot fill is a unit you have not scoped yet。

### 2. 默认开 fresh subagent

给新任务开新 subagent，带 consolidated scope：原始 brief、后续指令、上一个 agent 的报告和分支。只有新工作确实依赖旧 agent 的状态（本地 checkout、未提交改动、还在跑的进程）时才 resume。源码：`skills/poteto-mode/SKILL.md#Subagents`。

### 3. 选对 subagent 类型

playbook 步骤里的委派用 `poteto-agent`；`how`、`why`、`interrogate`、`reflect`、`swarm` 自带 `subagent_type`，不被覆盖。源码：`skills/poteto-mode/SKILL.md#Subagents`。

### 4. 父 agent 保留判断

子 agent 的产出要自己复核：看 diff、跑检查、写总结，不原样转述"我改好了"。源码：`skills/poteto-mode/SKILL.md#Subagents`。

### 5. `control surface` 不是父 agent 的控制权

pstack 的 `control surface` / `control skill` 指 `control-ui`、`control-cli` 这类驱动真实应用的东西。父 agent 保留的是 brief、todo 和最终判断；不要把这三样叫 `control surface`。

## 可见成果

一份九字段 brief，加一个明确的 subagent 类型与 fresh/resume 选择。

## 休息点

可以停在这里。下一步是把 subagent 的结果收回来，用 verdict 而不是自报来判断。

## 下一段

[08 用 verdict 证明](./08-prove-the-promise.md)：`prove-it-works` 看真实产物，`shipping` 要非作者 verifier 的 `PASS` 或 `PASS+NOTES`。
