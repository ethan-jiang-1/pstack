# pstack 术语表

只收 pstack 自身的术语。每个词给一句定义和源码位置；没有 pstack 对应物的话题不在这里发明名字。

## 入口与流程

**mode（`poteto-mode`）**：
主入口。读请求，匹配一个 playbook，把该 playbook 的步骤逐字复制进 todo，再按步骤调用其他 skill 或派 subagent。源码：`skills/poteto-mode/SKILL.md`。

**playbook**：
`poteto-mode` 下的步骤协议，共 23 个。匹配后整份复制，不概括。源码：`skills/poteto-mode/playbooks/`。

**skill**：
一个目录，含 `SKILL.md`，可带 `references/`、`playbooks/`、`scripts/`。共 54 个。源码：`skills/`。

**principle**：
24 个 `principle-*` skill，每个是一条可点名的判断规则。应用时读完整 leaf skill，并在回复里说明它改变了哪个决定。源码：`skills/poteto-mode/SKILL.md#Principles`。

**prompt 的五项**：
goal、done check、proof、what you already know、real constraints。pstack 不写 spec，意图压在这五项里。源码：`docs/guide/02-poteto-mode.md#What goes in a prompt`。

**todo 与 `skip`**：
todo 的最前面是匹配到的 playbook 步骤，逐字复制；不做的步骤留在列表里，写 `skip: <reason>`。源码：`skills/poteto-mode/SKILL.md#Playbooks`。

**brief**：
派给 subagent 的最小输入，字段是 GOAL / SCOPE / CONTEXT / ACCEPTANCE / VERIFY / TIMEBOX / FORBIDDEN / REPORT / STANDING；STANDING 把 `preferences.md` 带进去；cloud spawn 与 resume 逐字粘贴，local spawn 可按 store path 引用。源码：`skills/poteto-mode/playbooks/orchestrate.md#The brief`。

**frontier**：
PR 栈的合并前沿：最低的未合并 PR 是唯一要处理的，直到它合并。只在这个意义上用 frontier。源码：`skills/poteto-mode/playbooks/babysit.md`、`orchestrate.md#Stack safety`。

**ready work**：
`orchestrate` 每轮排空后重新计算的可开始工作。不要把它叫 frontier。源码：`skills/poteto-mode/playbooks/orchestrate.md#Steps`。

## 委派与模型

**subagent**：
由主 agent 派出的执行单元。playbook 步骤里的委派用 `poteto-agent`；`how`、`why`、`interrogate`、`reflect`、`swarm` 自带 `subagent_type`，不被覆盖。源码：`skills/poteto-mode/SKILL.md#Subagents`。

**fresh subagent**：
默认给新任务开新 subagent，带 consolidated scope（原始 brief、后续指令、上一个 agent 的报告和分支）。只有新工作依赖旧 agent 的本地 checkout、未提交改动或仍在跑的进程时才 resume。源码：`skills/poteto-mode/SKILL.md#Subagents`。

**role line**：
设置文件里“角色 → 模型”的一行，例如 `judgment and prose`、`swarm workers`。role 缺失时退回 skill 内默认值。源码：`skills/setup-pstack/SKILL.md`。

**model panel**：
`arena runners`、`architect runners`、`interrogate reviewers` 等角色行；列表长度决定 fan-out。源码：`skills/setup-pstack/SKILL.md`。

## 验证、评审与落地

**verdict**：
没写这份代码的 verifier 给出的 `PASS`、`PASS+NOTES` 或 `FAIL`。作者自证、绿 CI、bot approval 都不是 verdict。源码：`skills/poteto-mode/playbooks/shipping.md` step 1。

**patch-id**：
`git patch-id` 算出的补丁内容标识。rebase 会改 SHA，但补丁可能不变；旧 verdict 是否仍覆盖当前 head，看 patch-id。源码：`skills/poteto-mode/playbooks/shipping.md` step 3。

**contiguous verified run**：
从最低未合并 PR 往上，连续有 `PASS` 或 `PASS+NOTES` 的一段栈。第一个没有 `PASS` 或 `PASS+NOTES` 的 PR 就是停止点。源码：`shipping.md` step 2。

