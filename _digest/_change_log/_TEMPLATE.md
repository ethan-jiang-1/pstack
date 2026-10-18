# Sync NNNN: {descriptor}

> 复制本文件为新一条变更记录，替换所有占位符。编号递增。

**日期**: YYYY-MM-DD

**上游版本**: `x.y.z`（`plugin.json`）

**同步窗口**: `{start_hash_short}` → `{end_hash_short}`（共 N 个提交）

**本地合入**: `main` 跟随 `origin/main`，再 `git merge main` 到 `ethan`

## 核心主题

{一段话概括这一批上游变更的性质。}

## 按文件域拆解

### skills/ — 具体 skill 变更

| Skill | 变更 | 影响 | 意义 |
|---|---|---|---|
| `{skill-name}` | 新增 / 删除 / 改写 / frontmatter 变化 | {行为上有什么不同} | {为什么对解读重要} |

### playbooks/ — 流程变更

| Playbook | 变更 | 意义 |
|---|---|---|
| `{title}` | {步骤、门、失败出口的变化} | {路由或执行会怎样不同} |

### agents / docs / 配置

| 维度 | 变更 | 意义 |
|---|---|---|
| `agents/` | {poteto-agent、comment-sicko 的变化} | {子代理行为} |
| `docs/guide/` | {新增或改写的页面} | {操作教程是否仍与 digest 一致} |
| `plugin.json` / `.cursor-plugin/plugin.json` | {版本号、description} | {发布信号} |

## 对 `_digest/` 的影响

| 子目录 / 文件 | 影响程度 | 说明 |
|---|---|---|
| `system/{file}.md` | 🟢 / 🟡 / 🔴 | {具体影响} |
| `patterns/{file}.md` | 🟢 / 🟡 / 🔴 | {具体影响} |
| `principles/README.md` | 🟢 / 🟡 / 🔴 | {新增/删除/改名的 principle} |
| `playbooks/README.md` | 🟢 / 🟡 / 🔴 | {新增/删除/改名的 playbook} |
| `skills/{name}.md` | 🟢 / 🟡 / 🔴 | {具体影响} |
| `skills/README.md` | 🟢 / 🟡 / 🔴 | {索引是否要改} |

## 对 `_faq_on_digest/` 的影响

| FAQ 条目 | 影响程度 | 说明 |
|---|---|---|
| `{NN}-{slug}/` | 🟢 / 🟡 / 🔴 | {需要复查什么} |

## 对 `_handbook_for_agent_engineering/` 的影响

| 变更条目 | 手册层涉及位置 | 结论 | 说明 |
|---|---|---|---|
| {变更} | `manual/{file}.md` 的 {单元} | 🔴 已改 / 🟡 待改 / 🟢 不涉及 | {为什么；不涉及时写核对过哪里} |

## 优先级建议

1. **第一优先**: {先改哪些索引或路由页，为什么}
2. **第二优先**: {其次}
3. **可选**: {不紧急的}

## 核对记录

- [ ] 相对链接全部可解析
- [ ] skill / playbook / principle 计数与源码一致
- [ ] `plugin.json` 版本号已记录
- [ ] `README.md` 依赖表与 `_digest/system/distribution.md` 不矛盾
- [ ] 新增或删除的 skill 已在索引、单页、本记录三处同步
