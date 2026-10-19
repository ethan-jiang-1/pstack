# `correct`、`reflect` 与验证 skill

这组 skill 改的不是产品代码，是下一次工作发生的地方：仓库里的检查、skill 正文、项目本地的 verification skill。判断标准是：同类错误第二次出现时，环境已经让它无法再发生。

## `correct`

operator 反复纠正同一类错误时，改仓库，让下一个 agent 无法再犯。`correct` 假设每个贡献者都是只看过自己打开文件的 agent，会抄最近的例子，会走最短的能编译路径。

- 先归类：读最近的 commit、revert、review comment、agent 指令文件，以及解释 workaround 的注释。一类错误发生两次才算一类。
- 每类在能生效的最高层修：架构（每份状态一个 owner、每条任务一条受支持路径、删掉会被抄走的旧写法）→ 类型，或错误信息直接说该用哪个文件、类型、函数的 lint → 测试（修掉或删掉那种行为坏了还通过的测试）→ 文档或 agent 规则。文档放最后，因为跳过它不会有东西失败。
- 新检查必须在一个真实的历史错误上跑失败，本地和 CI 跑同一条命令。例外写在出错行上，带原因、到期时间和人的批准。
- agent 指令文件里留一张表，把每条规则和强制它的东西配对。operator 再次纠正时，先修错误再加规则；规则已在却没有任何东西强制它，就是重复，按最高层修。错误不可能再发生后删掉这条规则。

源码：[`skills/correct/SKILL.md`](../../skills/correct/SKILL.md)。

## `reflect`

1. 只在当前 workspace 的 session 目录里找这次对话的 transcript，不跨项目 glob。
2. 一条消息派三个 reviewer：judgment、tooling、divergent。judgment 与 divergent 读 `reflect judgment, divergent, synthesizer` 这行 role，tooling 读 `reflect tooling`。
3. synthesizer 通过 MCP 抽查引用，返回 Accepted / Rejected / Backlog。
4. 能由 lint、脚本、metadata flag 或运行时检查更可靠执行的 Accepted 项，移到 Backlog，依据是 `encode-lessons-in-structure`。
5. 完整展示三份清单，等 operator 明确批准后才应用 Accepted。skill 改动影响组织里每一个未来的 agent，所以不自动应用。
6. Backlog 直接进团队的 devex / backlog tracker，不等批准。
7. 应用时按每行的 Routing 走：一行 bullet 或收紧一句，父 agent 直接改；新段落、新表格或超过约 10 行，交给 harness 的 skill-authoring skill（Cursor 的 `create-skill`、Anthropic 的 `skill-creator`）跑 draft / test / iterate；`tune description` 与 `new skill via create-skill` 也交给它。

源码：[`skills/reflect/SKILL.md`](../../skills/reflect/SKILL.md)。

## `encode-lessons-in-structure`

第二次写同一条指令时，把它编码成 lint、metadata flag、运行时检查或脚本。需要判断、无法编码的教训，才把指令写得更显眼并加一个失败例子。推论是：能结构化的修法就只用结构化的修法，指令是症状。源码：[`skills/principle-encode-lessons-in-structure/SKILL.md`](../../skills/principle-encode-lessons-in-structure/SKILL.md)。

## `create-verification-skill` / `maintain-verification-skill`

`create-verification-skill` 生成项目本地的 `verify-<app>` skill：Launch、Doctor、Drive、Evidence、Cleanup、Helpers 六节，加 `features/` 下的 feature map。它先采访仓库而不是用户；drive 优先复用仓库已有的 Playwright / Cypress / expect / PTY / curl 路径；证据来自真实用户路径和真实副作用，不来自 internal setter。生成后必须自己完整跑一遍 launch、doctor、drive 一个 feature、capture、cleanup，并确认 cleanup 没吃掉证据。

`maintain-verification-skill` 是它的维护循环。先抽一个 outcome：clean、changed 或 blocked。每个 feature 一个只读 subagent 读源码，reconcile 后由 coordinator 亲自把每个 feature 至少驱动一次，最后最多开一个 PR 装已证明的修正。编辑范围只有 verification skill 自己的目录；产品行为真的坏了记成 product gap，不写进 map 当作文档修正。

源码：[`skills/create-verification-skill/SKILL.md`](../../skills/create-verification-skill/SKILL.md)、[`skills/maintain-verification-skill/SKILL.md`](../../skills/maintain-verification-skill/SKILL.md)、[`create-verification-skill` 单页](../skills/create-verification-skill.md)。

## 代价

- `correct` 要求证据。没有真实历史错误可复现，新检查只是猜测。
- `reflect` 慢。三个 reviewer 加一个 synthesizer，再加显式批准，比顺手改一句 prompt 贵得多。
- 自修改有风险。一次过拟合的 skill 修改会影响之后所有 agent，批准门不能省。
- `maintain-verification-skill` 不改产品代码。map 描述的行为如果已不存在，只能修文档或报 product gap。

## 关键洞察

1. 交付物包括仓库本身。只修当前 diff，同类错误会换一种形式回来。
2. 能失败的检查比一段提醒可靠。写第二条提醒前，先看能不能让第一条不可能被违反。
3. 改流程的人不能是唯一审流程的人。`reflect` 的批准门承担这次外部检查。
