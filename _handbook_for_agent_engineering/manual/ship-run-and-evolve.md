# 评审与落地

> 本章 4 个单元 · [手册目录](./README.md) · [上一章：验证与 verdict](./verify.md) · [下一章：环境](./evolve-the-environment.md)

## 这一章解决什么

verdict 证明"能用"，评审判断"值不值得合"，`shipping` 决定"哪一段可以落地"，`babysit` 把 PR 推到 merge-ready。pstack 到落地为止；部署、灰度、回滚、发布后监控不在 pstack 里，部署只是 **Always pause** 的一个不可逆动作。

| 单元 | 卡点 | 带走 |
|---|---|---|
| 5.1 | 评审要么挑刺，要么放行 | `interrogate` 的四桶结论 |
| 5.2 | 注释噪音和 workaround 混进 diff | `no-comments` 的删除与裁决 |
| 5.3 | 绿 CI 被当成可以合 | 非作者 verdict 与 contiguous verified run |
| 5.4 | PR 卡在 merge-ready 之前 | `babysit` / `session pickup` / `pause safely` |

---

## 5.1 `interrogate`

### 工作情境

一个 diff，测试绿了，执行者说"按 brief 做完了"。你读一遍，挑了几个命名问题；另一个模型可能看到你漏掉的权限路径。

`interrogate` 用模型多样性换独立性：同一段 diff、同一段 intent、同一套 rubric，发给多个 reviewer；再由 lead reviewer 按工程判断收成一个结论。源码：`skills/interrogate/SKILL.md`。

### 做法

1. 固定 scope：用户点名的文件或 diff；feature branch 上用 `git diff main...HEAD`；只提到"刚才那段工作"就收齐相关文件。
2. 冻结 intent：用一段话写清目标，来源是用户消息、commit message、PR 描述和代码本身。reviewer 的模板要求"不要质疑 intent，假设目标正确，挑战执行"。
3. 按 `interrogate reviewers` 的条目派 reviewer，一条消息里全部派出；每个 reviewer 是 `generalPurpose`、`readonly: true`，模型取配置值或默认表。
4. reviewer 拿 `references/rubric.md` 和 `references/code-quality-review.md`，输出带 severity、位置、证据的结构化发现。
5. 综合：合并描述同一问题的发现；两个以上模型独立提出的是最高信号；单个模型的发现降权；矛盾记录。
6. lead judgment：按 `references/lead-judgment.md` 过滤 nitpick、hypothetical、premature abstraction，把发现分进 Act On / Consider / Noted / Dismissed。
7. 输出固定七节：Intent、Reviewers、Act On、Consider、Noted、Dismissed、Agreement Map。

### 质量门

- 每个发现带提出它的模型和一句理由。
- `Dismissed` 写清否决原因，不是凑数。
- 不自动改代码；评审产出判断，改动是另一个决定。

---

## 5.2 `no-comments`

### 工作情境

diff 里混着叙述阶段的注释、注释掉的旧代码、解释 workaround 的长段落。它们不影响测试，但会误导下一位 agent。

`no-comments` 先派 Comment Sicko 只读评审，再由调用方裁决，最后由本 skill 做范围内最小的根因修复。源码：`skills/no-comments/SKILL.md`。

### 做法

1. 派 Comment Sicko，传 scope；它只报四项：碰过的文件、删除数、逐条一行的 `MUST KILL`、跳过项。
2. 调用方审报告与 diff：拒绝应用代码编辑、scope 逃逸、被例外保护的删除、错误的 `MUST KILL` 理由。
3. trivial 的 flag 直接做：删死路径、去掉参数、改用真实 API。
4. 需要形状时只跑一次 `architect`，停在 sketch；然后做范围内最小的根因修复，删掉每个被点名的 workaround。
5. 约束型注释（`do not remove`、`talk to X before changing`）：提出范围内最便宜的类型、运行时、测试或 CI lint 方案，等交互批准；无人值守时需预批准。
6. 根因在范围外时，落最小的范围内修复，其余报告为 open。

