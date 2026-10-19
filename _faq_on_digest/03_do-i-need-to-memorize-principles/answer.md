# principle 是词汇表，不是要背的清单

> [FAQ 首页](../README.md) · [问题原文](./question.md) · [principles 索引](../../_digest/principles/README.md)

## 直接回答

不用背。把它们当成一组**可点名的判断词汇**：先扫一遍知道"遇到这类问题有一个名字可以叫"，任务触发时再读完整的 leaf skill。一次任务通常只用少数几条（经验建议，不是源码规则），全用会互相抵消。

## 为什么不是背

- 24 条 leaf skill 各自有完整正文，索引只给"何时适用"。
- 名字本身就能激活预训练概念（`subtract before you add`、`prove it works`），不需要逐字记忆。
- 真正的要求不是记住，而是**应用时读全文，并在回复里说明它改变了哪个决定**。

## 按阶段找

| 你现在的困境 | 先看哪组 | 典型名字 |
|---|---|---|
| 不确定要建多少、要不要重做 | Core | `laziness-protocol`、`redesign-from-first-principles`、`subtract-before-you-add` |
| 状态、验证、兼容放在哪里 | Architecture | `model-the-domain`、`boundary-discipline`、`type-system-discipline`、`separate-before-serializing-shared-state` |
| 不确定什么算证明 | Verification | `prove-it-works`、`fix-root-causes`、`explain-the-number` |
| 并行工作时上下文被占满 | Delegation | `guard-the-context-window` |
| 同一条指令写了第二次 | Meta | `encode-lessons-in-structure` |

## 一次具体的使用

```text
use subtract before you add. delete the obsolete adapters first, then design what's left.
```

agent 应该：

1. 读 `principle-subtract-before-you-add` 的全文。
2. 按它重新检查当前计划。
3. 在回复里说"我先删了这三个适配器，因此新的接口只需要两个方法"，而不是只说"应用了 subtract before you add"。

第三步是防退化的关键。只报名字不说改变，等于 name-dropping。

## 冲突怎么办

规则会冲突。最典型的是 `laziness-protocol`（最小改动）与 `redesign-from-first-principles`（按目标架构重做）。pstack 没有给所有冲突排优先级，这时要显式判断：

- 当前问题是一次性补丁，还是长期结构？
- 保留兼容状态的代价是否大于重做？
- 哪条规则的适用条件真的被触发了？

把冲突写出来，比假装两条都满足更可靠。

## 常见误用

- **全用。** 24 条一起上，等于没有重点；一次用少数几条足够（经验建议，不是源码规则）。
- **当命令。** principle 不需要单独"调用"；它是读、判断、引用。
- **只点名。** 没有落到具体决定的引用，是装饰。
- **忽略适用条件。** 每条都有"何时触发"；不触发时硬套会制造无意义的改动。

## 继续阅读

- [当前行为的源码](../../skills/poteto-mode/SKILL.md)（**Principles** 一节）
- [principles 索引](../../_digest/principles/README.md)
- [principle 的用法](../../_digest/patterns/principles-as-vocabulary.md)
