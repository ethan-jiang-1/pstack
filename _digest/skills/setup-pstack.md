# `setup-pstack`

> 源码：[`skills/setup-pstack/SKILL.md`](../../skills/setup-pstack/SKILL.md)
> 调用：**M**（model-invoked，人和模型都能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

用一次对话决定"每个角色跑哪个模型、用多高的推理预算"，写成一个全局设置文件，之后所有 pstack skill 按它选模型。

## 它解决什么

模型默认值原本散落在每个 skill 的正文里：`how` 有 explorer 和 explainer 两个角色，`arena` 有 runners 和 cross-judge pool，mode 的五个代码 playbook 由四条角色线覆盖（`feature, refactoring`、`bug-fix`、`perf-issue`、`hillclimb`）。换一次模型要改好多处，而用户实际可用的 slug 又取决于账号与 harness。

`setup-pstack` 把这件事抽出来：一张角色到模型的表，加一个全局预算，再加两个别名。skill 只声明"这里需要一个判断模型"，不关心判断模型是谁。

## 核心机制

### 1. 先探测可用模型

以"本会话能传给 subagent 的 slug"为准（Cursor 的 `Task`、Claude Code 的 `Agent`、OpenCode 的 `task`、Codex 的 `spawn_agent`）。harness 如果有列出模型的命令（源码举例 `opencode models`、`pi --list-models`），可以用它补全。检测不到就请用户粘贴。

两条硬规则：不写任何未确认可用的真实 slug；`inherit-parent` 和 `auto` 永远合法，即使检测不到。

### 2. 读当前状态

默认映射就是后面写出的那份规则形状。文件已存在时，`# budget` 行和角色值就是当前选择；不在该形状里的角色行（源码举例 `how critics`）视为已退役，丢掉。

### 3. 问预算，标签固定

四个选项，标签是固定的：`unlimited — max reasoning`、`large — xhigh reasoning`、`medium — high reasoning`、`small — medium reasoning`。没有设置文件时说明 `large` 等于 skill 默认。用结构化提问工具，不用自由文本。

### 4. 预算是机械改写，不是重新选模型

effort token 是 slug 的最后一个 token，或在 `-fast` 之前的那个；阶梯是 `max > xhigh > high > medium > low`。目标档位不存在时，取同家族里不高于目标的最高档；再不行就把该角色标成"需要选择"。`inherit-parent` 和 `auto` 不改。

源码给了四个例子：`unlimited` 把 `claude-opus-5-5-xhigh` 变成 `claude-opus-5-5-max`；Grok 最高到 `xhigh`，所以 `grok-4.7-xhigh-fast` 保持不变；`large` 保持两个默认值；`small` 得到 `claude-opus-5-5-medium` 与 `grok-4.7-medium-fast`。重跑时保留用户自己改过的角色，包括按家族、列表或别名写下的值。

### 5. 展示角色并确认

列出每个角色和它的模型，把不在检测集合里的标成需要选择，同时列出读当前状态时丢掉的角色行。问接受还是改某几个角色，选项是检测到的模型加两个别名。

面板角色（`arena runners`、`architect runners`、`interrogate reviewers`）的值是列表，一项一个 subagent，列表长度决定扇出，别名项也算数。`arena cross-judge pool` 也是列表，但 Arena 只从中挑一个值，并尽量选与父会话模型不同家族的那个。`swarm workers` 是每个 worker 的默认模型。

### 6. 校验

写下去的每个真实 slug 必须在检测集合里；别名直接通过；有不可用的就停下重问。

### 7. 整文件重写

Cursor 写 `~/.cursor/rules/pstack-models.mdc`，带 `alwaysApply: true` 的 frontmatter；其他 harness 写 `~/.agents/pstack-models.md`，从第一个 `#` 行开始，不带 frontmatter。内容是一行一个角色，标签与 mode 用的一致，`# budget: <label> (<effort>)` 记录预算。删掉一行就回退到 skill 默认。整文件覆盖是为了让重跑幂等。

### 8. 收尾

告知写了哪个文件。Cursor 对新 session 生效；其他 harness 由 skill 在选模型时读取。然后是一个可选的提议：项目若没有 `verify-*` 或等效 harness，问一次要不要用 `/create-verification-skill` 生成；被拒就不再劝。

## 设计意图

- **角色是稳定接口。** "判断与文字"不会因为模型换代而改变，slug 会。
- **预算是一根全局旋钮。** 让"这次要跑多贵"变成一个人能回答的问题，而不是当前这十七个角色的独立设置。
- **幂等写入。** 覆盖整个文件，重跑不会叠加出重复行或半旧半新的配置。
- **不猜能力。** 检测不到就标记"需要选择"，比写一个看起来合理的 slug 更安全：错误的 slug 会在运行时才失败。
- **别名是 Auto 用户的出口。** `auto` / `inherit-parent` 让跟随父会话模型成为一个显式选择，而不是靠省略配置。
- **配置即成本。** 面板列表长度直接决定 subagent 数量，所以配置面板就是配置花费。

## 依赖关系

- **上游**：人直接调用 `/setup-pstack`；`poteto-help` 的 **Get set up** 一节把它作为第二步；[guide 第 1 页](../../docs/guide/01-setup.md)。
- **下游**：所有声明了角色默认值的 skill，包括 `how`、`why`、`arena`、`swarm`、`architect`、`interrogate`、`reflect`，以及 mode 的四个代码 playbook 角色。mode 的 **Harness** 一节要求在任务开始时读取设置文件（存在的话）。
- **共享**：角色标签、预算阶梯和降级规则，见 [role line](../system/model-panel.md)。
- **对照**：`create-verification-skill` 是可选的下一步，不属于模型配置；mode 的 **Subagents** 一节在设置文件之上做每次 spawn 的二次调整。

## 使用边界

- **写的是 home 目录，不是项目文件。** 全局生效，不进版本库，也不随项目走。
- **不改 skill 默认值。** 删行或删文件才是回退；文件里的值只覆盖，不修改源码。
- **Cursor 只对新 session 生效**；其他 harness 靠 skill 主动读取。
- **不校验角色名。** 拼错的行不会报错，下次会被当作退役角色丢掉。
- **预算标签不等于每个 slug 都达标。** `unlimited` 对 Grok 仍然是 `xhigh`。

## 值得注意

- **`unlimited` 名不副实。** 文件会记下 `# budget: unlimited (max)`，而 Grok 行仍是 `xhigh`。源码把它写成 fallback 规则，没有在标签上让步，所以读文件的人不能假设每一行都到了标签承诺的档位。
- **默认 slug 是快照。** 没有版本映射；默认值升级后，旧规则仍钉住旧 slug，需要用户自己发现并删行重跑（[guide 第 1 页](../../docs/guide/01-setup.md) 明确提醒了这一点）。
- **全局文件无法团队共享。** `setup-pstack` 只写这两个 home 路径，没有项目级写入；同一个仓库的协作者各配各的。
- **探测结果不可搬运。** 同一用户在不同 harness 里能传的 slug 集合不同，配置文件不能直接复制过去。

## 关键洞察

1. **把模型选择集中到一个文件，是让多模型工作可维护的前提。** 否则每个 skill 都要重复默认值。
2. **角色比模型名稳定。** 用角色做键，模型换代时只改值。
3. **预算是一根旋钮，不是每个角色的独立设置。** 它把成本变成一个可以讨论的量。
4. **幂等写入比重跑正确更容易保证。** 覆盖整文件消除了合并逻辑。
5. **不猜能力，宁可让人选。** 一个未确认的 slug 是延迟到运行时才暴露的错误。
