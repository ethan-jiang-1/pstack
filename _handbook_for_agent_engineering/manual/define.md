# 定义：把一个请求变成可执行的 prompt

> 本章 4 个实战单元 · [手册目录](./README.md) · [教程地图](../tutorial/README.md)

## 这一章解决什么

大多数返工不是从代码开始的，而是从一句没有被执行过的请求开始的：目标含糊，完成判据缺失，事实和猜测混在一起。等代码写出来，纠正的单位已经从一个词变成一个 diff。

pstack 没有单独的"定义"阶段，也没有一份要先写好的文档。它把请求压进 prompt 的五项，让 `poteto-mode` 匹配一个 playbook 并把步骤逐字复制进 todo，在向人提问之前先给岔口分类，交给别的 agent 时用 `brief` 的字段把意图固定住。pstack 不写 spec；离固定意图最近的产物，是 prompt 加 `multi-phase-plan` 的 plan 文件。（`docs/guide/02-poteto-mode.md`、`skills/poteto-mode/SKILL.md#Playbooks`）

四个单元对应四种卡点：

| 单元 | 卡点 | 带走 |
|---|---|---|
| 1.1 | 请求在脑子里，说不清什么算完成 | prompt 的五项 |
| 1.2 | 不知道这是 Investigation 还是 Feature，步骤散在聊天里 | 匹配到的 playbook 和逐字复制的 todo |
| 1.3 | 一到岔口就停下来问人 | `AskQuestion` 之前的分类 |
| 1.4 | 要交给另一个 agent，意图装不下 | `brief` 的字段；跨 PR 时的 plan 文件 |

四个单元不是流水线。一个 Bug fix 的 prompt 可能只写三项；一个跨多个 PR 的工作会在 1.3 停很多次；1.2 则在每次 `new task` 之后重来一次。

---

## 1.1 把请求放进 prompt 的五项

### 工作情境

请求可能来自 Slack、工单、会议或一句口头交代：

> "给导出功能加个筛选，只导出我选中的行。"

团队点头，各自开始理解：有人以为是在导出页加筛选器，有人以为是导出后过滤文件，有人以为要记住上次选择。每个人都对，但每个人想的不是同一件事。

pstack 不要求先开会澄清。它要求把这句话送进 `/poteto-mode`，并带上 prompt 能带的五项。（`docs/guide/02-poteto-mode.md#What goes in a prompt`）

### 带走什么

一个能被 `poteto-mode` 直接消费的 prompt，最多包含五项，每项一句话：

```text
goal：<哪里不对，或者你要什么>
done check：<能判过或判不过的完成判据>
proof：<要看到的真实输出：命令结果、流程视频、存储值、前后数字>
what you already know：<症状、复现步骤、日志行、链接>
real constraints：<repro first、先别改代码、零行为变化、我先看再继续>
```

五项不是模板填空。缺哪项，就在那句话里补哪项；补不出来，说明请求还没到能开工的程度。

### 做法

**第一步，写 goal。** 说哪里不对，或者要什么，不写 ceremony。（"Say the goal, not the ceremony"，`docs/guide/02-poteto-mode.md`）

**第二步，写 done check。** 它必须能判过或判不过。"做得更好一点"和"先干一个小时"都不是 done check。

**第三步，写 proof。** 要你真正会看的证据：真实命令的输出、一段流程视频、一个存储值、一组前后对比的数字。不要写"证明它能用"。

**第四步，写 what you already know。** 一个症状、一条复现步骤、一行日志、一个链接，都能省掉 agent 的一次搜索。已经知道的越具体，agent 越不需要猜。

**第五步，写 real constraints。** "repro first"、"先别改任何代码"、"零行为变化"、"我先看再继续"，每一条都会改变 agent 的动作。约束不是客气话，playbook 会照做。

