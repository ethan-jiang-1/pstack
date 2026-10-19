# `poteto-mode`

> 源码：[`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

`poteto-mode` 是一整套工作风格的入口，不是一个流程：它把"严谨、少写、验证过"拆成 24 条 principle 和 23 个 playbook（plugin 0.15.25 的计数）、一套 subagent 默认值、一份回复文体，再用一张触发表接到具体请求上。

## 它解决什么

同一个 agent 在同一天里，行为可以差很远：该先复现的 bug 直接改了，该独立验证的 PR 用绿 CI 顶替，subagent 用哪个模型全看当下顺手。人的约束写在 prompt 里，只能覆盖这一轮；下一轮换个说法，约束就没了。

`poteto-mode` 把这些约束挪进仓库，让它们有名字、有正文、能被回复引用。它要解决的不是"agent 不会做"，而是"agent 每次做的标准不一样"。

## 核心机制

### 1. **Non-negotiables** 是触发表

这一节排了 18 条"条件 → 该做什么"。它们不是步骤，而是在某个转移点必须想起的东西：

- 非平凡改动、"我们确定吗" → `how`；代码跨函数边界 → `architect`；并行扇出 → `swarm`；设计竞赛 → `arena`；有争议的设计 → `interrogate`。
- 任何文字表面（包括给用户的回复）→ `unslop`；提交前 → `deslop`；评审前 → `no-comments`；报性能数字前 → `benchmark-checklist`。
- PR 状态 → Babysit playbook，而不是 Cursor 的同名内置 skill；绿色栈要落地 → Shipping playbook，"绿不等于安全"。
- Bugbot 评论 → 怀疑姿态，按 [`references/bugbot-triage.md`](../../skills/poteto-mode/references/bugbot-triage.md) 分 fix / dismiss / ask；长任务或用户要离开 → `show-me-your-work` 的 decision trail。

每条都规定"在什么时候"，不规定"要做多少"。判断被挂在事件上，避免退化成一句笼统的"要谨慎"。

### 2. **Principles** 要求先读后引

24 条原则分五组：Core、Architecture、Verification、Delegation、Meta。每条有一个适用条件（"Refactoring, sizing a diff..."），叶 skill 名是 `principle-<title>`。

真正的机制在 **Non-negotiables** 开头：回复里要点名影响了决策的原则，以及它改变了哪个具体选择；只能引用本次会话里读过叶 `SKILL.md` 的原则。这把风格变成了可核对的主张：引一条没读过的原则，等于承认没读。

### 3. **Playbooks** 逐字进 todo

23 个 playbook 各是一份步骤表。匹配到之后，先把步骤原样复制进 todo list，再写任务特有的 todo；不做的步骤留着，标 `skip: <reason>`。没有合适的 playbook 时走 `figure-it-out` 设计一份一次性的；项目级、多天的持续工作走 Orchestrate。原样复制和三层兜底的细节见 [mode 与 playbook](../system/mode-and-playbooks.md)。

### 4. **Autonomy** 默认放行，只为不可逆动作停

默认"just do it"：可逆工作和外部动作（团队消息、ticket 更新、启动 eval）都不问。只有不可逆写入要停：强推共享分支、部署、删数据、给客户发消息。用户说"别停""我要睡了""跑完为止"会进入持续运行；Always pause 那几条不可逆写入不在放行范围内。

同一节还有一条容易被忽略的立场："No is an acceptable answer"。被问要不要做、被邀请加范围时给出真实判断；同意不是默认值。

### 5. **Subagents** 是契约，不是建议

- playbook 步骤里派出的 subagent 用 `poteto-agent`；但 `how`、`why`、`interrogate`、`reflect`、`swarm` 自带 `subagent_type`，不被覆盖。
- 每次派发的默认值：后台运行、agent 模式（只读会剥掉 MCP）、给文件指针而不内联内容、按角色指定模型。
- 模型从角色行或默认值起步，再按任务调：最难或最微妙的升一档，判断类降一档，例行工作降两档以上；只有 CI 或主干红了、有人正被堵住时才用 fast；验证是例外，固定用 `swarm workers` 的模型与速度。
- 默认用新 subagent，并给它合并后的范围：原始 brief、后续每条指令、上一个 agent 的报告和分支。只有新工作确实依赖旧 agent 里的状态（本地 checkout、未提交改动、还在跑的进程）时才续用。
- 父 agent 拥有每个 subagent 的产出：自己看 diff、自己写摘要，不转述。

### 6. **Writing the reply** 规定交付的文体

短句，一句一个意思；不出现长破折号；句中不用冒号连接；但内容不能因为短而变少。每条主张在同一句里带证据或标签：实测、推断、猜测。不编造链接、引用或 transcript。先讲对使用者和维护者的影响，再讲实现细节。

**Comments** 沿用同一套规则，并要求写的时候就写干净。验证脚本里不写叙述阶段的注释，用断言字符串记录步骤。

### 7. **Harness** 承认自己是 Cursor 优先

`Task`、`AskQuestion`、transcript 路径、skill 目录、模型设置文件都给了其他 harness 的映射；`poteto-agent` 未注册时，退化为"先读这个文件的绝对路径"的通用 subagent；没有 subagent 工具时自己按顺序做每个委派步骤；`deslop` 缺失时自己重读 diff 删 AI 痕迹，`control-cli`、`control-ui` 缺失时用现有自动化补（Playwright/CDP、tmux/PTY）。

## 设计意图

- **风格要能被点名，才可审计。** 原则有名字，回复要引用，引用又要求读过。三步把"我觉得这样更好"变成"我按这条规则改了这个选择"。
- **路由与执行不换手。** 选完 playbook 不交给另一个 agent，避免"选完就忘"。
- **约束贴着转移写。** 门放在"要提交了""要开 PR 了""要报数字了"这些具体动作上，而不是流程开头。
- **成本留在人手里。** 53/54 个 skill 是 user-invoked，唯一例外是 `setup-pstack`；mode 是这个选择的配套入口。
- **模型选择与工作流解耦。** 工作流说"这里需要一个判断模型"，`setup-pstack` 决定判断模型是谁。

## 依赖关系

- **上游**：人输入 `/poteto-mode` 或把它设成 Custom Mode；`poteto-help` 把它作为多数问题的默认答案。
- **下游**：23 个 playbook；24 个 principle skill；被点名的 `how`、`why`、`architect`、`arena`、`swarm`、`interrogate`、`reflect`、`unslop`、`technical-writing`、`no-comments`、`benchmark-checklist`、`show-me-your-work`、`figure-it-out`；需要开 PR 的 playbook 最后都走 `opening-a-pr`。
- **共享**：`setup-pstack` 写出的角色模型文件；`poteto-agent` 的 subagent 定义；`scripts/` 下的 `watch-pr`、`orch`、`check-plan.mjs`、`worktree-audit.sh`；`references/bugbot-triage.md`。
- **外部**：`cursor-team-kit` 的 `deslop`、`control-cli`、`control-ui`；Cursor 内置的 `/loop` 与 `create-skill`。
- **对照**：单点 skill 只负责一件事，mode 负责顺序、验收与收尾。

## 使用边界

- **不触发就不生效。** 用 Enter 附着只覆盖一条消息，随对话推进淡出；要持续生效得设成 Custom Mode，或每个任务重新输入。
- **小任务不值得。** 它会把 playbook 步骤写进 todo、读被点名的 skill、按角色派 subagent。`poteto-help` 的建议是：把 mode 留给需要严谨度的工作。
- **规则是文本，不是编译器。** "逐字复制步骤""用新 subagent"都靠执行者遵守，没有机械检查。
- **不要用它跑自己。** 源码明确：接收 `/poteto-mode` 的 chat 负责跑 playbook，不要派 `poteto-agent` 去跑 playbook。
- **Cursor 优先。** 其他 harness 靠 **Harness** 一节的映射；映射缺失时能力会退化，例如没有 subagent 工具就没有并行。

## 值得注意

- **常驻成本高。** 整份 `SKILL.md`（0.15.25 时 161 行），加被选中的 playbook，再加被点名的 skill，是每次运行的固定开销；设成 Custom Mode 后每轮都付。
- **引用原则有事后贴标签的诱惑。** "读过本 session 的叶文件"是自我约束，无法被外部验证；写得越顺，越难分辨真的按原则决策还是补一句引用。
- **两套并行的命名体系。** 24 条原则和 23 个 playbook 各自命名，靠人记。`poteto-help` 的表格在补这部分，但表格本身也要维护。
- **默认 slug 是快照。** `grok-4.7-xhigh-fast`、`claude-opus-5-5-xhigh` 会随模型发布过时；源码没有版本映射。
- **与 Cursor 内置 babysit 同名竞争。** 源码专门写了一条要求用 Babysit playbook，说明触发词冲突是真实存在的。

## 关键洞察

1. **把风格写成可引用的规则，才能被检查。** 名字、适用条件、引用义务，三件套让"按风格做"不再是印象分。
2. **"做什么"和"按什么顺序做"要分开。** 触发表管前者，playbook 管后者，都不换执行者。
3. **默认自主，只为不可逆动作设门。** 门越少，越可能被真的遵守；门放在哪里，比门有多少更重要。
4. **委派要同时给角色和模型。** 只说"派个 subagent"会把最贵的判断交给默认值。
5. **文体是协议的一部分。** 回复的形状、证据标签、不编造链接，和代码一样是可验收的交付物。
