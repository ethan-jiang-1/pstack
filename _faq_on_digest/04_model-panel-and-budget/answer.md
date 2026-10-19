# 先跑 `/setup-pstack`，预算选 large，角色别全填同一个模型

> [FAQ 首页](../README.md) · [问题原文](./question.md) · [role line](../../_digest/system/model-panel.md)

## 直接回答

跑一次 `/setup-pstack`，让它探测当前 harness 真正可用的 slug，预算先用 `large`（xhigh），角色保持"代码用快模型、判断与文字用强模型"的分工。面板角色保留至少两个不同家族，否则 `arena` 和 `interrogate` 的多样性收益会消失。

## 第一步：让它探测，不要手写

`setup-pstack` 的第一步是枚举当前会话能传给 subagent 的模型 slug。它能检测到时，以检测结果为准；检测不到时才让你粘贴。不要写一个你没有确认可用的 slug，否则每次 spawn 都会走降级。

## budget 怎么选

| 预算 | 目标 effort | 适合 |
|---|---|---|
| `unlimited` | max | 最难的设计、并发、微妙算法；成本最高 |
| `large` | xhigh | 默认；与 skill 内默认值一致 |
| `medium` | high | 日常判断类工作 |
| `small` | medium | 例行、机械、成本敏感 |

两个细节：

- 预算是**全局起点**，不是每个角色的最终值。每次 spawn 前仍会按任务升档或降档。
- 目标档位在某个家族不存在时，取同家族不高于目标的最高档。Grok 目前最高到 `xhigh`，所以 `unlimited` 不会把它变成 `max`。

## 角色不是按 skill 分

角色按**工作性质**分，多个 skill 共享一个角色：

- **代码**：feature、refactoring、bug-fix、perf-issue、hillclimb、swarm workers。
- **判断与文字**：judgment and prose、hardest tasks、why synthesizer、how explainer、reflect judgment, divergent, synthesizer。
- **探索**：how explorer、why investigators、reflect tooling。
- **面板**：arena runners、architect runners、interrogate reviewers、arena cross-judge pool。

所以改一个角色会影响多个 skill，这正是把配置集中到文件里的目的。

## 面板角色为什么是列表

列表的每一项对应一个 subagent：

- `arena runners` 两项 → 两个候选；列表长度就是候选数。
- `architect runners` 两项 → 两个设计视角。
- `interrogate reviewers` 两项 → 两个 reviewer。
- `arena cross-judge pool` 是池子，Arena 会尽量选一个与父模型不同家族的值；池里没有别的家族时用池内其他值。

如果把面板角色全填成同一个模型，流程仍然会跑，但"独立候选"和"对抗评审"变成了同一模型的多次采样。

## `auto` 与 `inherit-parent`

两者在运行时等价：都表示这个角色跟随父会话模型，spawn 时省略 `model`。它们不需要出现在探测到的 slug 列表里。

## 常见错误

1. **写未确认的 slug。** 每次运行都降级，且日志里到处都是 fallback。
2. **所有角色填同一个模型。** 省事，但多模型流程退化成单模型。
3. **把预算当成硬上限。** 它只是起点；mode 仍会按任务调整。
4. **手改错文件。** Cursor 是 `~/.cursor/rules/pstack-models.mdc`，其他 harness 是 `~/.agents/pstack-models.md`。
5. **忽略面板 fan-out 的成本。** 每个列表项都是一个完整 subagent，一次运行可能是十几个。

## 继续阅读

- [当前行为的源码](../../skills/setup-pstack/SKILL.md)
- [role line：角色、预算与降级](../../_digest/system/model-panel.md)
- [模型角色](../../_digest/patterns/model-diversity.md)
