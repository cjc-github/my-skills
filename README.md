# my-skills

个人技能库，用于存放和管理自定义的 Agent Skills，配合 Cursor 规则实现标准化的 AI 辅助工作流。

## 目录结构

```
my-skills/
├── README.md              # 项目说明
├── commit_history.json    # 提交历史记录（自动维护）
├── .cursor/
│   └── rules/
│       └── git-commit.mdc # git 提交规范（每次提交自动触发）
└── skills/
    └── commit_log.md      # 提交记录追踪 Skill
```

## Skills 说明

| 文件 | 用途 |
|------|------|
| `skills/commit_log.md` | 定义 git 提交规范：中文 commit message、自动编号、记录追踪 |

## Cursor Rules 说明

| 规则文件 | 触发时机 | 作用 |
|----------|----------|------|
| `.cursor/rules/git-commit.mdc` | 每次对话（alwaysApply） | 执行 git 提交时自动调用 `commit_log.md` 流程 |

## 工作流程

每次执行 git 提交时，AI 会自动：

1. 查询仓库当前提交总数，计算本次序号
2. 使用中文格式构造 commit message：`【第N次提交】类型（范围）：描述`
3. 执行提交并将记录追加到 `commit_history.json`
4. 输出提交摘要

## 扩展方式

在 `skills/` 目录下新增 `.md` 文件即可添加新技能，并在 `.cursor/rules/` 下创建对应 `.mdc` 规则文件来指定触发时机。
