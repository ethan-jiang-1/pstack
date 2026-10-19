# 单模型、无子代理：能用，但要把并行改成串行并自己补独立性

> [FAQ 首页](../README.md) · [问题原文](./question.md) · [harness 可移植性](../../_digest/system/harness-portability.md)

## 直接回答

能用。协议类 skill（`tdd`、`unslop`、`technical-writing`、`benchmark-checklist`、`how` 的简单路径）和单委托 playbook（Bug fix、Feature、Refactoring）不依赖多模型。真正失去的是**模型多样性带来的独立信号**，以及 `arena`、`interrogate`、`reflect` 的模型差异和 `shipping` 的独立 verifier。降级时要把 fan-out 改成串行，并用外部检查补回独立性。

## 仍然成立的能力

| 能力 | 为什么仍然成立 |
|---|---|
| `poteto-mode` 的步骤与门 | playbook 是文本协议，不依赖并行 |
| `bug-fix` / `feature` / `refactoring` | 单委托即可；把 subagent 换成自己在同一会话里做 |
| `how` 的简单路径 | 源码明确说简单问题一个 explainer 就够 |
| `tdd` | 红绿循环只需要一个执行者 |
| `unslop` / `technical-writing` | 写作规则与模型数量无关 |
| `benchmark-checklist` | 七问是检查表，一个人也能回答 |
| `show-me-your-work` | 轨迹可以手写；跨模型复查会缺，但 transcript 审计还在 |

## 失去的东西

- **候选竞争**：`arena` 的收益来自多个独立候选；单模型多次生成会共享同一个盲点。
- **对抗评审**：`interrogate` 的对抗信号明确来自模型差异，不是人设；单模型时只剩自我评审。
- **独立验证**：`shipping` 要求没写代码的 verifier；没有第二个 agent 时，这条门实际上被绕过。
- **并行覆盖**：`swarm` 的分区/竞速需要多个 worker；串行会显著变慢。
- **会话复盘**：`reflect` 的三个 reviewer（judgment、divergent 用 opus，tooling 用 grok）覆盖不同判断面；synthesizer 另派，也是 opus。单模型时视角仍在，跨模型差异消失。

## 怎样降级而不自欺

1. **按顺序扮演角色。** 没有子代理工具时，先做 explorer，再做 explainer；中间把发现写成文件，逼自己换视角。
2. **换会话做验证。** 写完代码后开一个新会话，只给它 brief 和 artifact，不给实现思路，让它按验收条件复核。
3. **优先用外部检查。** 测试、lint、类型、control surface、截图 diff 不依赖模型多样性，能补一部分独立性。
4. **明确记录缺口。** 在回复或轨迹里写"本次没有独立 verifier"，不要把作者自证写成 `PASS`。
5. **把 `shipping` 的门降级为人工门。** 如果没有人或第二个 agent 给 verdict，就不要自动合并；把栈推到 merge-ready，交给人。

## 成本变化

单模型省下的是 token，增加的是**验证的外部成本**。原本由第二个 agent 承担的独立复核，现在要由人、测试或 `control surface` 承担。如果这三者都没有，流程看起来仍然完整，证据强度已经下降。

## 继续阅读

- [当前行为的源码](../../skills/poteto-mode/SKILL.md)（**Harness** 与 **Subagents** 两节）
- [模型角色：收益与退化](../../_digest/patterns/model-diversity.md)
- [验证与 verdict](../../_digest/patterns/evidence-first.md)
