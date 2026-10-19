# 把 playbook 步骤放进 todo

> 教程 02 / 12 · 上一页：[01 用 mode 还是单 skill](./01-define.md) · 下一页：[03 先命名 data shape](./03-make-a-promise-observable.md)

## 眼前困惑

进了 `/poteto-mode`，它说"已匹配 Bug fix"。接下来最容易发生的，是它直接开始读代码、改代码。顺序一旦由临场发挥决定，复现、验证、评审这些门就会被跳过。

pstack 的做法是先把流程外化：匹配到 playbook 后，把那份 playbook 的步骤逐字复制进 todo。源码：`skills/poteto-mode/SKILL.md#Playbooks`。

## 现在动作

### 1. 让 mode 匹配一个 playbook

playbook 有 23 个，标题和描述就是匹配信号。你不需要先选；mode 读请求后选一个。源码：`skills/poteto-mode/playbooks/`。

如果请求里有明确信号，说清它：

```text
/poteto-mode the csv export drops its last row. repro first, then fix.
```

`repro first` 会把任务匹配到 Bug fix，而不是 Feature。

### 2. 看 todo 的前几项

匹配后，todo 的最前面应该出现 playbook 的步骤，逐字复制。以 Bug fix 为例（下面只摘每步的短标题，实际 todo 是整段复制）：

```text
1. Reproduce on the same surface yourself.
2. Bisect the hypotheses, using runtime evidence.
3. Plan the fix.
4. Verify on the same surface.
5. Stage the failing repro before the fix.
6. Opening a PR.
...
```

这些文字来自 playbook，不是 agent 的概括。概括会丢条件："reproduce it yourself" 被写成 "reproduce it"，责任主体就没了。

### 3. 跳过的步骤留在列表里

不做的步骤不能删，写一行 `skip: <reason>`：

```text
5. Stage the failing repro before the fix.   skip: test path is integration-heavy
```

这样"没做"和"忘了"可区分。不适用某个维度时，同理写 `n/a: <reason>`。

### 4. 再写任务特有的 todo

playbook 步骤之后，才是这次任务特有的项：哪个文件、哪个函数、哪个 fixture。顺序是"先协议，后细节"，不是反过来。

## 可见成果

一个 todo 列表，前几项是 playbook 步骤，带 `skip` / `n/a` 原因；后面才是任务特有项。你扫一眼就知道这次会经过哪些门。

## 休息点

可以停在这里。流程已经外化，接下来才是具体工作；下一页处理写代码前必须先定的事：data shape。

## 下一段

[03 先命名 data shape](./03-make-a-promise-observable.md)：在写逻辑之前，先定核心类型与数据结构，再让可观察结果有地方落。
