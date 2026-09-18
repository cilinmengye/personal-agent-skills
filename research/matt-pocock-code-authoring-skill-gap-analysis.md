# Matt Pocock 工程 Skills 与个人代码编写 Skills 的边界研究

> 研究日期：2026-09-16
>
> Matt Pocock 仓库快照：[`959a8e9`](https://github.com/mattpocock/skills/tree/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260)
>
> 范围：Matt Pocock 的 engineering skills、仓库中的 `andrej-karpathy-skills` 与 `code-comments`。本文只做研究，不修改 Skill。
>
> 实施状态：研究建议现已落实为 [`write-maintainable-code`](../skills/engineering/write-maintainable-code/) 与改造后的 [`code-comments`](../skills/engineering/code-comments/)；下文保留当时的对照依据。

## 结论

原 `andrej-karpathy-skills` 的四项原则大部分已经被 Matt 的完整工程流程覆盖：先澄清、限制范围、用最小实现前进、以测试和审查验证结果。继续保留它，只会让同一约束在多个 Skill 中重复，且难以判断谁是规范来源。

但 Matt 的体系仍存在一个明确空白：它擅长决定**做什么、以什么切片做、在哪里测试、如何审查**，却没有系统指导 Agent 在一次 red-green 循环的 green 阶段，**如何写出内部可理解、可维护、可移植且不过度设计的实现代码**。新的独立 Skill 应占据这个空白，而不是再次讲计划、TDD、目录结构或代码审查。

`code-comments` 也有独立价值。它应以“记录代码无法表达的契约与理由”为语义核心，同时保留对人友好的**条件性视觉结构**：项目 formatter 与既有风格优先；没有约定时才使用默认分隔；只有真实阶段转换或稳定顶层区域才增加视觉边界。问题不在视觉呼吸感本身，而在把个人偏好的固定数量误作跨语言正确性要求。

## 1. Matt 的体系实际在解决什么

Matt 把 engineering skills 分成用户显式调用的流程驱动器，以及模型可以按情境调用的参考或局部循环；官方 README 明确列出了这两组，并将主流程组织为“讨论与规格 → 票据 → 实现 → 审查”。这是一个**工程工作流系统**，而不是一本完整的编码风格手册。[工程 Skills README](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/README.md)

主链路可以概括为：

```text
grill-with-docs
      ↓
   to-spec
      ↓
  to-tickets ── 每张票是可验证的纵向切片
      ↓
  implement ─── 调用 TDD、类型检查和测试
      ↓
 code-review ── 分开检查 Standards 与 Spec
```

`ask-matt` 把这条链路和 wayfinder、triage、diagnosing-bugs、improve-codebase-architecture 等入口连接起来，并把 domain-modeling 与 codebase-design 定义为底层共享词汇。[`ask-matt`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/ask-matt/SKILL.md)

各层的主要职责是：

| 层次 | 代表 Skill | 主要约束 |
|---|---|---|
| 问题澄清 | `grill-with-docs`、`wayfinder`、`prototype`、`domain-modeling` | 暴露未知、明确术语，用可运行原型回答纸面上无法回答的问题 |
| 工作切片 | `to-spec`、`to-tickets` | 把决定持久化；用小而完整的 tracer-bullet 纵向切片，避免按技术层横切 |
| 实现编排 | `implement` | 从已决定的 spec/ticket 开始，驱动 TDD、类型检查、单测、全套测试和审查 |
| 行为反馈 | `tdd`、`diagnosing-bugs` | 通过 public seam 建立紧反馈；一条行为一个 red-green slice；调试先构造可复现信号 |
| 结构设计 | `codebase-design`、`improve-codebase-architecture` | 深模块、小接口、locality、deletion test，以及测试应通过接口发生 |
| 事后审查 | `code-review` | 独立检查 repo standards 与 spec；内置 Fowler smell baseline |

来源：[`to-tickets`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/to-tickets/SKILL.md)、[`implement`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/implement/SKILL.md)、[`tdd`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/tdd/SKILL.md)、[`codebase-design`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/codebase-design/SKILL.md)、[`code-review`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/code-review/SKILL.md)。

## 2. `andrej-karpathy-skills` 的重叠矩阵

已删除的 `andrej-karpathy-skills` 有四条原则。逐项对照后，结论如下：

| 现有原则 | Matt 中的覆盖 | 覆盖程度 | 判断 |
|---|---|---:|---|
| Think Before Coding | `grill-with-docs` 澄清；`wayfinder` 解决大型未知；`prototype` 用运行结果回答设计问题；`domain-modeling` 消除术语含糊；`to-spec` 固化决定 | 高 | 应删除重复。需要注意：`implement` 明确消费已经决定的工作，不负责重新采访或推翻计划。 |
| Simplicity First | TDD 只写使当前测试通过的代码并禁止 speculative feature；architecture scan 先做 YAGNI scope；code-review 检查 Speculative Generality、Middle Man 等 smell；codebase-design 不为单一 adapter 虚构 seam | 高 | “不提前设计未出现的需求”已被覆盖。 |
| Surgical Changes | 一次 `implement` 只处理一张 ticket；纵向切片限制工作范围；code-review 的 Spec 轴检查 scope creep | 中高 | 已覆盖任务边界，但 Matt 没有明确要求“每一行改动都可追溯至请求”。这可作为通用 guardrail，而不值得单独保留整个 Skill。 |
| Goal-Driven Execution | spec/ticket acceptance criteria；TDD 的 red-green 信号；diagnosing-bugs 的 red-capable command；code-review 的固定比较点 | 很高 | 已被更具体、可执行的循环取代。 |

Matt 的官方说明也确认：`implement` 只是把已决定的工作变成 commit；它运行 `tdd` 与 `code-review`，并不重新设计计划。[AI Hero：`implement`](https://www.aihero.dev/skills-implement) `tdd` 则强调一条测试、一份最小实现、再进入下一行为，而不是先批量想象全部测试。[AI Hero：`tdd`](https://www.aihero.dev/skills-tdd)

### 不应从旧 Skill 机械迁移的部分

- “如果不清楚就停下问”不应无条件发生在实现阶段；若上游 spec 已明确，重复提问会破坏流程分工。
- “越少代码越好”不能变成行数竞赛。复杂性可能应该被集中在深模块内部，从而让调用者更简单。
- “只改必要的行”不能阻止完成一个纵向切片所必需的跨层修改，也不能阻止清理由本次修改制造的 orphan。
- “先做计划”不能和 `implement` 再建立一套平行的计划协议。

## 3. 值得建立的新 Skill：代码实现内部的工程判断

### 3.1 Matt 没有系统覆盖的区域

以下判断并非说 Matt 的 Skills 会产出坏代码，而是说明其规范没有形成可执行、唯一的实现期指导。改造后的 [`write-maintainable-code`](../skills/engineering/write-maintainable-code/SKILL.md) 专门承担这一职责：

1. **实现内部的数据与控制流**：何时用表驱动/数据表示替代不断增长的条件分支，怎样让状态转换和不变量可见。
2. **策略与机制的分离**：何时一个 literal 是稳定的领域事实，何时是被硬编码的策略、环境或用户选择；如何避免把所有东西都反向配置化。
3. **副作用和错误路径**：在何处执行 I/O、修改状态、处理资源与失败，使 happy path 和 cleanup path 都能被局部理解。
4. **复用与依赖决策**：先搜索现有能力、标准库和项目依赖，同时评估新依赖的版本、安全、构建和维护成本，而不是把“不重复造轮子”理解成“总要加库”。
5. **可移植性与性能的条件性取舍**：默认使用标准、清晰的实现；只有仓库目标或测量证据表明这里是热路径时才专用化，并把平台特化封装在窄区域。
6. **实现期可读性**：命名、类型、局部抽象和注释怎样降低维护者恢复意图所需的推理，而不是满足固定格式。

这个空白在 Matt 的资料中是可以直接看见的：`tdd` 要求 green 阶段“只写足够通过测试的实现”，并把重构移出 red-green loop；`code-review` 之后只输出审查报告；`codebase-design` 明确是词汇与原则参考，不是具体实现过程。[`tdd`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/tdd/SKILL.md) [`code-review`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/code-review/SKILL.md) [AI Hero：`codebase-design`](https://www.aihero.dev/skills-codebase-design)

因此，新 Skill 最有价值的时机是：**行为、public seam 和任务范围已经确定，Agent 即将编写或重写 production implementation 时。**

### 3.2 建议的职责边界

| 新 Skill 应负责 | 新 Skill 不应负责 |
|---|---|
| 编码前短暂停顿，读取局部代码和项目规范，识别真实约束 | 重新采访用户或重写 spec |
| 搜索项目已有实现、标准库和已批准依赖 | 代替 `research` 做广泛技术选型 |
| 选择最简单但能局部化变化的表示和控制流 | 代替 `evolve-codebase-structure` 决定目录/package 演化 |
| 区分领域常量、配置、策略、运行时输入 | 一律禁止 literal，或把所有值配置化 |
| 让状态、不变量、错误与资源生命周期显式 | 代替 `tdd` 选择 seam 或编写测试流程 |
| 仅在有证据时引入抽象、依赖或平台优化 | 以未来可能变化为理由构建框架 |
| 完成当前 slice 后做一次实现质量自检 | 代替 `code-review` 的 Standards/Spec 审查 |

### 3.3 可吸收的软件工程原则

这些原则应被写成**决策规则**，而不是口号：

- **小而可组合，但不要制造浅模块。** Unix 的模块化与组合原则强调简单部件、干净接口和可连接性；Matt 的 deep-module 原则则提醒：外部接口应小，复杂性可以集中在内部。二者合并后的规则应是“让公开表面小而可组合，让相关实现保持 locality”，而不是“不断拆小文件”。[Eric S. Raymond《The Art of Unix Programming》：Unix 基本原则](https://www.catb.org/esr/writings/taoup/html/ch01s06.html) [`codebase-design`](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/codebase-design/SKILL.md)
- **简单不是最短，而是可推理。** 优先直白的数据流、低嵌套和少特殊分支；必要复杂性放在能够隐藏它的 owner 中。Unix 的 transparency、representation 与 robustness 原则主张把知识放进数据，让逻辑更简单，并通过透明和简单获得健壮性。[Unix 基本原则](https://www.catb.org/esr/writings/taoup/html/ch01s06.html)
- **按变化原因隐藏决策。** Parnas 的原始论文把模块化目标定义为灵活性、可理解性和缩短开发时间，并比较按处理步骤与按信息隐藏进行分解的差异。对“硬编码”的正确问题不是“有没有 literal”，而是“这个会变化的决定由谁拥有、改变它会扩散到哪里”。[Parnas 1972 原论文](https://doi.org/10.1145/361598.361623)
- **分离 policy 与 mechanism。** 用户选择、部署环境和产品策略不应散落在底层机制中；但只有真实变化轴才值得参数、配置或 adapter。[Unix Rule of Separation](https://www.catb.org/esr/writings/taoup/html/ch01s06.html)
- **先得到清晰基线，再优化已证实的热点。** Unix 的 optimization rule 是先原型/工作，再优化；GNU 官方标准也建议尽量使用标准接口，同时允许在确有维护性、能力或性能收益时使用扩展。对 GPU/推理项目，性能与设备特化可能就是需求，不应被“可移植性优先”机械否决；应把特化隔离并用测量守护。[Unix Rule of Optimization](https://www.catb.org/esr/writings/taoup/html/ch01s06.html) [GNU Coding Standards](https://www.gnu.org/prep/standards/standards.html)
- **复用前先搜索，采用依赖前评估代价。** Linux 官方规范明确提醒使用已有宏而非手写变体；外部依赖还会带来升级、安全和兼容成本，因此 NIH 与“依赖一切”都不是正确默认值。[Linux kernel coding style](https://docs.kernel.org/process/coding-style.html) [Go 官方依赖管理](https://go.dev/doc/modules/managing-dependencies)
- **先让代码表达意图，注释补上代码表达不了的事实。** 应注释不变量、锁/内存/顺序约束、单位、非显然取舍和性能原因；不应逐行复述语法。[Linux kernel coding style：Comments](https://docs.kernel.org/process/coding-style.html#commenting) [Google C++ Style Guide：Comments](https://google.github.io/styleguide/cppguide#Comments)

### 3.4 需要明确反对的误读

```text
小即是美      ≠ 每个函数都必须很短、每个概念都必须单独成文件
KISS          ≠ 用最快写出的硬编码补丁
避免硬编码    ≠ 所有 literal 都进入配置文件
复用          ≠ 为十行稳定逻辑引入重量级依赖
可移植性优先  ≠ GPU 热路径不得使用平台能力
原型优先      ≠ 每次小修改都先造一次 throwaway prototype
写注释        ≠ 给每个分支和逻辑阶段加标题
```

尤其要避免把“选择一种更复杂但更优雅的实现”作为目标。更好的说法是：**允许实现内部承担有价值的复杂性，只要它确实减少了调用者、未来修改和验证所承担的总复杂性。** 这与 deep module 的 leverage/locality 标准一致。

## 4. `code-comments` 的重叠与问题

研究时的旧版 `code-comments` 的正确核心是：说明 why，而不是复述 what；记录并发、内存、顺序、性能和单位等非显然契约；避免无信息量的 Args/Returns。这些内容与新的代码编写 Skill 有联系，但仍可单独存在，因为它面向的是**代码旁文档的内容和语义**，不是实现算法的选择。改造后的版本见 [`code-comments`](../skills/engineering/code-comments/SKILL.md)。

不过，研究时的版本不适合原样保留：

| 当前规则 | 证据与问题 | 建议 |
|---|---|---|
| 类内方法之间固定 2 个空行 | PEP 8 明确是 1 个空行；函数体内的空行应 sparingly 使用。[PEP 8](https://peps.python.org/pep-0008/#blank-lines) | 服从项目 formatter/style；保留视觉分隔，但删除跨语言固定数字。 |
| 每次“语义责任变化”都空一行，且多语句分支必须写前导注释 | Linux kernel 官方反而警告函数体内过多阶段注释，复杂到需要大量分段说明时应重新审视函数；Google 也要求避免明显注释。[Linux](https://docs.kernel.org/process/coding-style.html#commenting) [Google C++](https://google.github.io/styleguide/cppguide#Implementation_Comments) | 在 validation → mutation、setup → execution、execution → cleanup 等真实阶段转换处保留空行；分支注释只说明条件无法表达的业务意图、不变量或顺序原因。 |
| 所有类必须有 docstring；所有文件必须有固定 module header | PEP 8 要求 public API 文档，但 Google Python 明确指出测试等场景不应写只重复名字的 docstring。[Google Python Style Guide](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings) | 按 public contract、非显然语义与本地规范决定，不做无信息量 boilerplate。 |
| 固定 zone anchor 和 3 个空行 | 没有跨语言的一手规范支持，而且会与 formatter 和 repo conventions 冲突。 | 改为条件性默认：只用于已有多个稳定顶层区域、且拆分会损害 locality 的文件；优先项目 marker，不固定周围空行。 |
| CUDA 指针必须标 `[Device]`/`[Host]` | 明确内存可访问性很有价值，但 CUDA 还有 managed/unified/pinned memory；二元标签不是完整模型。[NVIDIA CUDA Programming Guide](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/programming-model.html#gpu-memory) | 记录实际 memory space、ownership、lifetime、alignment 与同步要求；优先采用项目既有注解/类型。 |
| 只允许固定的 WARNING/PERF/NOTE/TODO/FIXME token | token 是否有意义取决于项目工具、lint 和 issue 约定；强制替换所有其他标签可能破坏本地规范。 | 先读取仓库约定；没有约定时才使用一个小而一致的默认集合。 |

因此，`code-comments` 最合理的最终定位是：

> 在 Python/C++/CUDA 系统代码中记录调用者或维护者无法从类型和实现直接推导出的契约、理由、不变量、资源/内存归属、并发与性能约束，并服从项目现有格式与文档工具。

它可以负责**条件性视觉呼吸感**，但衡量标准是扫描性和信息增量，而不是固定空行与注释数量。真实阶段转换可以用空行，稳定顶层区域可以用 zone anchor；若代码需要大量标签才能读懂，仍应首先检查命名、控制流和职责边界。

## 5. 与 Matt Skills 的建议组合方式

新的代码编写 Skill 应是**用户显式选择的实现期参考**，而不是另一个上层 orchestration skill：

```text
Matt: grill / spec / tickets
              ↓
用户显式调用 implement + 个人代码编写 Skill
              ↓
Matt: TDD 决定行为反馈循环
个人 Skill: 约束 green implementation 的内部质量
可选 code-comments: 约束真正需要留下的代码旁文档与条件性视觉结构
              ↓
Matt: code-review 做 Standards + Spec 审查
```

推荐调用方式是由用户在同一请求中同时点名，例如：

```text
$implement $<new-code-authoring-skill> 实现 issue #42
```

涉及系统级注释契约时再增加：

```text
$implement $<new-code-authoring-skill> $code-comments 实现 issue #42
```

两个个人 Skill 都应增加：

```yaml
# agents/openai.yaml
policy:
  allow_implicit_invocation: false
```

同时应修改 frontmatter description，移除 `whenever`、`including small edits`、`consult before producing any code` 等自动触发语气。只有 YAML 禁止隐式调用而正文仍声称“任何写代码都必须使用”，会形成互相矛盾的使用契约。Matt 自己也通过 `agents/openai.yaml` 将 `implement`、`to-spec`、`to-tickets`、`improve-codebase-architecture` 等上层流程设为显式调用。[Matt engineering README](https://github.com/mattpocock/skills/blob/959a8e9f1edc3adbe2f7e3054bb6fbefa6696260/skills/engineering/README.md)

## 6. 后续改造的验收标准

### 新代码编写 Skill

- 不再复述 Karpathy 四原则或 Matt 主流程。
- 能在写代码前产生一个很短的 implementation choice，而不是长篇设计文档。
- 能区分稳定领域常量与错误硬编码。
- 能区分有价值的内部复杂性与不必要抽象。
- 对复用、依赖、可移植性和性能给出条件性判断，而非绝对命令。
- 不按文件行数、函数行数或抽象数量评分。
- 明确服从仓库现有标准、public seam、spec 与性能目标。

### `code-comments`

- 只对新增信息的注释提出要求。
- 保留条件性视觉结构，同时让 formatter、语言规范和项目约定决定精确空行与 marker 形式。
- 只在真实阶段转换处增加函数内间隔；zone anchor 只用于拆分会损害 locality 的稳定顶层区域。
- CUDA 文档覆盖 memory space、ownership、lifetime、alignment、同步，而不是只用 Host/Device 二分。
- 默认遵循本地 token、docstring 与文档工具约定。
- 不通过增加注释掩盖本应由命名、类型或控制流解决的问题。

## 最终判断

可以删除现有 `andrej-karpathy-skills` 的内容并将其位置改造成一个新的代码编写 Skill，但不建议继续使用原名字：Karpathy 原则是问题诊断的起点，不再能准确描述新 Skill 的职责。

新的 Skill 应填补一句话所概括的空白：

> **在行为与接口已经决定后，强迫 Agent 暂停直接堆代码，选择能够局部化变化、显式表达状态和约束、复用得当且不过度设计的最小可维护实现。**

`code-comments` 则应独立收窄为：

> **记录代码本身无法可靠表达、但未来维护者必须知道的事实，并用服从项目约定的条件性视觉结构帮助人快速扫描。**

这两个 Skill 与 Matt 体系是互补关系：Matt 管流程、反馈、接口与审查；个人 Skill 管实现内部的工程品味和代码旁契约。
