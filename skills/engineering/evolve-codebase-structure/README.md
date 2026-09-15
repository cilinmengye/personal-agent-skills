# Evolve Codebase Structure

`evolve-codebase-structure` 是一个由使用者主动启动的周期性架构审查 Skill。它关注的不是“某段代码能不能运行”，而是代码增长后是否仍然放在正确的文件、目录和包中。

它解决的典型问题是：Agent 知道测试应放进 `test/`、核心代码应放进 `project/core/`，却会继续向现有文件追加代码，不会主动判断 `project/core/` 内部是否已经出现了新的能力、生命周期或依赖边界。

## 它与 Matt Pocock Skill 的关系

本 Skill 的三阶段流程受 Matt Pocock 的 [`improve-codebase-architecture`](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture) 启发，但两者回答的问题不同：

| Skill | 核心问题 | 主要产物 |
| --- | --- | --- |
| `improve-codebase-architecture` | 哪些浅模块值得变深，接口和 seam 应如何改善？ | deepening 候选 |
| `evolve-codebase-structure` | 行为现在属于哪里，文件、目录、包和依赖方向应如何随项目生长？ | placement / split / merge / relocate / package-boundary 候选 |

二者互补。一个深模块完全可以由多个私有文件构成；“深模块”不等于“一个巨型文件”。本 Skill 专门补足从抽象原则到实际目录布局之间的落地判断。

本项目不是 Matt Pocock 仓库的官方扩展，也不要求向其仓库提交 PR。流程骨架注明来源，具体的结构判断、报告格式和文字均在本 Skill 中独立维护。

## 设计哲学

- **使用者掌握启动权。** 它只在你显式调用时运行，不会在普通编码任务中自动插入一次大型架构审查。
- **调查与施工分离。** 审查阶段不改源码，只在系统临时目录生成一份 HTML 报告；选定并讨论清楚后，才进入独立的实现流程。
- **证据高于整齐。** 依赖、变更历史、测试、构建与真实所有权比行数和目录对称更重要。
- **目录表达所有权。** 新行为归属于拥有其不变量和生命周期的模块，而不是归属于 Agent 当前打开的文件。
- **小公共接口，清晰私有实现。** 允许拆分实现文件，但不会为了缩短文件而增加公共接口或制造跳转。
- **允许“无需重构”。** 没有足够证据时，正确输出可以是当前结构合理，而不是为了填满报告硬造候选。

## 工作流

```text
显式调用
   ↓
1. Explore：检查树、依赖、历史、测试、构建和所有权
   ↓
2. HTML report：展示证据、候选结构、风险和首选项
   ↓  你选择一个候选
3. Grilling：逐轮解决所有权、接口、迁移和验证决策
   ↓  你确认共同理解
4. $to-spec → $to-tickets（需要时）→ $implement    # Codex
```

第 1、2 阶段只读取仓库。第 3 阶段仍不改源码，但在确有必要时可以通过 `domain-modeling` 更新 `CONTEXT.md`，或提议记录一份真正值得保留的 ADR。

## 一个必须承认的边界

周期性审查解决的是“发现并塑造结构工作”，不会自动阻止普通实现任务里的 append gravity。这个限制是有意的：如果把整套扫描、HTML 和访谈流程设为自动触发，每次小改动都会付出不必要的上下文和时间成本。

若你还希望 Agent 在每次实现时主动检查代码放置，正确做法是增加一个更小的、可由模型调用的 `code-placement` 参考 Skill，或在项目 `AGENTS.md` 中放一条指向仓库内结构规则的短指针。它只负责“实现前选位置、实现后复盘 touched area”，不生成报告也不启动 grilling。不要通过修改 Matt 的 `implement` 来混入这套逻辑；那会让个人变更与上游更新产生冲突。

当前 [`STRUCTURE-DESIGN.md`](STRUCTURE-DESIGN.md) 已保留这套判断语言。若后续创建 `code-placement`，应把共享规则提取为一个中立的单一来源，由两个 Skill 分别引用，避免复制两份逐渐漂移的规则。

## 如何调用

在 Codex 中显式选择或输入：

```text
$evolve-codebase-structure
```

最好附上方向，报告会更可执行：

```text
$evolve-codebase-structure 请检查 project/core。接下来准备增加多后端调度能力，哪些结构调整会让这次修改更容易？
```

周期复查时可以提供上次报告或基线：

```text
$evolve-codebase-structure 以上次 HTML 报告和 v0.8.0 为基线，只看新增或发生变化的结构问题。
```

报告默认位于系统临时目录，操作系统可能清理它。若要把它作为下次增量审查的基线，请把选定的报告复制到你自己的持久资料目录，并在下一次调用时传入路径；Skill 本身不会把审查产物写进项目仓库。跨周期报告会沿用候选 ID，并把条目标为 `new`、`changed`、`carried-forward`、`resolved` 或 `rejected`。

只想定期巡检而不进入访谈时，可以加上：

```text
$evolve-codebase-structure report-only，只生成报告，不进入 grilling。
```

## 什么时候运行

