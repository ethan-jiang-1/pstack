# 严格重写约束

目标：手册只使用 pstack 自身的概念。没有 pstack 对应物的方法元素，删除或降级为普通说明，不再发明名字。

## 允许的术语（只用这些）

- mode、`poteto-mode`
- playbook、`investigation`、`bug fix`、`feature`、`refactoring`、`multi-phase plan`、`prototype`、`visual parity`、`perf issue`、`hillclimb`、`runtime forensics`、`trace forensics`、`eval`、`babysit`、`shipping`、`autonomous run`、`orchestrate`、`autopilot-full`、`autopilot-stack`、`session pickup`、`pause safely`、`opening a PR`、`worktree cleanup`、`authoring a skill`
- skill、`architect`、`arena`、`swarm`、`interrogate`、`how`、`why`、`teach`、`recall`、`ketchup`、`sharpen`、`in-your-own-words`、`tdd`、`prove-it-works`、`benchmark-checklist`、`show-me-your-work`、`figure-it-out`、`correct`、`reflect`、`encode-lessons-in-structure`、`create-verification-skill`、`maintain-verification-skill`、`setup-pstack`、`unslop`、`technical-writing`、`no-comments`、`poteto-help`、`automate-me`、`blast-radius`、`make-bot-ui`
- principle、24 个 `principle-*`
- subagent、`poteto-agent`、`comment-sicko`
- brief、GOAL / SCOPE / CONTEXT / ACCEPTANCE / VERIFY / TIMEBOX / FORBIDDEN / REPORT
- prompt 的五项：goal、done check、proof、what you already know、real constraints
- frontier（只指 PR 栈的 merge frontier）、ready work
- verdict、`PASS` / `PASS+NOTES` / `FAIL`
- patch-id、contiguous verified run
- feature map
- decision trail / decision log（`decisions.tsv`）
- control surface / control skill（只指 `control-ui` / `control-cli`）
- Always pause
- todo、`skip: <reason>`
- role：`judgment and prose`、`hardest tasks`、`how explorer`、`how explainer`、`why investigators`、`why synthesizer`、`reflect tooling`、`reflect judgment, divergent, synthesizer`、`arena runners`、`arena cross-judge pool`、`swarm workers`、`architect runners`、`interrogate reviewers`

## 禁止的术语（不再使用）

request card、change card、word card、behavior card、behavior map、module shape card、feedback loop card、promise-to-evidence map、merge review packet、two-axis review、failure routing record、release risk decision、operational signal、friction evidence ledger、friction improvement proposal、promise retirement、accumulated corpus、work package、vertical slice、blocking edge、completion check、observation point、coverage boundary、independent oracle、evidence ladder、phase boundary、semantic conflict、authoritative artifact、artifact owner、task context packet、collaboration agreement、convergence and handoff、structural constraint、environment as product、direction drift、shared problem、interview、ready set、切口、控制面（用于父 agent 的委派控制时）、patch fingerprint

## 写法

1. 先给 pstack 的原词，再解释。例：“`brief` 的 SCOPE 字段……”，不写“任务上下文包”。
2. 不给概念起新名字。要描述一组动作时，直接写 skill 名、playbook 名或字段名。
3. 没有 pstack 对应物的话题（spec、发布、运行监控、承诺退役），写成“pstack 不覆盖这个；它的边界是……”，不发明方法。
4. 每个方法段落末尾给源码位置：`skills/<name>/SKILL.md`、`skills/poteto-mode/playbooks/<name>.md`、`agents/<name>.md`、`docs/guide/<n>.md`。
5. 解释可以抽象，但抽象后的句子必须能落回一个 pstack 机制；落不回去就删。
