# `poteto-help`

> 源码：[`skills/poteto-help/SKILL.md`](../../skills/poteto-help/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

把"pstack 该怎么用"变成一个可执行的下一步：判断你属于哪类需求，只回答那一节，给一段能直接发送的 prompt，并链到答案所在的文件。

## 它解决什么

pstack 有 54 个 skill、23 个 playbook、24 条 principle（plugin 0.15.25 的计数），其中 53 个只能由人触发。入口收得很紧（一个 `/poteto-mode` 加少数单点工具），代价是名字得靠人记。`poteto-help` 补的就是这部分记忆负担，但它的定位很窄：回答怎么用，不替用户跑。

这个区分有成本理由。一个 help 问题如果被当成任务执行，会拉起 playbook、subagent 和评审面板，而用户只是想知道该输入什么。所以整页的第一条行为约束就是"help 问题不要开工"。

## 核心机制

### 1. 先分类，只答一节

从消息和对话推断需求。明确的情境直接进对应小节，比如"哪个 skill 评审 PR"；仍不清楚时问一个多选题，五个选项：Get set up、Start a task、Pick a skill、Fix a run、Make pstack my own，然后只回答用户选中的那一节。不把整张地图倒给一个只想知道下一步的人。

### 2. 先查两个会改变答案的状态

- 设置文件不存在（Cursor 的 `~/.cursor/rules/pstack-models.mdc`，其他 harness 的 `~/.agents/pstack-models.md`）→ `/setup-pstack` 没跑过，每个角色都用默认模型。
- 项目里没有 `verify-*` 或现成的 app harness → agent 没有脚本化驱动应用的方式；问题涉及"证明改动有效"时提 `/create-verification-skill`。

只在状态真的改变答案时提。模型规则缺失且相关时，每个 chat 最多问一次是否现在配模型，给"现在/以后"两个选择；如果同时需要分类，两个问题一起问。

### 3. 帮助问题不执行工作

这是全页最硬的边界。对 help 问题：不要开始工作，给出 prompt，让用户自己发送，因为一次 pstack 运行会花真实 token。对"用 pstack 修这个 bug"这类请求：读 [`poteto-mode`](../../skills/poteto-mode/SKILL.md)，按它做，并提一次 Custom Mode 能让它常驻。

### 4. 六节地图

- **Get set up**：安装（Cursor 里 `/add-plugin pstack`；其他 harness 用 `npx skills add backnotprop/pstack`）、跑 `/setup-pstack`，然后用 `/poteto-mode` 给它一个目标和可判定的 done check，开始第一个任务。成本段落说明 token 花在 subagent 和评审面板上，以及四个降本手段：重跑 setup 选更小预算或更便宜模型、用 `auto` / `inherit-parent` 跟随 chat 模型、缩短面板列表、把 mode 留给需要严谨度的工作。
- **Start a task**：mode 匹配 playbook、复制步骤、`skip:` 保持可见；好 prompt 说目标与 done check，不列 skill（手写顺序会丢掉 playbook 保留的步骤）；Enter 只把 skill 附着到一条消息，Option/Alt+Enter 或 Use as Mode 才是 Custom Mode，才每轮都在。
- **Pick a skill**：一张"我想要 → skill"表，加上 close calls 和"不在 pstack 里"名单。表漏掉某个目录时，读它的 frontmatter 按 description 路由。
- **Playbooks and principles**：playbook 没有斜杠命令，靠描述任务或固定短语点名；pstack 没有 planning skill，跨阶段用 Multi-phase plan；principle 是单规则 skill，用户用名字操纵，也能用 `/principle-<name>` 按需加载。
- **Fix a run that went wrong**：症状到修法的表，覆盖 mode 淡出、问题被当成上一步、设置不生效、成本超预期、skill 未自动加载、并行 agent 互相覆盖、overnight 跑空、绿构建就报成功。
- **Make pstack my own**：`automate-me`、`reflect`、authoring 与 eval playbook，以及"坏 skill 在自己的 PR 里修"。

### 5. 真相在别处

正文只做映射，细节归被路由到的文件。两条规则：先读那个文件再引用；它和这张地图冲突时信它。链接指向安装后的 plugin 目录，用户可能打不开，所以给公开副本：`https://github.com/cursor/plugins/blob/main/pstack/` 加路径。

### 6. 回复契约

先给答案。最多一个示例 prompt（代码块），能套 [`references/recipes.md`](../../skills/poteto-help/references/recipes.md) 就套，然后链到那个文件。除非用户要整张地图，否则保持短。

## 设计意图

- **帮助也要算成本。** 回答一次 help 问题的开销应当远小于执行一次任务，否则人会不敢问。
- **一次只答一节。** 分类比重述全部内容便宜，也更接近用户此刻的问题。
- **地图与事实分离。** help 不拥有细节，只拥有路由；细节仍由各 skill 的正文负责。
- **给 prompt 而不是代跑。** 把"要不要花这笔钱"留在人手里，与 pstack 整体的 user-invoked 选择一致。
- **失败模式表优先于教程。** 用户来找 help，多数是已经卡住了。

## 依赖关系

- **上游**：人输入 `/poteto-help`；`poteto-mode` 的触发表不路由它，它是人的入口。
- **下游**：几乎每个 skill 的入口，尤其是 `setup-pstack`、`create-verification-skill`、`automate-me`、`reflect`；[`docs/guide/`](../../docs/guide/README.md) 的 01、02、06、07、08、09、10 页；`README.md`；两份 reference（`prompting.md`、`recipes.md`）。
- **共享**：`references/prompting.md` 展开"怎样措辞"，`references/recipes.md` 给出可复制样例，两者都是这一页的延伸材料。
- **对照**：`sharpen` 和 `in-your-own-words` 改变任务本身，help 只给入口；`recall` 与 `ketchup` 补上下文，help 不补。

## 使用边界

- **不是文档。** 它不复述机制；把细节抄进来只会制造第二份会漂移的事实。
- **不执行工作，也不写完整任务 brief。** 它给一段可发送的 prompt，但不接管后续。
- **假设已安装且目录布局完整。** 表格按当前 skill 集合写。
- **对"是否配过模型"只看文件是否存在**，不看内容是否过期、slug 是否仍可用。
- **外部工具的归属会变。** `cursor-team-kit`、`/loop`、`/create-skill` 的"不在 pstack 里"名单是当前快照。

## 值得注意

- **表格是维护面。** 几十行"我想要 → skill"加六节正文，每次增删 skill 都要同步，否则用户会被路由到不存在的名字。源码给了兜底（读 frontmatter），但没有机械检查。
- **两个来源不一致。** 公开副本指向 `cursor/plugins`，安装命令指向 `backnotprop/pstack`。镜像用户按提示读到的可能不是自己装的那份；源码没有解释这层关系。
- **一次只答一节牺牲全局观。** 想通读的人必须明说，页面只承诺"除非用户要整张地图"。
- **close calls 是人工判断。** 两个 skill 的边界一变，这里最先过期，而且过期了也不会报错。

## 关键洞察

1. **帮助页是路由器，不是百科。** 它的价值在"下一步输入什么"，不在"系统全貌"。
2. **给 prompt 而不是代跑，把花费决定留在人手里。** 这与整个 pstack 的 user-invoked 选择同源。
3. **分类问题比回答整张地图便宜。** 一次多选题换掉一次全量解释。
4. **指向源码加公开副本，是安装态文档最实际的引用方式。**
5. **失败模式表比教程更接近用户真正需要的东西。** 卡住的人找的是出口，不是入门。
