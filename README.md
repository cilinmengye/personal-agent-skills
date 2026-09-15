# Personal Agent Skills

这是我个人维护的 Agent Skills 合集。仓库将个人 Skill 与第三方上游仓库分开，方便独立安装、更新和长期演进。

## Skills

| Skill | 用途 | 状态 |
| --- | --- | --- |
| [`evolve-codebase-structure`](skills/engineering/evolve-codebase-structure/) | 周期性检查代码放置、文件拆分、目录演化和包边界 | 个人维护 |
| [`andrej-karpathy-skills`](skills/engineering/andrej-karpathy-skills/) | Andrej Karpathy 风格的通用 Agent 编程行为准则 | 从[原仓库](https://github.com/cilinmengye/andrej-karpathy-skills)原样迁移，待后续优化 |
| [`code-comments`](skills/engineering/code-comments/) | 约束代码注释的内容、位置和质量 | 从[原仓库](https://github.com/cilinmengye/code-comments)原样迁移，待后续优化 |

## 安装

安装整个仓库：

```powershell
npx skills@latest add cilinmengye/personal-agent-skills -g -a codex
```

只安装一个 Skill：

```powershell
npx skills@latest add cilinmengye/personal-agent-skills --skill evolve-codebase-structure -g -a codex -y
npx skills@latest add cilinmengye/personal-agent-skills --skill andrej-karpathy-skills -g -a codex -y
npx skills@latest add cilinmengye/personal-agent-skills --skill code-comments -g -a codex -y
```

Windows 不支持链接式安装时，可以在命令末尾增加 `--copy`。

## 仓库结构

```text
skills/
└─ engineering/
   ├─ evolve-codebase-structure/
   ├─ andrej-karpathy-skills/
   └─ code-comments/
```

三个 Skill 各自拥有独立的 `SKILL.md` 和说明文档。此次迁移不修改两个已有 Skill 的内容，也不删除或归档其原 GitHub 仓库；后续优化将在本仓库中单独进行。
