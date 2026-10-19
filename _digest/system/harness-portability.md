# harness 可移植性

## 一句话

pstack 按 Cursor 的模型写成，但把"换一个 harness 要改什么"写进了每个受影响的 skill：子代理工具、模型设置、会话记录位置、skill 目录、提问工具，以及缺哪个生态插件时怎样手工替代。这套写法让它在 Claude Code、Codex、Pi、OpenCode 上能跑，但可移植性来自**显式映射与降级**，不是抽象层。

## 映射表

| 能力 | Cursor | 其他 harness |
|---|---|---|
| 子代理 | `Task` | Claude Code `Agent`（`subagent_type: general-purpose`）；OpenCode `task`（`general`）；Codex `spawn_agent` |
| 云端 agent | `environment: "cloud"` | 本地独立 worktree |
| 子代理类型 | `poteto-agent` 已注册 | 未注册时派通用 subagent，prompt 第一句要求先读 `poteto-mode/SKILL.md` 全文 |
| 模型设置 | `~/.cursor/rules/pstack-models.mdc` | `~/.agents/pstack-models.md` |
| 推理档位 | 编码在 slug 里（`-xhigh`、`-fast`） | 子代理工具的 effort 选项，能设则设 |
| 会话记录 | system prompt 指定的 `agent-transcripts/` | Claude Code `~/.claude/projects/<slug>/`；Pi `~/.pi/agent/sessions/--<slug>--/`；Codex `~/.codex/sessions/` |
| 项目 skill 目录 | `.cursor/skills/` | `.claude/skills/`、`.pi/skills/`、`.agents/skills/` |
| 用户 skill 目录 | `~/.cursor/skills/` | `~/.claude/skills/`、`~/.pi/agent/skills/`、`~/.codex/skills/`、`~/.agents/skills/` |
| 结构化提问 | `AskQuestion` | Claude Code `AskUserQuestion`；OpenCode `question`；都没有时在聊天里给编号选项 |
| 无子代理工具 | 不适用 | Pi 无扩展时，按顺序自己做完每个委派步骤 |

这张表来自 `poteto-mode` 的 **Harness** 一节，以及各 skill 内重复的 **Other harnesses** 段。

## 生态插件的替代

pstack 有几个能力依赖 Cursor 生态，缺了要手工替代：

| 缺什么 | 替代 |
|---|---|
| `create-skill`（Cursor 内置） | Anthropic 的 `skill-creator`，或按 agentskills.io 的格式手写 |
| `deslop`（cursor-team-kit） | commit 前自己重读 diff，按 `unslop` 删 AI 痕迹 |
| `control-cli` / `control-ui`（cursor-team-kit） | Playwright 或 CDP 驱动 Web/Electron；tmux 或 PTY 驱动 CLI/TUI |
| `/loop`（Cursor / Claude Code 内置） | 定时 shell 命令等待，再检查 |
| Cursor cloud agent | 本地 worktree 中运行，父 agent 负责合流 |

替代方案不是等价物。云端 agent 的隔离性、控制技能的覆盖面、`/loop` 的唤醒语义，都会影响流程的可靠性。pstack 的处理方式是**明说降级**，而不是假装能力仍然存在。

## 镜像仓库做了什么

本仓库的 `main` 不是 Cursor 的原样拷贝。`MIRROR.md` 记录了本地编辑：

- `README.md` 顶部加一段镜像说明与安装方式，`mirror:end` 之后保持上游原文。
- 一些 skill 从 Cursor 专用改写为 harness 中立，例如把 `Task`、`agent-transcripts`、`cursor-team-kit` 的说明改成映射表。
- 目录名与 skill 名统一成 kebab-case（`poteto-mode`、`make-bot-ui`）。

这些编辑在同步时最容易被上游覆盖。`MIRROR.md` 的步骤要求先把 `main` 快进到 `origin/main`，再 merge 进 `ethan`；冲突时保留上游语义并重放本地措辞。

## 为什么可移植性没做成抽象层

pstack 没有引入"harness adapter"文件或能力探测脚本。它选择在正文里写条件分支：

```text
Task 在你的 harness 里是 Agent / task / spawn_agent。
环境是 cloud 时用云端；否则本地 worktree。
slug 不被接受时，用同家族最近的 slug，并说明。
```

好处是：agent 读到的就是当前要做的判断，不需要先理解一层抽象。坏处是：

- 同一段映射在 `how`、`why`、`reflect`、`swarm`、`interrogate` 里重复。
- 上游改一处，本地要跟五处。
- 没有单一事实来源。

这条路反过来走也自洽：把映射集中到一份共享文档，单一来源，代价是 agent 每次要先读一层抽象。pstack 选择就地写清楚，牺牲一致性换可执行性。

## 值得注意

- **"能跑"与"等价"不是一回事。** 没有云端 agent 时，`swarm` 的隔离假设变了；没有控制技能时，`shipping` 的独立验证会退化成 CI 加人工。
- **模型 slug 是最不可移植的部分。** 各 harness 的模型目录不同，pstack 只能用"同家族降级 + 说明"兜底。
- **`disable-model-invocation` 的执行方式仓库没有说明。** 见 [调用模型](./invocation-model.md)。
- **重复的映射段会漂移。** 修一处不等于修全部，这是当前结构的已知成本。

## 关键洞察

1. **可移植性可以是一组显式分支，不必是抽象层。** 代价是重复，收益是执行时不需要额外理解。
2. **降级要可见。** "用同家族最近的 slug 并说明"比悄悄换模型更可靠。
3. **能力缺失要写替代路径。** 只写"需要 control-ui"会把没装的人挡在门外。
4. **镜像仓库本身是移植工作的一部分。** 上游为 Cursor 写，本地为多 harness 改写，两者需要用分支隔离。

> 源码：[`poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Harness）、[`how`](../../skills/how/SKILL.md)、[`why`](../../skills/why/SKILL.md)、[`reflect`](../../skills/reflect/SKILL.md)、[`swarm`](../../skills/swarm/SKILL.md)、[`interrogate`](../../skills/interrogate/SKILL.md)。
