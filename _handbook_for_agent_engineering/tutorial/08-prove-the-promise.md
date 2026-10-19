# 用 verdict 证明

> 教程 08 / 12 · 上一页：[07 用 brief 派 subagent](./07-delegate-and-keep-control.md) · 下一页：[09 用 interrogate 评审](./09-review-for-merge.md)

## 眼前困惑

subagent 说"做完了"，测试也绿了。你能把这句话交给下一位吗？

pstack 的答案是不能。`prove-it-works` 要求验证真实产物；`shipping` 要求一个没写这份代码的 verifier 给出 `PASS`、`PASS+NOTES` 或 `FAIL`。作者自证、绿 CI、bot approval 都不是 verdict。源码：`skills/principle-prove-it-works/SKILL.md`、`skills/poteto-mode/playbooks/shipping.md` step 1。

## 现在动作

### 1. 用真实产物验证

`prove-it-works` 的规则是：运行功能、读实际值、看 diff。不要用代理指标。"能编译"不是证据；"测试绿了"只在测试覆盖了该行为时才是证据。

### 2. 拿到非作者 verdict

verifier 不能是写这份代码的人。它在真实表面上跑用户路径，parent against head，然后给一个 verdict：

- `PASS`：通过。
- `PASS+NOTES`：通过，但有需要注意的限制。
- `FAIL`：没有通过。

绿 CI 和 bot approval 不是 verdict，因为它们没有像用户一样跑这条路径。源码：`shipping.md` step 1。

### 3. 只有 `PASS` 或 `PASS+NOTES` 才算通过

落地时从最低未合并 PR 往上，取连续有 `PASS` 或 `PASS+NOTES` 的一段。第一个没有 `PASS` 或 `PASS+NOTES` 的 PR 就是停止点；它上面的 PR 默认不能先合。只有当一个 PR 既不包含也不依赖下面那段代码时，才可以先移到 trunk 单独落地，并且要先拿新的 verdict。源码：`shipping.md` step 2。

### 4. rebase 后检查 patch-id

rebase 会改提交 SHA，但补丁内容可能不变。用 `git patch-id` 比较 verdict 时的 SHA 和当前 head：补丁相同，旧 verdict 仍覆盖；补丁不同，先按 `shipping` step 3 的例外判断——只有测试、文档或 lint config 的差异，且 lane 在两边构建等价时，旧结果可以保留。改动本身仍要新检查和新评审。源码：`shipping.md` step 3。

### 5. 把证据写进回复

回复里给：改了什么、跑了什么、看到什么、哪些没覆盖。不要只写"已验证"。

## 可见成果

一个来自非作者 verifier 的 `PASS` 或 `PASS+NOTES`，加一条 patch-id 检查。它就是 verdict 加证据。

## 休息点

可以停在这里。verdict 有了，下一页处理评审：verdict 证明"能用"，`interrogate` 判断"值不值得合"。

## 下一段

[09 用 interrogate 评审](./09-review-for-merge.md)：多模型对抗评审、四个桶、lead judgment。
