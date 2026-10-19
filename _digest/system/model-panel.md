# role line 与模型设置

## 一句话

`setup-pstack` 把"哪个 role 用哪个模型"写进一份设置文件，一行一个 role。skill 派 subagent 时先读这份文件；role line 缺失时，退回该 skill 内写的默认值。0.15.25 的默认分工是代码 role 用快模型，判断与文字 role 用强模型。

## 设置文件

Cursor 写 `~/.cursor/rules/pstack-models.mdc`（带 `alwaysApply: true`），其他 harness 写 `~/.agents/pstack-models.md`。格式是一行一个 role：

```text
# budget: large (xhigh)
feature, refactoring: grok-4.7-xhigh-fast
judgment and prose: claude-opus-5-5-xhigh
arena runners: claude-opus-5-5-xhigh, grok-4.7-xhigh-fast
```

- 值可以是单个 slug、逗号分隔的列表，或 `inherit-parent` / `auto`。
- `arena runners`、`architect runners`、`interrogate reviewers` 的值是列表：每一项各跑一个 subagent，列表长度决定 fan-out 数量。
- `arena cross-judge pool` 也是列表，但 Arena 从中选一个值，并尽量让它的模型家族不同于父会话。
- `swarm workers` 是每个 worker 的默认模型，除非 race 或 comparison 按 arm 另配。
- 删掉一行，该 role 退回 skill 内写的默认值。

## 默认 role line

以下是 plugin `0.15.25`（2026-10-19）的默认值；slug 会随模型发布过时。

| role | 默认 | 用途 |
|---|---|---|
| `feature, refactoring` | `grok-4.7-xhigh-fast` | 功能与重构的代码委派 |
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
| `arena runners` / `arena cross-judge pool` | opus + grok | 候选生成 / 跨模型裁判池 |
| `swarm workers` | `grok-4.7-xhigh-fast` | 并行 worker |
| `architect runners` | opus + grok | 设计候选 |
| `interrogate reviewers` | opus + grok | 对抗评审 |

## 预算

`setup-pstack` 问四选一，然后把每个真实 slug 的 effort token 改成对应档位：

| 预算 | effort |
|---|---|
| unlimited | max |
| large | xhigh |
| medium | high |
| small | medium |

- effort token 是 slug 的最后一个 token，或在 `-fast` 之前的那个。阶梯是 `max` > `xhigh` > `high` > `medium` > `low`。
- 目标档位不存在时，取同家族里不高于目标的最高档；该家族没有可用 slug 时，把该 role 标成"需要选择"。Grok 目前最高到 `xhigh`，所以 `unlimited` 下 Grok 仍是 `xhigh`。
- `inherit-parent` 和 `auto` 不参与改写，表示这个 role 跑在父会话模型上。

## 每个 spawn 前再调整

设置文件给的是起点，不是终点。`poteto-mode` 的 **Subagents** 规则要求每次 spawn 前按任务再调：

- 最难或最微妙的工作：升一档，最高到 max。
- 判断类工作：降一档。
- 例行工作：降两档或更多。
- 只有 CI 或主干红了、有人正被结果堵住时才用 fast；否则用标准速度（同 slug 去掉 `-fast`）。
- 验证是例外：保持在 `swarm workers` 的模型与速度上，保证验证者与实现者的分工稳定。
- 可以改的只有 effort 和是否 fast，不能换家族，否则列表型 role line 会退化成同一个模型跑多遍。

## 降级

所有 skill 共享同一套写法：

1. 设置文件或该行缺失：用 skill 内的默认值。
2. 值是 `auto` / `inherit-parent`：省略 `model`，跟随父会话。
3. 调整后的 slug 被 harness 拒绝：用 role line 或默认值，并说明。
4. 默认值也被拒绝：用错误信息或 harness 模型列表里同家族最近的 slug，并说明。降级必须可解释，不悄悄换家族。

## 边界

- **默认 slug 是快照。** `claude-opus-5-5-xhigh`、`grok-4.7-xhigh-fast` 这类名字会随模型发布过时；源码没有版本映射表。
- **role 名是自由文本。** 设置文件没有 schema 校验，拼错 role 名不会报错，只会悄悄退回默认值。
- **effort 只有部分 harness 支持。** Claude Code 的 `Agent` 有 effort 选项；别处可能只能靠 slug 本身编码档位。
- **列表型 role line 的成本会叠加。** `arena runners`、`architect runners`、`interrogate reviewers` 各两项时，一次运行可能派出十几个 agent。

## 关键洞察

1. 把模型选择集中到一个文件，是让多 role 工作可维护的前提；否则每个 skill 都要重复默认值。
2. role 比模型名稳定。"判断与文字"这个 role 不会因为模型换代而改变。
3. 预算是一根全局旋钮，不是每个 role 的独立设置。这让"这次要跑多贵"变成一个可回答的问题。
4. 降级必须同家族。换家族会破坏列表型 role line 的差异，比降档更危险。

## 源码

- [`skills/setup-pstack/SKILL.md`](../../skills/setup-pstack/SKILL.md)（Steps 2-6）
- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Subagents）
- [`skills/arena/SKILL.md`](../../skills/arena/SKILL.md)、[`architect`](../../skills/architect/SKILL.md)
