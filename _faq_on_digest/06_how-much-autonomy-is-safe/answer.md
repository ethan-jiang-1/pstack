# 自主等于少问，不等于少证据

> [FAQ 首页](../README.md) · [问题原文](./question.md) · [自主与门](../../_digest/system/autonomy-and-gates.md)

## 直接回答

先用最小的授权：一个任务、一个 PR、明确的可逆范围。确认 `show-me-your-work` 的轨迹、独立验证和停止条件都成立后，再扩展到栈或队列。自主改变的是"要不要问人"，不改变"什么算完成"。

## 四种自主运行形态，不要混着给

| 形态 | 范围 | 不该做什么 |
|---|---|---|
| `Autonomous run` | 一个任务跑到一个可检查的谓词 | 不要把谓词改松来宣布成功 |
| `Autopilot-stack` | 一批改动形成待人工落地的栈 | 不要自动合并；落地的决定权在人 |
| `Autopilot-full` | 一个 PR 队列；owner 在验证后合并 | 不要越过 operator 明确保留的 gate |
| `Orchestrate` | 一个多天 program；coordinator 写 brief、主持 program，落地持续进行，不写代码 | 不要让 coordinator 直接改代码 |

"going to bed" / "run until done" 覆盖当前任务；换一个新任务要重新判断授权，不能继承。

## 自主不等于无门

| 门 | 什么时候仍然要停 |
|---|---|
| 复现 | Bug fix 必须先在该表面复现；够不到 `control surface` 时说明原因再请人 |
| 独立验证 | Shipping 要求没写代码的 verifier 给 `PASS` 或 `PASS+NOTES`；作者自证不算 |
| 基准 | 报告性能数字前走 `benchmark-checklist`：正式报告答七问；用户只要 ballpark 时至少答第 4、7 问，并说明只有一次运行 |
| 不可逆写入 | force-push 共享分支、部署、删数据、给客户发消息 |
| 合并 | 只有明确要求 merge/land/ship/merge when ready，或开启 `Autopilot-full` / `Orchestrate` 时，合并才在授权内；`Autopilot-stack` 只交付待人工落地的栈 |

## 审计的三个动作

1. **decision trail**：`show-me-your-work` 的 TSV，一行一个决定，证据是指针。只追加，错误用新行覆盖。
2. **transcript 审计**：交付前用原始会话记录对一遍，检查每行是否对应真实决定、证据是否能解析。
3. **跨模型复查**：派一个不同模型家族的 subagent 读轨迹与 transcript，回复末尾给 **Attention** 段，列出该看的行和风险。

三件都做，才叫"可审计的自主"。只做第一件，轨迹可能只是自述。

## 建议的渐进路径

1. **先跑一个只读任务**：Investigation 或 `how`，确认停止条件和回复形状。
2. **再跑一个有复现的 bug fix**：确认它真的先复现、在同表面验证。
3. **再跑一个单 PR 的 Shipping**：确认独立 verdict 和落地顺序。
4. **最后才是栈或队列**：此时你已经见过它怎样失败，知道该看哪几行。

## 红旗

- 轨迹里只有"完成"，没有决定和证据指针。
- 验证者是写代码的同一个 agent。
- 谓词从"所有 PR 合并"变成"主要 PR 合并"。
- 合并前既没有明确的 merge/land/ship 要求，也没有覆盖它的 grant（`Autopilot-full` / `Orchestrate`）。
- CI 绿、bot approval 被当成独立验证。
- 失败被写成 `INCONCLUSIVE` 后继续推进。

出现任意一条，先停下来补证据，不要扩大授权。

## 继续阅读

- [当前行为的源码](../../skills/poteto-mode/SKILL.md)（**Autonomy** 一节）
- [自主与门：门的失败出口](../../_digest/system/autonomy-and-gates.md)
- [验证与 verdict](../../_digest/patterns/evidence-first.md)
