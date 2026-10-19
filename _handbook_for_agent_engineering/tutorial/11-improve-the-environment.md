# 用 `correct` 与 `reflect` 改环境

> 教程 11 / 12 · 上一页：[10 把验证过的一段栈落地](./10-ship-observe-and-loop.md) · 下一页：[教程之后](./after-tutorial.md)

## 眼前困惑

一轮工作结束后，有两类东西值得留下：

- 同一类错误被 operator 纠正了第二次；
- 这次会话里有值得写回 skill 的经验。

pstack 用两个 skill 处理它们：`correct` 改仓库，让一类错误不再发生；`reflect` 把会话经验写回 skill。它们不是同一件事，也不要合并成一个流程。

## 现在动作

### 1. 用 `correct` 把重复错误变成结构

`correct` 先找 mistake classes：读 recent commits、reverts、review comments、agent instruction files、解释 workaround 的注释。一类错误出现两次，才算一个 class。源码：`skills/correct/SKILL.md`。

然后按"能工作的最高层"修：

1. 用架构消除它：每块状态一个 owner，每件事一条受支持路径，删掉旧方式和 agent 会抄的死代码。
2. 用类型让坏状态写不出来；类型表达不了时，用一条错误信息点名修法的 lint 或 CI 检查。
3. 用测试锁行为。
4. 文档与 agent 规则放最后，只用于判断类问题。

每条新检查都要证明它在一条真实历史错误上会失败；同一条命令本地和 CI 都跑。最后在 agent instruction file 里维护一张规则表，把每条规则与强制它的机制配对；规则已经不可能再犯时，删掉它。源码：`skills/correct/SKILL.md#Keep the rule table`。

### 2. 用 `reflect` 把会话经验写回 skill

`reflect` 只在本 workspace 找 transcript，不 glob 别人的目录。三个 reviewer 并行：

- judgment：命名具体事件背后的耐用原则。
- tooling：抓下个 agent 会重新推导的命令、路径、flag 事实。
- divergent：找反框架与二阶效应，包括"成功但侥幸"。

一个 synthesizer 用 MCP 抽查引用，返回 Accepted / Rejected / Backlog。Accepted 要等明确批准后才改 skill；Backlog 直接进团队的 devex / backlog tracker。源码：`skills/reflect/SKILL.md`。

### 3. 能编码的就编码

`encode-lessons-in-structure` 的规则是：第二次写同一条指令时，把它编码成 lint、元数据、运行时检查或脚本；判断类教训无法编码时，才写一段带失败例子的文字。源码：`skills/principle-encode-lessons-in-structure/SKILL.md`。

### 4. 需要项目本地验证时，用 `create-verification-skill`

如果项目没有可脚本化驱动真实应用的方式，用 `create-verification-skill` 生成 `verify-<app>` skill 和 feature map；之后用 `maintain-verification-skill` 维护。源码：`skills/create-verification-skill/SKILL.md`、`skills/maintain-verification-skill/SKILL.md`。

## 可见成果

一条在真实历史错误上会失败的检查，或一次 `reflect` 的 Accepted / Rejected / Backlog 结论。产出就是这两样。

## 休息点

教程到这里结束。你走完了一个 pstack 循环：prompt → mode/playbook → data shape → architect → 小步实现 → tdd → brief → verdict → interrogate → shipping → correct/reflect。

不需要每次都走满十二页。下次遇到具体卡点时，回到对应的一页，或者直接查手册。

## 下一段

[教程之后](./after-tutorial.md)：按卡点查手册的六章。
