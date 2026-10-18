# `show-me-your-work`

> 源码：[`skills/show-me-your-work/SKILL.md`](../../skills/show-me-your-work/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

给长跑或无人值守的工作留一份可审计的决策轨迹：单文件 TSV，一行一个决定，证据是指针而不是散文；只追加、不修改；交付前用本次运行的 transcript 对一遍，再让一个不同模型家族的 subagent 复查。

## 它解决什么

长任务结束后，人看到的是一段总结。总结可以很顺，也可以悄悄跳过失败、倒因为果、把"试过"写成"验证过"。工作越自主，这个风险越大，因为中间没有人在看。

决策轨迹把"我做了什么"变成可逐行检查的记录：每个决定有时间、有理由、有一个能打开的指针、有一个可判定的结果。它不替代 diff，也不替代测试；它补的是"为什么走这条路、试过什么、什么被否了"。

## 核心机制

### 1. 六列 TSV

一个文件，一行一个决定，单元格保持单行。`references/decision-log-template.tsv` 只有表头，复制它就是一份干净的日志。

- **ts**：ISO8601 时间戳。
- **phase**：阶段或工作流名称。
- **decision**：选了什么或做了什么，一行。
- **why**：平实语言的理由。驱动它的是哪条 principle 就直说，不要写术语标签。
- **evidence**：能证明它的链接或路径，commit SHA、PR 号、`file:line`、artifact / trace / 截图路径。永远不是一段话。
- **result**：结果或谓词状态，`tests green`、`reverted`、`pixel-diff 0`、`INCONCLUSIVE`、`open`。

读的时候从上往下、沿 evidence 指针点开抽查。GitHub 会把提交的 TSV 渲染成表格；终端里用 `column -s$'\t' -t decisions.tsv`。

### 2. 写行的纪律

每行按"跟同事讲你做了什么"来写：平实、具体、没有 AI 腔（`unslop` 同样适用于日志文本）。

用 helper `scripts/log.sh <logfile> <phase> <decision> <why> <evidence> <result>`。它盖时间戳、首次写表头、把单元格里的 tab / 换行 / CR 压掉、给以 `=`、`+`、`-`、`@` 开头的单元格加一个单引号。最后一条是公式注入防护：日志预期在表格软件里打开，而 evidence 可能来自 PR 标题、文件名或生成文本。手写 `printf` 也行，但要自己处理这些字节。

记决策点和 checkpoint，不是每个动作：选了一个 fork、一个单元完成及其验证结果、一次 pivot 或 revert 及其触发条件、一个暴露的 blocker、一个被修好的 gate。循环运行每个 iteration 一行。琐碎和自明的跳过。

### 3. `start` 行：run 的归属

一个 run 是一次 agent 对话，包括它后续的 turn 和任何对它的总结。pickup、替换上来的 agent、新开的 chat 都是新 run。

当一个 run 往已有行的日志里追加时，它的第一行 phase 是 `start`；在另一个 run 的 `start` 行之后，它的第一行也是 `start`。`start` 行点名在它之前、本 run 没有写的那些行的 `ts` 范围，evidence 点名本 run（例如 agent id）；它不用于其他任何用途。所以回到日志的 run 要先读最后几行，看有没有别的 run 写过。

### 4. 只追加，默认不提交

默认它是工作产物，不是提交物：放在工作目录的 `decisions.tsv`，多个 effort 并行时放 `.audit/<task-slug>.tsv`，不进 git。只有工作足够大、reviewer 需要轨迹才能信任结果时才提交；`figure-it-out` 这类运行通常到这个门槛。

错的决定用一行新记录取代，永远不编辑或删除历史。修正本身也是被记录的事件。证据优先来自已提交的脚本，而不是手工一次性生成的东西（`encode-lessons-in-structure`）。

### 5. 用 transcript 审计

交付前检查日志说了真话。读本次运行在 active workspace 会话目录里的 transcript：Cursor 是系统提示指定的 `agent-transcripts/`；Claude Code 是 `~/.claude/projects/<slug>/`，slug 是 workspace 路径里每个非字母数字字符变成 `-`；Pi 是 `~/.pi/agent/sessions/--<slug>--/`；Codex 是 `~/.codex/sessions/` 里首行 `payload.cwd` 等于 workspace 路径的那个文件。

不要 glob 所有项目的目录，那会读到无关的私人对话。

本次运行的每一段行，从它的某个 `start` 行（如果日志是它创建的，就从第一行）开始，到下一个别的 run 的 `start` 行结束。逐行对照三件事：每行是否对应真实发生过的决定或动作；每行的 evidence 是否能解析、是否真的显示了这行声称的东西；影响过工作但没记的 fork、pivot 或被放弃的方案是不是一个 gap，是就补上。

改日志，不改故事。审计永远不编辑或删除一行，即使是编造的。某行既不对应真实决定，也不对应真实动作，或者 claim / evidence 是错的，就加一行取代它，写实际发生的事和一个能解析的指针。审计不检查本 run 段之外的行；如果本 run 的工作表明其中某行是错的，像其他错误一样取代它。

### 6. 跨模型复查

交付前，在一个与做这份工作的模型不同家族的模型上派一个 subagent。自查不能替代它。它读轨迹和本次运行的 transcript，标出用户应该注意的东西：证据弱或缺失的决定、被跳过或只在口头上声称的验证、事后看有风险的抉择（premature、scope creep、盖住症状）、以及随手一翻会错过的 gap。

每个产出过轨迹的 run，回复都以 `Attention` 一节结尾。第一行是 `reviewed by <model>`，然后逐条 flag 指向具体行或时刻。`No flags` 是合法值，模型名不是。

## 设计意图

- **一个 canonical log。** 多个日志会互相矛盾；单文件才能被完整审计。
- **证据是指针。** 散文无法验证，路径可以；这条约束逼作者留下能打开的东西。
- **append-only 让修正也成为记录。** 可审计性来自历史不可改，而不是来自作者承诺诚实。
- **`start` 行把归属写进日志。** 没有它，run 无法区分自己写的和继承的行，审计范围就不成立。
- **transcript 审计把自报变成对照。** 日志可以编，同一 run 的原始记录不容易编。
- **跨模型复查查的是判断质量。** 审计查真伪，复查查"这个选择是否明智"，两者不能互相替代。
- **默认不提交。** 轨迹是工作产物；提交会带来评审噪音，只有需要建立信任时才付这个成本。
- **公式注入防护。** 日志预期在表格软件里打开，evidence 可能来自攻击者可控的文本。

## 依赖关系

- **上游**：`figure-it-out` 的 Phase D 把日志路由到这里，并且通常提交轨迹；`poteto-mode` 对长跑、自主、多阶段、或用户离开后复查的工作要求轨迹；`autonomous-run` 每次 iteration 写一行 checkpoint；`autopilot-full` 在 root agent 的 store 里开轨迹并每个单元后推送；`orchestrate` 用 `decisions.tsv` 作为 program 的轨迹；`hillclimb` 每次尝试写一行；`pause-safely` 指向已有轨迹而不是复制它。
- **下游**：跨模型复查的 subagent；PR reviewer 读提交后的 TSV。
- **共享**：`scripts/log.sh`、`references/decision-log-template.tsv`；`poteto-mode` 的 **Harness** 一节里的 transcript 路径表；`unslop`；principle `encode-lessons-in-structure`。
- **对照**：`ketchup` 是给人读的变更叙述，这个是给审计的行记录；`pause-safely` 的 resume note 是给下一个 agent 的上下文，它应当指向轨迹，而不是重复轨迹。其他 skill 把轨迹路由到这里、按名字引用，不重述列。

## 使用边界

- **短任务不要写。** 一次问答、一个小修不需要轨迹；提交更是例外。
- **不要记每个动作。** 逐动作日志会淹没信号；只记决策点和 checkpoint。
- **审计只覆盖本 run 的段。** 更早 run 的错误只有在本 run 的工作暴露它时才处理。
- **跨模型复查需要另一个家族可用。** 只有一个模型或没有 subagent 工具时，这一节退化成自报；源码没有给替代方案。
- **transcript 路径依赖 harness。** 路径和 slug 规则各 harness 一套，读错目录会得到错误的审计范围。
- **轨迹本身不是证据。** 它指向证据；没有可解析指针的行，在审计里会被取代。

## 值得注意

- **格式所有权与 `Hillclimb` 冲突。** 本 skill 说其他 skill 应把轨迹路由过来、不要重述列；`hillclimb` playbook 却在要求通过本 skill 开日志的同一段里，给 `decision.tsv` 列了一套自己的列（`id, hypothesis, change, before, after, delta, tests, verdict, note`），连文件名都与本 skill 的 `decisions.tsv` 不同。
- **`start` 行的归属依赖时序，但没有并发保护。** 两个 run 同时写同一份日志时，"本 run 没写的行"无法可靠界定。
- **跨模型复查的独立性是自报的。** 回复要求写 `reviewed by <model>`，但没有机制验证那个 subagent 真的跑了不同家族；面板退化成一个模型时，`Attention` 一节仍然会生成。
- **审计能力与隐私是同一条边界。** 源码明确禁止 glob 其他项目目录，因为会读到无关私聊；这意味着审计只能在当前 workspace 的记录上做。

## 关键洞察

1. **一行一个决定，证据是指针。** 轨迹的价值在于能沿指针复查，不在叙述完整。
2. **append-only 让修正也是记录。** 可审计性来自历史不可改。
3. **`start` 行把"谁写的"变成日志内的属性**，而不是日志外的记忆。
4. **自报要用原始记录对。** transcript 是最低成本的防伪。
5. **审计查真伪，复查查判断质量。** 两者分工，不能互相替代。
6. **格式归一个 skill 所有。** 否则每个流程都会长出一套自己的日志，最后没人能一次读完。
