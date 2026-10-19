# pstack 体系解读

这是对 [pstack](https://github.com/cursor/plugins/tree/main/pstack) 的结构化解读。它不逐句翻译 `SKILL.md`，而是回答四个更大的问题：这套系统怎样组织工作、为什么这样拆 skill、判断与证据在哪里收口，以及每个机制在源码里的位置。

> **一句话理解：** pstack 是一套把"agent 团队"按工程纪律组织起来的协议集。mode 负责选路，playbook 约束流程，principle 提供可点名的判断词汇，role line 与子代理承担并行，证据与验证负责收口。

## 五分钟心智模型

大多数工作沿一条主线前进：

```text
请求
  → poteto-mode：读任务，匹配一个 playbook
  → 把 playbook 的步骤逐字复制进 todo list（跳过的步骤要留一行原因）
  → 步骤内调用 skill（how / why / architect / tdd …）或派 subagent
  → 在真实表面验证（control skill、repro、benchmark checklist）
  → 按 unslop 规则写回复；开 PR 前跑 technical-writing
  → 长任务：show-me-your-work 留下可审计的 decision trail
```

其他入口不会另造一套流程，而是在合适的位置汇入：

- 不知道用哪个 skill：`poteto-help` 回答并给出可以直接发送的 prompt，但不替用户开工。
- 读代码、读历史：`how` 解释运行机制，`why` 追决策动机，`teach` 把两者讲成一段可理解的说明。
- 重建自己的工作上下文：`recall` 重建当前状态；接手别人的在途工作用 Session pickup。`ketchup` 用平实语言补齐这段对话的变化。
- 设计没把握：`architect` 先画形状，`arena` 让多个候选互相竞争，`interrogate` 做多模型对抗评审。
- 环境反复出同一种错：`correct` 先改架构让错误写不出来，再依次用类型、lint、测试、文档，`reflect` 把这次会话的经验写回 skill。

整套系统靠四条原则保持可控：

| 原则 | 解决什么问题 | 去哪里深入 |
|---|---|---|
| **路由边界** | 谁来选流程，mode、playbook 与单点 skill 的职责怎样分开 | [mode 与 playbook](./system/mode-and-playbooks.md) |
| **流程与门** | agent 何时可以从理解进入行动，何时必须停下补证据 | [自主与门](./system/autonomy-and-gates.md) |
| **委派与模型** | 什么时候值得派 subagent，怎样用模型差异换判断质量 | [role line](./system/model-panel.md) |
| **证据与收口** | 怎样算"做完了"，回复、PR 与 decision trail 各自留下什么 | [验证与 verdict](./patterns/evidence-first.md) |

完整图景见 [系统总览](./system/README.md)。不知道该找哪个 skill 时，直接看 [技能索引](./skills/README.md)。

## 选择阅读深度

### 只用五分钟

读 [系统总览](./system/README.md)。它把分发、路由、流程、词汇、委派与证据放在同一张地图上。

### 用二十分钟建立结构

1. [系统总览](./system/README.md)：先知道各层分别负责什么。
2. [机制总览](./patterns/README.md)：按 pstack 机制查 playbook、验证、委派与环境。
3. [技能索引](./skills/README.md)：按手头情境找到具体入口。

### 深入设计原理

1. [mode 与 playbook](./system/mode-and-playbooks.md)：请求怎样变成一条有步骤、有门的执行协议。
2. [调用模型](./system/invocation-model.md)：为什么几乎全部 skill 都只能由人触发。
3. [SKILL.md 结构解剖](./system/skill-anatomy.md)：frontmatter、正文、reference 与 harness 映射怎样落盘。
4. [role line](./system/model-panel.md)：角色、预算、模型家族与降级规则。
5. [分发与安装](./system/distribution.md)：插件、skills CLI 与部分安装的代价。
6. [harness 可移植性](./system/harness-portability.md)：Cursor 专用写法怎样映射到其他 agent。

## 按问题找入口

| 你想理解什么 | 先读 | 再读 |
|---|---|---|
| 这套系统整体怎样运转？ | [系统总览](./system/README.md) | [mode 与 playbook](./system/mode-and-playbooks.md) |
| 我现在该用哪个 skill 或 playbook？ | [技能索引](./skills/README.md) | [playbook 索引](./playbooks/README.md) |
| 为什么几乎每个 skill 都要手动触发？ | [调用模型](./system/invocation-model.md) | [FAQ：mode 还是单 skill](../_faq_on_digest/01_mode-or-single-skill/answer.md) |
| 24 条 principle 到底怎么用？ | [principles 索引](./principles/README.md) | [principle 的用法](./patterns/principles-as-vocabulary.md) |
| 多模型是不是必需的？ | [role line](./system/model-panel.md) | [模型角色](./patterns/model-diversity.md) |
| 怎样防止 agent 跳过复现或过早宣布完成？ | [验证与 verdict](./patterns/evidence-first.md) | [自主与门](./system/autonomy-and-gates.md) |
| 什么时候值得派 subagent？ | [Subagent 与 brief](./patterns/subagent-topology.md) | [子代理调用模型](./system/invocation-model.md) |
| 换到 Claude Code / Codex / Pi 还能用吗？ | [harness 可移植性](./system/harness-portability.md) | [FAQ：单模型、无子代理](../_faq_on_digest/02_single-model-no-subagents/answer.md) |
| 上游更新后，这些解读怎么跟上？ | [Change Log](./_change_log/README.md) | [维护约定](./MAINTAINING.md) |

## 目录边界

```text
_digest/
├── system/          系统级机制：分发、路由、调用、模型、结构、可移植性、自主
├── principles/      24 条 principle 的分组索引
├── playbooks/       23 个 playbook 的选择树与逐项说明
├── skills/          30 个非 principle skill 的单页解读
├── patterns/        按机制组织的解释页
├── figures/         机制图（手写 SVG）
└── _change_log/     上游同步记录与影响矩阵
```

`_digest` 是解释层，不是行为的单一事实来源。当前行为以仓库中的 `skills/**/SKILL.md` 与 `skills/poteto-mode/playbooks/` 为准；解读与源码冲突时，应修正解读。历史变化只记录在 [`_change_log`](./_change_log/README.md)，当前页尽量保持无时间依赖。

## 与 `docs/guide/` 的分工

pstack 自带的 `docs/guide/` 是**操作教程**：英文，面向"今天要跑一遍"的使用者，按 setup → mode → understand → design → build → verify → overnight，再到 principles → make it yours → recipes and pitfalls 的顺序走。

本目录是**结构与设计解读**：中文，面向想理解"为什么这样拆、代价是什么、每个机制在源码里的位置"的读者。两者会引用同一批 skill，但不互相复制：guide 教流程，digest 解释机制。

姊妹目录：

- [`_faq_on_digest/`](../_faq_on_digest/README.md)：按真实问题给决策。
- [`_handbook_for_agent_engineering/`](../_handbook_for_agent_engineering/README.md)：pstack 的概念与用法（教程 12 步 + 手册 6 章）。
- [`_change_log/`](./_change_log/README.md)：每次上游同步改变了什么。
