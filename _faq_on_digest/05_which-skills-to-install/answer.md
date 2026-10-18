# 可以只装几个，但先决定你要哪一层能力

> [FAQ 首页](../README.md) · [问题原文](./question.md) · [分发与安装](../../_digest/system/distribution.md)

## 直接回答

可以。最小可用集合取决于你要的是**写作纪律**、**理解与解释**，还是**完整 mode 流程**。前两层可以只装几个 skill；要跑 `poteto-mode`，就得接受它依赖大多数 skill 和全部 `principle-*`。

## 先选一层

| 你想要的 | 最小集合 | 说明 |
|---|---|---|
| 只清理英文、写 PR/文档 | `unslop`、`technical-writing`、`bro`、`ketchup` | `technical-writing` 依赖 `unslop`；`ketchup` 依赖 `bro` |
| 只理解代码与历史 | `how`、`why`、`teach`、`recall`、`blast-radius` | `teach` 依赖 `how` + `why`；`blast-radius` 还依赖 `arena` 和 `unslop` |
| 只做设计与对抗评审 | `architect`、`arena`、`interrogate` | `architect` 依赖 `arena` + `how` |
| 只做验证与交付 | `create-verification-skill`、`maintain-verification-skill`、`benchmark-checklist`、`show-me-your-work` | 两个验证 skill 成对使用 |
| 完整 mode 流程 | `poteto-mode` + 全部 `principle-*` + 大多数其他 skill | playbook 会在步骤里点名它们 |

## 依赖不是猜的

判断一个 skill 能不能单装，最可靠的办法是读它的正文，搜索：

- `../<name>/SKILL.md` 形式的路径；
- "the **\<name\>** skill" 形式的点名；
- "Read `references/...`" 形式的同目录依赖。

README 的依赖表是入口，但它是人工维护的，可能滞后。源码里的引用才是当前事实。

## 部分安装会在哪里断

1. **物理路径引用。** `../how/SKILL.md` 在目录被拆开移动后失效。
2. **`poteto-agent` 未注册。** 非 Cursor harness 需要按 Harness 映射用通用 subagent 替代。
3. **模型角色缺失。** 设置文件缺行时退回 skill 默认值；默认 slug 不存在时再降级。
4. **playbook 指向的 skill 不在场。** mode 能复制步骤，却无法执行那一步；正确行为是报告缺失并走 `skip`，不是自己发明。

只想试一个能力时，直接用单点 skill 比装一个残缺的 mode 更稳。

## 装到哪里

| Harness | 项目级 | 用户级 |
|---|---|---|
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Pi | `.pi/skills/` | `~/.pi/agent/skills/` |
| Codex / OpenCode / 其他 | `.agents/skills/` | `~/.agents/skills/` |

模型设置文件是另一条路径，见 [04](../04_model-panel-and-budget/answer.md)。

## 推荐路径

1. 先装一个写作包或理解包，跑一周，确认它真的被用上。
2. 需要多步流程时，再装完整 mode；不要一开始就全装，否则 23 个 playbook 的选择成本会先于收益出现。
3. 装 mode 时把全部 `principle-*` 一起装，否则 mode 的 Principles 索引会指向空文件。

## 继续阅读

- [当前行为的源码](../../skills/poteto-mode/SKILL.md)（**Playbooks** 与 **Harness**）
- [分发与安装：粒度、依赖与断点](../../_digest/system/distribution.md)
- [技能索引](../../_digest/skills/README.md)