**第六步，删掉 the how 和你的因果猜测。** 说清要达到什么，给 agent 留出比你选的更好的路径。把猜测中的原因留到 agent 复述问题之后再给；先说猜测会把搜索范围缩到你指的那一处。（`docs/guide/02-poteto-mode.md#What goes in a prompt`）

**第七步，报告很吵时，把复述作为第一步。** 长线程或模糊 bug 用：

```text
/poteto-mode read this thread. restate the underlying issue in your own words, in plain english. don't change any code yet.
```

误读会在复述里显形，此时纠正只花一条消息，而不是一次错误的修复。

**第八步，后续短说。** 对话已经带着上下文时，`/poteto-mode do it`、`continue`、`keep going until done` 就够。结构由 playbook 持有，不靠每轮把五项重写一遍。换话题时说 `new task`，让它重新匹配，而不是接着上一个 playbook 往下走。（`docs/guide/02-poteto-mode.md`）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 复述请求，指出五项里缺哪项 | 说清 goal 和 real constraints |
| 查仓库、日志、数据，补齐 what you already know | 确认 done check 和要看的 proof |
| 起草 done check 与 proof 的候选 | 做产品与偏好决定（见 1.3） |

agent 不应该用默认值填 goal 或 done check。它可以把缺项写成一句话问题；在明确的全自主授权下，它也可以先推进并报告。两种做法都按 1.3 的分类处理。

### 质量门

- 五项各能用一句话说清。
- done check 能判过或判不过。
- proof 是一个真实命令、视频、存储值或数字，不是"测试通过"。
- real constraints 每一条都会改变 agent 的动作。
- prompt 里没有 the how，也没有你先说出的因果猜测。
- 长线程的 prompt 把复述放在第一步。

### 进阶

prompt 最多五项，不是每次都要写满。小改动可以只留 goal、done check 和 proof，另外两项在对话里已经成立。

需要写满五项的情况：你说不清"不做什么"；你说不清怎样算完成；你要把证据留给自己以后看；或者这项工作会被交给另一个 agent。最后一种情况，五项会变成 `brief` 的一部分，见 1.4。

---

## 1.2 让 poteto-mode 匹配一个 playbook，把步骤复制进 todo

### 工作情境

prompt 发出去了，接下来最常见的问题不是"agent 不干活"，而是它按自己以为的类型干活：

- 一个只读问题被当成 Feature，开始改代码；
- 一个已经量过的慢查询被当成 Feature，没有先量 baseline；
- 一个跨多个 PR 的重构被当成一次 Refactoring，做到一半才发现要分栈；
- 长对话里换了话题，agent 还按上一个 playbook 的下一步走。

这些都不是模型能力问题，是入口没有把任务匹配到正确的 playbook。

### 带走什么

一个 todo：最前面是匹配到的 playbook 的步骤，逐字复制；后面才是这个任务自己的 todo。不做的步骤留在列表里，写一行 `skip: <reason>`。（`skills/poteto-mode/SKILL.md#Playbooks`）

比如任务是"用户重试后收到两条通知"，匹配到 Bug fix，todo 的前几项就是 Bug fix 的步骤，而不是 agent 现编的"先看通知代码"。

### 做法

**第一步，把请求交给 `poteto-mode`，不手工写步骤。** 它是入口：读请求，读 Principles 一节，匹配 23 个 playbook 中的一个，把该 playbook 的步骤逐字复制进 todo，再在步骤需要时调用其他 skill。（`skills/poteto-mode/SKILL.md`、`docs/guide/02-poteto-mode.md`）

**第二步，确认匹配对。** 常见路由是：

| 请求 | playbook |
|---|---|
| 只读问题：X 怎么工作、Y 为什么这样、要不要做 X | Investigation |
| 一个可复现的缺陷 | Bug fix |
| 新的或改变的行为 | Feature |
| 只改结构、行为不变 | Refactoring |
| 已测量到的慢 | Perf issue |
| 大工作或没有匹配 | figure-it-out |

