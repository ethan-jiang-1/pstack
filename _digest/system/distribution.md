# 分发与安装

## 一句话

pstack 的安装单位是**一个 skill 目录**，不是一整个插件。但 `poteto-mode` 依赖大多数目录同时在场，而 skill 之间又用物理路径互相引用，所以"只装几个"和"装上整套"是两种不同的系统。

## 三种分发形态

| 形态 | 适合 | 代价 |
|---|---|---|
| **Cursor 插件** `/add-plugin pstack` | Cursor 用户，想要自动更新和子代理类型注册 | 插件缓存路径由 harness 管理；本地改动不与上游合并 |
| **skills CLI** `npx skills add <owner>/pstack` | 跨 harness，只要一部分 skill，或想自己控制安装位置 | 依赖关系要自己看表；部分安装可能断链 |
| **直接拷贝目录** | 离线、审计、或改完自己发布 | 更新要自己做；路径布局必须保持 `skills/<name>/` |

三种形态装出来的内容相同：`skills/<name>/SKILL.md`，外加可选的 `references/`、`playbooks/`、`scripts/`。这是 [Agent Skills](https://agentskills.io) 的通用目录约定，pstack 没有私有打包格式。

## 安装粒度与依赖

仓库 README 把 skill 分成两类：互不调用、可以单装的，和必须成组安装的。

| Skill | 还需要 |
|---|---|
| `teach` | `how`、`why` |
| `why` | `how` |
| `technical-writing` | `unslop` |
| `architect` | `arena`、`how` |
| `blast-radius` | `arena`、`how`、`why`、`unslop` |
| `create-verification-skill` | `maintain-verification-skill` |
| `ketchup` | `bro` |
| `poteto-mode` | 全部 `principle-*` 与大多数其他 skill |
| `poteto-help` | 它要解释的那些 skill |

这张表是人工维护的，会漂移。判断某个 skill 能不能单装，最可靠的办法是读它的正文：出现 `../<name>/SKILL.md` 或"the **\<name\>** skill"时，它就有依赖。

## 装到哪里

| Harness | 项目级 | 用户级 |
|---|---|---|
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Pi | `.pi/skills/` | `~/.pi/agent/skills/` |
| Codex | `.agents/skills/` | `~/.codex/skills/` |
| OpenCode / 其他 | `.agents/skills/` | `~/.agents/skills/` |

模型设置文件是另一条路径：Cursor 读写 `~/.cursor/rules/pstack-models.mdc`，其余 harness 读写 `~/.agents/pstack-models.md`。它不属于 skill 安装，但缺了它，所有角色都会退回默认模型。

## 部分安装会在哪里断

1. **物理路径引用。** 很多正文用 `../how/SKILL.md` 这样的相对路径。目录被拆开移动，链接就断。
2. **`poteto-agent` 未注册。** 非 Cursor harness 需要按 `poteto-mode` 的 Harness 映射，用通用 subagent 加"先读这份 SKILL.md"的前缀替代。
3. **模型角色缺失。** 设置文件没写某一行，就走该 skill 的默认值；默认值里的 slug 在当前 harness 不存在时，还会再降级一次。
4. **playbook 指向的 skill 不在场。** mode 能把步骤复制进 todo，却无法执行那一步。此时正确行为是报告缺失并走 `skip` 出口，而不是自己发明一套。

`poteto-mode` 是依赖最重的入口。只想试一两个能力时，直接用对应的单点 skill（`unslop`、`bro`、`how`、`tdd`）比装一个残缺的 mode 更稳。

## 更新

- **插件**：由 harness 管理版本，`plugin.json` 的 `version` 是更新信号。
- **skills CLI**：重新运行 `npx skills add <owner>/pstack`，按目录重新安装（仓库只给出 `add` 这一条命令）。
- **本仓库镜像**：`main` 跟随上游，`ethan` 在 main 上叠加消化层。同步步骤见 [`_digest/_change_log/README.md`](../_change_log/README.md) 与 [`MIRROR.md`](../../MIRROR.md)。

## 值得注意

- **没有版本锁定。** skill 目录不带版本，安装多个来源时无法声明"我要 0.15.25 的 `how` 配 0.14 的 `unslop`"。跨版本混装靠自觉。
- **依赖表与 `poteto-mode` 的实际要求可能不同步。** 以正文中的路径引用为准，表只作入口。
- **镜像仓库的 README 顶部是本地编辑。** 同步上游时不要覆盖 `mirror:start`/`mirror:end` 之间那段，见 `MIRROR.md`。

## 关键洞察

1. **安装粒度是 skill，不是包。** 这让 pstack 可以被拆用，也把依赖管理交给了使用者。
2. **物理路径引用让"自包含目录"只在小范围内成立。** 单个 skill 目录自包含，但一组 skill 要一起搬。
3. **设置文件是运行时的第二个安装面。** 装好 skill 但没有模型配置，系统仍能跑，只是跑在默认值上。

> 源码：[`plugin.json`](../../plugin.json)、[`README.md`](../../README.md)、[`poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)（Harness）。
