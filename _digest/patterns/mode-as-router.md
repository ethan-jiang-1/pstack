# `poteto-mode` 的 playbook 路由

`poteto-mode` 只有一个入口：读请求，匹配一份 playbook，再把那份 playbook 的步骤逐字复制进 todo 的最前面，然后按步骤调用 skill 或派 subagent。选择与执行在同一个上下文里完成。

![请求如何经过 mode 进入 playbook](../figures/mode-routing.svg)

## 匹配与复制

- SKILL.md 的 Playbooks 一节列 23 个标题与各自适用情形。匹配后打开 `playbooks/<title>.md`，标题转 kebab-case（保留连字符，去掉其他标点）。条目自带文件的用它的文件：`playbooks/authoring-a-skill.md`、`playbooks/multi-phase-plan.md`、`playbooks/worktree-cleanup.md`。
- 步骤逐字复制，不概括。不做的步骤留在 todo 里，写一行 `skip: <reason>`。
- 没有自带的 playbook 覆盖，或工作大到横跨多个 call site 时，走 `figure-it-out`。工作是持续数天、几十上百个 subagent 的项目时，走 `orchestrate`。一个任务跑到某个 predicate 停，走 `autonomous-run`。
- SKILL.md 的 Non-negotiables 是第二处选择点：设计没把握进 `architect`，并行 fan-out 进 `swarm`，设计或代码 bakeoff 进 `arena`，有争议的设计进 `interrogate`，任何文字进 `unslop`。

源码：[`skills/poteto-mode/SKILL.md#playbooks`](../../skills/poteto-mode/SKILL.md#playbooks)、[`playbooks/`](../../skills/poteto-mode/playbooks/)。

匹配后的 todo 分两段：最前面是逐字复制的 playbook 步骤，之后才是任务特定的 todo。步骤里点名的 skill 自己定义动作；playbook 定义顺序、验证要求与结尾 reply 的形状。

逐字复制是为了可执行性，不是为了排版。bug fix 的 "Reproduce it yourself on the matching surface" 一旦概括成"先复现"，责任主体和"自己"都没了。

每份 playbook 的步骤与结尾的 reply 由那份文件定义；回复本身另受 Writing the reply 约束，见 [unslop 与 technical-writing](./prose-discipline.md)。

## 为什么这样组织

- 状态外化。todo 是最便宜的状态机，用户扫一眼就知道在第几步、哪一步被跳过。
- 协议可版本化。改流程就是改一个 markdown 文件，不改提示词工程，也不改代码。
- 匹配信号写在描述里。没有一个单独的 skill 负责匹配；每个标题和适用情形就是索引。

## 代价

- 23 个标题不是 23 个显而易见的区别。`babysit` 与 `shipping`、`autonomous-run` 与 `orchestrate` 的边界要读描述才知道。
- 选错会放大。长流程里选错 playbook，代价远大于一次普通回答选错。
- 依赖安装完整度。mode 需要大多数 skill 在场，少一个就可能走到断链的步骤。
- 没有机械校验。"逐字复制"靠执行者遵守，没有脚本比对 todo 与 playbook。

## 关键洞察

1. 选择和执行不换手，少一次交接损耗。
2. 步骤是状态，不是文档。搬进 todo 才真正约束执行。
3. 标题和适用情形就是索引，所以描述的准确性直接决定匹配质量。