其余 playbook 覆盖 metric 爬坡、运行时症状、trace、原型、视觉一致性、skill 编写与评估、长时间自主运行、把 PR 或栈推到 merge-ready、落地已验证的栈、PR 队列、项目级编排、session pickup、暂停、多 phase 或多 PR 计划、worktree 清理，以及每个 playbook 结尾的 opening a PR。完整集合在 `skills/poteto-mode/playbooks/`。（`docs/guide/02-poteto-mode.md`）

**第三步，不要手工列 skill 顺序。** 在 prompt 里写"先用 how，再用 architect，然后 arena"通常会把 playbook 本来会保留的步骤重排或丢掉。只有当你要覆盖某个具体选择时，才点名一个 skill。（`docs/guide/02-poteto-mode.md`）

**第四步，不做的步骤保留，写 `skip: <reason>`。** 删除步骤会让 todo 看起来干净，也让后来的人看不出你是有意跳过还是漏了。保留步骤加一行理由，两件事都留住了。

**第五步，换话题时说 `new task`。** 长对话会累积上一个任务的上下文；`new task` 让 `/poteto-mode` 重新匹配，而不是继续原来的 playbook。（`docs/guide/02-poteto-mode.md`）

**第六步，按规模选对入口。** 一个 agent 在当前会话预算内能完成的一项任务，走对应的 playbook；一次需要专门流程设计的复杂运行，走 `figure-it-out`；跨 phase 或跨 PR 的计划，走 `multi-phase-plan`；多天、多栈、几十到几百个 subagent 的常驻项目，走 `orchestrate`。（`skills/poteto-mode/SKILL.md#Playbooks`）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 匹配 playbook，复制步骤，逐步执行 | 在匹配错时用一句话纠正 |
| 把不做的步骤标成 `skip:` | 确认这确实不是本次任务的一部分 |
| 在 `new task` 后重新匹配 | 说清换的是什么任务 |
| 在 `figure-it-out`、`multi-phase-plan`、`orchestrate` 之间路由 | 决定工作要跨多久、要不要拆分 |

匹配错了，纠正的成本是一条消息；不纠正，成本是整轮按错误的步骤走完。

### 质量门

- todo 最前面是匹配到的 playbook 的步骤，逐字复制，不是摘要。
- 每个跳过的步骤都在列表里，带 `skip: <reason>`。
- 没有手写的 skill 顺序。
- 换任务后有 `new task`，playbook 重新匹配过。
- 多 PR 或多天的工作没有塞进单次 playbook。

### 进阶

`figure-it-out` 是兜底：没有 bundled playbook 适配时，它给这个任务设计一套专门的 playbook。`orchestrate` 不是"更大的 Feature"，它是把整个项目交给一个 coordinator chat；一个 agent 能在会话预算内做完的工作不该路由到那里。（`skills/poteto-mode/SKILL.md#Playbooks`）

---

## 1.3 在 AskQuestion 之前先分类

### 工作情境

agent 停在一个岔口：

- "筛选器放前端还是后端？"
- "缺失行是报错还是跳过？"
- "这个 API 该返回数组还是对象？"

它准备用 `AskQuestion` 把问题抛给人。人被打断，凭偏好选一个；或者 agent 不等回答，自己用默认值填了。

pstack 在 `poteto-mode` 的 Non-negotiables 里给这条岔口加了分类：能靠运行观察到的，不是人的问题；只有真正的产品与偏好决定，才轮到 `AskQuestion`。（`skills/poteto-mode/SKILL.md#Non-negotiables`）

### 带走什么

一个在提问之前写下的分类：

