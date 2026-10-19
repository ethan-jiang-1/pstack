# 教程之后：按下一道坎找手册

> 教程终点 · 上一页：[11 用 correct 与 reflect 改环境](./11-improve-the-environment.md) · [教程地图](./README.md) · [手册目录](../manual/README.md)

教程走完了一条 pstack 路径：prompt → mode/playbook → todo → data shape → `architect` → 切片 → `tdd` → brief → verdict → `interrogate` → `shipping` → `correct` / `reflect`。

真实工作不会每次都从头走一遍。你更可能卡在某一处，需要一个字段、一条规则或一个源码位置。手册按机制组织，不按教程顺序。

## 六种常见的下一道坎

| 你现在的状态 | 去哪里 |
|---|---|
| prompt 写不出来，或 done check 说不清 | [1.1 prompt 的五项](../manual/define.md) |
| 不知道该用 mode 还是单 skill | [1.2 mode 与 playbook](../manual/define.md) |
| 派了 subagent，回来失控 | [3.1 Subagents 默认值](../manual/delegate-and-collaborate.md) |
| 测试绿了，但不确定真实路径能用 | [4.2 prove-it-works](../manual/verify.md) |
| 评审要么挑刺要么放行 | [5.1 interrogate](../manual/ship-run-and-evolve.md) |
| 同一个错误反复出现 | [6.1 correct](../manual/evolve-the-environment.md) |

## 三条使用建议

1. 先读卡点表，不要顺读六章。
2. 拿到 skill 名、playbook 名或字段名后，回到源码确认；手册只解释，行为以 `SKILL.md` 为准。
3. 如果手册里没有你要的话题，先看它是不是 pstack 不覆盖：spec、发布、回滚、发布后监控都不在 pstack 里。

## 继续阅读

- [手册目录](../manual/README.md)
- [pstack 术语表](../CONTEXT.md)
- [源码映射](../PROVENANCE.md)
