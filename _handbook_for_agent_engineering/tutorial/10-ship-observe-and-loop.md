# 把验证过的一段栈落地

> 教程 10 / 12 · 上一页：[09 用 interrogate 评审](./09-review-for-merge.md) · 下一页：[11 用 correct 与 reflect 改环境](./11-improve-the-environment.md)

## 眼前困惑

PR 都绿了，评审也过了。团队说"可以合了"。但绿 CI 不是 verdict，评审通过也不证明真实用户路径能用。pstack 在这里有一条很窄的规则：只落地从最低未合并 PR 开始、连续通过独立验证的那一段。源码：`skills/poteto-mode/playbooks/shipping.md`。

## 现在动作

### 1. 先让 `babysit` 把 PR 推到 merge-ready

`babysit` 负责把 PR 或栈推到可合并：先声明 `drive` / `background` / `threads-only` / `check`，只处理最低未合并的 frontier，顺序是冲突、评审线程、CI。它不负责合并。源码：`skills/poteto-mode/playbooks/babysit.md`。

如果你要的是"check on X"，用 `check` 模式，不要默认 `drive`。

### 2. 让 `shipping` 独立验证

`shipping` 不信任绿 CI，也不信任作者自证。每个 PR 一个 verifier，verifier 不能是写这份代码的人，在真实表面跑用户路径，给出 `PASS`、`PASS+NOTES` 或 `FAIL`。源码：`shipping.md` step 1。

### 3. 找 contiguous verified run

从最低未合并 PR 往上走，停在第一个没有 `PASS` 或 `PASS+NOTES` 的 PR。它下面的 PR 构成 contiguous verified run；上面的 PR 默认不能先合。只有当一个 PR 既不包含也不依赖下面那段代码时，才可以先移到 trunk 单独落地，并且要先拿新的 verdict。源码：`shipping.md` step 2。

### 4. 检查 patch-id

rebase 会改 SHA，但补丁可能不变。用 `git patch-id` 比较 verdict 时的 SHA 和当前 head；补丁不同时，只有测试、文档或 lint config 的差异且 lane 在两边构建等价时，旧结果才可以保留。改动本身仍要新检查和新评审。源码：`shipping.md` step 3。

### 5. 自底向上，一次合一个

先合连续段最低的未合并 PR。合并后读回状态，确认 base 是主干、状态是 merged；再把下一个 PR 的 base 移到主干。落地过程中不要改栈的拓扑。源码：`shipping.md` step 4、5。

### 6. pstack 到这里为止

部署、灰度、回滚、发布后监控不在 pstack 里。部署只是 **Always pause** 清单里的一个不可逆动作：force-push 到共享分支、部署、删除数据、给客户发消息。源码：`skills/poteto-mode/SKILL.md#Autonomy`。

不要把这些写成 pstack 的能力，也不要用一个新的流程名把它们包装起来。需要发布时，那是 pstack 之外的工程决定。

## 可见成果

一段从最低未合并 PR 开始、连续有 `PASS` 或 `PASS+NOTES` 的栈，按 patch-id 检查过，自底向上合完。

## 休息点

可以停在这里。落地完成后，下一页处理怎样让下一次工作更容易：`correct` 和 `reflect`。

## 下一段

[11 用 correct 与 reflect 改环境](./11-improve-the-environment.md)：把重复错误变成结构，把会话经验写回 skill。
