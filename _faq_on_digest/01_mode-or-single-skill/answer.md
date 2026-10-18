# 用 mode 还是单 skill：看任务有没有"顺序"

> [FAQ 首页](../README.md) · [问题原文](./question.md) · [mode 与 playbook](../../_digest/system/mode-and-playbooks.md)

## 直接回答

任务需要**多步顺序、验证门或交付收口**时用 `/poteto-mode`；只需要一个动作、结果形状已经清楚时直接调那个 skill。拿不准就用 mode，它的成本主要是 token 和安装完整度，不是学习成本。

## 先看任务有没有顺序

mode 真正提供的不是"更聪明"，而是三件事：

1. **选路**：从 23 个 playbook 里选一个。
2. **状态**：把步骤逐字复制进 todo，跳过要写原因。
3. **收口**：验证、PR、回复形状都有固定要求。

如果任务只需要一个动作，这三件事都是额外开销。

## 三种典型选择

| 任务 | 建议 | 理由 |
|---|---|---|
| 修一个有复现的 bug | mode → Bug fix | 必须先复现、二分、修、在同表面验证；顺序不能省 |
| 做 feature、重构、性能修复 | mode → 对应 playbook | 有设计、委派、验证、提交多个阶段 |
| 把一个栈推到可合并或落地 | mode → Babysit / Shipping | 有独立 verdict、frontier、落地顺序 |
| "X 怎么工作"、查设计动机 | 单用 `how` / `why` / `teach` | 只读，交付一个带引用的答案 |
| 对一个改动做对抗评审 | 单用 `interrogate` | 它本身就是完整流程，不需要外层编排 |
| 清理英文、写 PR 正文 | 单用 `unslop` / `technical-writing` | 一次编辑动作，没有跨阶段状态 |
| bug 有便宜的测试路径，只想先红后绿 | 单用 `tdd` | 目标单一，且 mode 的 Bug fix 会在需要时调用它 |
| 任务还没说清 | 先 `sharpen`，再进 mode | 先把目标、规格松紧和外部检查想清楚 |

## mode 的成本与前提

- **安装完整度**：mode 的步骤会点名很多 skill；只装一部分时可能走到不存在的 skill。
- **成本**：一次 mode 运行可能派多个 subagent，token 远高于一次单 skill 调用。
- **选择风险**：mode 直接选 playbook，不做交互式澄清；选错的代价在长流程里会放大。

如果这三项都可接受，mode 的收益是稳定：同样的任务每次都走同一条有门的路径。

## 不确定时的默认

用 mode。理由不是它更"高级"，而是它会在 step 0 就把流程外化，你可以在 todo 出现后立刻判断"这不对"，然后打断或换 playbook。直接调单 skill 时，顺序错了往往要到结果出来才发现。

只想要一个只读答案时，可以明确说"只调查，不改代码"，让 mode 走 Investigation；它不会开 PR。

## 继续阅读

- [当前行为的源码](../../skills/poteto-mode/SKILL.md)
- [23 个 playbook 的选择树](../../_digest/playbooks/README.md)
- [模式：Mode as Router](../../_digest/patterns/mode-as-router.md)
