# my-skills

个人技能库，用于存放和管理自定义的 Agent Skills，配合 Cursor 规则实现标准化的 AI 辅助工作流。

## 目录结构

```
my-skills/
├── README.md                              # 项目说明
├── commit_history.json                    # 提交历史记录（自动维护）
├── .agents/
│   └── skills/
│       └── libfuzzer/
│           └── SKILL.md                   # libFuzzer 模糊测试技能（跨平台 Agent 规范）
├── .cursor/
│   ├── rules/
│   │   └── git-commit.mdc                # git 提交规范（每次对话自动触发）
│   └── skills/
│       └── libfuzzer -> .agents/skills/   # 符号链接，复用 .agents 中的技能
└── skills/
    └── commit_log.md                      # 提交记录追踪 Skill（被 Rule 调用）
```

### 目录说明

| 目录 | 用途 | 适用范围 |
|------|------|---------|
| `.agents/skills/` | 跨平台 Agent Skills 存储，遵循通用 `.agents/` 目录规范 | 所有支持该规范的 AI Agent |
| `.cursor/skills/` | Cursor 专用技能目录，通过符号链接复用 `.agents/` 中的技能 | Cursor Agent |
| `.cursor/rules/` | Cursor 规则文件，控制技能的自动触发时机 | Cursor Agent |
| `skills/` | 自定义流程型技能，由 Rule 文件间接调用 | Cursor Agent |

---

## Skills 清单

### 1. libFuzzer 模糊测试

| 属性 | 值 |
|------|-----|
| 文件路径 | `.agents/skills/libfuzzer/SKILL.md` |
| Cursor 入口 | `.cursor/skills/libfuzzer/SKILL.md`（符号链接） |
| 类型 | fuzzer |
| 用途 | 基于 LLVM 的覆盖引导模糊测试，适用于 C/C++ 项目 |

**触发方式：**

- **自动触发**：当对话中涉及 C/C++ 模糊测试、fuzzing、libFuzzer、Clang sanitizer 等关键词时，Cursor Agent 会自动匹配并加载该技能
- **手动触发**：在 Cursor 对话中直接描述需求即可，例如：
  - "帮我对这个 C++ 项目做模糊测试"
  - "用 libfuzzer 编写 fuzz harness"
  - "帮我给这个函数写一个 fuzzing 测试用例"

**主要能力**：
- 编写 libFuzzer harness（`LLVMFuzzerTestOneInput`）
- 使用 Clang sanitizer 编译并运行模糊测试
- Corpus 管理与最小化
- 崩溃分析与复现
- 与 AFL++ 的兼容迁移指导

---

### 2. 提交记录追踪

| 属性 | 值 |
|------|-----|
| 文件路径 | `skills/commit_log.md` |
| 关联规则 | `.cursor/rules/git-commit.mdc` |
| 用途 | 规范化 git 提交：中文 commit message、自动编号、历史记录追踪 |

**触发方式：**

- **自动触发**：`.cursor/rules/git-commit.mdc` 设置了 `alwaysApply: true`，每次 Cursor 对话都会加载该规则。当用户执行 git 提交操作时，Agent 会自动读取 `skills/commit_log.md` 并执行完整流程
- **手动触发**：在 Cursor 对话中请求提交即可，例如：
  - "帮我提交代码"
  - "更新 git"
  - "git commit"

**自动执行流程**：
1. 查询仓库提交总数，计算本次序号
2. 构造中文 commit message：`【第N次提交】类型（范围）：描述`
3. 执行 `git commit`
4. 追加记录到 `commit_history.json`
5. 输出提交摘要

---

## Cursor Rules 说明

| 规则文件 | 触发时机 | 作用 |
|----------|----------|------|
| `.cursor/rules/git-commit.mdc` | 每次对话（`alwaysApply: true`） | 执行 git 提交时自动调用 `skills/commit_log.md` 流程 |

---

## 触发机制详解

### 自动触发（Cursor Agent 智能匹配）

Cursor Agent 通过以下机制自动触发技能：

1. **Rules（规则）**：`.cursor/rules/*.mdc` 文件中设置 `alwaysApply: true` 的规则，在每次对话中都会生效。当检测到匹配场景（如 git 提交），自动调用关联的 Skill
2. **Skills（技能）**：`.cursor/skills/*/SKILL.md` 中的 `description` 字段会被 Agent 用于语义匹配。当用户意图与描述匹配时，自动加载执行

### 手动触发（用户主动描述需求）

在 Cursor Agent 对话中用自然语言描述需求即可。Agent 会根据语义理解匹配最合适的 Skill。

**有效的触发话术示例：**

| 想要触发的 Skill | 可以这样说 |
|-----------------|-----------|
| libFuzzer | "对这个 C++ 函数做 fuzz 测试"、"写一个 libfuzzer harness" |
| 提交记录追踪 | "帮我提交代码"、"更新 git"、"commit 一下" |

---

## 扩展方式

### 添加新技能

1. 在 `.agents/skills/` 下创建新目录和 `SKILL.md` 文件
2. 在 `.cursor/skills/` 下创建符号链接指向该技能
3. 如需自动触发，在 `.cursor/rules/` 下创建对应 `.mdc` 规则文件

```bash
# 示例：添加名为 my-new-skill 的技能
mkdir -p .agents/skills/my-new-skill
# 编写 SKILL.md（包含 name 和 description 前置元数据）
ln -s ../../.agents/skills/my-new-skill .cursor/skills/my-new-skill
```

### SKILL.md 基本格式

```markdown
---
name: skill-name
description: 简要说明技能用途和触发场景
---

# 技能名称

## 使用说明
...
```

`description` 字段是 Agent 自动匹配的关键，需要明确写出**做什么**和**何时使用**。
