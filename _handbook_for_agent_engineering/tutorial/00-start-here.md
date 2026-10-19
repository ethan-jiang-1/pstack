# 从这里开始：把 prompt 写成 pstack 能接住的样子

> 教程 00 / 12 · 上一页：[教程地图](./README.md) · 下一页：[01 用 mode 还是单 skill](./01-define.md)

## 眼前困惑

你给 agent 一句"给导出功能加个筛选，只导出我选中的行"，它很快给了方案，甚至直接改了文件。但你看完不确定：它是不是理解成了"给导出页加一个筛选器"？没勾选任何行时会导出什么？列顺序会不会变？

pstack 不要求你先写一份 spec。`docs/guide/02-poteto-mode.md` 的原话是 "You don't write a spec." 它建议把意图尽量放进 prompt 的最多五个位置，让 agent 有足够约束开工。

## 现在动作

### 1. 写 goal

说清要改什么，或者想要什么。一句话就够：

```text
导出的 CSV 从昨天开始会丢掉最后一行。
```

### 2. 写 done check

如果写了 done check，它必须能通过或失败。"更好用"不是判据。

```text
done means the 60k-row fixture exports every row.
```

### 3. 写你要看到的 proof

要真实命令输出、录屏、存下来的值，或 before/after 数字。

```text
show me the row counts before and after.
```

### 4. 写你 already know 的信息

症状、复现步骤、日志行、链接。每一条都省掉一次搜索。

```text
failing job id is 4812.
```

### 5. 写 real constraints

会改变 agent 行为的硬约束：`repro first`、`don't change any code yet`、`zero behavior change`、`let me review before proceeding`。

```text
repro first, then fix.
```

把它们拼起来：

```text
/poteto-mode the csv export drops its last row since yesterday's deploy.
failing job id is 4812. repro first, then fix.
done means the 60k-row fixture exports every row.
show me the row counts before and after.
```

## 可见成果

一条可以直接发给 `/poteto-mode` 的 prompt，最多包含五项：goal、done check、proof、what you already know、real constraints。

写不出 done check 也可以进 mode：用复述或一个 Prototype 先把问题显形。但一旦要验收，done check 必须能通过或失败。

## 休息点

可以停在这里。你已经有了一条 pstack 能接住的 prompt，不需要先写 spec，也不需要先建一套术语。

## 下一段

下一页 [01 用 mode 还是单 skill](./01-define.md) 处理入口选择：什么时候用 `/poteto-mode`，什么时候直接调一个 skill。
