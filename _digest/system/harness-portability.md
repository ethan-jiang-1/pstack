# harness 可移植性

## 一句话

pstack 按 Cursor 的模型写成，但在 `poteto-mode` 的 **Harness** 一节，以及 `how`、`why`、`reflect`、`swarm`、`interrogate` 各自的 **Other harnesses** 段里，逐项写了换 harness 时要换成什么。可移植性来自这些显式映射和降级，不来自中间抽象。

## 映射

| 能力 | Cursor | 其他 harness |
|---|---|---|
| subagent 工具 | `Task` | Claude Code `Agent`（`subagent_type: general-purpose`）；OpenCode `task`（`general`）；Codex `spawn_agent` |
| 云端 agent | `environment: "cloud"` | 本地独立 worktree |
| `poteto-agent` | 已注册的子代理类型 | 未注册时派通用 subagent，prompt 第一句要求先读 `poteto-mode/SKILL.md` 全文 |
| 模型设置 | `~/.cursor/rules/pstack-models.mdc` | `~/.agents/pstack-models.md` |
| 推理档位与速度 | 编码在 slug 里（`-xhigh`、`-fast`） | 用 subagent 工具的 effort 选项；工具没有就跳过 |
| 会话记录 | system prompt 指定的 `agent-transcripts/` | Claude Code `~/.claude/projects/<slug>/`；Pi `~/.pi/agent/sessions/--<slug>--/`；Codex `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`；OpenCode `~/.local/share/opencode/storage/` |
| 项目 skill 目录 | `.cursor/skills/` | `.claude/skills/`、`.pi/skills/`、`.agents/skills/` |
| 用户 skill 目录 | `~/.cursor/skills/` | `~/.claude/skills/`、`~/.pi/agent/skills/`、`~/.codex/skills/`、`~/.agents/skills/` |
| 结构化提问 | `AskQuestion` | Claude Code `AskUserQuestion`；OpenCode `question`；都没有时在聊天里给编号选项 |
| 无 subagent 工具 | 不适用 | Pi 无扩展时，按顺序自己做完每个委派步骤 |

`arena` 和 `architect` 不重复这张表：它们只写设置文件路径，并复用 `arena` 的 runner 规则。

## 缺生态插件时的替代

| 缺什么 | 替代 |
|---|---|
| `create-skill`（Cursor 内置） | Anthropic 的 `skill-creator`，或按 agentskills.io 的 Agent Skills 格式手写 |
| `deslop`（cursor-team-kit） | commit 前自己重读 diff，按 `unslop` 删 AI 痕迹 |
| `control-cli` / `control-ui`（cursor-team-kit） | Web/Electron 用 Playwright 或 CDP；CLI/TUI 用 tmux 或 PTY |
| `/loop`（Cursor / Claude Code 内置） | 用定时 shell 命令等待，再检查 |
| Cursor cloud agent | 在本地 worktree 中运行 |

替代方案不是等价物。云端 agent 的隔离性、control skill 的覆盖面、`/loop` 的唤醒语义，都会影响流程的可靠性。pstack 的处理方式是明说降级，而不是假装能力仍在。

## 为什么写在正文里

pstack 没有 harness adapter 文件，也没有能力探测脚本。它在正文里写条件分支：

```text
Task 在你的 harness 里是 Agent / task / spawn_agent。
environment 是 cloud 时用云端；否则本地 worktree。
slug 不被接受时，用同家族最近的 slug，并说明。
```

好处是 agent 读到的就是当前要做的判断，不需要先理解一份中间抽象。代价是同一段映射在五个 skill 里重复，上游改一处，本地要跟五处，没有单一来源。

## 边界

- **"能跑"与"等价"不是一回事。** 没有云端 agent 时，`swarm` 的隔离假设变了；没有 control skill 时，`shipping` 的独立验证只能靠现有自动化或人工。
- **模型 slug 是最不可移植的部分。** 各 harness 的模型目录不同，pstack 只能用"同家族降级 + 说明"兜底。
- **`disable-model-invocation` 的执行方式仓库没有说明。** 见 [调用模型](./invocation-model.md)。
- **重复的映射段会漂移。** 修一处不等于修全部，这是当前结构的已知成本。

## 关键洞察

1. 可移植性可以是一组显式分支，不必是中间抽象。代价是重复，收益是执行时不需要额外理解。
2. 降级要可见。"用同家族最近的 slug 并说明"比悄悄换模型更可靠。
3. 能力缺失要写替代路径。只写"需要 control skill"会把没装的人挡在门外。
4. 映射写在正文里，读者读到的就是当前要执行的判断；代价是没有单一来源。

## 源码

- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Harness）
- [`skills/how/SKILL.md`](../../skills/how/SKILL.md)、[`why`](../../skills/why/SKILL.md)、[`reflect`](../../skills/reflect/SKILL.md)、[`swarm`](../../skills/swarm/SKILL.md)、[`interrogate`](../../skills/interrogate/SKILL.md)
- [`skills/arena/SKILL.md`](../../skills/arena/SKILL.md)、[`architect`](../../skills/architect/SKILL.md)
