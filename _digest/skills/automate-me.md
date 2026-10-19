# `automate-me`

> 源码：[`skills/automate-me/SKILL.md`](../../skills/automate-me/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [`poteto-mode` 的 Harness](../patterns/harness-portability.md)

## 一句话

从你自己的 transcript 里挖出反复出现的工作约定，起草或修订一个只属于你的 `<handle>-mode` skill；它只负责排序 create-skill、unslop 和一次内联挖掘，不自己发明 skill 的形状。

## 它解决什么

人说不清自己的工作风格。说出来的通常是理想版本，历史里留下的是实际版本。而 pstack 的 `poteto-mode` 恰好是一个人的风格：playbook、路由、模型角色这套机器可以换人，风格不能。

`automate-me` 的产物是"这个人希望 agent 怎样工作"的可执行文件，例如 `jay-mode`、`priya-mode`。它不是通用最佳实践，也不是某个人风格的复制；它对 poteto-mode 的要求是读它的粒度，不读它的内容，源码写得直白：`The user's rules are not the same as poteto-mode's`。

它同时解决第二类需求：风格会漂移，所以要有 update 形态，只挖 skill 上次修改之后的历史，保留没被推翻的小节。

## 核心机制

### 1. 先看有没有现成的 mode skill

递归找 `*-mode/SKILL.md`，范围包括项目级目录（`.cursor/skills/`、`.claude/skills/`、`.pi/skills/`、`.agents/skills/`）与用户级目录（`~/.cursor/skills/`、`~/.claude/skills/`、`~/.pi/agent/skills/`、`~/.codex/skills/`、`~/.config/opencode/skills/`、`~/.agents/skills/`）。mode skill 可能放在 `<skills>/<handle>/` 这样的分类目录里，所以查找必须递归。

存在时用结构化提问确认是 update 还是 start fresh，除非用户已经说了"更新我的 skill"。update 会改变后面每一步：挖掘范围收窄到 `git log -1 --format=%cI <path>` 之后；提问从"要抓什么"变成"变了什么、缺什么"；起草改为原地编辑，保留用户没有推翻的小节，只为真正新的规则加小节。

### 2. 挖历史

前置动作和 `reflect` 一样：只在本 workspace 的 transcript 目录里找，不跨项目。然后按时间片并行派 subagent，源码给的例子是最近 2-4 周切成 3 片，每片返回带证据指针的结构化模式列表。

默认的六个信号：回复偏好（长度、语气、格式、被要求"说人话"的纠正）、委派习惯（subagent、模型、专用流程、并行度）、验证姿态（什么算 done、单测还是活体复现、要不要 reviewer）、代码与文字纪律（风格、引用的 principle、lint 与格式化工具）、流程约定（worktree、commit、PR、评审与合并工具）、元偏好（中途修 skill、主动提议新 skill）。

跨片交叉验证是这一步的判据：2 片及以上出现才算高置信，孤立信号默认丢掉。

### 3. 直接问人

挖掘只能看到发生过的事，看不到"想要但还没发生"的偏好。这一步要求用结构化多选（`AskQuestion`、`AskUserQuestion`、`question`），而不是让用户从零打字：1-2 个问题，每个 4-6 个选项，类别题允许多选；先问"哪些方面最重要"，再对选中的方面追问具体选项，最后留一个自由文本兜底。源码明确禁止一次抛 20 个问题。

### 4. 聚类成小节

把两路信号合成小节。常见的有八类：response style、autonomy、understand first、subagents、prose / code discipline、review and verify、process、skills。`poteto-mode` 在这里作为粒度参照，只读形状。

### 5. 交给 create-skill 起草

起草本身不是这个 skill 的职责。placement 规则：保留已有 mode skill 的分类位置；新的 mode 在仓库已有 `<handle>` 个人目录时写 `<skills>/<handle>/<handle>-mode/SKILL.md`，否则默认 `<skills>/<handle>-mode/SKILL.md`（项目内）或用户级 skill 目录。

frontmatter 有三条硬约束：`description` 触发器是 handle + `/<handle>-mode` + "work in their style"，不用 "write code"、"review PR" 这类泛词；YAML 必须是单个 scalar，按 create-skill 的规则写；默认 `disable-model-invocation: true`，只有用户明确要每轮生效才关掉。

### 6. 迭代文字，然后落地

每一行都过 `unslop` 和 create-skill 的写作规则，反复给用户看并收反馈，预期多轮。源码的措辞是 cut ruthlessly，一句定位很实用：`A mode skill is not a manual`。

落地在 off main 的 worktree 里，提交并开 PR，不直接 push main。

### 7. 护栏

源码单列了一组反过拟合规则。同一条偏好一次被说、另一次被反着说，就是噪音，要多次出现才固化。不耍聪明：不复述其他 skill 的内容、不造隐喻、不为 agent 读者写"诗意"的句子。引用不内联：用户依赖的其他 skill 与 principle 文档以路径出现，不贴摘录。小节最小：只在用户有具体的、非默认的规则时才成节，"communicate clearly" 不算一节。祈使句用 "the user" 或 "the human"，不用作者名字。没有流程规则可写就跳过 Process 这一节，不为对称凑数。

## 设计意图

- **两条采集通道，是因为两种证据。** 历史给"实际做过什么"，提问给"还没发生过但想要什么"。只有前者会漏掉新意图，只有后者会得到理想化的自述。
- **跨片交叉验证把"一次偏好"变成"一个模式"。** 与 `reflect` 的 convergence 判据同源：单次出现不算规律。
- **update 用 git 时间而不是"上次会话"。** `git log -1 --format=%cI <path>` 给出一条可复现的下界，不依赖"上次会话"这种没有记录的概念。
- **把自己定位成编排器。** skill 形状归 create-skill，文字归 unslop，automate-me 只决定顺序和输入。这样两边的规则更新不需要它跟着改。
- **默认 U。** 个人风格应该是显式选择，而不是悄悄套到每一轮对话上。

## 依赖关系

- **协作**：`create-skill`（Cursor 内置）或 Anthropic 的 `skill-creator`，两者都没有时按 agentskills.io 的 Agent Skills 规范手写。第 5 步完全依赖它，第 6 步要它和 `unslop` 一起。
- **协作**：[`unslop`](../../skills/unslop/SKILL.md) 提供文字纪律，第 6 步逐行应用。
- **协作**：`poteto-mode` 的 Subagents 一节决定挖片 subagent 的推理档位与速度；`poteto-mode` 本身只作为粒度示例。
- **共享**：各 harness 的 transcript 路径表与 `reflect` 的定位规则同源，两份各写一遍，没有共享模块。
- **下游**：产出的 `<handle>-mode` skill 与 `poteto-mode` 并存，README 把它描述成"保留 pstack 作为基础，再拥有自己的路由 skill"。
- **索引关系**：README 把 `automate-me` 列入"不调用其他 pstack skill"的一组，而 SKILL.md 正文明确编排 `unslop` 与 `create-skill`。两者要一起读：正文里的引用是软依赖，源码只为缺少 create-skill 的情况写了替代路径，没有为缺少 `unslop` 写。

## 使用边界

- **不写通用最佳实践。** 产物只描述这个人，写进"团队都应该这样"的内容属于别的 skill。
- **不做任务型 skill。** 只想让 agent 会做某件事，用 `create-skill` 就够，不需要挖掘。
- **不吞一个窄工作流。** "我怎样写 commit message"是一份普通 skill，撑不起一个 mode。
- **需要这个 workspace 有历史。** 没有 transcript 时挖掘这一步无料可挖，而源码没有给替代路径，结果只能靠提问通道和用户自评。
- **update 依赖 git。** 不在 git 仓库里时 `git log -1` 这条时间下界失效，只能退化成全量重挖或人工指定日期。
- **产物质量取决于外部 skill。** 没有 create-skill 和 unslop 时仍能跑完，但形状与文字都靠自己把住。

## 值得注意

- **六类信号是一个世界观。** 它假定工作风格可以切成回复、委派、验证、纪律、流程、元偏好六块；不适配这六块的习惯没有"其他"桶可落，会被漏掉。
- **挖掘成本随历史长度增长，且没有上限。** 源码只给"2-4 周切 3 片"的示例，历史更长时切几片、每片多深，由执行者临场决定。
- **"问人"这一步的质量取决于选项设计。** 4-6 个选项的封闭题能降低门槛，也会把用户的表达限制在预设范围内；源码用最后那个自由文本兜底，但没有要求把兜底内容与选项结果同等对待。
- **验收方式是 vibe-check。** 源码承认这是主观产物、create-skill 式 benchmark 不适用，只在触发准确率出问题时才跑 description 优化。这与 pstack 其他地方"验证与 verdict"的姿态不同，理由成立，但意味着 mode skill 的质量没有可复现的检查。

## 关键洞察

1. **风格不能靠自述采集。** 历史是唯一有证据的样本，提问只补历史没有覆盖的意图。
2. **挖和问是两种证据，不是两种方法。** 一个回答"你实际怎样工作"，一个回答"你想怎样工作"。
3. **编排器不复制被编排者的规则。** 形状与文字各归其主，编排层只承担顺序。
4. **个人风格默认显式触发。** 把 `disable-model-invocation: true` 设为默认，等于承认风格是一种选择而不是默认值。
