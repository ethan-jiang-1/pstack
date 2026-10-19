# pstack 手册：结构与进度

> **状态：严格重写中（2026-10-19）。** 目标不是写一本独立方法书，而是把 pstack 自己的概念讲清楚。没有 pstack 出处的章节删除；没有 pstack 对应物的名字不再发明。写作约束见 [`STYLE.md`](./STYLE.md)，术语见 [`CONTEXT.md`](./CONTEXT.md)。

## 教程：12 步

| 步骤 | 讲什么 | 主要源码 |
|---|---|---|
| 00 | prompt 的五项：goal、done check、proof、what you already know、real constraints | `docs/guide/02-poteto-mode.md` |
| 01 | 用 `/poteto-mode` 还是单 skill；AskQuestion 什么时候该问 | `skills/poteto-mode/SKILL.md#Non-negotiables` |
| 02 | mode 匹配 playbook，把步骤逐字复制进 todo，跳过写 `skip` | `skills/poteto-mode/SKILL.md#Playbooks` |
| 03 | 先命名 data shape；可观察结果写在 plan 的 verify block | `principle-model-the-domain`、`feature.md` |
| 04 | `architect` Phase B：public surface、`not implemented` 草图、design red flags | `skills/architect/SKILL.md` |
| 05 | `feature` step 3 的 throughput checkpoint；`multi-phase-plan` 的 one PR = one change | `feature.md`、`multi-phase-plan.md` |
| 06 | `tdd` 的红绿回路，以及测试路径不便宜时跳过 | `skills/tdd/SKILL.md` |
| 07 | 用 brief 派 subagent；`poteto-agent`、fresh、角色行 | `poteto-mode#Subagents`、`orchestrate.md#The brief` |
| 08 | `prove-it-works`；`shipping` 的 verdict、PASS / PASS+NOTES / FAIL、patch-id | `principle-prove-it-works`、`shipping.md` |
| 09 | `interrogate`：scope、frozen intent、rubric + code-quality、四个桶 | `skills/interrogate/SKILL.md` |
| 10 | `shipping` 落地；`babysit` 推到 merge-ready；部署只是 Always pause | `shipping.md`、`babysit.md` |
| 11 | `correct` 与 `reflect`：把重复错误和会话经验写回结构 | `skills/correct/SKILL.md`、`skills/reflect/SKILL.md` |

教程后导航：`tutorial/after-tutorial.md`。

## 手册：6 章

| 章 | 单元 | 主要源码 |
|---|---|---|
| 1 Prompt、mode 与 todo | 1.1 prompt 五项；1.2 mode/playbook；1.3 todo、skip 与回复；1.4 AskQuestion 分类 | `docs/guide/02`、`poteto-mode` |
| 2 Architect、feature 与 multi-phase-plan | 2.1 data shape；2.2 architect Phase B；2.3 feature step 3；2.4 multi-phase plan | `architect`、`feature.md`、`multi-phase-plan.md` |
| 3 Subagent 与 brief | 3.1 Subagents 默认值；3.2 brief 八字段；3.3 fresh / resume；3.4 orchestrate roles | `poteto-mode#Subagents`、`orchestrate.md` |
| 4 验证与 verdict | 4.1 tdd；4.2 prove-it-works；4.3 benchmark-checklist；4.4 verdict / patch-id / contiguous run | `tdd`、`prove-it-works`、`benchmark-checklist`、`shipping.md` |
| 5 评审与落地 | 5.1 interrogate；5.2 no-comments；5.3 shipping；5.4 babysit / session pickup / pause safely | `interrogate`、`no-comments`、`shipping.md`、`babysit.md`、`session-pickup.md`、`pause-safely.md` |
| 6 环境 | 6.1 correct；6.2 reflect；6.3 encode-lessons-in-structure + create/maintain-verification-skill | `correct`、`reflect`、`encode-lessons-in-structure`、`create-verification-skill` |

## 明确不覆盖

pstack 不写 spec，不做发布 / 灰度 / 回滚 / 发布后监控，也没有承诺退役记录。它停在 verified landing；部署只是 **Always pause** 里的一个不可逆动作。这些话题不建章节。

## 进度

- [x] `STYLE.md`、`CONTEXT.md`、`README.md`
- [x] tutorial 00–11 与 after-tutorial（重写中，逐页验收）
- [x] manual 1–6（重写中，逐页验收）
- [ ] 32 张图：保留有 pstack 标签的，删除或重画依赖自造概念的
- [ ] `PROVENANCE.md`：逐单元源码映射，去掉 EXTENSION 分类
- [ ] 全量术语扫描：禁止词 0 命中
- [ ] 链接、锚点、SVG、计数检查
- [ ] 独立 review 并即时修正
