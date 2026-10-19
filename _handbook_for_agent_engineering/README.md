# pstack 手册

这是一份讲 pstack 自身概念的手册：`poteto-mode` 怎样把请求匹配到 playbook，brief 怎样写，verdict 怎样产生，`shipping` 在哪里停下，`correct` 与 `reflect` 怎样改环境。它不发明 pstack 之外的方法名，也不把外部方法映射成 pstack 的机制。

> **状态：严格重写完成（2026-10-19），spot REVIEW 的 4 条 findings 已修正。** 教程与手册只保留 pstack 的 skill、playbook、字段、角色与状态；没有 pstack 依据的章节已删除，32 张依赖自造标签的图已删除。结构见 [PLAN.md](./PLAN.md)，术语见 [CONTEXT.md](./CONTEXT.md)。

## 两种读法

| 你现在的状态 | 从这里开始 | 你会得到什么 |
|---|---|---|
| 第一次用 pstack，想知道一次任务怎样从 prompt 走到落地 | [教程](./tutorial/README.md) | 一条连续路径：prompt 五项 → mode/playbook → todo → architect/tdd → brief → verdict → shipping → correct/reflect |
| 已经用过，想查某个机制 | [手册](./manual/README.md) | 按机制组织的源码解释与操作边界 |

## 教程

教程从 [00 用 prompt 说清一件事](./tutorial/00-start-here.md) 走到 `11`，每页只处理一个 pstack 机制，并给出来源：

- `00–04`：prompt 五项、mode 与单 skill、playbook 步骤进 todo、named data shape、`architect` 的 public surface。
- `05–08`：`feature` step 3 / `multi-phase-plan`、`tdd`、brief 与 subagent、`prove-it-works` 与 verdict。
- `09–11`：`interrogate`、`shipping`、`correct` / `reflect`。

## 手册

手册按 pstack 的机制组织，不按"第二课程"组织：

1. Prompt、mode 与 todo
2. `architect`、`feature` 与 `multi-phase-plan`
3. Subagent 与 brief
4. 验证与 verdict
5. 评审与落地
6. 环境：`correct`、`reflect`、`encode-lessons-in-structure`

## 范围

**涵盖**：pstack 的 mode、playbook、skill、principle、subagent、brief、verdict、patch-id、contiguous verified run、feature map、decision trail、session pickup、pause safely、shipping、correct、reflect。

**不涵盖**：pstack 没有的 spec 生命周期、发布 / 灰度 / 回滚 / 发布后监控、独立的多人协作协议。pstack 停在 verified landing；部署只是 **Always pause** 里的一个不可逆动作。

## 约定

- pstack 的 skill、playbook、字段、角色、状态名保留原写法，不翻译成新名字。
- 中文只作解释；不出现"我们把 X 叫作 Y"的构造。
- 每个具体断言给源码位置：`skills/<name>/SKILL.md`、`skills/poteto-mode/playbooks/<name>.md`、`agents/<name>.md`、`docs/guide/<n>.md`。
- 写不出 pstack 出处的段落，删除或改成一句"pstack 不覆盖这个"。

术语表：[`CONTEXT.md`](./CONTEXT.md) · 写作约束：[`STYLE.md`](./STYLE.md) · 结构计划：[`PLAN.md`](./PLAN.md) · 源码映射：[`PROVENANCE.md`](./PROVENANCE.md)。
