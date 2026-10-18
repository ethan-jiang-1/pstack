# `maintain-verification-skill`

> 源码：[`skills/maintain-verification-skill/SKILL.md`](../../skills/maintain-verification-skill/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [证据优先](../patterns/evidence-first.md)

## 一句话

定期把项目里的验证 skill 与它的 feature map 重新对准真实应用：每个 feature 文件派一个只读读者从源码解释，协调者再独占驱动一次覆盖全部 feature 的 live pass，最后最多交付一个只含已证明修正的 PR。

## 它解决什么

feature map 描述的是用户视角的行为，而应用一改，它就开始失真。失真有两种，处理方式相反：map 说应用会做某件事而应用不做了，可能是文档漂移（改 map），也可能是产品回归（报告，不要用文档盖住）。混在一起修，回归会被写成新文档。

逐句校对 map 的成本不可接受，所以这个 skill 换了计量单位：源码写的是 `The unit of rigor is the feature, not every sentence`。每个 feature 至少被覆盖两次，一次从源码读，一次在真实应用里走，但不要求每条 bullet 都单独验证一遍。

它还解决"一次维护跑完到底该留下什么"的问题。`## Outcomes` 把结果分成 `clean`（覆盖完整、没有值得交付的东西）、`changed`（一个 PR）、`blocked`（覆盖没跑完或修不能安全落地）。`clean` 明确不建分支、不开 PR。

## 核心机制

### 0. 先认目标

找项目本地的验证 skill：body 里有 launch/drive 小节、带 feature map 的那个（通常在 `.cursor/skills/`、`.claude/skills/`、`.pi/skills/` 或 `.agents/skills/` 下的 `verify-*/`）。多个候选就问是哪个；一个都没有就停下并指向 `/create-verification-skill`，不发明维护对象。

### 1. feature map 索引卫生

读 map 的 README，glob 同目录文件，修 missing、extra、duplicate、dead 条目。源码把这步定性为 lightweight：不生成新的清单，只让索引与文件系统一致。

### 2. source wave：每个 feature 一个只读读者

每个 feature 文件一个 read-only subagent，同一批并发。它从源码回答"这个用户可见 feature 怎么工作"，带引用标出可能的 doc drift，并返回一条可执行的 live 验证 recipe。回报固定四段：feature summary、source entry points、likely drift 或 none、one recipe。子 agent 不驱动应用，也不改文件。

### 3. reconcile：把发现压成少数应用状态

每个 feature 都必须有返回。重叠的 recipe 合并成尽可能少的 app state，这是 live pass 时长的决定因素。只抽查被引用的 drift，不重新证明已经 clean 的声明。再扫最近的 churn，找 map 漏掉的用户可见面；把一个面判为 missing 之前必须有具体源码路径。

### 4. live pass：协调者独占驱动

源码写明"即使源码看起来干净也要跑"。协调者拥有全部驱动权，并按验证 skill 自己的 launch model 决定形态：长驻 server 或 UI 用一个长活实例串行驱动，短命 CLI 每个 drive 一个隔离 session。每个 feature 至少走一次，全程守三条不变量：

- 驱动一个自上次异常以来没做过 health check 的实例是禁止的：首驱前 doctor，按 session 计量时每个新 session doctor，任何失败驱动之后再 doctor；doctor 看不见的失败（进程健康但 UI 卡死）要复位到已知状态或重启。
- 已捕获的证据必须在每次 cleanup 后仍然存在，并且到命名位置实际检查，不靠假设。
- 一次驱动启动的东西不得活得比这次驱动的有用期更长。失败迭代的残留要清；共享实例只清残留，不清实例。

doctor 因 skill 漂移而失败也按漂移处理：在 edit scope 内修，重试一次，只重启这次修复作废的东西。最后在本次 run 的最后一驱（包括为修复而重跑的驱动）之后 teardown。

### 5. triage：三分类，只有一个进 PR

用户视角描述错了或缺失，是 doc drift，改 map。行为正常但 harness 驱动不了，是 harness gap，补 harness；补完要重新 live 驱动才能 ship。应用本身坏了，是 product gap，记录给用户，留在本 PR 之外。三条路径的分界，就是这次 run 能改什么的分界。

### 6. ship or stop

`changed` 才开 PR，且开之前重读每个被改文件。`clean` 和 `blocked` 都不开 PR，报告 outcome 和真实覆盖。run notes（覆盖了哪些 feature、哪些不可达及前提、确认的漂移、outcome）留在 scratch 位置，不提交。

## 设计意图

- **以 feature 为原子。** 句子级校对没有终点，feature 级覆盖有终点，也和用户能感知的失败对齐。
- **读与碰分离。** source wave 只读、并行；live pass 由协调者独占、串行。并行读可以扩展，并行驱动同一个应用只会互相污染状态。
- **recipe 合并。** 子 agent 只返回一条 recipe 而不是整套操作，父 agent 才有机会把 N 个 recipe 压成少数几个 app state；否则 live pass 会退化成 N 次独立会话。
- **把"没东西可交"设成一等结果。** 没有 `clean`，维护循环就会奖励为了交差而编出来的 PR。
- **edit scope 是防自欺装置。** 只允许改验证 skill 自己的目录，产品行为只能报告。这条把"文档跟不上"与"产品坏了"强制分开。

## 依赖关系

- **上游**：[`create-verification-skill`](../../skills/create-verification-skill/SKILL.md) 生成 target：`SKILL.md` 的 Launch、Doctor、Drive、Evidence、Cleanup 小节，以及 `features/` 下每个 feature 文件（四个 H2：`Sub-features`、`How to get to it (user POV)`、`Driving it with <harness>`、`Gotchas`）。根 `README.md` 的依赖表把这两个 skill 配成一对安装。
- **上游**：`poteto-mode` 的 Subagents 规则决定 source reader 的模型档位、后台运行与 fresh 策略。
- **共享接口**：feature map 的四个 H2 同时是生成端与维护端的契约。维护端读的是生成端写下的形状，任何一边改形状都会让另一边的假设失效。
- **对照**：`swarm` 可以按 feature-map 条目切分一次全量 pass；`benchmark-checklist` 负责性能数字，不在这里。
- **调度**：[`docs/guide/06-verify-and-ship.md`](../../docs/guide/06-verify-and-ship.md) 建议每天一次，最好由 scheduled automation 触发，说明这个 skill 被设计成可无人值守运行。

## 使用边界

- **不能代替首次生成。** 没有目标就停，源码明确要求指向 `create-verification-skill`，不允许临时造一个 map。
- **不修产品代码。** 发现回归只报告；想让 agent 顺手修，那是另一个任务。
- **一次 run 的交付上限是一个 PR。** 大幅重写 map 要么不做，要么全部塞进同一个 PR。
- **覆盖依赖应用可启动。** 需要 auth、entitlement、特定 OS 或外部状态的 feature 只能判 `verified-unreachable`，而判定必须带具体前提和尝试过的路径。
- **没有并行化 live pass 的开关。** 串行是规定，不是默认值。

## 值得注意

- **live pass 的时长与 feature 数线性相关。** 协调者独占驱动换来了没有并发污染，代价是 feature 多的项目一次 run 会很长，而源码没有给"只跑一部分 feature"的合法形态。
- **`verified-unreachable` 是一个宽松出口。** 源码要求写清前提与路径，但没有任何第三方复核这条判断，它容易变成"这个 feature 我放弃了"的委婉写法。
- **doctor 有自知之明。** 源码自己承认它看不见健康进程上的卡死 UI，只能靠 reset/relaunch 兜底；兜底是否执行取决于协调者的判断，不取决于检查。
- **一个 PR 的上限与"小 PR"倾向冲突。** map 结构性整理和若干条修正会被压进同一个 PR，只有开 PR 前的"重读每个改动文件"在把关。

## 关键洞察

1. **维护的 rigor 单位决定了它能否持续运行。** 选 feature 而不是句子，才有可能每天跑一次。
2. **读与碰分开，是为了让并行只发生在安全的一侧。** 只读探索可以 fan-out，状态变更必须单点。
3. **允许"没有可交付变更"是循环存活的前提。** `clean` 不建分支，维护者才不需要制造工作。
4. **edit scope 比任何提醒都可靠。** 把"不要用文档盖住回归"写成目录边界，比写一句行为规则更难违反。
