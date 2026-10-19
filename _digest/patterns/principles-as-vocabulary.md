# principle 的用法

24 个 `principle-*` skill 不是调用命令。它们是可点名的判断规则：用户说出名字，agent 重新检查当前决定，并在回复里说明这条规则改变了哪个选择。

## `poteto-mode` 的 Principles 一节

- 24 条按 Core / Architecture / Verification / Delegation / Meta 分组，每个条目给一句"何时适用"。
- 多步任务开始时先读 Principles 这份索引，任务触发哪条再读哪条的 leaf；不在触发时把所有 leaf 读一遍。
- 条目名转 kebab-case 就是 leaf skill 名：`Laziness Protocol` → `principle-laziness-protocol`。条目自带名字的按它给的：`principle-sequence-verifiable-units`。
- 应用某条 principle 时读完整 leaf skill。回复里点名每条影响了决定的 principle，并写出它改变的具体选择。只引用这次 session 读过的 leaf。

源码：[`skills/poteto-mode/SKILL.md#principles`](../../skills/poteto-mode/SKILL.md#principles)、[principles 索引](../principles/README.md)。

## 名字怎样改变执行

[`docs/guide/08-principles.md`](../../docs/guide/08-principles.md) 的说法是：你不调用 principle，你用它的名字改变工作方向。名字背后是一条已经读过的完整规则，所以一个短语比一段解释更精确。

典型用法：

- 要加适配器之前：`use subtract before you add`，先删旧适配器，再在剩下的基础上设计。
- 只有 build 通过就想宣布完成：`apply prove it works`，去真实流程里跑一遍并贴出写下的记录。
- 两个并行尝试要写同一个分支：`separate before serializing shared state`，各自一个 worktree。
- 同类错误反复出现：`encode lessons in structure`，把它变成 lint 或检查，而不是再写一段提醒。

源码：[`skills/principle-*`](../../skills/)、[`docs/guide/08-principles.md`](../../docs/guide/08-principles.md)。

## 冲突要显式选

规则之间不总是一致。`laziness-protocol` 倾向最小改动，`redesign-from-first-principles` 倾向按目标架构重做；两者都能引用时，必须说明这次为什么选一条。冲突不是缺陷，它逼出一次显式判断。

## 代价

- 24 个名字要记、要选、要在回复里正确引用。不熟的人读起来像术语表。
- 名字会被当口号。没有"改变了哪个决定"的约束，点名就退化成 name-dropping。
- 不是强制。没有 lint 检查 principle 是否被遵守，只有回复里的自述。
- 规则之间可能冲突，靠人判断。

## 关键洞察

1. 可点名的规则比长段落更适合指挥。一个短语就能重定向工作。
2. 引用义务是防退化的关键。名字必须落到决定上，否则只是术语。
3. 24 条不需要每次全用，只在触发时用对那几条。
