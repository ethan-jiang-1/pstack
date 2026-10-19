# 用 mode 还是单 skill

> 教程 01 / 12 · 上一页：[00 从这里开始](./00-start-here.md) · 下一页：[02 把 playbook 步骤放进 todo](./02-give-words-a-home.md)

## 眼前困惑

prompt 写好了，下一步是输入 `/poteto-mode`，还是直接调 `how`、`tdd` 这样的单 skill？

`poteto-mode` 会匹配 playbook、复制步骤、按角色派 subagent，成本更高；单 skill 更轻，但你要自己知道用哪个、顺序是什么。

## 现在动作

### 1. 看任务有没有顺序

需要多步顺序、验证门或交付收口时，用 `/poteto-mode`。只需要一个动作、结果形状已经清楚时，直接调那个 skill。

| 任务 | 用什么 | 原因 |
|---|---|---|
| 修一个有复现的 bug | mode → Bug fix | 先复现、再修、在同表面验证，顺序不能省 |
| 做 feature、重构、性能修复 | mode → 对应 playbook | 有设计、委派、验证多个阶段 |
| 把栈推到可合并或落地 | mode → Babysit / Shipping | 有独立 verdict、frontier、落地顺序 |
| "X 怎么工作"、查设计动机 | 单用 `how` / `why` / `teach` | 只读，交付一个带引用的答案 |
| 对抗评审一个 diff | 单用 `interrogate` | 它本身就是完整流程 |
| 清理英文、写 PR 正文 | 单用 `unslop` / `technical-writing` | 一次编辑动作，没有跨阶段状态 |
| bug 有便宜的测试路径，只想先红后绿 | 单用 `tdd` | 目标单一；Bug fix 会在需要时调用它 |
| 任务还没说清 | 先 `sharpen`，再进 mode | 先把目标、规格松紧和外部检查想清楚 |

拿不准就用 mode。它的成本是 token 和安装完整度，不是学习成本。源码：`skills/poteto-mode/SKILL.md#Playbooks`。

### 2. 该问人的时候再问

mode 要求先分类，不要把不确定都丢给人。源码：`skills/poteto-mode/SKILL.md#Non-negotiables`。

- 答案能通过跑一下观察到（行为、时序、布局、输出、性能、eval 是否分得开）：这不是人的问题。用 Prototype playbook 做出来，让结果决定。
- 只读 Investigation，交付物是带引用的答案：留在 Investigation，用证据回答，不做探针。
- 真正的产品、偏好或方向选择，实验无法定论：问人。
- 全自主授权下，授权覆盖的选择：直接决定、执行、报告，不请求确认。
- 只有 operator 能做的选择，但在授权下：取一个默认值推进，完整报告，并说明 operator 可以让你改成什么。
- operator 明确保留的 gate，或 Always pause 清单：仍然要等人。

一条硬规则：不给简写 token。"回复 A 表示同意"这类设计被禁止；人用自己的话回答。

### 3. 被问"要不要做 X"时，可以说不

推荐不是验证。被邀请加范围时给真实判断；同意不是默认值。源码：`skills/poteto-mode/SKILL.md#Autonomy`。

## 可见成果

一句明确的入口选择：`/poteto-mode` 或某个单 skill，以及一个理由（有没有顺序、有没有门、要不要交付）。

## 休息点

可以停在这里。你现在知道该进 mode 还是直接调单 skill；进 mode 后，下一页处理 playbook。

## 下一段

[02 把 playbook 步骤放进 todo](./02-give-words-a-home.md)：mode 匹配 playbook 后，第一步不是写代码，而是把步骤逐字复制进 todo。
