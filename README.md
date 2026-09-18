# Personal Agent Skills

这是我个人维护的 Agent Skills 合集。仓库将个人 Skill 与第三方上游仓库分开，方便独立安装、更新和长期演进。

## Skills

| Skill | 用途 | 状态 |
| --- | --- | --- |
| [`evolve-codebase-structure`](skills/engineering/evolve-codebase-structure/) | 周期性检查代码放置、文件拆分、目录演化和包边界 | 个人维护 |
| [`write-maintainable-code`](skills/engineering/write-maintainable-code/) | 在行为与接口确定后，选择最小可维护实现 | 个人维护；替代旧 `andrej-karpathy-skills` |
| [`code-comments`](skills/engineering/code-comments/) | 改善 Python/C++/CUDA 系统代码的契约注释与条件性视觉结构 | 个人维护 |

本次改造的 `write-maintainable-code` 与 `code-comments` 都由使用者显式调用，不会自动注入普通编码任务。

## 安装

安装整个仓库：

```powershell
npx skills@latest add cilinmengye/personal-agent-skills -g -a codex
```

只安装一个 Skill：

```powershell
npx skills@latest add cilinmengye/personal-agent-skills --skill evolve-codebase-structure -g -a codex -y
npx skills@latest add cilinmengye/personal-agent-skills --skill write-maintainable-code -g -a codex -y
npx skills@latest add cilinmengye/personal-agent-skills --skill code-comments -g -a codex -y
```

Windows 不支持链接式安装时，可以在命令末尾增加 `--copy`。

## 仓库结构

```text
skills/
└─ engineering/
   ├─ evolve-codebase-structure/
   ├─ write-maintainable-code/
   └─ code-comments/
```

三个 Skill 各自拥有独立的 `SKILL.md`、使用说明和显式调用配置。`code-comments` 的 CUDA 细节按需放在 supporting reference 中。

## 与 Matt Pocock Skills 组合

本仓库不复制 Matt Pocock 的规划、TDD 和审查流程。推荐由使用者在同一请求中显式组合：

```text
$implement $write-maintainable-code 实现已确认的 ticket。
$implement $write-maintainable-code $code-comments 实现系统代码 ticket，并维护契约注释。
$tdd $write-maintainable-code 通过已确认的 public seam 实现一个 red-green slice。
```

Matt 的 `implement` 会按其自身流程编排 TDD、测试和最终 `code-review`；`write-maintainable-code` 负责 green implementation 内部的工程选择，`code-comments` 负责代码旁契约与可扫描性。本仓库这两个显式调用 Skill 不会互相调用，因此需要哪一个就由你在请求中点名哪一个。

## 旧名称迁移

`andrej-karpathy-skills` 已从本仓库删除。其与 Matt 工作流重叠的部分不再保留；实现期仍有独立价值的判断已迁移到 `$write-maintainable-code`。若本机曾安装旧 Skill，请从安装目录移除旧副本，再按上面的命令安装新 Skill。