| 岔口的答案 | 处理 |
|---|---|
| 能靠运行观察：行为、时机、布局、输出、性能，甚至一个 eval 能不能区分 | 用 Prototype playbook 做出来，让结果决定 |
| 只读 Investigation，交付物是有引用的回答 | 留在 Investigation 里，用证据回答，不搭草图 |
| 真正的产品与偏好决定，没有实验能定 | `AskQuestion` |
| 全自主授权覆盖的决定 | 直接决定、执行、报告；不要求回复一个词，也不提供备选 |
| 授权下只有 operator 能做的决定 | 取默认值推进，完整报告，并用平实语言说明 operator 可以让你改成什么 |
| `Always pause` 列表里的不可逆动作，或 operator 点名的 gate | 停下等人 |

### 做法

**第一步，把岔口写成一句问题。** "前端过滤还是后端过滤"是选项；问题是"在 5 万行、跨页勾选、权限过滤三个条件下，哪种实现能在两周内交付且不引入数据错误"。

**第二步，问一句：运行一次能不能回答？** 行为、时机、布局、输出、性能，甚至一个 eval 能不能区分两个方案，都是可以观察的事实。能观察，就不是人的问题。（`skills/poteto-mode/SKILL.md#Non-negotiables`）

**第三步，能观察就做 Prototype。** 用 `playbooks/prototype.md` 搭出最小草图，让结果决定。原型是丢弃式工具，输出是决定加一次性产物，不是可交付代码。（`skills/poteto-mode/playbooks/prototype.md`）

**第四步，只读 Investigation 不要搭草图。** 如果任务本来就是 Investigation，交付物是一条有引用的回答，就留在 Investigation 里，从证据回答。（`skills/poteto-mode/SKILL.md#Non-negotiables`）

**第五步，剩下的才问。** 只有真正的产品与偏好决定、没有实验能定时，才用 `AskQuestion`。问的时候给选项和你的推荐；问题本身要能被人用一句话回答。

**第六步，在全自主授权下按两档处理。** 授权覆盖的决定，直接决定、执行、报告；只有 operator 能做的决定，取一个默认值推进，完整报告，并说明 operator 可以让你改成什么。operator 用自己的话回答；不要给一个要人打回来的缩写 token。（`skills/poteto-mode/SKILL.md#Non-negotiables`）

**第七步，`Always pause` 不因授权消失。** force-push 到共享分支、部署、删除数据、给客户发消息，仍然停下等人。（`skills/poteto-mode/SKILL.md#Autonomy`）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 把岔口分类，判断能否用运行回答 | 做产品与偏好决定 |
| 用 Prototype 做草图并让结果决定 | 定风险容忍度与优先级 |
| 在授权覆盖时直接决定并报告 | 在授权下被告知默认值与可改回的方式 |
| 在 `Always pause` 前停下 | 批准不可逆动作 |

关键不是"少问人"，而是"只问人真正能回答的"。一个能跑一次就知道答案的问题，问人只会把决定变成偏好。

### 质量门

- 岔口能用一句问题说清。
- 分类写明运行什么就能回答它；能运行回答的没有变成 `AskQuestion`。
- 只读 Investigation 没有为了回答而搭草图。
- 授权下的决定有报告：选了什么、为什么、operator 可以怎么改。
- 不可逆动作停在 `Always pause`。
- 没有给人一个缩写 token 让他打回来。

### 进阶

`multi-phase-plan` step 2 把这条规则用在写计划之前：每个开放问题先跑 Prototype，保留分支、SHA 和截图，写进 plan 的 Appendix A；只有产品与偏好决定才问 operator，并且给选项。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

这也是本章和下一章之间最常用的一次交接：1.3 决定一件事能不能靠做出来回答；第 2 章的 `architect` 与 `arena` 把设计形状做出来。

---

## 1.4 交给别人时写 brief；pstack 不写 spec

> **边界。** pstack 不写 spec。`docs/guide/02-poteto-mode.md` 的原话是 "You don't write a spec"，pstack 根目录的 `README.md` 写 "the best spec is code"。意图的近处是 prompt 的五项；跨多个 PR 时是 `multi-phase-plan` 的 plan 文件；交给 subagent 时是 `brief`。

