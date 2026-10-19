# 先命名 data shape

> 教程 03 / 12 · 上一页：[02 把 playbook 步骤放进 todo](./02-give-words-a-home.md) · 下一页：[04 用 architect 画 public surface](./04-design-a-testable-shape.md)

## 眼前困惑

请求说"给导出加个筛选，只导出选中的行"。实现者开始想组件、按钮、事件，但没人说清"选中"在数据里是什么，"导出结果"又是什么形状。写到一半才发现：页面传的是当前页勾选，导出器按全部勾选处理，两边各自都对，合起来不对。

pstack 在 Non-negotiables 里要求：任何 code 之前，先命名 data shape，再选组织它的结构。源码：`skills/poteto-mode/SKILL.md#Non-negotiables`，具体判断见 `skills/principle-model-the-domain/SKILL.md`。

## 现在动作

### 1. 写下输入

```text
Selection = { orderIds: string[] }
ExportFormat = "csv" | "xlsx"
```

不要先写"筛选器组件"。先写这次操作真正接收的数据。

### 2. 写下输出

```text
ExportResult =
  | { ok: true; rows: number; file: Blob }
  | { ok: false; reason: "no-rows" | "permission" | "too-large" }
```

输出里带上失败出口。只写 happy path，测试就只能测 happy path。

### 3. 让非法状态构造不出来

"选中"要么是"当前页的若干行"，要么是"全部筛选结果"，不能靠一个可空的 `selectedRows` 加注释表达。用判别式、非空类型或单独的类型分支，让矛盾状态在类型层就写不出来。源码：`skills/typescript-best-practices/SKILL.md`。

### 4. 把可观察结果写进 plan 的 You see 与 Verify

如果这次工作会走 `multi-phase-plan`，每个 PR 都有 `You see` 和三条 verify lane：unit、live、perf。`You see` 写用户可观察的结果；Verify 写具体命令或表面。PR 只有在 unit、live、perf 都勾上时才算 verified；测试不是充分验证。

```text
You see: 勾选 3 行导出，下载文件恰好 3 行，表头与全量导出一致。
Verify, unit: 运行 export-e2e；断言 rows === 3。
Verify, live: 在真实导出页勾选 3 行，下载并打开文件，数行数。
```

源码：`skills/poteto-mode/playbooks/multi-phase-plan.md`、`skills/poteto-mode/playbooks/feature.md`。

### 5. 用 unit 测试和真实表面各给一条证据

data shape 定完后，验证分两层：有便宜测试路径就写一条 unit 测试；`tdd` 只在它的触发条件成立时用（教程 06）。用户可见行为还要在真实表面跑一条路径。两者不是二选一：测试只证明分支行为，真实表面证明用户路径。验证方式在下一页和教程 06 展开。

## 可见成果

一张输入 / 输出 / 失败出口的 data shape，加 `You see` 与 unit / live / perf 三处验证（不适用时写 `n/a`）。

## 休息点

可以停在这里。你现在有类型和判据；下一页处理跨函数边界时怎样先画形状。

## 下一段

[04 设计一个可测试的形状](./04-design-a-testable-shape.md)：代码跨函数边界时，先用 `architect` 画 public surface。
