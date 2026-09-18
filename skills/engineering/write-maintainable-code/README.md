# Write Maintainable Code

`write-maintainable-code` 是一个由使用者显式调用的实现期 Skill。它填补的是这样一个空白：行为、接口和任务范围已经确定，但 Agent 即将写 green implementation 时，仍需要判断数据表示、控制流、硬编码、复用、抽象、副作用以及性能/可移植性取舍。

它不重新做需求澄清、spec、测试 seam、目录架构审查或最终 code review。它关心的是：**当前行为如何用最少但足够可维护的概念表达出来。**

## 它约束什么

- 简单性按概念、分支、依赖、隐藏状态和公共接口衡量，而不是按行数衡量。
- 先区分领域不变量、部署配置、产品策略和局部实现细节，再判断 literal 是否属于硬编码。
- 复用遵循“项目已有能力 → 标准库 → 已有依赖 → 小型本地实现 → 谨慎新增依赖”。
- 抽象必须隐藏真实变化、集中知识或降低调用者复杂性；单一实现不预建未来框架。
- 状态转换、副作用、资源生命周期和错误路径应有清楚的 owner。
- 默认选择清晰可移植的基线；性能契约或测量证据可以证明隔离的专用实现合理。
- 允许当前功能所必需的局部 prefactor，但不顺手清理无关区域。

生成代码、vendor 和明确的 throwaway prototype 不适用这套维护成本假设。

## 如何调用

单独用于一个已经说清楚的修改：

```text
$write-maintainable-code 请实现已确认的批处理重试策略。
```

它不会自动调用其他用户 Skill。需要完整施工流程时，由你在同一请求中明确组合：

```text
$implement $write-maintainable-code 实现 issue #42；测试 seam 以 ticket 中的 public API 为准。
```

希望显式进行 red-green 循环时：

```text
$tdd $write-maintainable-code 通过 Scheduler.submit 的 public seam 实现新的取消行为。
```

系统代码还需要整理契约注释时，再加入：

```text
$implement $write-maintainable-code $code-comments 实现 CUDA buffer pooling ticket。
```

这里的职责顺序是：Matt 的 `implement` 会编排施工、`tdd` 与最终 `code-review`；TDD 拥有 red → green 时序，本 Skill 只在 red 已建立后约束 green implementation 的内部选择，`code-comments` 管代码旁契约。

## 非平凡修改会先给出什么

Agent 在开始首次非平凡 implementation 前会给出一个很短的选择说明；如果正在使用 TDD，则先建立 failing red test，再在 green implementation 前给出：

```text
Owner and invariant:
Representation and control flow:
Reuse/dependency choice:
Portability/performance posture:
Necessary local prefactor: none / list:
```

这不是第二份设计文档。它的作用是让“为什么用这种实现”在代码落地前可见，并阻止 Agent 仅因为当前文件已经打开就继续堆叠代码。

## 从旧 Skill 迁移

`andrej-karpathy-skills` 已被删除，不再是本仓库的安装项。它的澄清、最小范围、验证目标等原则已由 Matt Pocock 的完整流程更具体地覆盖；仍有独立价值的“green 阶段怎样写代码”被收敛到 `$write-maintainable-code`。

旧调用：

```text
$andrej-karpathy-skills
```

新调用：

```text
$write-maintainable-code
```

该 Skill 同时在 `SKILL.md` 和 `agents/openai.yaml` 中声明为显式调用，普通编码任务不会自动注入它。
