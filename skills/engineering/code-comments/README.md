# Code Comments

`code-comments` 是一个由使用者显式调用的系统代码文档 Skill，面向 Python、C++ 与 CUDA。它同时保留两类对人类维护者有价值的信号：

1. 代码本身无法可靠表达的契约、理由、不变量、单位、所有权和顺序；
2. 服从 formatter 与项目风格的条件性视觉结构。

它不再把固定空行数量、每个分支的标题注释或每个私有 helper 的模板 docstring 当作跨语言真理。具体空行首先服从仓库 formatter 和语言规范；在函数内部，真正的阶段转换仍然可以用空行增强扫描性。稳定的大型文件也可以使用 zone anchor，但只有在拆文件会损害 locality 时才这样做。

这与官方规范并不冲突：PEP 8 对 Python 顶层定义、类内方法和函数内部空行给出不同指导；PEP 257 关注公共可导入对象的文档契约；Linux C 风格也有自己的函数间空行约定。可读性是共同目标，固定数字不是跨语言共同规则。

- [PEP 8：Blank Lines](https://peps.python.org/pep-0008/#blank-lines)
- [PEP 257：Docstring Conventions](https://peps.python.org/pep-0257/)
- [Linux kernel coding style：Functions](https://docs.kernel.org/process/coding-style.html#functions)
- [CUDA Programming Guide：Programming Model](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/programming-model.html#gpu-memory)

## 如何调用

整理一个文件的契约与注释：

```text
$code-comments 请检查 scheduler.py：补充调用者需要知道的契约，并改善真实阶段之间的视觉结构。
```

与实现 Skill 组合时，由使用者同时点名；两个 Skill 不会互相自动调用：

```text
$write-maintainable-code $code-comments 实现新的请求迁移路径，并记录锁顺序和错误恢复契约。
```

与 Matt Pocock 的完整流程一起使用：

```text
$implement $write-maintainable-code $code-comments 实现 CUDA graph capture ticket。
```

其中 Matt 的 `implement` 会编排 `tdd` 与最终 `code-review`，`write-maintainable-code` 管实现内部选择，本 Skill 只管代码旁文档和可扫描性。若改善注释暴露出需要改命名、类型、控制流、接口或模块的问题，单独调用本 Skill 时只报告；联合调用时交给实现 Skill 处理。

## CUDA 不再使用简单二分

CUDA 指针的真实契约不止 Host/Device。公共 kernel 和 wrapper 在类型无法表达时，应记录实际 memory space（例如 device global、host pinned、managed）、读写权限、ownership、lifetime、aliasing、shape/stride/dtype/alignment、stream/synchronization 以及有证据的性能前提。标量参数记录单位与范围，不贴 `[Host]` 标签。

该 Skill 同时在 `SKILL.md` 和 `agents/openai.yaml` 中声明为显式调用，因此普通代码编辑不会自动注入它。
