# `unslop` 与 `technical-writing`

pstack 把回复、PR 正文、提交信息、决策日志和代码注释都当成要交付的文字。`poteto-mode` 的 Non-negotiables 规定：任何要交付的文字，包括回复本身，走 `unslop`；docs、RFC、readme、PR 描述和 commit message 走 `technical-writing`；commit 前走 `deslop`（缺失时见 Harness）。

## `unslop`

规则编号是稳定 id，其他 skill 会引用（`poteto-mode` 引用 `unslop` rule 14）。删掉的规则留空号，所以不能重排。

- 删掉 `highlighting…`、`ensuring…` 这类现在分词装饰。
- 不用 `serves as` / `boasts` 代替 `is` / `has`。
- 不用 `not just X, but Y`。
- 不强行凑三点。同义词不要轮换。
- 不用长破折号；冒号只用于列表前，不做句中连接。
- 不用抽象隐喻名词（`substrate`、`wedge`、`north star`）。
- 说机制或数字，不说感觉。
- 主动语态；砍副词；用平实词；不要压缩到读者要解码。

源码：[`skills/unslop/SKILL.md`](../../skills/unslop/SKILL.md)、[`unslop` 单页](../skills/unslop.md)。

## `technical-writing`

四层标准叠在一起：Diátaxis 选文档类型，Google developer style 写句子，STE 约束指令，Global English 消除歧义。三条总则压在四层之上：删掉不出力的词；用短的日常词；规则让句子更糟时换一种写法或不动。

- 一份文档只取一种类型：tutorial、how-to、reference、explanation，不要混。
- 对读者说 you，用现在时；说清谁做什么；条件放在指令前。
- 一条指令一句；warning 放在它保护的步骤前。
- 不留两种读法：`only` 靠近它修饰的词，`it` / `this` 指向明确。
- PR 描述与 commit message 是写作：除 Diátaxis 外每层都适用。PR 正文是一分钟的 briefing，不贴 swarm 日志、SHA 列表或指标表。
- codebase 就是词表：写真实的 symbol、file、flag、command。不发明 jargon。

源码：[`skills/technical-writing/SKILL.md`](../../skills/technical-writing/SKILL.md)、[`technical-writing` 单页](../skills/technical-writing.md)。

## `poteto-mode` 的 Writing the reply

- 短陈述句，一句一个意思。
- 任何地方不用长破折号；冒号不做句中连接。
- 简洁不等于删内容：playbook 的 reply 要求的细节、权衡、选择、未决项都要留。
- 先说对用户和下一个维护者的影响，再说实现细节。
- 不编造链接、引用或 transcript 记录，只链这次 session 产出的或读过的。
- 每条断言在同一句里带证据或标签：measured、inferred、guess。

源码：[`skills/poteto-mode/SKILL.md#writing-the-reply`](../../skills/poteto-mode/SKILL.md#writing-the-reply)。

## 注释

同一个规则：只保留代码本身说不出的 non-obvious why。禁止 `// Phase 1: add cards` 这类叙述阶段的注释；步骤由 assertion 或 log 字符串体现，例如 `assert(ok, 'persisted across restart')`。源码：[`skills/poteto-mode/SKILL.md#comments`](../../skills/poteto-mode/SKILL.md#comments)、[`no-comments` 单页](../skills/no-comments.md)。

## 代价

- 规则多，全部遵守要持续注意力。
- `unslop` 有些规则是绝对的，`technical-writing` 留了"规则让句子更糟时换一种写法"的口子，两套硬度不一致。
- 靠自检执行。没有 linter 检查回复；`deslop` 缺失时只能自己重读 diff。
- 边写边干净比事后清理难。`poteto-mode` 明确说 draft 后的清理不能去掉这些写法。

## 关键洞察

1. 写作规则保护的是读者能否快速判断，不是文字是否好看。
2. 稳定 id 让规则可引用。没有编号，别的 skill 只能说"那条关于冒号的规则"。
3. 先写干净比事后清理可靠。套话一旦进了草稿，清理时容易保留骨架。
