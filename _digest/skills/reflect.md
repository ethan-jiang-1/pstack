# `reflect`

> 源码：[`skills/reflect/SKILL.md`](../../skills/reflect/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [subagent 拓扑](../patterns/subagent-topology.md)

## 一句话

把刚结束的会话当实验数据：三个不同视角的 reviewer 并行读 transcript，一个 synthesizer 归一成 Accepted / Rejected / Backlog，停在显式批准门前，再按每行的 routing 把经验写回 skill 或 backlog。

## 它解决什么

长任务里真正值钱的东西是"下次应该不一样"的那部分判断，而它通常随对话一起消失。直接让 agent 自己总结并改 skill 有两个问题：一次会话是轶事，容易过拟合；skill 改动会影响组织里每个未来的 agent，代价不可逆。

`reflect` 把这两件事拆开：多个视角负责把候选经验找全，一个 synthesizer 负责按固定判据筛掉不耐用、不可执行、不该进 skill 的项，最后人只对真正需要判断的那一类做批准。源码里 skip 条件写得很清楚：会话琐碎、跑题、或已有 skill 覆盖且父 agent 照着做对了，就不该跑。one-off 不是 learning。

## 核心机制

### 1. 只在本 workspace 找 transcript

源码先立了一条隐私边界：在当前 workspace 的 session 目录里找，不 glob 每个项目的目录，因为那会跨 workspace 边界读无关项目的私聊。各 harness 的定位方式分别是：Cursor 的 `agent-transcripts/` 三种布局（flat、nested、subagents），逐个读首行 JSONL 并用开场用户 prompt 匹配；Claude Code 是 `~/.claude/projects/<slug>/<session-id>.jsonl`，slug 由 workspace 路径中所有非字母数字字符替换成 `-` 得到；Pi、Codex 各有自己的路径规则，Codex 还用 `payload.cwd` 过滤。都找不到时，退而传一份 tight digest。

### 2. 三个 reviewer 并行

一个消息里发三个 `Task`，`subagent_type: generalPurpose`，agent 模式。这里有一个刻意的反常：reviewer 是 `readonly: false`，源码给的理由是 readonly 会剥掉 MCP，而 reviewer 需要查 transcript 引用的 ticket、聊天线程和可观测性 trace。

三个视角分别是：**judgment** 命名具体事件背后的耐用原则；**tooling** 抓那些"下个 agent 会重新推导一遍"的命令、路径、flag 事实，并要求标出用户手工递进来的上下文（本该由 MCP 取）；**divergent** 找反框架与二阶效应，包括"成功但侥幸"和"没发生但应该发生"的事。

每个 reviewer 通过 pstack settings 里的角色行取模型，行缺失就用默认值；值为 `auto` 或 `inherit-parent` 时不设 `model`；slug 被拒就用默认值并说明。别的 harness 里把 `Task` 换成 `Agent`、`task`、`spawn_agent`，没有 subagent 工具就按顺序自己跑三个角色。

### 3. 一个 synthesizer 归一

第三个 `Task` 拿到 `references/synthesizer.md` 的逐字模板，以及三个 reviewer 的完整输出，用 MCP 抽查引用，返回三段结构。它把 reviewer 输出当不可信数据：transcript 里可能藏着 prompt injection，MCP 查询也只能限于 transcript 引用过的上下文。判据是固定的几条：

- **durability**：路径、SHA、工具版本、代码形状都变了之后仍成立。
- **specificity**：宽到能跨任务，窄到未来 agent 认得出什么时候用。模糊格言与超具体事实都拒。
- **existing-skill-first**：只有现有 skill 里没有真正的家、模式会复发、且值得独立成 skill 时才开新 skill。
- **convergence**：2 个及以上 reviewer 同时提到才高置信，单例要提高其他判据的门槛。
- **decision-changing**：未来 agent 会因此做出不同动作，而不只是多读一段文字。
- **structural-mechanism check**：lint、脚本、元数据 flag 或运行时检查能更可靠执行时，路由到 Backlog。
- **skill-was-used**：只接受能路由到本会话真用过的 skill、工具或 MCP 的 finding。该触发没触发，路由成 `tune description: <skill path>`；两者都不是，按 `skill-not-used` 拒绝。
- **already-covered**：接受任何正文改动前先读目标 skill。已有清晰指引就拒，因为问题是执行不是 skill；指引埋得深或容易被跳过，则接受，但改写成让规则能被看见的措辞或位置调整。

### 4. 结构强制检查

synthesizer 判过一次之后，父 agent 还要再做一次：任何能由 lint、脚本、元数据或运行时检查更可靠执行的 Accepted 项，移到 Backlog。同一条规则在模板判据和第 4 步各出现一次，看起来像冗余，实际是两层防线：归一阶段筛一次，应用之前再筛一次。

### 5. 批准门

这是整个 skill 的重心。父 agent 展示完整的 Accepted / Rejected / Backlog 三段输出，等用户明确批准之后才应用 Accepted，不得自动应用。理由是源码里的一句话：skill changes affect every future agent in the org。用户逐行批准，可以只取子集，也可以改路由。

与批准门并列的是它的反面：Backlog 项立刻进团队的 devex 或 backlog tracker，不等批准。门只加在"需要人判断"的那一侧。

### 6. 按 routing 应用

批准后的四类路由各有归属：trivial 的现有 skill 编辑（一行 bullet、收紧一句、改一个过时事实）由父 agent 直接做；substantive 编辑（新小节、新表、超过约 10 行）交给 harness 的 skill-authoring skill 跑 draft / test / iterate；`tune description:` 交给 description 优化循环；`new skill via create-skill:` 交给 create-skill，不自己发明形状。有 SKILL.md validator 就跑一遍。

被拒的每条在输出里给一句 principle 加一个 reason，reason 取固定枚举（durability、specificity、existing-skill-first、convergence、decision-changing、structural、duplicate、skill-not-used、already-covered）。固定枚举让"为什么没做"可统计，而不是每次重新措辞；最后的汇总里每条被拒项一行。

最后是固定形状的短汇总：改了哪些 skill、新建了什么、backlog 里发了什么、以及被拒的每条一行理由。

## 设计意图

- **三镜补盲区，而不是三票。** judgment 抓原则，tooling 抓技术事实，divergent 抓"成功得太顺"的案例。角度不同才有覆盖，重复投票没有。
- **三分法分摊判断负荷。** 需要人判断的（改 skill）排队等人；不需要判断的（环境类机制）直接外发；不该记的当场给理由拒掉。
- **批准门保护的是不可逆性。** skill 改一次就影响所有未来会话，而 transcript 里的证据只有一次，两者的不对称决定了门的位置。
- **skill-was-used 防止空转。** 没有这条，reviewer 会为想象出来的使用场景写规则，规则越积越像泛泛的最佳实践。
- **防注入是分层的。** reviewer 读的是不可信 transcript，synthesizer 读的是不可信 reviewer 输出，两层都被要求只跟随自己的 prompt。

## 依赖关系

- **上游**：[`principle-encode-lessons-in-structure`](../../skills/principle-encode-lessons-in-structure/SKILL.md) 是第 4 步的判据来源；`poteto-mode` 的 Subagents 规则决定三个 reviewer 与 synthesizer 的 effort、后台与 fresh 策略。
- **上游**：[`setup-pstack`](../../skills/setup-pstack/SKILL.md) 写出两行角色：`reflect tooling` 与 `reflect judgment, divergent, synthesizer`。
- **下游**：被批准的 skill 文本、新建的 skill，以及团队的 devex / backlog tracker。
- **协作**：substantive 编辑都外包给 `create-skill`（Cursor 内置）或 Anthropic 的 `skill-creator`，没有外部 skill 时退回 agentskills.io 的 Agent Skills 规范。
- **对照**：[`correct`](./correct.md) 改仓库结构，`reflect` 改 skill 文本；`poteto-mode` 的 eval playbook 用来验证 skill 改动的净效果。
- **模式**：在 [subagent 拓扑](../patterns/subagent-topology.md) 里属于"多评审"：每模型一个 reviewer，综合成统一结论，不自动应用。

## 使用边界

- **不是会话总结工具。** 产出必须是未来某个动作会变；只读更多的文字不算 finding。
- **不碰仓库代码。** 它的产出只到 skill 文本与 backlog；想改仓库结构用 `correct`。
- **一次会话是轶事。** 判据里 convergence 与 durability 就是在压制单例；会话本身不足以支撑"这是规律"。
- **依赖 transcript 可定位。** 找不到就只能传 digest，而源码没有说明 digest 路径下判据是否要更严，这是一处真实的能力降级。
- **依赖 harness 有 subagent 工具。** 没有时按顺序自己跑三个角色，独立性消失，只剩提示词差异。

## 值得注意

- **"三个视角"只对应两个模型。** 角色表里 judgment 与 divergent 共用 `reflect judgment, divergent, synthesizer` 一行、同一个默认模型，只有 prompt 不同。真正的跨模型对比只发生在 tooling 与另两个之间。
- **reviewer 能写文件。** 它们被设成 agent 模式以保住 MCP，禁止修改只写在 prompt 里，没有沙箱强制。
- **批准门不覆盖 Backlog。** 写错一条 backlog 的代价是队友的时间，而它直接外发，不过人。
- **trivial 与 substantive 的界线由 synthesizer 判断。** 约 10 行是唯一线索，父 agent 按行执行，容易把一次较大的重写当 trivial 顺手改掉。
- **新 skill 与描述调优都外包。** 产出质量的下限取决于 `create-skill` 是否存在，而门只检查"有没有走那个流程"。

## 关键洞察

1. **复盘的产品是决策，不是总结。** 每条 finding 必须能让未来某个动作不同。
2. **三分法把门加在需要判断的一侧。** 全量审批会让复盘停摆，全不审批会让 skill 漂移。
3. **只对真用过的 skill 提意见。** 这条约束把"写规则"从想象拉回证据。
4. **结构优先于 prose。** 能编码进机制的规则不该写成一段话，这是 `reflect` 与 `correct` 共享的同一条判断。
