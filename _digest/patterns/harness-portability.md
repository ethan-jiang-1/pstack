# `poteto-mode` 的 Harness

`poteto-mode` 为 Cursor 写成。它的 Harness 一节逐项给出换 harness 时读 Cursor 专用指令的映射：子代理工具、`poteto-agent`、模型设置、会话记录、skill 目录、提问工具、生态插件。判断自己在哪个 harness，看这些路径哪个存在。

## 映射

### 子代理

`Task` 在 Claude Code 是 `Agent`（`subagent_type: general-purpose`），OpenCode 是 `task`（`general`），Codex 是 `spawn_agent`。工具没有的参数直接丢掉。`environment: "cloud"` 只表示 Cursor 云端 agent；别处把该 agent 放到它自己的 worktree 里跑。没有子代理工具时（例如未装扩展的 Pi），把每个委派步骤按顺序自己做。

`poteto-agent` 没注册成 subagent 的地方，派一个通用 subagent，prompt 第一句要求它先完整读 `poteto-mode/SKILL.md`，包括 Principles。

### 模型设置

Cursor 自动加载 `~/.cursor/rules/pstack-models.mdc`。别处由 `/setup-pstack` 写 `~/.agents/pstack-models.md`，task 开始时如果存在就先读。slug 在本地不解析时，用最接近的等价物。Cursor 把推理档位和速度写进 slug（`-xhigh`、`-fast`）；别处用子代理工具的 effort 选项，工具设不了的就跳过。

### 会话记录

只读当前 workspace 的 session，绝不读别的项目。Claude Code 在 `~/.claude/projects/<slug>/*.jsonl`，`<slug>` 是 workspace 路径里每个非字母数字字符换成 `-`。Pi 在 `~/.pi/agent/sessions/--<slug>--/*.jsonl`。Codex 在 `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`，只保留首行 `payload.cwd` 等于 workspace 路径的文件。OpenCode 在 `~/.local/share/opencode/storage/`。

### skill 目录

项目：`.cursor/skills/`、`.claude/skills/`、`.pi/skills/`、`.agents/skills/`。用户：`~/.cursor/skills/`、`~/.claude/skills/`、`~/.pi/agent/skills/`、`~/.codex/skills/`、`~/.agents/skills/`。

### 提问

`AskQuestion` 在 Claude Code 是 `AskUserQuestion`，OpenCode 是 `question`。都没有时在聊天里给编号选项。

### 生态插件

`create-skill` 是 Cursor 内置。别处用 Anthropic 的 `skill-creator`，或按 agentskills.io 的 Agent Skills 格式写。`deslop`、`control-cli`、`control-ui` 来自 `cursor-team-kit`。没有 `deslop`，就在 commit 前自己重读 diff，按 `unslop` 删。没有 control skill，就用现有自动化驱动应用：Web 与 Electron 用 Playwright 或 CDP 浏览器，CLI 与 TUI 用 tmux 或 PTY。`/loop` 在 Cursor 与 Claude Code 内置，没有就用定时 shell 命令等下一次检查。

源码：[`skills/poteto-mode/SKILL.md#harness`](../../skills/poteto-mode/SKILL.md#harness)。

## 降级要自报

映射不是等价替换。各 skill 在落不回去时都要求把降级说出来：

- `arena`：配置的条目被拒，该 runner 在同家族默认模型上跑，并说明。
- `interrogate`：配置的条目被拒，该 reviewer 用同家族默认值，并说明；拒绝默认值时才从有效 slug 里找最近的。
- `swarm`：slug 被拒时用默认模型并说明。`auto` 或 `inherit-parent` 的 worker 跑在父 chat 的模型上。
- `reflect`：slug 被拒时用默认值并说明；`auto` 或 `inherit-parent` 时省略 `model`。

源码：[`skills/arena/SKILL.md`](../../skills/arena/SKILL.md)、[`skills/interrogate/SKILL.md`](../../skills/interrogate/SKILL.md)、[`skills/swarm/SKILL.md`](../../skills/swarm/SKILL.md)、[`skills/reflect/SKILL.md`](../../skills/reflect/SKILL.md)。

## 代价

- 同一段映射在多处重复。上游改一处，本地要跟多处。
- 没有单一事实来源，几份映射会逐渐不一致。
- "能跑"不等于"等价"。没有云端 agent，隔离假设就变了；没有 control skill，验证走的是另一条路径。

## 关键洞察

1. 换 harness 时要改的是工具名、路径和缺失能力的替代，不是 playbook 的步骤。
2. 显式映射比抽象层更容易执行，代价是重复。
3. 降级不说出来，就会变成错误的证据。
