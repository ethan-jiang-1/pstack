# 用 `tdd` 先看见红

> 教程 06 / 12 · 上一页：[05 把工作切成可委派的单元](./05-cut-work-into-delegable-units.md) · 下一页：[07 用 brief 派 subagent](./07-delegate-and-keep-control.md)

## 眼前困惑

bug 修到最后，测试绿了。但绿可能只证明测试和实现互相一致：如果测试是从实现反推出来的，它会在实现错的时候一起错；如果测试只覆盖顺利路径，它会在边界上沉默。

`tdd` 的规则是：用户明确要求 TDD、失败测试或回归测试，或 bug 有便宜清晰的本地测试目标时，先写一个会失败的回归测试，再改实现。源码：`skills/tdd/SKILL.md`。

## 现在动作

### 1. 确认测试路径值得走

`tdd` 明确说不要硬造测试。测试需要大面积 harness、脆弱的 mock、慢的端到端、production-only 状态、模糊的复现步骤或大量无关 fixture 时，跳过新测试，改用最接近的可用验证。源码：`skills/tdd/SKILL.md#If a Failing Test Is Impractical`。

### 2. 理解 bug

写下 intended behavior、current behavior、受影响路径、最小可观察复现。源码：`skills/tdd/SKILL.md#Workflow` step 1。

### 3. 选最窄的可执行检查

优先用这条 codepath 已有的单元、组件、集成或回归测试。没有现成路径时，不要从零搭一套。step 2。

### 4. 先写会失败的测试

测试要断言 intended behavior，不镜像实现。step 3。

### 5. 在修之前运行它

确认它因为正确的原因失败。如果它通过，或因为无关原因失败，先修测试或复现。step 4。

### 6. 改实现

做最小的 production change，保持附近契约。step 5。

### 7. 再跑一次

确认测试现在通过。step 6。

如果 `tdd` 不适用，仍然要有一条可重复的检查：在真实表面跑用户路径，把命令、输入、输出记下来。这条检查由 `prove-it-works` 和 `shipping` 的 verdict 接着管。

## 可见成果

一条先失败、后通过的回归测试，或一条明确写出"为什么不写测试"的真实表面检查。

## 休息点

可以停在这里。你现在有一条会变红的检查；下一页把工作交给 subagent。

## 下一段

[07 用 brief 派 subagent](./07-delegate-and-keep-control.md)：委派不是把任务丢出去，而是写一份能让陌生执行者独立完成的 brief。
