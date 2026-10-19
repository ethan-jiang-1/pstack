# Sync 0001: baseline

**日期**: 2026-10-19

**上游版本**: `0.15.25`

**同步窗口**: `6f131f1`（`origin/main` 的当前 tip）

**本地合入**: 从 `main` 新建 `ethan`，在 ethan 上建立消化层。`main` 不变。

> 历史说明：这份 log 记录严格重写前的基线；其中 "patterns / 8 页 / 图" 是当时的目录状态，不是当前的解释框架。当前结构见 [`../README.md`](../README.md)。

## 核心主题

这是消化层的基线，不是一次上游变更。它固定三件事：pstack 在 2026-10-19 的形状、消化层的第一版结构、以及后续同步要对照的起点。

基线时的形状：

- 54 个 skill：30 个功能/元 skill，24 个 `principle-*`。
- 23 个 playbook，挂在 `poteto-mode/playbooks/` 下。
- 2 个 subagent 定义：`poteto-agent`、`comment-sicko`。
- 10 章 `docs/guide/`，外加一个索引页。
- `automations/benny/` 是独立的 Slack 自动化包，不注册为 slash skill。
- 54 个 skill 中 53 个设置 `disable-model-invocation: true`，唯一例外是 `setup-pstack`。

## 按文件域拆解

### skills/

| Skill | 变更 | 影响 | 意义 |
|---|---|---|---|
| 全部 54 个 | 基线，未改动 | 无 | 后续同步以此为对照 |

### playbooks/

| Playbook | 变更 | 意义 |
|---|---|---|
| 全部 23 个 | 基线，未改动 | 后续同步对照步骤与门的变化 |

### agents / docs / 配置

| 维度 | 变更 | 意义 |
|---|---|---|
| `agents/poteto-agent.md` | 基线 | 子代理读取 mode 全文的约定 |
| `agents/comment-sicko.md` | 基线 | `no-comments` 的评审 agent |
| `docs/guide/` | 基线 | 操作教程与 digest 的分工起点 |
| `plugin.json` | `0.15.25` | 后续同步的版本信号 |

## 对 `_digest/` 的影响

| 文件 | 影响程度 | 说明 |
|---|---|---|
| `README.md` | 🔴 新建 | 五分钟心智模型与阅读路径 |
| `MAINTAINING.md` | 🔴 新建 | 页面契约、分支拓扑、同步流程 |
| `system/*.md`（8 页） | 🔴 新建 | 总览、分发、路由、调用、模型、结构、可移植性、自主 |
| `patterns/*.md`（8 页） | 🔴 新建 | 入口、词汇、多样性、拓扑、证据、文字、环境、移植 |
| `principles/README.md` | 🔴 新建 | 24 条分组索引 |
| `playbooks/README.md` | 🔴 新建 | 23 个流程的选择树与逐项说明 |
| `skills/README.md` + 30 页 | 🔴 新建 | 非 principle skill 的单页解读 |
| `figures/*.svg`（8 张） | 🔴 新建 | 系统图与模式图 |
| `_change_log/` | 🔴 新建 | 本记录、模板与维护流程 |

## 对 `_faq_on_digest/` 的影响

| FAQ 条目 | 影响程度 | 说明 |
|---|---|---|
| `01_mode-or-single-skill/` | 🔴 新建 | mode 与单点 skill 的分工 |
| `02_single-model-no-subagents/` | 🔴 新建 | 单模型、无子代理时的可用性 |
| `03_do-i-need-to-memorize-principles/` | 🔴 新建 | principle 的使用方式 |
| `04_model-panel-and-budget/` | 🔴 新建 | role line 与预算 |
| `05_which-skills-to-install/` | 🔴 新建 | 安装粒度与依赖 |
| `06_how-much-autonomy-is-safe/` | 🔴 新建 | 自主边界与审计 |

## 对 `_handbook_for_agent_engineering/` 的影响

| 变更条目 | 手册层涉及位置 | 结论 | 说明 |
|---|---|---|---|
| 基线建立 | 全书 | 🟡 骨架 | 只完成 README、CONTEXT、PLAN 与两个索引；正文待写 |

## 优先级建议

1. **第一优先**: `_digest` 的 system、patterns、principles、playbooks、skills 索引与 30 个单页，让解释层先完整。
2. **第二优先**: 6 个 FAQ 条目，覆盖安装、模式选择、单模型、原则、预算、自主。
3. **可选**: handbook 正文；等 pstack 的使用经验积累后再写，避免抄 guide。

## 核对记录

- [x] 相对链接全部可解析
- [x] skill / playbook / principle 计数与源码一致（54 / 23 / 24）
- [x] `plugin.json` 版本号已记录（0.15.25）
- [x] `README.md` 依赖表与 `_digest/system/distribution.md` 不矛盾
- [x] 新增或删除的 skill 已在索引、单页、本记录三处同步
