# my-skills

个人技能库，用于存放和管理自定义的 Agent Skills。

## 目录结构

```
my-skills/
├── README.md              # 项目说明
├── commit_history.json    # 提交历史记录（自动生成）
├── .cursor/
│   └── rules/
│       └── git-commit.mdc # git 提交规范（自动触发）
└── skills/
    └── commit_log.md      # 提交记录追踪 Skill
```

## skills 目录

`skills/` 目录用于存放各类自定义技能，每个技能通常包含一个 `SKILL.md` 文件，描述该技能的用途、使用场景及操作步骤。

## 使用方式

将技能文件放入 `skills/` 目录下对应的子文件夹中，即可在 Cursor 中引用和使用。
