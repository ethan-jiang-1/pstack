# `typescript-best-practices`

> 源码：[`skills/typescript-best-practices/SKILL.md`](../../skills/typescript-best-practices/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [系统总览](../system/README.md)

## 一句话

把 `type-system-discipline` 的原则翻译成 TypeScript 的 16 条语法级规则，分成表示、信任边界、闭合与派生、相邻纪律四组。每条规则在正文里只有一行，例子和细化放在 `references/patterns.md`。

## 它解决什么

TypeScript 里"类型检查通过"和"值是对的"是两件不同的事。`as`、`any`、可选字段袋、手写的逐属性 type guard，都会把验证责任混进形状里：编译器被说服了，运行时没有。同一个团队里 TS 写法差异还会变成 review 争论，因为每个人对"够不够类型安全"的直觉不同。

这个技能的作用是把原则落成可引用的具体形状：用 `kind` 判别式而不是布尔加可选字段，用 `unknown` 而不是 `any` 接外部数据，用 schema 推断类型而不是维护重复的 interface 和 guard。它给出的不是新理论，而是一张可以逐条指认的写法表。

## 核心机制

### 1. 先应用原则

正文第一行就要求先应用 `type-system-discipline` 这个 principle skill。表格是这个原则的 TypeScript 实例，不是替代品。`boundary-discipline` 也单独出现在边界规则里。原则管判断，这张表管语法。

### 2. 四组规则

十六条规则分成四组。**表示**：discriminated unions、branded types、constructive modeling、simplest total type。**信任边界**：`unknown`、schemas before guards、no `as`、type guards、narrowing hierarchy、boundary validation。**闭合与派生**：exhaustiveness、`satisfies`、schema-derived types。**相邻纪律**：object args、real tests、structured telemetry，这一组不直接从类型原则推出来。

### 3. 表示轴的动作

判别式让变体共享一个字段名、每个值唯一，于是矛盾状态构造不出来，而不是靠可选字段加运行时检查。branded types 统一用 `& { readonly __brand: "X" }` 的形状，在边界验证一次，下游直接信任。constructive modeling 用 `[T, ...T[]]` 表示非空、`[T, T][]` 表示偶数长度、`start` 加 `duration` 表示区间，让坏值在构造阶段就不存在。`simplest total type` 是反向约束：只要 `T[]` 上每个操作都是全函数，就保留 `T[]`；只有当松类型逼出 `!`、cast 或 "should never happen" 抛错时，才加强到 `NonEmpty<T>`。

### 4. 信任边界的顺序

外部数据一律 `unknown`，包括 RPC、`JSON.parse`、`postMessage`、IPC、文件内容、环境变量、数据库结果。先用仓库已有的运行时 schema 库并从中推断类型（如 `z.infer`），再考虑手写 guard。guard 必须真的验证它声称的事，撒谎的 guard 比 `as` 更糟，因为 bug 藏在一个声称安全的名字后面。narrowing 有明确层级：判别式 switch、`in`、`typeof` 或 `instanceof`、用户定义 guard、`as`，从最好到最后。解析只发生在数据进入的位置，`Record<string, unknown>` 到那次 parse 为止，内部信任类型。

### 5. 闭合与派生

default 分支里写 `const _exhaustive: never = x;`，新增变体时编译器报错。`satisfies` 用在校验值但不放宽字面量类型的地方。声明新 interface 之前，先看 `Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof` 能不能从已有类型派生。

### 6. 正文与例子分工

`SKILL.md` 只有表格和一行指向 `references/patterns.md` 的说明。真正需要判断的细节都在例子文件里：brand 必须沿用同一形状、判别字段名选一个并固定、`TimeRange` 用 `start` 加 `durationMs` 且默认不要品牌化、`safeParse` 用于失败是预期分支、不要为一个 guard 新增 schema 依赖、`as` 的重构路径按缺失判别式、过宽类型、未类型化边界、确实无法表达四类排查。

### 7. 作用域声明

frontmatter 里有 `paths: ["**/*.ts", "**/*.tsx"]`，把技能限定在 TypeScript 文件。但同一个 frontmatter 又设了 `disable-model-invocation: true`，所以它不会自己加载。`docs/guide/05-build-and-clean.md` 明确写了 "It doesn't load on its own"，需要在任务涉及 `.ts` 或 `.tsx` 时点名 `/typescript-best-practices`。`paths` 在这里表达范围，不是触发器。

## 设计意图

- **原则与语法分离。** `type-system-discipline` 是语言无关的判断，这张表是 TypeScript 的语法。换语言时原则留下，表需要重写。
- **最短的正文。** 表格让 agent 以很低成本加载全部规则，细节按需进 `references/`，这也是仓库的通用结构。
- **表示优先于校验。** 构造不出来的非法值不需要运行时 guard，错误被提前到编译期。
- **反向规则防过度建模。** `simplest total type` 阻止 branded type 与 `NonEmpty` 变成反射动作，避免把简单签名复杂化。
- **边界只解析一次。** 内部信任类型，深层调用链不重复校验，这是 `boundary-discipline` 的同一条判断。
- **相邻纪律一起放进表里。** 测试与遥测放在类型表末尾，因为 TS 代码最常撒谎的地方是 mock 和 `console.log`，不是类型标注。

## 依赖关系

- **先应用 `type-system-discipline`。** 这是正文的硬要求；`boundary-discipline` 在边界行里被点名，`patterns.md` 开头也指向两者。
- **例子文件是正文的一部分。** `SKILL.md` 末尾只有一句 "Examples: `references/patterns.md`"，但离开它，几条绝对规则没有例外说明。
- **被谁引用。** `poteto-help` 的技能表、`docs/guide` 的 build 章节与 README 的 skill 表都指向它；`poteto-mode` 的常规路由列表里没有它，所以它主要靠用户点名或指南提示。
- **独立安装的声明有出入。** 仓库 README 把它列在"不调用其他 pstack skill、可单独安装"的一组，但正文第一行要求先应用一个 principle skill。这个冲突是文本层面的；实际上 principle 是判断词汇，缺了它表仍然可用，只是少了那层判断。

## 使用边界

- **只覆盖 TypeScript 文件。** `paths` 限定 `.ts` 与 `.tsx`；命名、格式、lint 配置不在范围内，它不是 style guide。
- **不自动加载。** 需要在任务开始时点名，否则 agent 不会主动套用这张表。
- **不能只读表。** "No `as` casts" 这类绝对规则在 `patterns.md` 里有真实的逃逸口：确实无法表达时用 branded type 或 `satisfies`。只读 `SKILL.md` 会把它当成无条件禁令。
- **两条相邻规则依赖仓库现状。** "schemas before guards" 要求仓库已有 schema 库，并写明不要为一个 guard 新增依赖；"structured telemetry" 偏好结构化 logger，没有要求仓库已有。没有这些设施的项目只能退回到手写 guard 与项目现有日志。
- **热路径豁免最好有证据。** object args 在 per-frame render、tokenizer、parser 上不适用；源码没有要求证明，"这是热路径"的判断按 `explain-the-number` 的原则最好来自测量，而不是感觉。
- **不能替代边界设计。** 它规定解析发生的位置，不规定边界应该画在哪里；后者是 `boundary-discipline` 与 `architect` 的问题。

## 值得注意

- **`paths` 在禁用模型调用后是惰性的。** 字段看起来像自动作用域，实际不会带来任何自动加载。读者容易以为"改 `.ts` 时它自动生效"，而指南明确说不是。
- **表格压掉了论证。** 十六条一行一条，读起来干净，但"为什么"和"例外"都在另一个文件。两份文件必须一起读，否则绝对规则会变成教条。
- **相邻纪律容易被跳过。** 读者为类型而来，`real tests` 与 `structured telemetry` 两行在表尾，最可能被略过，而它们恰好是 TS 代码里最常见的谎言来源。
- **两条核心规则互相拉扯。** "constructive modeling" 要求用类型排除非法值，"simplest total type" 要求别过度加强。判据是"松类型是否逼出了 `!`、cast 或抛错"，这条判据本身仍需要判断。
- **README 的独立声明与正文不一致。** 一个说可单独安装，一个说先应用原则技能。降级是平滑的，但声明应当对齐。

## 关键洞察

1. **表示法优先于校验。** 能让坏值构造不出来，就不需要运行时 guard。
2. **边界解析一次，内部信任类型。** 重复校验是把类型系统当成不可信，也把成本摊到每条调用链。
3. **要最简的全函数类型，不是最强的类型。** 加强只在松类型逼出谎言时发生。
4. **撒谎的 guard 比 `as` 更危险。** 它带着"已验证"的名字，bug 因此更难被发现。
5. **原则管判断，表管语法。** 换语言时前者留下，后者重写。
