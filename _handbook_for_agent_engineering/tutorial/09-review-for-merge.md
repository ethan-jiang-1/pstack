# 评审变更是否值得合入

> 教程 09 / 12 · 上一页：[08 证明兑现的是原承诺](./08-prove-the-promise.md) · 下一页：[10 把验证过的一段栈落地](./10-ship-observe-and-loop.md)

## 眼前困惑

P2 的导出 PR 已经打开。diff 不大，测试绿了，执行者说"按 brief 做完了"。

你读 diff，很快落入两种模式之一：

- **挑刺**：命名不够好、这个函数可以拆、注释可以删。改完这些，PR 更整齐，但没人回答它到底对不对。
- **放行**：测试绿、CI 绿、bot approve。三天后才发现无权限的 403 路径没有处理。

问题不在于你读得不够仔细，而在于读的人只有一个模型。你自己的盲点会原样留在结论里，而一个没写这份代码的模型不会共享这些盲点。

pstack 用 `interrogate` 做这件事：同一段 diff、同一段 intent，发给多个模型；每个模型独立挑错；再由 lead reviewer 按工程判断收成一个结论。它不投票，也不自动改代码。

源码：`skills/interrogate/SKILL.md`。

## 现在动作

### 1. 先固定 scope

`interrogate` 的第一步是把"审什么"定死：

- 用户点名了文件和 diff，就用那个；
- 在 feature branch 上，用 `git diff main...HEAD`（或对应 base）拿完整 changeset；
- 只提到"刚才那段工作"，就把相关文件收齐。

把 diff 和 reviewer 理解代码所需的上下文文件放在一起。范围定下来之后，所有 reviewer 看同一份，不凭记忆各自选。

源码：`skills/interrogate/SKILL.md` Step 1。

### 2. 把 intent 写成一段，然后不再改

派 reviewer 之前，用一段话写清这次改动要做什么。材料是用户的话、commit message、PR 描述（如果存在）和代码本身。

写不出来就停下来问，不要带着模糊的 intent 开审。写出来之后，这段文字原样进每个 reviewer 的 prompt，审的过程中不改。条件变了，是重新开一次 `interrogate`，不是悄悄改 intent。

源码：`skills/interrogate/SKILL.md` Step 2。

### 3. 按 `interrogate reviewers` 行派 reviewer

要它跑，得明确说 `interrogate`，或 `challenge this`、`tear this apart` 这类触发词。它不会自己冒出来。

一条消息里全部派出，每个 entry 一个 reviewer，标成 Reviewer A、B……，与 entry 数量对齐。条目来自设置文件里的 `interrogate reviewers` 行；文件或这一行缺失时用默认表。

每个 reviewer：

- `readonly: true`，只能读，不能改；
- 拿到同一份模板：intent、diff、`references/rubric.md` 的 rubric、`references/code-quality-review.md` 的 code-quality lens；
- 不分配"安全专家""性能专家"这类人设。对抗性来自模型不同，不来自你给它的角色。

没有 subagent 工具的 harness，就由你自己按顺序一个一个跑，prompt 和模型保持不变。

源码：`skills/interrogate/SKILL.md` Step 3、`references/reviewer-prompt.md`。

### 4. 合成：先合并重复，再看谁独立提出

reviewer 陆续回来后，先解析每一条发现，把描述同一问题的条目合并，并记下是哪些模型提的。

判断信号强弱只有一个可靠依据：**两个以上模型独立提出同一条**。这是最高信号。只有一个模型提出的也要读，但按单条信号的权重处理。两个模型明确给出相反结论的，记下来；它是 lead 判断的上下文，不是噪音。

源码：`skills/interrogate/SKILL.md` Step 4。

### 5. lead reviewer 把发现放进四个桶

lead reviewer 是资深工程师，不是中性聚合器。每条发现进一个桶，写上提出它的模型和一句理由：

| 桶 | 放什么 |
|---|---|
| **Act On** | 影响正确性、安全或可维护性的真实问题，按实际目标衡量会挡住一个真实的 PR |
| **Consider** | 成立，但现在处理它的成本是否值得还不确定；值得让用户看见 |
| **Noted** | 技术上成立，但现在不可行动：依赖上下文、过早优化、影响太小 |
| **Dismissed** | 错的、挑刺的、或缺上下文的；给一句理由 |

几个常见误判要挡住：

- reviewer 只看过一段代码和一段 intent，不知道哪些是临时脚手架、哪些约束在代码之外、下一个 PR 会补什么。lead 有完整对话，用上它。
- "如果这里传 null 呢"只有调用方真的能传 null 才算发现。沿调用链追到调用点；上游已经校验或类型不允许，就 Dismissed。
- "我会换个写法"不是缺陷。除非 reviewer 指出当前写法具体哪里会坏，否则 Dismissed。
- 安全与正确性的发现即使只有一个模型提出，也要多查一遍再驳回。

Act On 超过 5 条，通常说明没有在过滤，而不是代码真有 5 个阻塞问题。Dismissed 不是凑数：把驳回的和理由摆出来，用户才能在你判断错的地方推翻你。

