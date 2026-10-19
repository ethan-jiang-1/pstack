# 模型面板

## 一句话

pstack 把"用哪个模型"从每个 skill 里抽出来，变成一张**角色到模型**的映射表，再加一个全局推理预算。skill 只声明自己需要什么角色；`setup-pstack` 决定每个角色跑在哪个 slug 上。

## 设置文件

Cursor 写 `~/.cursor/rules/pstack-models.mdc`（带 `alwaysApply: true`），其余 harness 写 `~/.agents/pstack-models.md`。格式是一行一个角色：

```text
# budget: large (xhigh)
feature, refactoring: grok-4.7-xhigh-fast
judgment and prose: claude-opus-5-5-xhigh
arena runners: claude-opus-5-5-xhigh, grok-4.7-xhigh-fast
```

- 值可以是单个 slug、逗号分隔的列表，或 `inherit-parent` / `auto`。
- 列表只对"面板"角色有意义：`arena runners`、`architect runners`、`interrogate reviewers` 的每一项各跑一个 subagent；列表长度决定 fan-out 数量。
- 删掉一行，该角色退回 skill 内写的默认值。

## 角色与默认值

下表是 pstack `0.15.25` 的默认值；slug 会随模型发布过时。

| 角色 | 默认 | 用途 |
|---|---|---|
| `feature, refactoring` | `grok-4.7-xhigh-fast` | 代码委派 |
| `bug-fix` | `grok-4.7-xhigh-fast` | 修复实现 |
| `perf-issue` | `grok-4.7-xhigh-fast` | 性能修复 |
| `hillclimb` | `grok-4.7-xhigh-fast` | 指标爬坡循环 |
| `judgment and prose` | `claude-opus-5-5-xhigh` | 判断与文字 |
| `hardest tasks` | `claude-opus-5-5-xhigh` | 跨切面设计、并发、微妙算法 |
| `how explorer` | `grok-4.7-xhigh-fast` | 代码探索 |
| `how explainer` | `claude-opus-5-5-xhigh` | 机制解释 |
| `why investigators` | `grok-4.7-xhigh-fast` | 证据检索 |
| `why synthesizer` | `claude-opus-5-5-xhigh` | 动机综述 |
| `reflect tooling` | `grok-4.7-xhigh-fast` | 会话复盘的工具视角 |
| `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` | 复盘判断、发散、综合 |
| `arena runners` | opus + grok | 候选生成 |
| `arena cross-judge pool` | opus + grok | 跨模型裁判池 |
| `swarm workers` | `grok-4.7-xhigh-fast` | 并行 worker |
| `architect runners` | opus + grok | 设计候选 |
| `interrogate reviewers` | opus + grok | 对抗评审 |

默认值表达一个明确的分工假设：**代码交给快模型，判断和文字交给强模型**。这不是能力排名，而是成本与质量的分配。

## 预算阶梯

`setup-pstack` 问你四选一，然后把每个真实 slug 的 effort token 改成对应档位：

| 预算 | effort |
|---|---|
| unlimited | max |
| large | xhigh |
| medium | high |
| small | medium |

规则细节：

- effort token 是 slug 的最后一个 token，或在 `-fast` 之前的那个。
- 阶梯是 `max` > `xhigh` > `high` > `medium` > `low`。
- 目标档位不存在时，取同家族里不高于目标的最高档。Grok 目前最高到 `xhigh`，所以 `unlimited` 下 Grok 仍是 `xhigh`。
- `inherit-parent` 和 `auto` 不参与改写，表示"用父会话的模型"。

## 每个 spawn 前的二次调整

设置文件给的是**起点**，不是终点。mode 的规则要求每次 spawn 前按任务再调：

- 最难或最微妙的工作：升一档，最高到 max。
- 判断类工作：降一档。
- 例行工作：降两档或更多。
- 只有 CI/主干红了、有人被结果堵住时才用 fast；否则用标准速度（同 slug 去掉 `-fast`）。
- 验证是例外：保持在 `swarm workers` 的模型与速度上，保证验证者与实现者的分工稳定。

可以改的只有 effort 和是否 fast，**不能换家族**。否则多模型面板会退化成同一个模型跑两遍。

## 降级规则

所有 skill 共享同一套写法：

1. 设置文件或该行缺失 → 用 skill 内的默认值。
2. 值是 `auto` / `inherit-parent` → 省略 `model`，跟随父会话。
3. 选中的 slug 被 harness 拒绝 → 用同家族默认值，并在回复里说明。
4. 默认值也被拒绝 → 用错误信息或 harness 模型列表里同家族、同档位最近的 slug；仍不行就报告。

这套规则的价值在于：**降级是可解释的**，不会悄悄换成另一个家族的模型。

## 为什么强调模型差异

`arena`、`interrogate`、`reflect` 的收益来自不同模型看到不同的东西；`swarm` 的主收益是并行覆盖，模型竞速只是可选的额外维度。

- `arena` 让多个候选竞争，跨模型裁判选择基础版本。
- `interrogate` 的对抗信号来自模型多样性，不来自给每个模型分配"人设"。
- `reflect` 用三个不同视角复查同一段会话。
- `swarm` 的 worker 可以按切片或按模型分组比赛。

同一个模型跑三次，得到的是三次相似的判断；不同模型跑三次，才可能得到独立证据。

## 值得注意

- **默认 slug 是快照。** `claude-opus-5-5-xhigh`、`grok-4.7-xhigh-fast` 这类名字会随模型发布过时；源码没有版本映射表。
- **effort 只有部分 harness 支持。** Claude Code 的 `Agent` 有 effort 选项；别处可能只能靠 slug 本身编码档位。
- **角色是自由文本。** 设置文件没有 schema 校验，拼错角色名不会报错，只会悄悄退回默认值。
- **面板成本会叠加。** `arena runners` 两项、`architect runners` 两项、`interrogate reviewers` 两项，再加上每个候选自己的 subagent，一次运行可能是十几个 agent。

## 关键洞察

1. **把模型选择集中到一个文件，是让多模型工作可维护的前提。** 否则每个 skill 都要重复默认值。
2. **角色比模型名稳定。** "判断与文字"这个角色不会因为模型换代而改变。
3. **预算是一根全局旋钮，不是每个角色的独立设置。** 这让"这次要跑多贵"变成一个可回答的问题。
4. **降级必须同家族。** 换家族会破坏面板的多样性假设，比降档更危险。

> 源码：[`setup-pstack/SKILL.md`](../../skills/setup-pstack/SKILL.md)、[`poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Subagents）。
