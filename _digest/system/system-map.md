# 系统页索引

## 一句话

这个文件只是 `_digest/system/` 的页面索引，不是 pstack 的机制、role 或 playbook。pstack 的主入口仍然是 `poteto-mode`：它匹配一个 playbook，把步骤逐字复制进 todo，再按步骤调用 skill 或派 subagent。

## 页面

| 页面 | 回答什么 | 源码起点 |
|---|---|---|
| [系统总览](./README.md) | 一次运行怎样从请求走到验证与收口 | `skills/poteto-mode/SKILL.md` |
| [mode 与 playbook](./mode-and-playbooks.md) | 请求怎样匹配到 23 个 playbook 之一，gate 写在哪 | `skills/poteto-mode/SKILL.md#Playbooks`、`playbooks/` |
| [调用模型](./invocation-model.md) | 谁可以触发 skill，53 / 54 为何只由人触发 | `skills/*/SKILL.md` 的 frontmatter |
| [SKILL.md 结构](./skill-anatomy.md) | 一个 skill 目录里有什么，references 与 scripts 怎样用 | `skills/`、`agents/` |
| [role line 与模型设置](./model-panel.md) | `setup-pstack` 写什么，subagent 怎样选模型 | `skills/setup-pstack/SKILL.md` |
| [harness 可移植性](./harness-portability.md) | 换 harness 要替换哪些工具、路径与插件 | `skills/poteto-mode/SKILL.md#Harness` |
| [Autonomy 与 gate](./autonomy-and-gates.md) | 什么时候直接做、问人、停下，decision trail 记什么 | `skills/poteto-mode/SKILL.md#Autonomy` |
| [分发与安装](./distribution.md) | pstack 装到哪里，部分安装会断在哪 | `README.md`、`plugin.json`、`MIRROR.md` |

## 主流程

`poteto-mode` → 匹配一个 playbook → 把步骤逐字复制进 todo → 按步骤调用 `how`、`why`、`architect`、`tdd` 等 skill，或按 role line 派 subagent → 在真实产物上验证 → 需要时用 `Opening a PR` 收尾。`shipping` 的独立 verdict 与 `show-me-your-work` 的 decision trail 决定能不能停。

## 单点入口

`how`、`why`、`teach`、`recall`、`ketchup`、`sharpen`、`correct`、`reflect`、`automate-me`、`poteto-help` 不需要先跑 mode。`Session pickup` 是 playbook，经 mode 进入。`how` 与 `why` 会被 playbook 步骤点名；`ketchup` 出现在 `Autopilot-full` 与 `Autopilot-stack` 的 Reply 里。

23 个 playbook 的逐项入口见 [playbook 索引](../playbooks/README.md)；24 个 `principle-*` 见 [principles 索引](../principles/README.md)；54 个 skill 见 [技能索引](../skills/README.md)。

## 源码

- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)
- [`skills/poteto-mode/playbooks/`](../../skills/poteto-mode/playbooks/)
- [`skills/setup-pstack/SKILL.md`](../../skills/setup-pstack/SKILL.md)
- [`plugin.json`](../../plugin.json)