### 工作情境

意图已经清楚，你要把它交给另一个 subagent。它没有你的聊天记录，看不到你刚才读的文件，也不能在卡住时回头问你一句。它只能读你给它的那段文字。

旧习惯是写一份 spec，把目标、行为、边界、验收都固定下来。pstack 不这么做，它把同样的意图放进三个更靠近执行的位置：prompt、plan 文件和 `brief`。（`docs/guide/02-poteto-mode.md`、`skills/poteto-mode/playbooks/orchestrate.md#The brief`）

### 带走什么

一个填满字段的 `brief`：

```text
GOAL        一句话，结果，一个没有聊天记录的人也能执行
SCOPE       这个 unit 可以写的路径；不可以写的路径；独占的 worktree 或分支
CONTEXT     文件和 PR 的指针；依赖的上游报告逐字粘贴，因为 worker 看不到兄弟
ACCEPTANCE  可检查的判据，一行一条
VERIFY      确切的命令或 control skill 路径，加上已知的坑
TIMEBOX     运行时间上限；到点返回部分发现并停下，不要跑过头
FORBIDDEN   不 retarget 或 merge PR、不 rebase、不 force-push、不修 scope 外的代码，加 unit 专属的禁令
REPORT      status、branch、head SHA、PR、verdict、实际跑了什么、偏离、建议的后续
```

`orchestrate` 的每次 spawn 和 resume 还会逐字粘贴 `preferences.md`。字段填不满，说明这个 unit 还没有被 scope 好；缺字段是拒绝 spawn 的条件。（`skills/poteto-mode/playbooks/orchestrate.md#The brief`）

跨多个 PR 时，离意图最近的固定产物是 `multi-phase-plan` 的 plan 文件：每个 PR 一节，一个 PR 是一个变更，带自己的证据。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

### 做法

**第一步，先判断要不要计划。** 一两个文件、路径明显时，`multi-phase-plan` step 1 说跳过 plan，直接说"这次不需要 plan"然后停。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

**第二步，跨 phase 或跨 PR 时写 plan 文件。** 把 `multi-phase-plan` 的 skeleton 复制进 plan 文件，逐个填占位符；每个 PR 一节；一个 PR 是一个变更，带自己的证据（`sequence-verifiable-units`）。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

**第三步，写 GOAL。** 一句话，结果，能被一个没有聊天记录的人执行。不要写"帮我把导出弄一下"。

**第四步，写 SCOPE。** 可以写哪些路径，不可以写哪些路径，独占哪个 worktree 或分支。没有 scope，并行工作会互相覆盖。

**第五步，写 CONTEXT。** 文件和 PR 用指针；这个 unit 依赖的上游报告逐字粘贴，因为 worker 看不到兄弟 agent。依赖不只是顺序，它还是上下文转发；没有声明的上游上下文，worker 只能猜。

**第六步，写 ACCEPTANCE 和 VERIFY。** ACCEPTANCE 一行一条，能检查；VERIFY 给确切命令或 control skill 路径，加上已知的坑。`orchestrate` 的验证按 unit 规模缩放：一条便宜命令就让 worker 跑完报输出，昂贵、要判断或高 blast-radius 的才上独立的 verifier。（`skills/poteto-mode/playbooks/orchestrate.md`）

**第七步，写 TIMEBOX、FORBIDDEN、REPORT。** 到时间上限就返回部分发现并停；FORBIDDEN 至少包含不 retarget、不 merge、不 rebase、不 force-push、不修 scope 外。REPORT 固定成 status、branch、head SHA、PR、verdict、跑了什么、偏离、后续。

**第八步，按 unit 大小收放。** 一条命令的 unit 把模板压成一段，仍要点名 goal、scope、verify 命令和 report 形状。给两行编辑套 4KB 的脚手架，写和遵守的成本都高于编辑本身。（`skills/poteto-mode/playbooks/orchestrate.md#The brief`）

