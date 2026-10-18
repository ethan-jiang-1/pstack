# Change Log / 上游同步记录

这个目录记录每次从上游同步 pstack 到本仓库的变更摘要，以及这些变更对消化层的影响。

## 与当前解读的边界

`_change_log/` 回答"从什么变成什么、为什么需要改解读"；[`_digest/system/`](../system/README.md)、[`_digest/patterns/`](../patterns/README.md)、[`_digest/skills/`](../skills/README.md) 只描述当前行为与当前结构。

版本对比、旧行为、同步批次和"新增/删除/改写"等时间性解释写在这里。同步完成后，当前解读页要改写成不依赖历史背景也能成立的说明。

## 为什么需要这个

pstack 以上游 PR 与版本号的形式持续演进：新增 skill、重写 playbook、改模型默认值、调整 harness 映射。这些变更会让 `_digest` 的解释过时，也会让 FAQ 的结论失效。没有一份同步记录，就不知道哪些页面需要复查。

## 每条记录包含

- 同步窗口（日期 + 上游版本号 + commit 跨度）
- 核心主题
- 按文件域的变更拆解（skills / playbooks / agents / docs / 配置）
- 对 `_digest` 的影响评估（🟢🟡🔴）
- 对 `_faq_on_digest` 的影响评估
- 对 `_handbook_for_agent_engineering` 的影响评估
- 优先级建议

## 命名约定

```text
NNNN-{descriptor}.md
```

- `NNNN`：递增序号，4 位补零。
- `descriptor`：简短、kebab-case，描述这次同步的主题或版本。
- 补充的执行计划用下划线前缀，例如 `_plan-N-{描述}.md`；只有影响面很大时才单独建文件。

## 目录结构

```text
_change_log/
├── README.md
├── _TEMPLATE.md
├── 0001-baseline.md     # 消化层建立时的上游基线
└── 后续每次同步按编号递增
```

## 每次同步的流程

1. 更新同步线：

   ```bash
   git switch main
   git pull --ff-only origin main
   ```

2. 记录旧位置，方便写跨度：

   ```bash
   git log --oneline -1 main
   git diff --stat <上次同步的短 SHA>..main
   ```

3. 并入消化层：

   ```bash
   git switch ethan
   git merge main
   ```

4. 读上游 diff，按 [`_TEMPLATE.md`](./_TEMPLATE.md) 写新一条记录，编号递增。

5. 按影响矩阵更新 `_digest/`、`_faq_on_digest/`、`_handbook_for_agent_engineering/`。先改索引与路由页，再改单页。

6. 一致性检查：相对链接、skill/playbook/principle 计数、`plugin.json` 版本、`README.md` 依赖表。

7. 提交并推送：

   ```bash
   git push origin main ethan
   ```

## 写作要求

- 变更记录可以引用 PR、commit、版本号；当前解读页不要依赖这些。
- 影响矩阵里每个 🔴/🟡 都要写"需要改什么"，不能只标颜色。
- 不涉及也要写"核对过哪里"，避免下次重复判断。
- 删除或重命名的 skill，要在影响矩阵里单独列一行，并检查所有 `../<name>/SKILL.md` 引用。
