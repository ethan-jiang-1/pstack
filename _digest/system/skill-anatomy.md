# SKILL.md 结构

## 一句话

一个 skill 是 `skills/<name>/` 目录：`SKILL.md` 定义 frontmatter 和执行协议，可选 `references/`、`playbooks/`、`scripts/`。`agents/` 在仓库根目录，放 subagent 定义。没有额外的 Codex 元数据文件；harness 差异写在正文的 **Harness** 或 **Other harnesses** 段里。

## 目录

```text
skills/<name>/
├── SKILL.md
├── references/     # 按需加载的 prompt、模板、评分表
├── playbooks/      # 只有 poteto-mode 有：23 个流程分支
└── scripts/        # 可执行工具
```

`agents/poteto-agent.md` 和 `agents/comment-sicko.md` 是 subagent 定义，由 harness 注册，不在 skill 目录里。

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

| 字段 | 作用 |
|---|---|
| `name` | 调用名，必须与目录名一致（kebab-case） |
| `description` | 面向人的识别文本，也写触发条件 |
| `disable-model-invocation` | 默认值，表示只有人能触发；53 / 54 写了它，只有 `setup-pstack` 省略 |
| `icon` / `color` | Cursor 的 UI 装饰，`poteto-mode` 使用 |
| `paths` | 按文件类型限定，`typescript-best-practices` 用 `["**/*.ts", "**/*.tsx"]` |

没有 `agents/openai.yaml`，也没有 `argument-hint`。frontmatter 没有提交时校验器。

## 正文里有什么

正文没有固定模板，常见的有：

- **编号步骤。** playbook 和大多数 skill 用编号步骤推进状态，步骤带完成条件与失败出口。例：`bug-fix` step 1 在 matching surface 上复现；`shipping` step 1 要独立 verdict；`babysit` step 1 先声明 mode。
- **Phase 名。** `arena`、`swarm`、`reflect` 用 Phase A/B/C 或 1-6，阶段名本身就是进度状态。
- **规则表。** `unslop`、`typescript-best-practices`、`benchmark-checklist` 是规则表，不推进状态，只约束判断。`unslop` 的编号是稳定 id，删一条要留空号，别处会引用 "unslop rule 14"。
- **`Reply:` 行。** 规定交付文本必须包含什么。`bug-fix` 要求 what was broken、root cause、fix、how you verified，并逐字粘贴失败到通过的输出。

## references

| Skill | 按需材料 | 何时读 |
|---|---|---|
| `architect` | `runner-prompt.md`、`rationale-template.md`、`design-red-flags.md` | 派设计候选、写理由、查红旗 |
| `how` | `explainer-prompt.md`、`explorer-prompt.md` | 派探索者、解释者 |
| `why` | `epistemics.md`、`investigator-prompt.md`、`source-playbook.md`、`sources/*.md`、`synthesizer-prompt.md` | 派 investigator、按源取证据、写置信度与综合 |
| `interrogate` | `reviewer-prompt.md`、`rubric.md`、`code-quality-review.md`、`lead-judgment.md` | 派评审、综合结论 |
| `reflect` | `judgment-reviewer.md`、`tooling-reviewer.md`、`divergent-reviewer.md`、`synthesizer.md` | 会话复盘 |
| `no-comments` | `comment-sicko.md` | 派 Comment Sicko |
| `create-verification-skill` | `feature-map-example/` | 生成 feature map |
| `show-me-your-work` | `decision-log-template.tsv` | 建 decision trail |

pointer 通常写成"读 `references/x.md`，把它逐字作为 prompt"，并说明触发条件。这比"详见"更强，因为模型知道何时该读、读了要做什么。

## scripts

pstack 把一部分协议做成脚本，而不是散文：

- `show-me-your-work/scripts/log.sh`：追加一行 TSV，处理表头、转义和公式注入。
- `poteto-mode/scripts/orch/orch.ts`：orchestrate 的队列与 frontier 记账。
- `poteto-mode/scripts/watch-pr/`：GitHub PR 状态轮询。
- `poteto-mode/scripts/check-plan.mjs`：计划检查。

脚本是可重跑的证据，对应 `build-the-lever` 原则：能让脚本每次同样执行的事，不要靠手。

## 边界

- **harness 映射在五个 skill 里重复。** `how`、`why`、`reflect`、`swarm`、`interrogate` 各写了一段 Other harnesses；`arena`、`architect` 只写设置文件路径。重复带来漂移风险。
- **物理路径引用。** 正文直接写 `../how/SKILL.md`，目录一动就断。
- **没有格式校验器。** frontmatter 拼错、description 缺引号、相对链接失效，都不会在提交时被拦住。
- **`disable-model-invocation` 在各 harness 的执行方式仓库没有说明。** 见 [调用模型](./invocation-model.md)。

## 关键洞察

1. 身份在 frontmatter，行为在正文，深度在 reference；三者对应不同的加载时机。
2. 步骤推进状态，规则表约束判断；混在一起会让协议失去可检查性。
3. `Reply:` 也是协议的一部分。规定交付文本的形状，是防止"完成了"空口无凭的低成本手段。
4. 稳定 id 比漂亮编号重要。`unslop` 的规则号被别处引用，就不能重排。

## 源码

- [`skills/`](../../skills/) 下的 `SKILL.md`、frontmatter、`references/` 与 `scripts/`
- [`skills/poteto-mode/SKILL.md`](../../skills/poteto-mode/SKILL.md)
- [`skills/show-me-your-work/scripts/log.sh`](../../skills/show-me-your-work/scripts/log.sh)
- [`agents/poteto-agent.md`](../../agents/poteto-agent.md)、[`agents/comment-sicko.md`](../../agents/comment-sicko.md)