- 大功能开始前：围绕即将触碰的区域问“怎样让这次修改更容易？”
- 一轮功能快速增长或版本发布后：检查本轮变化是否形成新的内部能力。
- 同一个文件频繁冲突、反复膨胀，或不断加入不同依赖时。
- 新增运行时、硬件后端、语言、部署单元或独立维护责任时。
- 遗留项目补测试前：先判断现有所有权和测试接口是否合理。

对于高频开发项目，可以每两周或每月做一次有范围的检查；稳定项目按季度或在上述事件发生时运行即可。周期不是目标，及时发现“结构已跟不上变化”才是目标。

## 它不会做什么

- 不根据固定行数自动拆文件。
- 不为了目录好看而预建未来分类。
- 不把一个大文件机械切成许多互相跳转的小文件。
- 不在审查过程中直接重构源码。
- 不替代 `codebase-design` 对接口和 seam 的设计，也不替代后续的 spec、TDD 和 code review。

## 与 Matt Pocock Skills 一起安装

推荐把两个来源保持独立，并全局安装到 Codex。Matt 的仓库作为上游订阅，你的 GitHub 仓库作为个人扩展；不要把个人 Skill 塞进 Matt 的安装目录，也不需要 fork 他的仓库。

先安装 Matt 的 Skills：

```powershell
npx skills@latest add mattpocock/skills -g -a codex
```

至少选择 `codebase-design`、`grilling`、`domain-modeling`。若要使用完整交接流程，也选择 `setup-matt-pocock-skills`、`to-spec`、`to-tickets`、`implement`、`tdd` 和 `code-review`。安装整个集合也可以。

第一次在某个项目使用 Matt 的主流程时，按其仓库说明运行一次 setup。Codex 使用 `$`，使用 slash command 的客户端使用 `/`：

```text
$setup-matt-pocock-skills       # Codex
/setup-matt-pocock-skills       # slash-command client
```

它负责确定 issue tracker、标签和文档位置；这类项目配置不应该由个人结构审查 Skill 再维护一份。`to-spec` 与 `to-tickets` 会向这个已配置的 tracker 发布内容；若你不想创建外部 issue，可以选择本地文件 tracker，或保留 Structure Decision Brief 并在另一个任务中直接请求实现。

交接调用语法如下：

| Codex | slash-command client |
| --- | --- |
| `$to-spec` | `/to-spec` |
| `$to-tickets` | `/to-tickets` |
| `$implement` | `/implement` |

按当前 Matt 上游版本，`implement` 会使用 `tdd`、结束前运行 `code-review`，并提交当前分支。运行前应把这些源码修改与 Git 副作用视为明确的施工阶段，而不是本 Skill 的审查阶段。

再从你自己的 GitHub 仓库安装本 Skill：

```powershell
npx skills@latest add <github-owner>/<personal-skills-repo> --skill evolve-codebase-structure -g -a codex -y
```

更新两个来源已经安装的 Skill：

```powershell
npx skills@latest update -g
```

只更新个人 Skill：

```powershell
npx skills@latest update evolve-codebase-structure -g
```

Skills CLI 默认优先使用链接式安装；Windows 环境不支持链接时可在安装命令中增加 `--copy`。使用复制式安装后，把 GitHub 仓库作为唯一源码：修改、commit、push，再运行 update。直接修改安装目录的内容会在下次更新时丢失。

## 建议的个人仓库结构

```text
personal-agent-skills/
├─ README.md
├─ LICENSE
└─ skills/
   └─ engineering/
      └─ evolve-codebase-structure/
         ├─ SKILL.md
         ├─ README.md
         ├─ STRUCTURE-DESIGN.md
         ├─ HTML-REPORT.md
         └─ agents/
            └─ openai.yaml
```

`personal-agent-skills` 是你维护的上游；`~/.codex/skills/` 只是安装目标。两个 GitHub 仓库可以独立更新，因为 Skill 名称不同，不会产生同名覆盖。

## 调用策略

Codex 的 [`agents/openai.yaml`](agents/openai.yaml) 设置了 `policy.allow_implicit_invocation: false`，因此 Codex 不会把它作为自动触发 Skill 注入普通任务。它是一个真正的“上层、使用者调用”的流程 Skill：运行中可以使用 `codebase-design`、`grilling` 和 `domain-modeling` 这些下层参考 Skill，但会把 `to-spec` 等其他上层 Skill 留给你显式启动。

Matt 的 Skill 在 `SKILL.md` frontmatter 使用 `disable-model-invocation: true`。该字段是特定客户端采用的扩展，并不在当前 Agent Skills 标准 frontmatter 字段集合中，当前 Codex 校验器也会拒绝它。因此本 Skill 的 Codex 版本只使用 `agents/openai.yaml` 表达相同策略。若将来同时发布面向其他 Agent 的版本，应在该客户端的配置层声明显式调用，而不要让一个未经目标客户端验证的字段承担跨客户端语义。

## Attribution

The survey → HTML report → grilling workflow shape was inspired by Matt Pocock's MIT-licensed [`improve-codebase-architecture`](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture). This is an independent personal extension focused on physical code organization.
