# `create-verification-skill`

> 源码：[`skills/create-verification-skill/SKILL.md`](../../skills/create-verification-skill/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

把一个项目"怎样像用户一样驱动真实应用并留下证据"的知识，生成成本地的 `verify-<app>` skill：给下一个 agent 冷启动时读的 Launch、Doctor、Drive、Evidence、Cleanup、Helpers 说明，加一张按用户功能组织的 feature map，并且交付前自己完整跑通一次。

## 它解决什么

agent 能改代码，却常常无法证明行为。每个项目都有一套隐含知识：怎么起、用哪个命令点、看到什么才算对、证据放哪、跑完怎么收。这些散在 README、Makefile、某个人的快捷键和口头约定里。没有它，agent 只能写一次性脚本，或者让用户自己点一遍。

生成的 skill 是给下一个 agent 的，不是给人的。它会在一无所知的情况下、任务进行到一半时打开这个文件，所以不能留 placeholder，不能假设读者知道上下文，命令必须是真的能跑的。源码把这句写在开头：你写的是生成器的输出，读者是一个从没见过这个 app 的 agent。

## 核心机制

### 1. 采访仓库，不采访用户

五个问题从代码库回答，只有代码观察不到的才问人。**Surface**：用户实际碰什么，web UI、CLI/TUI、桌面、API、移动端还是库；一个仓库可能有多个，选主要的并记下其余的。**Run**：项目自己的 dev 命令、端口、env、seed 数据、auth。**Drive**：先找现有 harness（Playwright 或 Cypress 规格、expect 脚本、PTY helper、可 curl 的端点、debug port），再退回通用方案（web 与 Electron 用浏览器或 CDP，CLI/TUI 用 tmux 或 PTY，服务用 HTTP）。**Observe**：能留下什么证据，截图、终端 transcript、响应体、日志、exit code、数据库状态。**Isolate**：两个实例能否并存；不能就在生成的 skill 里直说，拒绝同时驱动一个共享实例胜过污染用户的会话。

### 2. 基线坏了先修

checkout 不能 build 或 start 时，先处理它，或者精确报告。写成对着坏基线操作的 skill 会教出错误步骤。与目标无关的缺失资产（API 从不服务的静态目录、示例配置）可以由生成的 skill 创建，但要明确标成 verification scaffolding，并在 cleanup 里删除。

### 3. 生成的 skill 有固定骨架

YAML frontmatter 必须有 `name: verify-<app>` 和一段说明 app、surface、何时使用的 description，没有 frontmatter 这个 skill 不会注册。正文分六块。

- **Launch**：精确启动命令与 ready 信号。必须从空 data dir 到达可用状态，包括 migration 与 seed。ready 指可用，不是进程有响应；数据库健康检查通过时，init 脚本可能还在跑。要含 teardown。短命的 CLI 或 TUI 没有常驻 server，launch 指 build 一次（或装依赖），之后每次 drive 各开独立的 PTY 或 tmux 会话。
- **Doctor**：一个只读检查，回答"这个实例值不值得驱动"：进程在、构建版本对、端口是我们占的、auth 有效。任何地方看起来不对时先跑它。
- **Drive**：这份仓库真实的 selector 与命令，不是示例。优先稳定 handle：ARIA 标签、data attribute、prompt 字符串、route 路径，而不是坐标和 tab 顺序。
- **Evidence**：证明标准。走真实用户路径，不用内部 setter 或 test-only endpoint；同时留下动作和结果状态，不能只截最后屏；可见结果之外还要验证副作用（写了文件、插了行、发了消息）；mock 只放在生产边界已经隔离外部系统的地方。用 dry-run 或 test mode 时，要观察它实际跳过了什么（文件、网络、git ref），不要相信名字，有些 dry-run 仍然访问网络或打开浏览器。
- **Cleanup**：怎么拆掉这次运行创建的实例。永远不要按进程名 kill，只 kill 自己启动的。清理移除实例与 scratch state，不移除证据：proof artifact 要活过 teardown，放在 skill 指定的位置。
- **Helpers**：随 skill 附带的脚本必须可执行，调用方式写在正文里。需要读者逆向的 helper 不算 helper。

### 4. 种下 feature map

创建 `features/README.md` 和每个可识别用户功能一个文件，先从路由、命令、菜单或文档里挑 3 到 5 个。形状照 `references/feature-map-example/`。每个功能文件用四个固定 H2：`Sub-features`、`How to get to it (user POV)`、`Driving it with <harness>`、`Gotchas`。内容只回答用户视角的四件事：这是什么、怎么到达、怎么驱动、什么可观察终态证明它工作。

seed 数据缺少某个旅程需要的前置条件时，旅程报失败，原因写进该功能的 `Gotchas`，绝不为了让旅程通过而削弱它。map 是仓库长期维护的验证来源：map 列了多个入口时，只驱动一个方便入口的 proof 不算完整。

### 5. 交付前自证一次

按它自己的说明跑一遍：launch、doctor、驱动 map 里的一个功能（一个就够，map 的存在就是为了后续运行覆盖其余）、抓证据、清理。清理之后确认证据还在原位，吃掉 proof 的 cleanup 直接判这一步失败。每次失败的迭代也要跑 cleanup，别让坏尝试留下进程和端口。没有执行过的生成物是草稿，不是交付物。

如果在 stand-in 上验证（容器代替全新 VM，本地 stack 代替远端），要列出 stand-in 做了而真实目标不会做的每一步。这些步骤在真实目标上跑过之前都是未验证的。

### 6. 交接维护

指向 `/maintain-verification-skill`，让 map 随 app 变化继续诚实。用户不问，就不要主动建议运行频率。

## 设计意图

- **语言与框架无关。** 技能只要求五种能力，不绑定工具；现有 harness 优先，避免为一个仓库引入第二套测试设施。
- **写给冷读者。** 精确命令、真实 selector、ready 判据，都是为了让一个没有上下文的 agent 不必猜测。
- **ready 是可用，不是活着。** 这是验证 flaky 最常见的来源，所以写进了 Launch 的定义里。
- **证据与清理分离。** proof artifact 是产物，scratch state 是垃圾，两者的生命周期必须不同。
- **自证一次。** 生成器执行自己的输出，把"可执行"从声称变成事实；只跑一个功能是成本上限，其余交给 map 和维护循环。
- **隔离默认。** 宁可拒绝双驱动，也不冒污染用户会话的风险。
- **map 是维护单元。** 后续维护按功能而不是按句子进行，所以种子文件必须结构化。

## 依赖关系

- **下游是 `maintain-verification-skill`。** 它接手 map 的日常维护；仓库 README 的依赖表要求这两个 skill 一起安装。
- **上游触发者。** `setup-pstack` 在项目既没有 `verify-*` skill 也没有现成 harness 时提示一次并调用它；`poteto-help` 在"怎么证明改动"类问题里提到它；`docs/guide/06-verify-and-ship.md` 展示在 prompt 里点名生成的 skill。
- **内部依赖例子目录。** 第 3 步指向 `references/feature-map-example/`。这个目录同时是 `README.md`、`search.md`、`create-note.md` 三份文件的组合，示范 index 与单功能文件的关系。
- **与 control skill 的关系。** 例子里的 `control-notes` 是项目自带的驱动 CLI；指南在讲 control CLI 时紧接着指向 `principle-build-the-lever`，但 `SKILL.md` 本身没有点名这个 principle。
- **并行使用。** 指南里提到 `swarm` 可以按 feature-map 条目拆分整轮验证并汇总结果。

## 使用边界

- **项目已有 harness 或 `verify-*` skill 时不该用它。** description 的适用条件就是"没有脚本化方式证明 UI、CLI 或服务行为"。
- **它不保证行为正确。** 它提供可驱动、可观察、可取证的能力；某次改动是否真的对，仍由那一次 proof 判断。
- **依赖项目能启动。** 起不来时它先修或报告，这部分工作会越出"生成 skill"的范围。
- **生成的 skill 是项目本地资产。** 路径随 harness 变化（`.cursor/skills/`、`.claude/skills/`、`.pi/skills/`、`.agents/skills/`），换 harness 或 monorepo 多 app 时要重新选目录。
- **初始 map 必然不完整。** 只种 3 到 5 个功能，覆盖率靠后续维护补。
- **不覆盖 CI 集成与产品测试。** 它生成的是驱动与取证说明，不是测试套件。

## 值得注意

- **完整性要求与 3 到 5 个种子上限并存。** 一方面要求"map 列了多个入口时一个入口的 proof 不完整"，另一方面又只种 3 到 5 个功能。这是有意的分期，但读者容易把种子 map 当成完整清单。
- **正文的 section 清单与例子 index 不完全对齐。** 例子 README 里有 baseline preconditions、driving conventions、proof and skip reporting 三块，`SKILL.md` 的六块骨架没有要求生成这些。照着 `SKILL.md` 生成和照着例子生成，会得到结构不同的索引。
- **修基线是范围扩张。** 为了让生成的 skill 可用，生成器可能要先修 build 或启动，这部分工作量与归属没有定义。
- **stand-in 的未验证步骤只被列出。** 技能要求诚实登记，但没有机制保证它们在真实目标上补跑。
- **清理与证据的边界靠人工。** 如果 cleanup 按目录删 scratch，很容易连 artifacts 一起删；技能只给了"清理后确认证据还在"的检查。
- **无人值守时采访无法完成。** 源码要求只问代码观察不到的事，但没规定问不到时是报告、默认还是停止。

## 关键洞察

1. **采访仓库，不采访用户。** 大部分答案已经在 build 脚本、路由和现有测试里。
2. **写给冷启动的 agent。** 无 placeholder、精确命令、明确 ready 判据，这三件事决定了生成物能不能被直接执行。
3. **ready 意味着可用。** 进程活着、健康检查通过，都不等于应用可以接受用户操作。
4. **没跑过的生成物是草稿。** 自证一次是交付门槛，不是可选项。
5. **一个入口的 proof 不等于整张 map。** map 把"验证"从一次点击变成可维护的覆盖范围。
6. **只 kill 自己启动的，证据必须活过清理。** 这两条保护的是别人的会话和这次的结论。
