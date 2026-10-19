# 调用模型

## 一句话

pstack 的 skill 几乎都只能由人触发。截至 plugin `0.15.25`（2026-10-19），54 个 skill 里有 53 个的 frontmatter 写了 `disable-model-invocation: true`，只有 `setup-pstack` 省略了它。模型不会自己发现并调用 skill；路由由人输入 `/poteto-mode` 或单点命令，或 mode 在 playbook 步骤里点名后读对应的 `SKILL.md`。

## 两种身份

| | user-invoked（默认） | model-invoked（例外） |
|---|---|---|
| 谁能触发 | 只有人 | 人或模型 |
| frontmatter | `disable-model-invocation: true` | 省略该字段 |
| description 的读者 | 浏览菜单或文档的人 | 判断是否调用的模型 |
| 模型可见成本（按字段语义） | 不进入隐式调用索引 | 参与 skill 发现 |
| 能否被其他 skill 隐式触发 | 不能 | 能 |
| pstack 中的数量 | 53 / 54 | 1 / 54（`setup-pstack`） |

pstack 的 `description` 仍写得像触发条件（"Use for /architect, 'architect this'…"），但它的主要读者是人：帮人在菜单里认出它，或让 mode 在正文里点名时有个稳定的名字。

## 这样做的结果

1. **成本可预测。** 一次 pstack 运行会花真实 token，还会派 subagent。让模型自行决定何时启动一整套流程，等于把成本决定权交出去。
2. **顺序可以被步骤写死。** Bug fix 必须先复现，Shipping 必须有独立 verdict。隐式调用无法保证顺序；显式点名可以。
3. **入口可以很少。** 人只需要记住 `/poteto-mode` 和少数单点工具，其余由 mode 在步骤里带出来。
4. **误触发少。** `tdd` 的 description 明确说只在用户要求 TDD 或 bug 有便宜的本地测试目标时用。模型若能自行调用，这条边界很难守住。

代价是人的记忆负担：不跑 mode 时，得知道 `how`、`why`、`arena` 这些名字。`poteto-help` 和 `docs/guide/` 在补这部分。

## 跨 skill 调用怎样发生

pstack 不用调用名做符号引用，而是用正文点名加物理路径：

- mode 的 **Non-negotiables** 写触发条件："Nontrivial change → the **how** skill"。
- playbook 的步骤写 "Seed them with `how`"、"`architect` first"。
- 一些正文直接给路径：`../how/SKILL.md`、`../principle-prove-it-works/SKILL.md`。

执行者的实际动作通常是：读到点名，打开同仓的 `SKILL.md`，按里面的协议做。具体走 Skill 工具、斜杠命令还是文件读取，取决于 harness。

两个后果：

- **同仓布局是契约。** `skills/<name>/` 的相对位置不能随便改；部分安装容易断链。
- **名字稳定，路径脆弱。** 读者可以记住 `how`，但安装器必须保证 `../how/SKILL.md` 真的存在。

## `setup-pstack` 为什么是例外

它没有写 `disable-model-invocation`。从行为看，这说得通：配置模型是低频、可逆、经常在"第一次用"时被建议的动作。但源码没有解释这个例外，所以这里只把它记为观察，不当作设计意图。

## 跨 harness 的差异

仓库没有说明 `disable-model-invocation` 在各 harness 里的执行方式；能确认的只有 53 个 skill 写了它。以下是常见 harness 的一般性观察，不是仓库的承诺：

- 在 Claude Code 里，模型无法通过 Skill 工具调用这些 skill，"只能人来触发"是硬约束。
- 在 Cursor 里，用户通过 `/` 菜单附着 skill；"不能隐式调用"更多是产品行为，不是同一条代码路径。
- 在没有 Skill 工具的 harness 里，正文点名会退化成读文件，约束力来自指令遵循，不是访问控制。

这也是 [harness 可移植性](./harness-portability.md) 要单独写一页的原因：调用语义不是一个跨 harness 常量。

## 关键洞察

1. 调用身份是成本开关，不只是分类标签。53 个 skill 设成 user-invoked，等于把"什么时候花钱"留在人手里。
2. user-invoked 不等于零上下文。skill 被触发后正文照样加载；省掉的是 description 常驻模型索引的那部分。
3. 路由需要替代品。关掉隐式发现，就必须给一个 `poteto-mode` 这样的单一入口，否则人的记忆负担会失控。
4. 物理路径引用是可移植性的债。单仓安装时最省事，拆装、换 harness、改目录时最脆弱。

## 源码

- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Non-negotiables、Playbooks）
- [`skills/setup-pstack/SKILL.md`](../../skills/setup-pstack/SKILL.md)
- [`skills/tdd/SKILL.md`](../../skills/tdd/SKILL.md)
- [`docs/guide/02-poteto-mode.md`](../../docs/guide/02-poteto-mode.md)
