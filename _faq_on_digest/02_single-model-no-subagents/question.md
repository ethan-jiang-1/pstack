# 只有一个模型、没有子代理，pstack 还能用吗？

> [FAQ 首页](../README.md) · [直接回答](./answer.md)

## 问题要解决什么

pstack 的默认叙事是多模型 role line 和大量 subagent：代码交给快模型，判断交给强模型，`arena`、`interrogate`、`swarm`、`reflect` 都靠 fan-out。但很多 harness 只有一个模型，或者没有子代理工具（例如没有扩展的 Pi），或者用户不想为一次任务开十几个 agent。

这个问题讨论这种条件下哪些能力仍然成立、哪些降级、以及降级后怎样保住证据。不讨论具体模型选型。

## 不在本问题范围内

- role line 怎么配置：见 [04](../04_model-panel-and-budget/answer.md)。
- 哪些 harness 支持什么工具：见 [harness 可移植性](../../_digest/system/harness-portability.md)。
- 没有安装完整 pstack：见 [05](../05_which-skills-to-install/answer.md)。
