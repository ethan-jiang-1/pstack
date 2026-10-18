# pstack FAQ

这里回答"现在该怎么做"的具体问题。要理解系统为什么这样设计，读 [系统导读](../_digest/README.md)；要确认某个 skill 今天究竟会做什么，回到对应的 [`SKILL.md`](../skills/) 或 [`docs/guide/`](../docs/guide/README.md)。三者分工不同：FAQ 给决策，digest 给模型，源码给当前行为。

> 基线：pstack `0.15.25`，2026-10-19。涉及版本、模型 slug 或默认值的答案，都要按当前仓库重新核对。

## 从问题进入

| 你的问题 | 先读 | 再深入 |
|---|---|---|
| 我该用 `/poteto-mode`，还是直接调单个 skill？ | [01：mode 还是单 skill](./01_mode-or-single-skill/answer.md) | [mode 与 playbook](../_digest/system/mode-and-playbooks.md) |
| 只有一个模型、没有子代理，还能用吗？ | [02：单模型、无子代理](./02_single-model-no-subagents/answer.md) | [harness 可移植性](../_digest/system/harness-portability.md) |
| 24 条 principle 要背下来吗？ | [03：principle 怎么用](./03_do-i-need-to-memorize-principles/answer.md) | [principles 索引](../_digest/principles/README.md) |
| 模型面板怎么配，budget 选哪个？ | [04：模型与预算](./04_model-panel-and-budget/answer.md) | [模型面板](../_digest/system/model-panel.md) |
| 能只装几个 skill 吗？要装哪些？ | [05：安装范围](./05_which-skills-to-install/answer.md) | [分发与安装](../_digest/system/distribution.md) |
| 全自动、自主运行安全吗？怎么审？ | [06：自主边界](./06_how-much-autonomy-is-safe/answer.md) | [自主与门](../_digest/system/autonomy-and-gates.md) |

## 阅读路径

**只想马上开始。** 先读 [05](./05_which-skills-to-install/answer.md) 决定装什么，再读 [01](./01_mode-or-single-skill/answer.md) 决定用 mode 还是单 skill。模型配置见 [04](./04_model-panel-and-budget/answer.md)。

**想理解完整系统。** 先读 [系统总览](../_digest/system/README.md)，再读 [mode 与 playbook](../_digest/system/mode-and-playbooks.md) 和 [证据优先](../_digest/patterns/evidence-first.md)。需要落到具体动作时，回到本目录。

**想跑长任务或自主运行。** 从 [06](./06_how-much-autonomy-is-safe/answer.md) 开始，再读 [`orchestrate`](../skills/poteto-mode/playbooks/orchestrate.md) 与 [`shipping`](../skills/poteto-mode/playbooks/shipping.md)。先确认验证与审计，再谈自动化程度。

**想改 pstack。** 读 [`_digest/MAINTAINING.md`](../_digest/MAINTAINING.md) 和 [`authoring-a-skill`](../skills/poteto-mode/playbooks/authoring-a-skill.md)，再用 [`reflect`](../skills/reflect/SKILL.md) 决定改动是否值得。

## 问题的边界

（编号即目录号。新增问题从下一个空闲编号开始。）

### 入口与成本

- **01** [用 mode 还是单 skill？](./01_mode-or-single-skill/question.md)
- **02** [单模型、无子代理时的可用性](./02_single-model-no-subagents/question.md)

### 判断与配置

- **03** [principle 的用法](./03_do-i-need-to-memorize-principles/question.md)
- **04** [模型面板与预算](./04_model-panel-and-budget/question.md)

### 安装与治理

- **05** [安装范围与依赖](./05_which-skills-to-install/question.md)
- **06** [自主边界与审计](./06_how-much-autonomy-is-safe/question.md)

## 以后怎样追加问题

1. 用 [`_TEMPLATE.md`](./_TEMPLATE.md) 建 `NN_slug/` 目录，编号递增。
2. `question.md` 写读者真正会问的问题，不是代号。
3. `answer.md` 先给一句可执行结论，再解释条件、例外和下一步。
4. 只有一个维度会让主回答明显变长时，才加补充页。
5. 结论必须与 `README.md`、相关 `SKILL.md` 和相邻 FAQ 一致；涉及版本或模型名时标注基线。
