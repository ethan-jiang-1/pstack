# 维护约定

这份约定管的是本仓库的**消化层**：`_digest/`、`_faq_on_digest/`、`_handbook_for_agent_engineering/`。它不描述 pstack 本身的行为，只描述这些解释材料怎样与上游同步、怎样保持不自相矛盾。

## 一句话

源码说当前行为，digest 说结构与理由，FAQ 说现在怎么做，handbook 说 pstack 的概念与用法，change log 说历史。五者不能互相冒充。

## 五层边界

| 位置 | 回答什么 | 事实来源 | 允许写时间性内容 |
|---|---|---|---|
| `skills/**/SKILL.md`、`skills/poteto-mode/playbooks/`、`agents/`、`docs/guide/` | 今天实际会怎样 | 它们自己 | 只在自身的 CHANGELOG/历史里 |
| [`_digest/`](./README.md) | 系统怎样组织、为什么这样拆 | 以上源码 | 否，只写当前态 |
| [`_faq_on_digest/`](../_faq_on_digest/README.md) | 我此刻该怎么做 | 结论对齐源码 | 否，只写当前态 |
| [`_handbook_for_agent_engineering/`](../_handbook_for_agent_engineering/README.md) | pstack 的概念与用法 | pstack 源码，逐页映射 | 否，只写当前态 |
| [`_change_log/`](./_change_log/README.md) | 从什么变成什么、为什么 | 上游 diff 与 PR | 是，历史只在这里 |

一条硬规则：**digest 与源码冲突时，改 digest，不改源码。** 如果源码本身可疑，在 `_change_log` 或 FAQ 里记为观察，不要把它写成"设计意图"。

## 分支拓扑

```text
origin/main          上游镜像线：Cursor 的 pstack 内容 + 镜像侧编辑
        │
        ├── main        本仓库的同步线，跟随 origin/main
        │
        └── ethan       main + 本目录（digest / FAQ / handbook / 图）
```

- `main` 上不写消化层。它要能随时快进或合并到上游的新版本。
- `ethan` 上不直接抄上游文件。上游更新一律先落到 `main`，再 merge 进 `ethan`。
- 冲突的含义是：上游改了本地也改过的同一行。保留上游的新语义，再按本仓库的措辞重放本地编辑。

## 每次同步的流程

1. 更新同步线。

   ```bash
   git switch main
   git pull --ff-only origin main
   ```

2. 看清这次变了什么。

   ```bash
   git log --oneline <上次同步的短 SHA>..main
   git diff --stat <上次同步的短 SHA>..main
   ```

   重点看 `skills/**/SKILL.md`、`skills/poteto-mode/playbooks/`、`docs/guide/`、`plugin.json` 的版本号。新增或删除 skill、改 frontmatter、改 playbook 步骤，都会影响解读。另外按 `MIRROR.md` 第 6 步刷新 `skills/no-comments/references/comment-sicko.md`，并扫描上游 diff 里新增的 Cursor-only 指令。

3. 并入消化层。

   ```bash
   git switch ethan
   git merge main
   ```

4. 写一条变更记录。复制 [`_change_log/_TEMPLATE.md`](./_change_log/_TEMPLATE.md)，编号递增，填三张影响矩阵：`_digest`、`_faq_on_digest`、`_handbook_*`。

5. 按影响矩阵改页面。先改索引（`skills/README.md`、`playbooks/README.md`、`principles/README.md`）和路由相关页（`system/mode-and-playbooks.md`），再改单页。

6. 做一致性检查：所有相对链接能解析；计数（skill 数、playbook 数、principle 数）与 `plugin.json` 版本一起看；`README.md` 的依赖表和 `_digest/system/distribution.md` 不互相矛盾。

7. 提交并推送两条线。

   ```bash
   git push origin main ethan
   ```

## 页面契约

### 系统页与机制页

固定回答四件事，顺序可以变：

1. **一句话**：这一页的核心判断。
2. **机制**：它实际怎样运转，尽量引用源码位置。
3. **边界**：什么时候不成立、代价是什么、容易误读什么。
4. **关键洞察**：这一节最容易被忽略的源码事实。

### 单 skill 页

```markdown
# `skill-name`

> 源码：[`skills/skill-name/SKILL.md`](../../skills/skill-name/SKILL.md)
> 调用：**U**（只有人能触发）｜**M**（模型也能触发）
> 索引：[技能索引](./README.md)

## 一句话
## 它解决什么
## 核心机制
## 设计意图
## 依赖关系
## 使用边界
## 关键洞察
```

`值得注意` 一节只在确有代价时添加，例如 Cursor 绑定、依赖物理路径、与其他 skill 重叠。不要为了显得客观而每页都加。

### FAQ 页

沿用 [`_faq_on_digest/_TEMPLATE.md`](../_faq_on_digest/_TEMPLATE.md)：`question.md` 记录范围与前提，`answer.md` 先给可执行结论，再给条件、例外和下一步。固定标题只有"直接回答"和"继续阅读"。

## 写作约定

- 正文以中文为主。关键术语第一次出现写作 `English（中文）`，例如 `verdict（结论）`；同页重复不再加注。pstack 自带的 skill、playbook、角色名和命令保留原写法，不加翻译。
- 允许批评，但批评必须指向具体机制或代价，并给出证据位置。不写"感觉""可能不太好"。
- 不复制 pstack 的行为规则。解释"为什么这样定"，规则本身让读者去读 `SKILL.md`。
- 不把会变的东西写成永恒事实：skill 数、模型 slug、默认预算、上游版本都要带日期或版本号。
- 引用来源优先用相对链接指向源码文件；行号会漂移，页码内用小节名定位。

## 不变量

1. 每个解释页都有指回源码的链接。
2. 历史只写在 `_change_log`；当前页改写后要能脱离历史独立成立。
3. FAQ 与 digest 不互相抄结论：FAQ 负责决策，digest 负责机制。
4. handbook 只讲 pstack 自身的概念，不引入外部方法。
5. 新增或删除非 principle 的 pstack skill 时，三处必须同时改：`_digest/skills/README.md`、对应单页、`_change_log` 的最新记录；`principle-*` 没有单页，改 `_digest/principles/README.md` 和 change-log 计数。
6. 所有相对链接在提交前实际打开过。
