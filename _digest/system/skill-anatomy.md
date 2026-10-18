# SKILL.md 结构解剖

## 一句话

pstack 的一个 skill 是一个目录：`SKILL.md` 定义身份与执行协议，`references/` 放按需加载的材料，`playbooks/` 放 mode 的流程分支，`scripts/` 放可执行工具。没有额外的 Codex 元数据文件，harness 差异直接写在正文里。

## 目录结构

```text
skills/<name>/
├── SKILL.md
├── references/          # 按需加载的模板、评分表、评审 prompt
├── playbooks/           # 只有 poteto-mode 有：23 个流程分支
├── scripts/             # 可执行工具，如 log.sh、orch.ts、watch-pr
└── templates/           # 可复制的空白 artifact
```

`agents/` 在仓库根目录，不在 skill 里：`poteto-agent.md` 和 `comment-sicko.md` 是 subagent 定义，由 harness 注册。

## Frontmatter

```yaml
---
name: poteto-mode
description: poteto's agent style for concise, detailed responses, ...
disable-model-invocation: true
icon: crown
color: yellow
---
```

字段分工：

- `name`：调用名，必须与目录名一致（kebab-case）。
- `description`：面向人的识别文本，也承担 playbook 之外的触发说明。
- `disable-model-invocation`：pstack 的默认值，表示只有人能触发。见 [调用模型](./invocation-model.md)。
- `icon` / `color`：Cursor 的 UI 装饰，`poteto-mode` 使用。
- `paths`：按文件类型限定，`typescript-best-practices` 用 `["**/*.ts","**/*.tsx"]`。

没有 `agents/openai.yaml`，也没有 `argument-hint`。Codex 元数据在 pstack 里不存在；跨 harness 的说明写在正文。

## 正文的四种内容

### 1. 步骤型协议

playbook 和大多数 skill 用编号步骤推进状态。好的步骤带可检查的完成条件与失败出口，例如：

- `bug-fix` step 1：在该表面上自己复现；控制面够不到时才请用户，并给出理由。
- `shipping` step 1：每个 PR 由没写代码的 verifier 给出 `PASS`/`PASS+NOTES`/`FAIL`。
- `babysit` step 1：先声明四种模式之一，再开始轮询。

### 2. 阶段型协议

`arena`、`swarm`、`reflect` 用 Phase A/B/C 或 1-6 的阶段名，每个阶段有产物和退出条件。阶段名本身就是进度状态。

### 3. 参考型内容

`unslop` 是一张编号规则表；`typescript-best-practices` 是一张规则表；`benchmark-checklist` 是七问清单。这些内容不推进状态，只约束判断。

`unslop` 的编号是**稳定 id**：其他 skill 引用"rule 13""rule 26"，删掉一条要留空号，不能重排。

### 4. 回复契约

很多 skill 末尾有 **Reply:** 行，规定这一次交付必须包含什么。例如 `bug-fix` 要求"what was broken, root cause, fix, how you verified"，并逐字粘贴失败到通过的输出。

这让"做完"有可检查的文本形状，而不只是"模型说完成了"。

## Reference 的组织

| Skill | 按需材料 | 何时读 |
|---|---|---|
| `architect` | `runner-prompt.md`、`rationale-template.md`、`design-red-flags.md` | 派设计候选、写理由、查红旗 |
| `how` | `explainer-prompt.md`、`explorer-prompt.md` | 派探索者/解释者 |
| `why` | `epistemics.md` | 写置信度与措辞 |
| `interrogate` | `reviewer-prompt.md`、`rubric.md`、`code-quality-review.md`、`lead-judgment.md` | 派评审、综合结论 |
| `reflect` | 三个 reviewer prompt + synthesizer | 会话复盘 |
| `no-comments` | `comment-sicko.md` | 派 Comment Sicko |
| `blast-radius` | 如何与 `how`/`why` 配合 | 追跨模块影响 |
| `create-verification-skill` | `feature-map-example/` | 生成 feature map |

pointer 通常写成"读 `references/x.md`，把它逐字作为 prompt"，并说明触发条件。这比"详见"更强，因为模型知道何时该读、读了要做什么。

## Scripts 与可执行证据

pstack 把一部分协议做成脚本，而不是散文：

- `show-me-your-work/scripts/log.sh`：追加一行 TSV，处理表头、转义和公式注入。
- `poteto-mode/scripts/orch/orch.ts`：orchestrate 的队列与 frontier 记账。
- `poteto-mode/scripts/watch-pr/`：GitHub PR 状态轮询。
- `poteto-mode/scripts/check-plan.mjs`：计划检查。

脚本是可重跑的证据。这对应 `build-the-lever` 原则：能让脚本每次同样执行的事，不要靠手。

## 体量分布

- 大多数 skill 在 40-160 行。
- `poteto-mode` 161 行，是最长的单文件，因为它要同时承担路由、自主、子代理、写作和 harness 映射。
- playbook 大多 10-25 行，`orchestrate` 112 行、`multi-phase-plan` 155 行，因为它们是完整协议。

体量不是复杂度指标。`tdd` 只有 42 行，但它约束的是一个高频、容易自欺的动作。

## 与其他文档层的关系

| 层 | 读者 | 作用 |
|---|---|---|
| `SKILL.md` | agent | 稳定执行协议 |
| `docs/guide/` | 人 | 操作教程，按第一次使用排序 |
| `_digest` | 人 | 结构与设计解释 |
| `README.md` | 人 | 安装、依赖、playbook 一览 |

四层不应互相复制。`SKILL.md` 说"怎样做"，guide 说"第一次怎样走"，digest 说"为什么这样拆"。

## 值得注意

- **harness 映射在多个文件里重复。** `how`、`why`、`reflect`、`swarm`、`arena`、`interrogate`、`architect` 都各写了一段"其他 harness 怎样映射 Task"。重复带来漂移风险。
- **物理路径引用。** 正文直接写 `../how/SKILL.md`，目录一动就断。
- **没有格式校验器。** frontmatter 拼错、description 缺引号、相对链接失效，都不会在提交时被拦住。
- **`disable-model-invocation` 的语义跨 harness 不一致。** 见 [调用模型](./invocation-model.md)。

## 关键洞察

1. **身份在 frontmatter，行为在正文，深度在 reference。** 三者对应不同的加载时机。
2. **步骤与参考是两种内容。** 步骤推进状态，参考约束判断；混在一起会让协议失去可检查性。
3. **回复也是协议的一部分。** 规定交付文本的形状，是防止"完成了"空口无凭的低成本手段。
4. **稳定 id 比漂亮编号重要。** `unslop` 的规则号被别处引用，就不能重排。
