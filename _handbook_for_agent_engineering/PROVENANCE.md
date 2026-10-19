# pstack 手册：源码映射

这本手册的每一页都必须能落到 pstack 的源码。落不回去的话题，不写；pstack 没有的机制，明确说"不覆盖"，不起新名字。

## 教程

| 页面 | 讲的 pstack 机制 | 源码 |
|---|---|---|
| `tutorial/00-start-here.md` | prompt 五项：goal、done check、proof、what you already know、real constraints | `docs/guide/02-poteto-mode.md#What goes in a prompt` |
| `tutorial/01-define.md` | mode 与单 skill 的选择；AskQuestion 分类；可观察的事实不问人 | `skills/poteto-mode/SKILL.md#Non-negotiables` |
| `tutorial/02-give-words-a-home.md` | mode 匹配 playbook；步骤逐字进 todo；`skip: <reason>` | `skills/poteto-mode/SKILL.md#Playbooks` |
| `tutorial/03-make-a-promise-observable.md` | 先命名 data shape；可观察结果写进 plan 的 You see 与 Verify（unit / live / perf） | `principle-model-the-domain`、`skills/poteto-mode/playbooks/feature.md`、`multi-phase-plan.md` |
| `tutorial/04-design-a-testable-shape.md` | `architect` Phase B：public surface、`not implemented` 草图、design red flags | `skills/architect/SKILL.md`、`references/design-red-flags.md` |
| `tutorial/05-cut-work-into-delegable-units.md` | `feature` step 3 throughput checkpoint；`multi-phase-plan` 的 one PR = one change | `playbooks/feature.md`、`playbooks/multi-phase-plan.md` |
| `tutorial/06-build-a-feedback-loop.md` | `tdd` 的 red-green；测试路径不便宜时跳过 | `skills/tdd/SKILL.md` |
| `tutorial/07-delegate-and-keep-control.md` | Subagents 默认值；`poteto-agent`；brief；fresh subagent | `skills/poteto-mode/SKILL.md#Subagents`、`playbooks/orchestrate.md#The brief` |
| `tutorial/08-prove-the-promise.md` | `prove-it-works`；verdict；`PASS` / `PASS+NOTES` / `FAIL`；patch-id | `skills/principle-prove-it-works/SKILL.md`、`playbooks/shipping.md` |
| `tutorial/09-review-for-merge.md` | `interrogate`：scope、intent、rubric + code-quality、四个桶 | `skills/interrogate/SKILL.md` 与 `references/` |
| `tutorial/10-ship-observe-and-loop.md` | `shipping` 的独立 verdict、contiguous run、patch-id、bottom-up；`babysit`；部署是 Always pause | `playbooks/shipping.md`、`playbooks/babysit.md`、`skills/poteto-mode/SKILL.md#Autonomy` |
| `tutorial/11-improve-the-environment.md` | `correct` 的 mistake classes 与阶梯；`reflect` 的三 reviewer、Accepted/Rejected/Backlog、批准门 | `skills/correct/SKILL.md`、`skills/reflect/SKILL.md` |
| `tutorial/after-tutorial.md` | 指向手册章节的导航 | — |

## 手册

| 单元 | 讲的 pstack 机制 | 源码 |
|---|---|---|
| 1.1 | prompt 五项 | `docs/guide/02-poteto-mode.md` |
| 1.2 | mode 与 playbook；23 个 playbook 的匹配；todo 与 `skip` | `skills/poteto-mode/SKILL.md#Playbooks` |
| 1.3 | AskQuestion 分类：可观察的事实不问人；operator-only 才问 | `skills/poteto-mode/SKILL.md#Non-negotiables` |
| 1.4 | brief 九字段与 pstack 不写 spec 的边界 | `playbooks/orchestrate.md#The brief`、`docs/guide/02-poteto-mode.md` |
| 2.1 | named data shape | `principle-model-the-domain`、`skills/poteto-mode/SKILL.md#Non-negotiables` |
| 2.2 | `architect` Phase B / public surface / design red flags | `skills/architect/SKILL.md` |
| 2.3 | `feature` step 3 throughput checkpoint 与 `multi-phase-plan` 的 per-PR 计划 / You see / Verify | `playbooks/feature.md`、`playbooks/multi-phase-plan.md` |
| 3.1 | Subagents 默认值：后台、agent 模式、文件指针、角色模型 | `skills/poteto-mode/SKILL.md#Subagents` |
| 3.2 | brief 九字段 | `playbooks/orchestrate.md#The brief` |
| 3.3 | fresh subagent / resume 的边界 | `skills/poteto-mode/SKILL.md#Subagents` |
| 3.4 | `orchestrate` roles、`preferences.md`、one writer per worktree | `playbooks/orchestrate.md` |
| 4.1 | `tdd` workflow 与 skip | `skills/tdd/SKILL.md` |
| 4.2 | `prove-it-works` | `skills/principle-prove-it-works/SKILL.md` |
| 4.3 | `benchmark-checklist` 七问与 ballpark 例外 | `skills/benchmark-checklist/SKILL.md` |
| 4.4 | verdict、`PASS+NOTES`、patch-id、contiguous verified run | `playbooks/shipping.md` |
| 5.1 | `interrogate` scope / intent / rubric / 四桶 | `skills/interrogate/SKILL.md` |
| 5.2 | `no-comments` 与 Comment Sicko | `skills/no-comments/SKILL.md`、`agents/comment-sicko.md` |
| 5.3 | `shipping` | `playbooks/shipping.md` |
| 5.4 | `babysit` / `session pickup` / `pause safely` | `playbooks/babysit.md`、`session-pickup.md`、`pause-safely.md` |
| 6.1 | `correct` | `skills/correct/SKILL.md` |
| 6.2 | `reflect` | `skills/reflect/SKILL.md` |
| 6.3 | `encode-lessons-in-structure`；`create-verification-skill` / `maintain-verification-skill` 与 feature map | 对应 SKILL.md |

## 明确不覆盖

- **spec**：pstack 不写 spec。`docs/guide/02-poteto-mode.md` 的原话是 "You don't write a spec"；根 `README.md` 说 "the best spec is code"。手册只讲 prompt 五项和 `multi-phase-plan` 的计划文件。
- **发布 / 灰度 / 回滚 / 发布后监控**：pstack 停在 verified landing。部署只是 **Always pause** 里的一个不可逆动作（`skills/poteto-mode/SKILL.md#Autonomy`）。

这些话题如果出现在别的方法里，不要写成 pstack 的能力；手册里只保留一句边界说明。
