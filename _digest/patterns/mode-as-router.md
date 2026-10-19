# Mode as Router 模式

## 是什么

pstack 不把 23 个工作流程做成 23 个可以随便进的命令，而是让一个入口 `poteto-mode` 先读请求、选一个 playbook，再把那份 playbook 的步骤逐字搬进 todo。入口负责选路，playbook 负责协议，两者合起来才是一次执行。

![请求如何经过 mode 进入 playbook](../figures/mode-routing.svg)

## 源码里的形状

1. `poteto-mode` 的 **Playbooks** 一节列出 23 个标题与各自适用情形。
2. 匹配后打开对应的 playbook 文件，把步骤复制进 todo，"跳过的步骤留 `skip: <reason>`"。路径默认按标题推导；条目自带文件的，用它给的文件（`authoring-a-skill`、`multi-phase-plan`、`worktree-cleanup`）。
3. 步骤里的 skill 调用、验证要求、回复形状都由那份 playbook 定义。
4. 没有合适 playbook 时，走 `figure-it-out`；是持续 program 时走 `Orchestrate`。

关键细节是**逐字复制**。它不是排版偏好，而是防止模型在转述时丢掉条件。例如 bug fix 的 "Reproduce it yourself on the matching surface" 一旦被概括，责任主体和"自己"就没了。

## 为什么有效

- **选择与执行在同一上下文里。** 选完流程立刻执行，不需要把意图交接给另一个 agent。
- **状态外化。** todo list 是最便宜的状态机，用户扫一眼就知道在第几步、哪一步被跳过。
- **协议可版本化。** 改流程就是改一个 markdown 文件，不需要改提示词工程或代码。
- **兜底有层次。** 窄 playbook、一次性设计（`figure-it-out`）、持续 program（`Orchestrate`）各管一个规模。

## 代价

- **发现成本。** 23 个标题不是 23 个显而易见的区别；`babysit` 与 `shipping`、`autonomous-run` 与 `orchestrate` 的边界要读描述才知道。
- **选错会放大。** 长流程里选错 playbook，代价远大于一次普通回答选错。
- **依赖安装完整度。** mode 需要大多数 skill 在场，少一个就可能走到断链的步骤。
- **没有机械校验。** "逐字复制"靠执行者遵守，没有脚本比对 todo 与 playbook。

## 怎么迁移

1. 先画出工作从请求到可交付结果的几种形状，不要先起 23 个命令名。
2. 给每种形状写一份**有步骤、有门、有失败出口**的协议文件；步骤要短到能整段复制。
3. 选一个主入口负责匹配，并把匹配信号写清楚（任务类型、已有 artifact、用户说出的验收方式）。
4. 设计三层兜底：最窄的协议、一次性定制、持续 program。
5. 允许跳过，但要求留下原因。

## 关键洞察

1. **路由器同时是执行器，减少了交接损耗。**
2. **步骤是状态，不是文档。** 把它搬进 todo，才真正约束执行。
3. **路由质量取决于描述质量。** 没有单独的 router skill 时，每个 playbook 的标题和描述就是索引。
