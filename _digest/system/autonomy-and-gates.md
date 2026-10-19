# 自主与门

## 一句话

pstack 的默认姿态是**在可逆范围内不问人**，只在不可逆、真正的产品取舍或死路前停下。但"自主"不是"没有门"：每个高风险转移都有自己的可检查条件，长任务还要留下可审计的 decision trail。

## 自主的三层

### 1. 直接做

使用任何 MCP 工具；可逆的工作和外部动作（团队聊天、工单更新、启动 eval）不需要先问。这条对应 `never-block-on-the-human` 原则：可逆的事先做，做完再让人纠正。

### 2. 必须暂停

不可逆的写入：

- force-push 到共享分支
- 部署
- 删除数据
- 给客户发消息

这份清单只列不可逆写入，范围很窄。它只保护那些"做错了无法用下一次提交挽回"的动作。

### 3. 会话覆盖

"Don't stop" / "going to bed" / "run until done" / "be fully autonomous" 会把授权延长到整个任务。`Autopilot-full` 与 `Autopilot-stack` 是这种授权的结构化形态。

## 提问的分类

pstack 不允许把所有不确定都丢给人。mode 要求先分类：

| 不确定的类型 | 处理 |
|---|---|
| 跑一下就能观察的事实（行为、时序、布局、输出、性能、eval 是否分得开） | 不是人的问题。用 Prototype playbook 做出来，让结果决定 |
| 只读调查，交付物是一个带引用的答案 | 留在 Investigation，用证据回答，不做探针 |
| 真正的产品、偏好或方向选择，实验无法定论 | 问人 |
| 在完整自主授权下，且授权覆盖这个选择 | 自己做，报告结果，不请求确认 |
| 只有 operator 能做的选择，但在授权下 | 用一个默认值推进，完整报告，并说明"你可以让我改成什么" |
| 人明确命名的 gate，或 Always pause 清单 | 仍然要等人 |

两条细节很关键：

- **不给简写 token。** "回复 A 表示同意"这类设计被禁止；人用自己的话回答。
- **推荐不是验证。** 被问"要不要做 X"时，可以说不、可以反对、可以说"这不值得占位置"。附和不是默认值。

## 门在哪里

自主与门不是矛盾：门保护的是那些"agent 容易自欺"的转移。

| 门 | 可检查条件 | 失败出口 |
|---|---|---|
| Bug fix step 1 | 在该表面上实际复现 | 合成触发、收紧条件、加 instrumentation；只有控制面够不到时才请用户，并说明原因 |
| Bug fix step 2 | 存活假设有运行时证据 | 继续二分，不猜 |
| Shipping step 1 | 每个 PR 有非作者的 verifier verdict | 没有 verdict 的 PR 不进入落地序列 |
| Shipping step 2 | 从最低未合并 PR 向上连续的 verified run | 遇到第一个没有 `PASS` 或 `PASS+NOTES` 的就停 |
| Babysit step 1 | 已声明 `drive`/`background`/`threads-only`/`check` | 未声明默认 `drive` |
| Benchmark | 七问都有真实运行的证据 | 只有一次运行就说只有一次；选项比较不算 ballpark |
| Feature step 3 | 四项 throughput checkpoint 都有值或 `n/a: <reason>` | 缺项不能进 fan-out |
| 长任务 | `show-me-your-work` 有轨迹，且与 transcript 对得上 | 补一行 supersede，不删历史 |

这些门的共同点：**条件能被工具输出、artifact、用户确认或状态标签证实**。"我觉得理解了"不是条件。

## Decision trail（决策轨迹）

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

这把"自主"从"相信我"变成"沿着证据检查我"。

## 自主的边界条件

- **可逆性判断由 agent 做，但错了要认。** 一个被误判为可逆的删除，仍然是事故。
- **授权有范围。** "going to bed" 覆盖这个任务，不覆盖下一个任务；新任务重新判断。
- **门不因授权消失。** 用户说"全自动"不等于放弃 shipping 的独立验证；它只改变了"要不要问"，没有改变"什么算完成"。
- **死路要报。** 谓词达不到时说清楚，不要把谓词改松再宣布成功。

## 值得注意

- **Always pause 清单与 playbook 的授权有张力。** `Shipping` 会合并 PR，`Autopilot-full` 会落地队列；合并算不算不可逆写入，源码没有统一回答。安全读法是：只有在用户明确要求 land/ship 或开启 `Autopilot-full` 时，合并才在授权内；`Autopilot-stack` 只交付待人工落地的栈。
- **"不问人"依赖 agent 能识别什么是产品取舍。** 分类表写得很细，但仍是判断题。
- **decision trail 是自报的。** 所以才有 transcript 审计和跨模型复查；单靠轨迹本身不能证明轨迹是真的。

## 关键洞察

1. **自主的对立面不是谨慎，而是无证据。** 可逆的事快速做，不可逆的事停下来，两者都要求证据。
2. **门的价值在于失败出口。** 只有"必须满足"没有"不满足时去哪"，门会变成死胡同。
3. **轨迹是自主的配套成本。** 跑得越久，越需要一种让人事后能检查的形态。
4. **授权改变问不问，不改变算不算完成。** 把这两件事分开，是长任务不漂移的关键。

> 源码：[`poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Autonomy）、[`bug-fix`](../../skills/poteto-mode/playbooks/bug-fix.md)、[`shipping`](../../skills/poteto-mode/playbooks/shipping.md)、[`babysit`](../../skills/poteto-mode/playbooks/babysit.md)、[`feature`](../../skills/poteto-mode/playbooks/feature.md)、[`benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md)、[`show-me-your-work`](../../skills/show-me-your-work/SKILL.md)。
