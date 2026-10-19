# Autonomy 与 gate

## 一句话

pstack 的自主写在 `poteto-mode` 的两处：**Autonomy** 规定可逆工作直接做、不可逆写入必须停；Non-negotiables 的提问分类规定遇到 fork 时先判断它属于哪一类。判断标准不是"要不要问人"，而是这个 call 属于哪一类、当前有没有覆盖它的 grant。

## 五个类别

| 类别 | 行为 | 出处 |
|---|---|---|
| **full grant** | "Don't stop" / "going to bed" / "run until done" / "be fully autonomous" 把授权延长到整个任务。`Autopilot-full` 是它的结构化形式；`Autopilot-stack` 只授权构建，落地仍归 operator。grant 覆盖的 call 自己做，报告结果，不请求确认。 | `SKILL.md#Autonomy`、`#Non-negotiables` |
| **product / preference calls** | 真正的产品、偏好或方向选择，实验无法定论。没有 grant 时问人，给选项，人用自己的话回答；grant 覆盖时自己决定并报告。 | `#Non-negotiables` |
| **operator-only calls** | 只有 operator 能做的选择。没有 grant 时仍要等 operator；在 grant 下用一个默认值推进，完整报告，并说明 operator 可以让你改成什么。 | `#Non-negotiables` |
| **named gates** | operator 明确点名的 gate，即使有 grant 也要等。例：`multi-phase-plan` 的 review gate；`Autopilot-full` 里 operator 声明属于自己的 item。 | `playbooks/multi-phase-plan.md`、`playbooks/autopilot-full.md` |
| **Always pause** | 不可逆写入：force-push 到共享分支、部署、删除数据、给客户发消息。 | `SKILL.md#Autonomy` |

grant 不覆盖的 call 按 product/preference call、operator-only call、named gate 或 `Always pause` 处理。还有一类不是人的问题：跑一下就能观察的事实（行为、时序、布局、输出、性能、eval 是否分得开）。它归 `Prototype` playbook，或用 `Investigation` 直接回答，不走提问。只读 Investigation 的交付物是带引用的答案，不做探针。

## 提问的细节

- operator 用自己的话回答。不给"回复 A 表示同意"这类 shorthand token。
- 推荐是判断，不是附和。"不做"、"这不值得占位置"都是可接受的回答。
- 先分类再问。把所有不确定都丢给人，和从不停下来问，都是偏离。

## gate 写在步骤里

| gate | 可检查条件 | 失败出口 |
|---|---|---|
| Bug fix step 1 | 在 matching surface 上用 control skill 实际复现 | 合成触发、收紧条件、加 instrumentation；只有 control surface 够不到目标时才请用户，并给出具体原因 |
| Bug fix step 2 | 存活的机制有运行时证据 | 继续二分，不猜 |
| Shipping step 1 | 每个 PR 有一个没写这份代码的 verifier 的 `PASS` 或 `PASS+NOTES` | 没有 verdict 或只有 `FAIL` 的 PR 不进入落地序列 |
| Shipping step 2 | 从最低未合并 PR 向上连续的 verified run | 遇到第一个没有 `PASS` 或 `PASS+NOTES` 的就停；独立 PR 需新 verdict 才能单独落地 |
| Babysit step 1 | 已声明 `drive` / `background` / `threads-only` / `check` | 未声明默认 `drive`；小 PR 或 docs-only 默认 `check` |
| Feature step 3 | 四项 throughput checkpoint 都有值或 `n/a: <reason>` | 缺项不能进 fan-out |
| Benchmark | `benchmark-checklist` 的七问都有真实运行的证据 | 只有一次运行就说只有一次；选项比较不算 ballpark |
| 长任务 | `show-me-your-work` 的轨迹与 transcript 对得上 | 补一行 supersede，不删历史 |

这些 gate 的共同点是：条件能被工具输出、artifact、用户确认或状态标签证实。"我觉得理解了"不是条件。

## Decision trail

长任务、自主运行、或用户离开后要复查的工作，用 `show-me-your-work` 留一个 TSV：

```text
ts  phase  decision  why  evidence  result
```

规则：

- 只追加，不修改、不删除。错误用新行覆盖，并指向正确的证据。
- evidence 是指针（commit SHA、PR 号、`file:line`、artifact 路径），不是一段话。
- 一个 run 是一段对话；pickup、替换 agent、新会话都算新 run，第一行用 `phase=start`。
- 交付前用 transcript 审计轨迹：每行是否对应真实决定或动作，证据是否能解析。
- 再派一个不同模型家族的 subagent 复查轨迹，回复末尾给 **Attention** 段，列出该看的行与风险。

## 边界

- **可逆性判断由 agent 做，但错了要认。** 一个被误判为可逆的删除，仍然是事故。
- **授权有范围。** "going to bed" 覆盖这个任务，不覆盖下一个任务；`Autopilot-full` 的 grant 只覆盖队列里的 item。
- **gate 不因授权消失。** 用户说"全自动"不等于放弃 shipping 的独立验证；它只改变了"要不要问"，没有改变"什么算完成"。
- **死路要报。** 谓词达不到时说清楚，不要把谓词改松再宣布成功。
- **Always pause 与 landing 有张力。** `shipping` 会合并 PR，`Autopilot-full` 会落地队列；源码没有统一回答合并算不算不可逆写入。安全读法是：只有用户明确要求 merge/land/ship/merge when ready，或开启 `Autopilot-full` / `Orchestrate` 时，合并才在授权内；`Autopilot-stack` 停在待 operator 落地的栈。
- **decision trail 是自报的。** 所以才有 transcript 审计和跨模型复查；单靠轨迹本身不能证明轨迹是真的。

## 关键洞察

1. 自主的对立面不是谨慎，而是无证据。可逆的事快速做，不可逆的事停下来，两者都要求证据。
2. gate 的价值在于失败出口。只有"必须满足"没有"不满足时去哪"，gate 会变成死胡同。
3. 轨迹是自主的配套成本。跑得越久，越需要一种让人事后能检查的形态。
4. 授权改变问不问，不改变算不算完成。

## 源码

- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Autonomy、Non-negotiables）
- [`playbooks/bug-fix.md`](../../skills/poteto-mode/playbooks/bug-fix.md)、[`shipping.md`](../../skills/poteto-mode/playbooks/shipping.md)、[`babysit.md`](../../skills/poteto-mode/playbooks/babysit.md)、[`feature.md`](../../skills/poteto-mode/playbooks/feature.md)
- [`playbooks/multi-phase-plan.md`](../../skills/poteto-mode/playbooks/multi-phase-plan.md)、[`autopilot-full.md`](../../skills/poteto-mode/playbooks/autopilot-full.md)、[`autopilot-stack.md`](../../skills/poteto-mode/playbooks/autopilot-stack.md)
- [`skills/benchmark-checklist/SKILL.md`](../../skills/benchmark-checklist/SKILL.md)
- [`skills/show-me-your-work/SKILL.md`](../../skills/show-me-your-work/SKILL.md)