源码：`skills/interrogate/SKILL.md` Step 5、`references/lead-judgment.md`。

### 6. rubric 和 code-quality lens 在问什么

rubric 有六个 lens，不是每条都适用于每个改动：

- **正确性**：边界输入、错误处理、状态与竞态、幂等。怀疑有 bug 时沿执行路径追出调用链，不要只说"这里可能是 null"。
- **根因还是症状**：guard clause 是否盖住了更深的不变量破坏，retry 是否在掩盖坏掉的契约，类型转换是否在压住建模错误。能用类型、lint 或运行时检查让错误写法不可能，就不要只留一句注释。
- **结构完整性**：校验是否在系统边界，抽象层级是否混在一起，耦合是否让下一次改动更难，是否留下新旧两条并行 API。
- **验证**：测试测的是行为还是实现；bug fix 有没有对应测试；集成边界有没有全路径测试；是在检查真实值，还是在检查 mtime、缓存这类代理指标。
- **复杂度预算**：能不能更简单而不丢正确性或清晰度；只有一个调用点的抽象、为不存在的场景准备的配置、死代码，都算负债。简单更好，除非简单是错的。
- **安全**：把输入路径追到危险 sink（SQL、shell、eval、innerHTML），看鉴权缺口、日志和报错里的 secret、安全路径上的 TOCTOU。

code-quality lens 在此之上要求主动找结构性简化，而不是停在命名：

- 有没有一次"code judo"式的重构，行为不变，但整段分支、helper、条件分支或层直接消失；
- 一个 PR 是否把文件从 1000 行以下推到 1000 行以上而没有强理由；
- 是否在已有流程里塞进临时的条件分支，把原有路径缠成 spaghetti；
- 逻辑是否留在它该在的那一层，是否复用了已有的 helper，而不是在共享代码里散落 feature 检查；
- 类型和边界是否干净：不必要的 optional、`unknown`、`any`、大量 cast，通常说明不变量没写清。

它的 approval bar 很直接：行为看起来对，不构成 approve 的理由。上面这些是 presumptive blocker，除非作者能给出理由。

源码：`skills/interrogate/references/rubric.md`、`skills/interrogate/references/code-quality-review.md`。

### 7. 它不决定合入

`interrogate` 交出的是 Act On / Consider / Noted / Dismissed 的结论，不是 `PASS` / `PASS+NOTES` / `FAIL`，也不自动应用任何修改。

合入的门在 `shipping` step 1：一个没写这份代码的 verifier，用匹配的 control skill 按用户的方式跑一遍改动，parent 对 head，然后在这个 PR 上给出 `PASS`、`PASS+NOTES` 或 `FAIL`。作者自证、绿 CI、bot approve 都不是 verdict。

所以"这个改动值不值得合入"有两个分开的答案：`interrogate` 说该修什么，`shipping` 的 verdict 说这个 patch 能不能落地。你既是作者又是 lead reviewer 时，你的阅读仍然不是 verdict。

源码：`skills/poteto-mode/playbooks/shipping.md` step 1。

## 可见成果

一份 `interrogate` 的结论，格式就是它的 Output Format：

```text
Intent
> 运营在订单列表勾选行后只导出勾选行；无权限时给出可读提示；5 万行以上的等待策略不在本次范围。

Reviewers
- Reviewer A: <model>, 4 findings
- Reviewer B: <model>, 6 findings

Act On
- 无权限的 403 路径没有页面提示（A、B 都提，A 追到了调用链）
- 超过 5 万行时 missing 行被静默跳过（B 提，A 未反驳）

Consider
- 重复的按钮组件可以合并，但会牵动另一个页面，适合单独一个 PR

Noted
- 导出器的内部命名和现有模块不一致（成立，但本次改动没有加重它）

Dismissed
- 建议把整个导出流程改成队列（A 提；没有说明当前同步路径的具体故障，属于"我会换个写法"）

Agreement Map
- A、B 都指向无权限路径，这是最高信号；只有 B 提到 missing 静默跳过，但给出了复现路径，按 Act On 处理。
```

改完 Act On 之后，重新跑一次 `interrogate`，因为旧结论只覆盖旧的 diff。

## 休息点

可以停在这里。你已经有了一个多模型审过的结论：哪些必须改、哪些值得考虑、哪些驳回，以及为什么。

`interrogate` 不自动应用修改，所以现在是你决定采纳哪几条的时刻。如果发现的是行为本身错了，不要在 review 里顺手改行为；回到产生这段行为的 playbook（`feature`、`bug fix`、`refactoring`），把行为改对，再重新 `interrogate`。

## 下一段

下一页 [10 把验证过的一段栈落地](./10-ship-observe-and-loop.md) 处理 review 之后的半段：`babysit` 把 PR 推到 merge-ready，`shipping` 用独立 verdict 确认每个 PR，再自底向上把连续通过的一段落地。pstack 停在那里，没有发布、监控和回滚。

想先看完整方法，查 [手册：验证](../manual/verify.md)；想回到教程起点，见 [教程地图](./README.md)。
