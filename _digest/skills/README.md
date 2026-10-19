# 技能索引

这里收录 30 个非 principle skill 的解读。24 条 `principle-*` 不逐个建页，见 [principles 索引](../principles/README.md)。

标记说明：**U** = user-invoked，只有人显式触发；**M** = model-invoked，人和模型都可触发。pstack 里 53/54 是 **U**，唯一没有设置 `disable-model-invocation` 的是 `setup-pstack`。

> 基线：本目录的计数与默认模型 slug 以 pstack `0.15.25`（2026-10-19）为准；这些值会随版本变化，页面里的具体 slug 是快照。

## 我现在该从哪里开始

| 当前情境 | 入口 | 下一站 |
|---|---|---|
| 不知道用哪个 skill，或 pstack 怎么配 | [`poteto-help`](./poteto-help.md) **U** | 一段可直接发送的 prompt |
| 要做一件需要严谨工程流程的事 | [`poteto-mode`](./poteto-mode.md) **U** | 匹配 playbook 并执行 |
| 第一次配置模型与预算 | [`setup-pstack`](./setup-pstack.md) **M** | 写出角色到模型的设置文件 |
| 任务还没说清，先想想要什么 | [`sharpen`](./sharpen.md) **U** | 专家路径、真实目标、规格松紧 |
| 只想让 agent 复述它理解的目标 | [`in-your-own-words`](./in-your-own-words.md) **U** | 一段自己的话复述 |
| 上一段回复太绕，需要人话 | [`bro`](./bro.md) **U** | 去掉术语的重述 |
| 离开一阵，要补这段对话的变化 | [`ketchup`](./ketchup.md) **U** | 平实语言的变更摘要 + 行动项上下文 |

## 理解与解释

| 情境 | 入口 | 产出 |
|---|---|---|
| "X 是怎么工作的"、改代码前先建心智模型 | [`how`](./how.md) **U** | 带文件定位的架构说明 |
| "为什么这样做"、追溯决策与权衡 | [`why`](./why.md) **U** | 带引用的动机阅读 |
| 要把一个子系统讲给一个人听懂 | [`teach`](./teach.md) **U** | 组合 `how` 与 `why` 的一段说明 |
| 重建自己最近的工作上下文 | [`recall`](./recall.md) **U** | 当前状态简报与下一步 |
| 一个改动可能在别处弄坏什么 | [`blast-radius`](./blast-radius.md) **U** | 跨模块影响与要证明的那一条事实 |

## 设计与对抗

| 情境 | 入口 | 产出 |
|---|---|---|
| 要定类型、签名、模块边界 | [`architect`](./architect.md) **U** | 带 `not implemented` 的设计草图与理由 |
| 一个非平凡产物需要多个候选 | [`arena`](./arena.md) **U** | 多候选 + 跨模型裁判 + 手工 graft |
| 一个改动需要多模型对抗评审 | [`interrogate`](./interrogate.md) **U** | 分桶后的综合 verdict，不自动改代码 |
| 多个切片、竞速或覆盖矩阵要并行 | [`swarm`](./swarm.md) **U** | 一份收敛后的结果报告 |

## 构建与清理

| 情境 | 入口 | 产出 |
|---|---|---|
| bug 有便宜、清晰的测试路径 | [`tdd`](./tdd.md) **U** | 先红后绿的回归测试 |
| 想清掉注释里的噪音 | [`no-comments`](./no-comments.md) **U** | Comment Sicko 的发现与处理 |
| 任何英文写作表面 | [`unslop`](./unslop.md) **U** | 去掉 AI 痕迹的文本 |
| 文档、RFC、README、PR 描述、提交信息 | [`technical-writing`](./technical-writing.md) **U** | 按四层标准写出的技术文本 |
| 读或改 `.ts` / `.tsx` | [`typescript-best-practices`](./typescript-best-practices.md) **U** | 类型纪律与边界解析 |

## 验证与交付

| 情境 | 入口 | 产出 |
|---|---|---|
| 要报告一个性能数字 | [`benchmark-checklist`](./benchmark-checklist.md) **U** | 七问逐项有真实运行证据 |
| 项目没有可脚本化驱动真实应用的方式 | [`create-verification-skill`](./create-verification-skill.md) **U** | 项目本地的 `verify-*` skill |
| 验证 skill 与 feature map 开始腐烂 | [`maintain-verification-skill`](./maintain-verification-skill.md) **U** | 一次审计与至多一个修正 PR |
| 长任务、自主运行或离开后要复查 | [`show-me-your-work`](./show-me-your-work.md) **U** | 可审计的 TSV decision trail |
| 没有更窄的 playbook 可用 | [`figure-it-out`](./figure-it-out.md) **U** | 一份一次性 rigor playbook |

## 让环境变好

| 情境 | 入口 | 产出 |
|---|---|---|
| operator 反复纠正同一类错误 | [`correct`](./correct.md) **U** | 架构/类型/lint/测试层的新约束 |
| 一次会话里有值得留下的经验 | [`reflect`](./reflect.md) **U** | Accepted / Rejected / Backlog 与 skill 修改 |
| 想把个人工作习惯变成 skill | [`automate-me`](./automate-me.md) **U** | 一个定制的 `-mode` skill |

## 其他

| 情境 | 入口 | 产出 |
|---|---|---|
| 要做一个点击后唤醒 Grok Bot 的 UI | [`make-bot-ui`](./make-bot-ui.md) **U** | 带服务端发送的页面与部署说明 |

## 怎样读单篇

每篇保持同一骨架：

1. **一句话**：先判断它是否解决你的问题。
2. **它解决什么**：触发它的真实困境。
3. **核心机制**：实际步骤、阶段或规则。
4. **设计意图**：为什么这样切，而不是只讲怎么操作。
5. **依赖关系**：上游、下游、共享能力。
6. **使用边界**：什么时候不该用。
7. **关键洞察**：这一 skill 最容易被忽略的源码事实。

解读页顶部链接对应的 `SKILL.md`。行为细节以源码为准；这里负责解释结构与设计理由。
