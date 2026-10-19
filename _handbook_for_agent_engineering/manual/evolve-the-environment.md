# 环境：`correct` 与 `reflect`

> 本章 3 个单元 · [手册目录](./README.md) · [上一章：评审与落地](./ship-run-and-evolve.md)

## 这一章解决什么

一轮工作结束后，有两类东西值得留下：

- 同一类错误被 operator 纠正了第二次；
- 这次会话里有值得写回 skill 的经验。

pstack 用 `correct` 改仓库，让一类错误不再发生；用 `reflect` 把会话经验写回 skill。两者都不是"复盘文档"，也不产出新的方法名。

| 单元 | 卡点 | 带走 |
|---|---|---|
| 6.1 | 同一个错误反复出现 | 一条在真实历史错误上会失败的检查 |
| 6.2 | 会话经验随对话消失 | Accepted / Rejected / Backlog 与批准门 |
| 6.3 | 能编码的规则还停留在文字 | 结构约束，或项目本地的验证 skill |

---

## 6.1 `correct`

### 工作情境

operator 第三次说"你又把 missing 行静默丢了"。每次修完当前 diff，下次换个 agent 又犯。

`correct` 的做法是把仓库改成"下一个 agent 无法再犯"。源码：`skills/correct/SKILL.md`。

### 做法

1. 找 mistake classes：读 recent commits、reverts、review comments、agent instruction files、解释 workaround 的注释。一类错误出现两次，才算一个 class。
2. 按"能工作的最高层"修：
   - 架构：每块状态一个 owner，每件事一条受支持路径，删掉旧方式和 agent 会抄的死代码。
   - 类型：让坏状态写不出来；类型表达不了时，加一条错误信息点名修法的 lint 或 CI 检查。
   - 测试：锁行为。
   - 文档与 agent 规则：放最后，只用于判断类问题。
3. 每条新检查都要证明它在一条真实历史错误上会失败。
4. 同一条命令本地和 CI 都跑。
5. 例外写在违规那一行上，带理由、到期日、人的批准。
6. 在 agent instruction file 里维护规则表：每条规则配一个强制它的机制；规则已经不可能再犯时，删掉它。

### 质量门

- 一个 class 一个 commit。
- 每条检查都能在真实历史错误上变红。
- 规则表里没有"只有文字、没有强制者"的行。

---

## 6.2 `reflect`

### 工作情境

一段长会话结束，里面有几条值得写回 skill 的经验。直接让 agent 总结并改 skill 有两个问题：一次会话是轶事；skill 改动会影响组织里每个未来的 agent。

`reflect` 把这两件事分开：多个视角找候选，一个 synthesizer 按固定判据筛，最后人只批准需要判断的那一类。源码：`skills/reflect/SKILL.md`。

### 做法

1. 只在本 workspace 找 transcript，不 glob 别人的目录。
2. 三个 reviewer 并行，agent 模式（`readonly: false`，否则 MCP 被剥掉）：
   - judgment：命名具体事件背后的耐用原则。
   - tooling：抓下个 agent 会重新推导的命令、路径、flag 事实。
   - divergent：找反框架与二阶效应，包括"成功但侥幸"。
3. synthesizer 用 MCP 抽查引用，按固定判据筛：
   - durability、specificity、existing-skill-first、convergence、decision-changing；
   - structural-mechanism check：能由 lint、脚本、元数据或运行时检查更可靠执行的，路由到 Backlog；
   - skill-was-used：只接受能路由到本会话真用过的 skill、工具或 MCP 的 finding；如果 skill 该用却没用，路由到 `tune description: <skill path>`；两者都不满足，拒绝为 `skill-not-used`；
   - already-covered：接受任何正文改动前先读目标 skill。
4. 展示完整的 Accepted / Rejected / Backlog，等明确批准后才应用 Accepted。
5. Backlog 直接进团队的 devex / backlog tracker，不等批准。
6. 按 routing 应用：trivial 编辑父 agent 直接做；substantive 编辑交给 skill-authoring；`tune description:` 交给 description 优化；`new skill via create-skill:` 交给 create-skill。

### 质量门

- 每个 Accepted 都有 durability、specificity、convergence 的依据。
- 没有自动应用 skill 修改。
- Backlog 有 owner 和去向。

---

## 6.3 `encode-lessons-in-structure` 与验证 skill

### 工作情境

你第二次写同一条"不要用 console.log"。写第三遍时，它还是会被跳过。

`encode-lessons-in-structure` 的规则是：第二次写同一条指令时，把它编码成 lint、元数据、运行时检查或脚本；判断类教训无法编码时，才写一段带失败例子的文字。源码：`skills/principle-encode-lessons-in-structure/SKILL.md`。

### 做法

- 能由类型表达的，用类型。
- 能由 lint 或 CI 表达的，用 lint 或 CI，错误信息点名修法。
- 能由运行时检查表达的，用运行时检查。
- 能由脚本表达的，用脚本；脚本可重跑，比手更可靠。
- 只能靠判断的，写短、写具体，并附一个失败例子。

### 项目本地验证

如果项目没有可脚本化驱动真实应用的方式，用 `create-verification-skill` 生成 `verify-<app>` skill 和 feature map；之后用 `maintain-verification-skill` 维护。源码：`skills/create-verification-skill/SKILL.md`、`skills/maintain-verification-skill/SKILL.md`。

### 质量门

- 每条规则能回答"违反时谁会失败"。
- 编码后的检查在真实历史错误上变红。
- 判断类规则有失败例子，不是一句口号。

---

## 本章与 pstack 的边界

pstack 没有 spec 生命周期；`correct` 会删掉不再需要的规则，`docs/guide/04` 说落地后删除计划文件。除此之外，不建额外流程。
