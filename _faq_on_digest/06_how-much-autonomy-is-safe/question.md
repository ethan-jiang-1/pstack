# 全自动、自主运行安全吗？该怎样审？

> [FAQ 首页](../README.md) · [直接回答](./answer.md)

## 问题要解决什么

pstack 支持从 `Autonomous run` 到 `Orchestrate`、`Autopilot-full`、`Autopilot-stack` 的一系列自主形态。它们能在人离开时继续工作，也能合并或落地 PR。对第一次用的人来说，问题不是"能不能开"，而是"授权到什么程度、留下什么证据、出问题怎样发现"。

这个问题讨论授权边界与审计方式。不讨论某个 playbook 的逐步操作，也不讨论具体 forge 命令。

## 不在本问题范围内

- 每个自主 playbook 的完整步骤：见 [playbook 索引](../../_digest/playbooks/README.md)。
- 决策轨迹的字段细节：见 [`show-me-your-work`](../../skills/show-me-your-work/SKILL.md)。
- 模型与成本：见 [04](../04_model-panel-and-budget/answer.md)。
