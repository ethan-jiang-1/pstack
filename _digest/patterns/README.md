# 机制索引

这一组页面按 pstack 的机制组织，不按 skill 逐条罗列。每页对准一个可以指认的位置：一节 SKILL.md、一份 playbook、一个字段，或设置文件里的一行 role。

| 机制 | 页面 | 源码 |
|---|---|---|
| Playbooks 一节把请求匹配到 23 份步骤协议 | [poteto-mode 的 playbook 路由](./mode-as-router.md) | [`SKILL.md#playbooks`](../../skills/poteto-mode/SKILL.md#playbooks) |
| 24 个 `principle-*` 在回复里被点名 | [principle 的用法](./principles-as-vocabulary.md) | [`SKILL.md#principles`](../../skills/poteto-mode/SKILL.md#principles) |
| role line 与列表长度决定模型和 fan-out | [模型角色](./model-diversity.md) | [`setup-pstack`](../../skills/setup-pstack/SKILL.md) |
| `poteto-agent` 与 `orchestrate` 的 brief 字段 | [Subagent 与 brief](./subagent-topology.md) | [`orchestrate.md#the-brief`](../../skills/poteto-mode/playbooks/orchestrate.md#the-brief) |
| verdict、patch-id 与 decision trail | [验证与 verdict](./evidence-first.md) | [`shipping.md`](../../skills/poteto-mode/playbooks/shipping.md) |
| `unslop`、`technical-writing` 与回复的写法 | [unslop 与 technical-writing](./prose-discipline.md) | [`unslop`](../../skills/unslop/SKILL.md) |
| `correct`、`reflect` 与项目本地的 verification skill | [correct、reflect 与验证 skill](./environment-as-product.md) | [`correct`](../../skills/correct/SKILL.md) |
| Harness 一节的跨 harness 映射 | [poteto-mode 的 Harness](./harness-portability.md) | [`SKILL.md#harness`](../../skills/poteto-mode/SKILL.md#harness) |

## 按问题找

| 你想知道 | 先读 | 接着读 |
|---|---|---|
| 一句请求怎样变成一条可执行、带门的流程？ | [poteto-mode 的 playbook 路由](./mode-as-router.md) | [playbook 索引](../playbooks/README.md) |
| 怎样用一个名字改变 agent 的判断？ | [principle 的用法](./principles-as-vocabulary.md) | [principles 索引](../principles/README.md) |
| 这里的多个模型具体指什么？ | [模型角色](./model-diversity.md) | [role line](../system/model-panel.md) |
| 什么时候派 subagent，怎样写 brief？ | [Subagent 与 brief](./subagent-topology.md) | [调用模型](../system/invocation-model.md) |
| 怎样防止"测试绿了"被当成完成？ | [验证与 verdict](./evidence-first.md) | [自主与门](../system/autonomy-and-gates.md) |
| 回复和 PR 正文为什么也算交付？ | [unslop 与 technical-writing](./prose-discipline.md) | [`unslop` 单页](../skills/unslop.md) |
| 怎样让下一次运行起步更好？ | [correct、reflect 与验证 skill](./environment-as-product.md) | [`reflect` 单页](../skills/reflect.md) |
| 换 harness 时哪些东西会坏？ | [poteto-mode 的 Harness](./harness-portability.md) | [分发与安装](../system/distribution.md) |

## 读这些页时的约定

- 每个机制段落末尾给源码位置。事实以源码为准，页面只解释为什么这样拆。
- 页面里的名字只有 skill、playbook、字段、role 和状态，没有为方便叙述另起的名字。
- 计数会变。skill 数、playbook 数、模型 slug 以 `plugin.json` 和源码为准，这里不重复维护。

## 相关入口

[系统总览](../system/README.md)、[技能索引](../skills/README.md)、[playbook 索引](../playbooks/README.md)。要把一个机制放回完整流程，先读 [mode 与 playbook](../system/mode-and-playbooks.md)。