**第九步，不要 resume-chain 一个 brief。** 新工作默认开新 subagent，带 consolidated scope：原始 brief、后续指令、上一个 agent 的报告和分支。只有新工作严格依赖旧 agent 的本地 checkout、未提交改动或还在跑的进程时才 resume。（`skills/poteto-mode/SKILL.md#Subagents`）

### 人机边界

| agent 可以做 | 人必须做 |
|---|---|
| 起草 brief，指出缺哪个字段 | 定 GOAL 与 FORBIDDEN |
| 查代码与上游报告，填 CONTEXT | 给 ACCEPTANCE 里属于产品判断的判据 |
| 写 plan 文件并跑检查脚本 | 给 operator 的 explicit go |
| 派 worker 与 verifier | 接受 plan 的范围与顺序 |

`multi-phase-plan` 把交付点写死：把 plan 路径和检查脚本输出贴回来，然后停；执行等 operator 明确的 go，按 plan 里点名的执行 playbook 跑。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

### 质量门

- brief 的每个字段都填满；填不满的字段被当成"这个 unit 还没 scope 好"。
- GOAL 能被一个没有聊天记录的人执行。
- SCOPE 同时写了可写与不可写。
- ACCEPTANCE 一行一条、可检查；VERIFY 是确切命令或 control skill 路径。
- plan 文件里一个 PR 一节，一个 PR 一个变更，带自己的证据。
- 计划执行前有 operator 的 explicit go。
- 没有另写一份 spec。

### 进阶

plan 文件是让 owner 一行一行跑、让 operator 从证据审的 checklist；plan 本身就是交付物，写 plan 的阶段不实现。（`skills/poteto-mode/playbooks/multi-phase-plan.md`）

这也是为什么 pstack 不需要 spec：能固定意图的东西已经分散在 prompt、plan 文件和 `brief` 里，而代码本身是最终的那份。

---

## 四个单元怎样接起来

```text
请求
  → prompt 的五项（1.1）
  → poteto-mode 匹配 playbook，步骤逐字进 todo（1.2）
  → 岔口先分类：Prototype / Investigation / AskQuestion / 全自主（1.3）
  → 交给 subagent 时写 brief；跨 PR 时写 multi-phase-plan 的 plan 文件（1.4）
  → 第 2 章：data shape 与 architect 的 Phase B 草图
```

小改动可能只走 1.1 和 1.2。高风险岔口会在 1.3 停很久。1.4 只在意图要离开这个会话时出现。

## 常见失败

| 失败 | 表现 | 修正 |
|---|---|---|
| 请求只有一句话 | prompt 缺 done check 和 proof | 回到 1.1，补五项 |
| playbook 匹配错 | 只读问题开始改代码 | 用 `new task` 重新匹配，或换一个 playbook |
| 手写 skill 顺序 | playbook 的步骤被重排或丢掉 | 回到 1.2，只写 goal 与约束 |
| 跳过步骤不留痕 | 后来的人分不清是有意还是漏了 | 保留步骤，写 `skip: <reason>` |
| 能跑一次的问题去问人 | 人凭偏好选了一个 | 回到 1.3，用 Prototype 让结果决定 |
| brief 缺字段 | worker 用猜的 | 补齐字段；缺字段就拒绝 spawn |
| 想先写一份 spec | 意图被复制到另一份会过期的文档 | 用 prompt 的五项、plan 文件和 brief |

## 继续阅读

- [设计与计划：让工作可兑现、可委派](./design-and-plan.md)
- [教程 01：从一句模糊的话开始](../tutorial/01-define.md)
- [教程 02：给共同语言一个位置](../tutorial/02-give-words-a-home.md)
- [术语表：prompt 的五项、playbook、brief、plan 文件](../CONTEXT.md)
