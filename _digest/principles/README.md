# Principles 索引

`pstack` 把 24 条工程判断做成 24 个 `principle-*` skill。它们不是要逐个调用的命令，而是一组**可点名的词汇**：用户说一个名字，agent 就知道要重新检查哪一类决定。

> 你不需要背下全部。先扫一遍，记住"遇到这类问题时，有一个名字可以叫"，需要时再回来查。

## 怎么用

| 用法 | 例子 |
|---|---|
| 中途纠偏 | `use subtract before you add. delete the obsolete adapters first.` |
| 要求证据 | `apply prove it works. run the real import flow and show me the written records.` |
| 阻止共享写入 | `separate before serializing shared state. give each attempt its own worktree.` |
| 复盘时指认 | "这次失败是 premise 问题，不是第三个修法不够好。" |

`poteto-mode` 的规则是：应用某条 principle 时读完整 leaf skill，并在回复里点名它改变了哪个决定。只报名字不说改变，等于没有应用。

## Core：建多少、何时重想

| Principle | 何时触发 | 一句话 |
|---|---|---|
| [`laziness-protocol`](../../skills/principle-laziness-protocol/SKILL.md) | 重构、估 diff 大小、想加抽象或层 | 偏向删除和解决问题的最小改动 |
| [`foundational-thinking`](../../skills/principle-foundational-thinking/SKILL.md) | 写逻辑之前 | 先定核心类型与数据结构、脚手架与功能的顺序、并发者共享什么 |
| [`redesign-from-first-principles`](../../skills/principle-redesign-from-first-principles/SKILL.md) | 把新需求塞进既有设计 | 假设它从第一天就存在，重新设计 |
| [`attack-the-premise`](../../skills/principle-attack-the-premise/SKILL.md) | 两个以上同前提的修法都失败 | 写下前提，直接测它，不要写第三个修法 |
| [`subtract-before-you-add`](../../skills/principle-subtract-before-you-add/SKILL.md) | 要加东西、重构或重写 | 先删死重，再在更简单的基础上建 |
| [`minimize-reader-load`](../../skills/principle-minimize-reader-load/SKILL.md) | 评审难以追踪的代码 | 数层数与隐藏状态，折叠单调用 wrapper，缩小可变作用域 |
| [`outcome-oriented-execution`](../../skills/principle-outcome-oriented-execution/SKILL.md) | 有明确阶段边界的重写/迁移 | 收敛到目标架构，不保留一次性兼容状态 |
| [`experience-first`](../../skills/principle-experience-first/SKILL.md) | 产品、UX、功能范围取舍 | 用户愉悦优先于实现便利 |
| [`exhaust-the-design-space`](../../skills/principle-exhaust-the-design-space/SKILL.md) | 没有先例的新交互或架构决定 | 做 2-3 个竞争原型，比较后再定 |
| [`build-the-lever`](../../skills/principle-build-the-lever/SKILL.md) | 任何非平凡工作 | 建工具（codemod、脚本、生成器）而不是手工做；工具是可重跑的产物 |

## Architecture：状态、验证、兼容放在哪里

| Principle | 何时触发 | 一句话 |
|---|---|---|
| [`model-the-domain`](../../skills/principle-model-the-domain/SKILL.md) | 写有状态逻辑、大量分支、重复形状假设 | 用状态机、类型模型、表/注册表、reducer、边界或正确的集合编码领域 |
| [`boundary-discipline`](../../skills/principle-boundary-discipline/SKILL.md) | 接验证、错误处理、框架适配器 | 守卫集中在系统边界，内部信任类型，业务逻辑保持纯函数 |
| [`type-system-discipline`](../../skills/principle-type-system-discipline/SKILL.md) | 设计类型或签名 | 让非法状态无法表示；品牌化原始类型；在边界解析外部数据 |
| [`make-operations-idempotent`](../../skills/principle-make-operations-idempotent/SKILL.md) | 命令、生命周期步骤、会遇崩溃和重试的循环 | 收敛到同一个终态 |
| [`migrate-callers-then-delete-legacy-apis`](../../skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) | 引入新内部 API 且旧调用者还在 | 一波迁移并删除 |
| [`separate-before-serializing-shared-state`](../../skills/principle-separate-before-serializing-shared-state/SKILL.md) | 并发者可能写同一文件、分支、key 或对象 | 先消除共享，再加协调 |

## Verification：什么算证明

| Principle | 何时触发 | 一句话 |
|---|---|---|
| [`prove-it-works`](../../skills/principle-prove-it-works/SKILL.md) | 任务之后、宣布完成之前 | 验证真实产物，不用代理指标 |
| [`fix-root-causes`](../../skills/principle-fix-root-causes/SKILL.md) | 调试 | 追到系统第一次出错的位置，在那里修 |
| [`sequence-verifiable-units`](../../skills/principle-sequence-verifiable-units/SKILL.md) | 多步工作、扫荡、迁移、堆叠提交/PR | 每个小单元以一次检查结束，验证后再下一个 |
| [`test-behavior-not-implementation`](../../skills/principle-test-behavior-not-implementation/SKILL.md) | 写、改或保留测试 | 断言用户可观察的行为，内部改变时测试仍应通过 |
| [`explain-the-number`](../../skills/principle-explain-the-number/SKILL.md) | 信任、报告或依据一个测得的数字之前 | 找出它的限制，并排除"测的不是你以为的工作" |

## Delegation：并行时怎样不互相污染

| Principle | 何时触发 | 一句话 |
|---|---|---|
| [`guard-the-context-window`](../../skills/principle-guard-the-context-window/SKILL.md) | 上下文被大输出、长文件、重复读取、fan-out 规划占满 | 批量工作交给 subagent，主线程只留摘要 |
| [`never-block-on-the-human`](../../skills/principle-never-block-on-the-human/SKILL.md) | 想为可逆工作问"要不要做 X" | 先做，呈现结果，让人纠正 |

## Meta：让环境自己变好

| Principle | 何时触发 | 一句话 |
|---|---|---|
| [`encode-lessons-in-structure`](../../skills/principle-encode-lessons-in-structure/SKILL.md) | 第二次写同一条指令 | 优先编码成 lint、元数据、运行时检查或脚本；判断类教训无法编码时，才写一段带失败例子的文字 |

## 值得注意

- **24 条不是检查表。** 一次任务通常只用少数几条（经验建议，不是源码规则）；全用会互相抵消。
- **规则会冲突。** `laziness-protocol` 与 `redesign-from-first-principles` 在"小改还是重做"上经常对立；冲突要显式判断，不要假装都满足。
- **名字是入口，不是结论。** 点名的价值在于让双方回到同一条规则，而不是赢得争论。
- **不是强制检查。** 没有工具验证 principle 是否被遵守，只有回复里的自述和评审时的人工判断。