**feature map**：
项目本地的 `verify-<app>/features/`：一个 README 索引加每个用户可见 feature 一个文件。由 `create-verification-skill` 生成，`maintain-verification-skill` 维护。源码：`skills/create-verification-skill/SKILL.md`。

**decision trail / decision log**：
`show-me-your-work` 的决策记录：默认 `decisions.tsv`，一行一个决定（ts、phase、decision、why、evidence、result），只追加。概念层叫 decision trail，文件叫 decision log。源码：`skills/show-me-your-work/SKILL.md`。

**control surface / control skill**：
驱动真实应用的 control surface，例如 `control-ui`、`control-cli`，或项目本地的 `verify-*` skill。不要用它指父 agent 的委派控制。源码：`skills/poteto-mode/SKILL.md#Harness`、`skills/create-verification-skill/SKILL.md`。

**Always pause**：
不可逆写入：force-push 到共享分支、部署、删除数据、给客户发消息。源码：`skills/poteto-mode/SKILL.md#Autonomy`。

## 环境

**`correct`**：
把 operator 反复纠正的同一类错误变成仓库里最强的可行约束：架构 → 类型 / lint → 测试 → 文档；每条新检查要证明它在真实历史错误上会失败。源码：`skills/correct/SKILL.md`。

**`reflect`**：
三个 reviewer（judgment、tooling、divergent）并行读 transcript，synthesizer 归一成 Accepted / Rejected / Backlog；Accepted 等明确批准后才改 skill，Backlog 直接进 tracker。源码：`skills/reflect/SKILL.md`。

**`encode-lessons-in-structure`**：
第二次写同一条指令时，把它编码成 lint、元数据、运行时检查或脚本；判断类教训无法编码时，才写一段带失败例子的文字。源码：`skills/principle-encode-lessons-in-structure/SKILL.md`。

**`create-verification-skill` / `maintain-verification-skill`**：
生成和维护项目本地的 `verify-<app>` skill 与 feature map，让“像用户一样驱动应用”成为仓库的一部分。源码：`skills/create-verification-skill/SKILL.md`、`skills/maintain-verification-skill/SKILL.md`。

## 落地与接管

**`shipping`**：
独立验证每个 PR，只落地从最低未合并 PR 开始连续通过的那一段，自底向上一次一个。pstack 到落地为止。源码：`skills/poteto-mode/playbooks/shipping.md`。

**`babysit`**：
把 PR 或栈推到 merge-ready：声明 `drive` / `background` / `threads-only` / `check`，只处理最低未合并的 frontier，不负责合并。源码：`skills/poteto-mode/playbooks/babysit.md`。

**`session pickup`**：
接管别人未完成的工作：读原轨迹，重建操作状态，路由到对应 playbook。`recall` 不接这个场景。源码：`skills/recall/SKILL.md`、`skills/poteto-mode/playbooks/session-pickup.md`。

**`pause safely`**：
在原子步骤边界干净暂停：取消 nested subagent，不启动新工作，不推送（除非已经推过），未提交改动做成 `wip:` 提交，在上下文之外写恢复说明。源码：`skills/poteto-mode/playbooks/pause-safely.md`。

**`opening a PR`**：
需要开 PR 的 playbook 的共享收尾：worktree、提交节奏、PR 标题与六节正文、forge、栈规则。源码：`skills/poteto-mode/playbooks/opening-a-pr.md`。

**`worktree cleanup`**：
回收磁盘：worktree 与陈旧的 iOS 模拟器；clean、merged、not-in-use 可直接删，`wip` 和使用中先停。源码：`skills/poteto-mode/playbooks/worktree-cleanup.md`。

## pstack 不覆盖

pstack 不写 spec，不做发布 / 灰度 / 回滚 / 发布后监控。它停在 verified landing；部署只是 **Always pause** 里的一个不可逆动作。这些话题如果出现在外部方法里，不要把它们写成 pstack 的机制。
