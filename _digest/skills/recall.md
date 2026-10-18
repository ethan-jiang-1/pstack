# `recall`

> 源码：[`skills/recall/SKILL.md`](../../skills/recall/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

在开始或接手续做之前，从两条记录里重建最近的工作上下文——自己的聊天记录，和同一批代码在别人名下的共享记录——再用 live state 校验，最后交回一份"现在在哪、下一步做什么"的紧简报。

## 它解决什么

新开的 chat 没有旧 chat 的记忆。旧 transcript 里有做过什么、决定了什么，但它只记住意图。一个功能的 bug 尾巴大多不在你的 transcript 里：用户反复报的症状、上线又回滚的修复、还在 prod 里烧的错误，这些都记在共享记录里，署名可能是别人。只读 transcript，会得到一个"我当时以为做完了"的版本。

`recall` 同时读两条记录，并规定用 `git` 和 `gh` 校验浮出来的 PR、分支和 ticket，让简报描述现在，而不是描述当时的印象。

## 核心机制

### 1. 先分类再路由

要恢复某个具体的旧 chat，走 `session-pickup` playbook，不是这里；把重复习惯变成 skill 是 `automate-me`；要一份人类可读的工作总结是另一件事。用户如果已经给了完整状态胶囊（路径、分支、改动内容），直接用它，跳过挖掘。

### 2. 先锁范围，再搜索

固定三件事：窗口（"recent" 是真实区间，默认最近 7 天）、主题（如果点名了的话）、workspace（默认当前 workspace，未经要求绝不读别的项目 transcript）。把范围复述回去。不允许把 "all" 悄悄改成 "recent N"。

### 3. 并行挖自己的聊天记录

transcript 是 JSONL，位置按 harness 不同：Cursor 在 `~/.cursor/projects/<slug>/agent-transcripts/`，Claude Code 在 `~/.claude/projects/<slug>/`，Pi 在 `~/.pi/agent/sessions/`，Codex 在 `~/.codex/sessions/YYYY/MM/DD/`（不按 workspace 分组，要按第一行的 `payload.cwd` 过滤），OpenCode 是 `~/.local/share/opencode/storage/` 下的 JSON。slug 规则各 harness 也不同：有的是去掉前导斜杠、把 `/` 换成 `-`，有的把所有非字母数字换成 `-`。

然后按范围分片，派并行 subagent。每个 subagent 被要求：按真实修改时间 `ls -t` 排序而不是按 UUID 名；先 grep 主题，再只读命中的 chat 的相关区域；跳过当前 chat 和明显噪音（subagent、eval、test chat）。每个 chat 返回同一套字段：topic、用户目标、决定、未结线程、挣扎与纠正、产物（PR、ticket、分支），并引用 chat UUID。只有一两个 chat 时不用 fan-out，直接搜。原始 transcript 留在 subagent 里，主线程只拿发现。

### 4. 扫共享记录是默认动作

只要主题点名了 feature、file、subsystem、area 或 bug，就要做，包括"我在 X 上的工作"这种听起来只关于自己的问法。做法是交给 `why` 的 source investigators，但把问题从"当初为什么这样建"改成"当前状态是什么、试过什么没成立、用户还在报什么"；复用它的 per-source playbook，与聊天挖掘并行，并继承它的姿态：一个源一个 investigator、空结果是发现、缺 MCP 就说明。只有纯活动回忆（"我这周做了什么"）且没有点名目标时才跳过。

### 5. 用 live state 校验

把挖掘和扫描里浮出来的 PR、分支、ticket 用 `git` 和 `gh` 查一遍。如果答案取决于 agent 实际做了什么（跑过哪些工具、读过哪些文件、撞过哪些错），读完整 transcript，而不是本地裁剪过的副本。

### 6. 按契约写简报

先是 Capsule，最多 5 条 bullet，说明这摊工作是什么、整体到哪一步。然后 Threads，每行一条，前缀恰好一个状态标签：`[merged #N]`、`[open PR #N]`、`[in flight <branch>]`、`[verified, uncommitted]`、`[reverted #N]`、`[planned, not started]`；没有标签就是还没标完。然后 Problems，最多 5 条反复出现的问题，要包含用户反复报的症状和上线后回滚的修复，让下一次尝试从上次失败的地方开始。最后 Next move，一个具体的动作。相邻的 feature 或 ticket 除非阻塞当前工作，否则不写。超出一屏时先砍细节，再砍线程。

整个简报过 `unslop`；chat 里的发现按 UUID 引用，共享记录里的发现按来源引用（PR 号、ticket ID、chat permalink、错误追踪 issue）；公开输出前脱敏。

## 设计意图

- **两条记录互相校正偏差。** transcript 记住意图，共享记录记住后果；缺一条就会得到偏斜的当前态。
- **subagent 是上下文防火墙。** 原始 transcript 不进主线程，是上下文预算管理，不只是并行优化。
- **固定字段加状态标签把简报变成状态板。** 每行可判定，扫一眼就知道哪些线程还开着。
- **live state 是最终仲裁者。** transcript 会过期，PR 和分支不会说谎。
- **锁范围是对挖掘类任务的硬约束。** 这类任务最常见的失败是越读越多，范围复述让偏离可见。

## 依赖关系

- **上游**：`why` 的 source investigator、per-source playbook 和调查姿态；`poteto-mode` 的 Subagents 规则（推理档位、快慢、模型）；各 harness 的 transcript 布局。
- **下游**：后续工作，`recall` 本身不产出代码。它产出的是简报和下一步。
- **对照**：`ketchup` 只看本 chat 自用户上一条消息之后；`session-pickup` 恢复单个具体 chat 或接管在飞的工作；`automate-me` 把习惯变成 skill。
- **共享**：`unslop` 负责语言，`git` 与 `gh` 负责校验。

## 使用边界

- **只重建"最近"。** 默认 7 天；更早的范围要显式给。
- **依赖 transcript 存在且可读。** 换机器、换 harness、清理过会话，第一条记录就没了。
- **布局是硬编码的。** Codex 不按 workspace 分组，必须读每个文件第一行的 `payload.cwd`；OpenCode 是 JSON 存储而不是 JSONL。这些路径会随 harness 版本漂移。
- **不跨项目。** 未经要求不读别的 workspace 的 transcript。
- **不做决定、不续做工作。** 它的产出是简报；要不要接着做是下一个 turn 的事。
- **公开输出前必须脱敏。** transcript 和 issue 里都可能有私有上下文。

## 值得注意

- **隐私与成本。** 为了重建上下文可能扫大量 transcript；源码用 scope 和 subagent 隔离控制成本，但"默认扫共享记录"意味着会触碰 issue 和 chat 里的他人内容。脱敏只写在输出前，没有读取时的过滤。
- **六个状态标签覆盖不了混合态。** 一个线程同时"已合并"又有后续未提交修复时，只能拆成多行或选一个主导状态，源码没有给拆分规则。
- **harness 路径表是某一时刻的快照。** 五个 harness 里 Codex 和 OpenCode 的布局与其余四个明显不同，这既说明它认真做过移植，也说明每加一个 harness 就要维护一条硬编码路径。
- **brief 没有置信度字段。** 它借用了 `why` 的 investigator 和"空结果是发现"的姿态，却没有继承五级置信度；简报区分了来源（chat UUID 还是 PR、ticket），读者仍要自己判断哪句是核实过的、哪句只是旧 chat 里的说法。

## 关键洞察

1. **上下文有两条记录。** 只读自己的 transcript，会把"我以为完成了"当成当前状态。
2. **把原始语料留在 subagent 里。** 这是上下文预算管理，不是并行技巧。
3. **状态标签是压缩。** 它强制每条线程可判定，也让缺口（还没标签）显形。
4. **先锁范围再搜索。** 对任何挖掘型 agent 任务，这都是最有效的成本控制。
5. **live state 优先于记忆。** 记忆说明意图，仓库说明事实。