### 质量门

- 评审没有顺手改应用代码。
- 每个删除有理由；例外有证明。
- 约束型注释要么被编码，要么被报告为 open。

---

## 5.3 `shipping`

### 工作情境

栈绿了，bot approve 了。团队说"可以合了"。

`shipping` 的规则是：只落地从最低未合并 PR 开始、连续通过独立验证的那一段。源码：`skills/poteto-mode/playbooks/shipping.md`。

### 做法

1. 每个 PR 一个非作者 verifier；verifier 在真实表面跑用户路径，给 `PASS`、`PASS+NOTES` 或 `FAIL`。
2. 从最低未合并 PR 往上，停在第一个没有 `PASS` 或 `PASS+NOTES` 的 PR；它下面是 contiguous verified run。上面的 PR 默认不能先合；只有当一个 PR 既不包含也不依赖下面那段代码时，才可以先移到 trunk 单独落地，并且要先拿新的 verdict。
3. rebase 后用 `git patch-id` 比较 verdict 时的 SHA 和当前 head；补丁不同时，只有测试、文档或 lint config 的差异且 lane 在两边构建等价（不计 noise）时，旧结果才可以保留；没有 build output 的 lane（如 dev server）必须重跑，其他漂移也要重验。改动本身仍要新检查和新评审。
4. 自底向上，一次合一个；合并后读回状态，确认 base 是主干、状态是 merged。
5. 落地过程中唯一允许的拓扑改动是 `shipping` step 4 的 next-PR retarget 与配套的 `--force-with-lease`；其他 rebase、retarget 或 force-push 不改。

### 质量门

- 只有 `PASS` / `PASS+NOTES` 进入落地段。
- patch-id 覆盖当前 head。
- 停止点之上的 PR 没有在未拿新 verdict 的情况下先合。

### pstack 的边界

pstack 停在 verified landing。部署、灰度、回滚、发布后监控不在 pstack 里；部署只是 **Always pause** 的一个不可逆动作。源码：`skills/poteto-mode/SKILL.md#Autonomy`。不要把这些写成 pstack 的机制。

---

## 5.4 `babysit`、`session pickup`、`pause safely`

### `babysit`

把 PR 或栈推到 merge-ready。先声明模式：

- `drive`：跑到 merge-ready。
- `background`：不阻塞地 triage，适合还在执行的计划。
- `threads-only`：只处理 review comments。
- `check`：一次状态检查加报告。

未声明默认 `drive`；小或 docs-only PR 用 `check`。只处理最低未合并的 frontier；顺序是冲突、评审线程、CI；CI 失败先分类再重试；bugbot 评论按证据逐条处置。不负责合并。源码：`skills/poteto-mode/playbooks/babysit.md`。

### `session pickup`

接管别人未完成的工作：读原轨迹（transcript、cloud URL 或已推送分支），重建操作状态，diff 已完成与待办，然后路由到对应 playbook。不要重做已经完成的工作。源码：`skills/poteto-mode/playbooks/session-pickup.md`。

### `pause safely`

在原子步骤边界干净暂停：取消 nested subagent，不启动新工作，不推送（除非已经推过），未提交改动做成 `wip:` 提交，在上下文之外写恢复说明。只用于明确暂停，不用于"继续"。源码：`skills/poteto-mode/playbooks/pause-safely.md`。

### 质量门

- `babysit` 声明了模式，停在 frontier，不合并。
- `session pickup` 先读原轨迹，不重做已完成的工作。
- `pause safely` 留下可恢复的 `wip:` 提交和恢复说明。

---

## 本章与 pstack 的边界

pstack 到落地为止。本章只讲 `interrogate`、`no-comments`、`shipping`、`babysit`、`session pickup`、`pause safely`；部署是 Always pause，不是本章的流程。
