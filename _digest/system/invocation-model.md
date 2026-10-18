# 调用模型

## 一句话

pstack 做了一个很彻底的选择：**几乎全部 skill 都只能由人显式触发**。54 个 skill 里有 53 个写了 `disable-model-invocation: true`，唯一的例外是 `setup-pstack`。模型不会自己发现并调用 skill；路由由人输入 `/poteto-mode`，或在 playbook 里被点名后读取对应的 `SKILL.md`。

## 两种身份

| | user-invoked（pstack 的默认） | model-invoked（pstack 的例外） |
|---|---|---|
| 谁能触发 | 只有人 | 人或模型 |
| 前端写法 | `disable-model-invocation: true` | 省略该字段 |
| description 的读者 | 浏览 `/` 菜单或文档的人 | 判断是否调用的模型 |
| 持续的模型可见成本 | 不进入模型的隐式调用索引 | 参与模型的 skill 发现 |
| 能否被其他 skill 隐式触发 | 不能 | 能 |
| pstack 中的数量 | 53 / 54 | 1 / 54（`setup-pstack`） |

在 pstack 里，`description` 仍然写得很像触发条件（"Use for /architect, 'architect this'…"），但它的主要读者是人：帮你在菜单里认出它，或者让 mode 在正文里点名时有个稳定的名字。

## 为什么这样选

1. **成本可预测。** 一次 pstack 运行会花真实 token，还会派 subagent。让模型自行决定何时启动一整套流程，等于把成本决定权交出去。
2. **顺序是产品的一部分。** bug fix 必须先复现，shipping 必须有独立 verdict。隐式调用无法保证顺序；显式点名可以。
3. **入口可以很少。** 用户只需要记住 `/poteto-mode` 和少数单点工具，其余由 mode 在步骤里带出来。
4. **避免误触发。** `tdd` 的 description 明确说"只在用户要求 TDD 或 bug 有便宜的本地测试目标时用"。如果模型可以自行调用，这条边界很难守住。

代价是人的记忆负担：不跑 mode 时，你得知道 `how`、`why`、`arena` 这些名字。`poteto-help` 和 `docs/guide/` 就是在补这部分。

## 跨 skill 调用怎样发生

pstack 不用"调用名"做符号引用，而是用**正文点名 + 物理路径**：

- mode 的 **Non-negotiables** 列出触发条件："Nontrivial change → the **how** skill"。
- playbook 的步骤写"Seed them with `how`"、"`architect` first"。
- 一些正文直接给出路径：`../how/SKILL.md`、`../principle-prove-it-works/SKILL.md`。

因此执行者的实际动作通常是：读到点名，打开同仓的 `SKILL.md`，按里面的协议做。具体是走 Skill 工具、斜杠命令还是文件读取，取决于 harness。

这带来两个后果：

- **同仓布局是契约。** `skills/<name>/` 的相对位置不能随便改；这也是 [分发与安装](./distribution.md) 里说部分安装容易断链的原因。
- **名字是稳定的，路径是脆弱的。** 读者可以记住 `how`，但安装器必须保证 `../how/SKILL.md` 真的存在。

作为对比，Matt Pocock 的体系用 model-invoked engine 加 thin wrapper：共享协议允许被其他 skill 触发，人类入口只是一层薄壳。pstack 把整个目录都设成 user-invoked，用一个 mode 承担路由，用 playbook 承担顺序。两种选择都自洽，但优化目标不同：前者优化复用与组合，后者优化可预测与成本。

## `setup-pstack` 为什么是例外

它没有写 `disable-model-invocation`。从行为看，这说得通：配置模型是低频、可逆、且经常在"第一次用"时被建议的动作，允许模型提出或执行它风险不高。但源码没有解释这个例外，所以这里只把它记为观察，不当作设计意图。

## 跨 harness 的差异

`disable-model-invocation` 是 Claude Code 认的 frontmatter 字段。Cursor、Codex、Pi 等 harness 是否强制执行、以什么方式强制执行，并不统一。因此：

- 在 Claude Code 里，模型无法通过 Skill 工具调用这些 skill，"只能人来触发"是硬约束。
- 在 Cursor 里，用户通过 `/` 菜单附着 skill；"不能隐式调用"更多是产品行为，不是同一条代码路径。
- 在没有 Skill 工具的 harness 里，正文点名会退化成"读文件"，约束力来自指令遵循，不是访问控制。

这也是为什么 [harness 可移植性](./harness-portability.md) 要单独写一页：调用语义不是一个跨 harness 常量。

## 关键洞察

1. **调用身份是成本开关，不只是分类标签。** 把 53 个 skill 设成 user-invoked，等于把"什么时候花钱"留在人手里。
2. **user-invoked 不等于零上下文。** skill 被触发后正文照样加载；省掉的是 description 常驻模型索引的那部分。
3. **路由需要替代品。** 关掉隐式发现，就必须给一个 `poteto-mode` 这样的单一入口，否则人的记忆负担会失控。
4. **物理路径引用是可移植性的债。** 它在单仓安装时最省事，在拆装、换 harness、改目录时最脆弱。
