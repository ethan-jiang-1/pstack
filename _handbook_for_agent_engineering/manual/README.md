# pstack 手册目录

手册按 pstack 的机制组织。不要顺读；从你卡住的地方进入，拿到一个 skill、playbook 或字段，再回到工作。

> 状态：严格重写完成（2026-10-19）。只保留 pstack 的 skill、playbook、字段、角色与状态；没有 pstack 依据的章节不建。结构见 [PLAN.md](../PLAN.md)，术语见 [CONTEXT.md](../CONTEXT.md)。

## 按卡点进入

| 你现在的感觉 | 先读 | 你会拿到 |
|---|---|---|
| 请求说不清，不知道 prompt 写什么 | [1.1 prompt 的五项](./define.md) | 一条能进 mode 的 prompt |
| 不知道用 mode 还是单 skill | [1.2 mode、playbook 与 todo](./define.md) | 入口选择、todo 与 skip |
| 不知道该不该问人 | [1.3 AskQuestion 分类](./define.md) | 问 / 不问的判断 |
| 写代码前不知道 data shape | [2.1 先给数据一个形状](./design-and-plan.md) | 输入、输出、失败出口 |
| 代码跨函数边界，不知道接口怎么定 | [2.2 architect Phase B](./design-and-plan.md) | public surface 与 `not implemented` 草图 |
| 工作太大，不知道怎样切 | [2.3 throughput checkpoint 与 plan](./design-and-plan.md) | `feature` step 3 与 `multi-phase-plan` |
| 派了 subagent，回来失控 | [3.1 Subagents 默认值](./delegate-and-collaborate.md) | 默认值与父 agent 复核 |
| brief 写完还是做偏 | [3.2 brief 九字段](./delegate-and-collaborate.md) | GOAL / … / REPORT / STANDING |
| 不知道 fresh 还是 resume | [3.3 fresh、resume 与角色](./delegate-and-collaborate.md) | 判断与 role line |
| 多个 agent 跑多天 program | [3.4 orchestrate roles](./delegate-and-collaborate.md) | coordinator / worker / verifier 与单一 stacker |
| 测试绿了但不确定兑现 | [4.1 tdd](./verify.md) / [4.2 prove-it-works](./verify.md) | 红绿检查或真实表面证据 |
| 性能数字没有方法 | [4.3 benchmark-checklist](./verify.md) | 七问与 ballpark 例外 |
| 绿 CI 被当成可以合 | [4.4 verdict 与 patch-id](./verify.md) | 非作者 verdict 与 contiguous run |
| 评审要么挑刺要么放行 | [5.1 interrogate](./ship-run-and-evolve.md) | 四桶结论 |
| 注释噪音混进 diff | [5.2 no-comments](./ship-run-and-evolve.md) | 删除与裁决 |
| PR 卡在 merge-ready 之前 | [5.3 shipping](./ship-run-and-evolve.md) / [5.4 babysit](./ship-run-and-evolve.md) | 落地顺序与模式 |
| 同一个错误反复出现 | [6.1 correct](./evolve-the-environment.md) | 一条会失败的检查 |
| 会话经验留不下来 | [6.2 reflect](./evolve-the-environment.md) | Accepted / Rejected / Backlog |
| 能编码的规则还停留在文字 | [6.3 encode-lessons-in-structure](./evolve-the-environment.md) | 结构约束或验证 skill |

## 六章

### [1 定义：prompt、mode 与 todo](./define.md)

1.1 prompt 的五项。
1.2 mode、playbook 与 todo。
1.3 AskQuestion 分类。
1.4 brief；pstack 不写 spec。

### [2 设计与计划：data shape、architect、feature](./design-and-plan.md)

2.1 先给数据一个形状。
2.2 `architect` Phase B。
2.3 `feature` step 3 与 `multi-phase-plan`。

### [3 委派：Subagent 与 brief](./delegate-and-collaborate.md)

3.1 Subagents 默认值。
3.2 brief 九字段。
3.3 fresh、resume 与角色。
3.4 `orchestrate` 的 roles、one writer 与单一 stacker。

### [4 验证与 verdict](./verify.md)

4.1 `tdd`。
4.2 `prove-it-works`。
4.3 `benchmark-checklist`。
4.4 verdict、patch-id 与 contiguous verified run。

### [5 评审与落地](./ship-run-and-evolve.md)

5.1 `interrogate`。
5.2 `no-comments`。
5.3 `shipping`。
5.4 `babysit`、`session pickup`、`pause safely`。

### [6 环境：`correct` 与 `reflect`](./evolve-the-environment.md)

6.1 `correct`。
6.2 `reflect`。
6.3 `encode-lessons-in-structure` 与验证 skill。

## 怎么读

1. 先看卡点表，找到最接近的一行。
2. 读对应单元，拿走 skill 名、字段或判断规则。
3. 只有需要完整机制、例外或源码位置时，再读同章其他单元。
4. 教程是另一条读法，见 [教程地图](../tutorial/README.md)。
