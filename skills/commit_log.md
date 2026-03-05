# Skills：提交记录追踪

## 适用场景

每次执行代码提交时，必须同时触发本 Skill，确保：
- Commit message **全部使用中文**
- 自动标注这是该仓库的**第几次提交**
- 将提交信息追加记录到 `commit_history.json`，形成可追溯的提交日志

---

## 执行步骤

### 步骤 1：查询当前提交序号

使用 `git_commit_count` 工具查询仓库当前的总提交数，得到序号 N，本次提交将标注为「第 N+1 次提交」。

```
Action: git_commit_count
Action Input: {"repo_path": "<仓库路径>"}
```

### 步骤 2：构造中文 Commit Message

Commit message **必须全部使用中文**，格式如下：

```
【第N次提交】<类型>（<范围>）：<简短描述>

提交背景：<说明为什么要做这次改动>
改动内容：<具体改了什么>
影响范围：<影响哪些模块或功能>
```

**类型对照表：**

| 英文 type | 中文类型 |
|-----------|---------|
| fix       | 修复缺陷 |
| feat      | 新增功能 |
| refactor  | 代码重构 |
| test      | 测试补充 |
| chore     | 构建维护 |
| docs      | 文档更新 |

**完整示例：**

```
【第3次提交】修复缺陷（用户接口）：修复用户信息接口空值导致的500错误

提交背景：线上反馈 /api/user/info 接口在传入不存在的用户ID时返回500
改动内容：在 get_user_info 函数中增加数据库查询结果的空值检查，返回404明确错误信息
影响范围：routers/user.py 中的 get_user_info 函数
```

### 步骤 3：执行提交

```
Action: git_commit
Action Input: {
  "repo_path": "<仓库路径>",
  "message": "【第N次提交】<中文完整描述>",
  "files": null
}
```

### 步骤 4：追加提交记录

提交成功后，使用 `append_commit_log` 工具将本次提交信息写入 `commit_history.json`。

```
Action: append_commit_log
Action Input: {
  "repo_path": "<仓库路径>",
  "commit_number": N,
  "commit_hash": "<提交返回的 hash>",
  "message": "<完整的中文 commit message>",
  "changed_files": ["<文件1>", "<文件2>"]
}
```

### 步骤 5：输出提交摘要

提交并记录完成后，向用户输出提交摘要：

```
✅ 第 N 次提交完成
   Hash：xxxxxxx
   信息：【第N次提交】...
   修改文件：x 个
   记录已追加至：commit_history.json
```

---

## 注意事项

- **Commit message 必须全部使用中文**，包括类型、范围、描述、正文
- 序号从仓库第一次提交开始累计，不受分支影响（以仓库总提交数为准）
- `commit_history.json` 存放在仓库根目录，需纳入版本控制
- 若 `commit_history.json` 不存在，自动创建并初始化为空数组
- 禁止直接 push 到 main/master（Guardrails 会自动拦截）

---

## commit_history.json 格式

```json
[
  {
    "序号": 1,
    "hash": "a3f9c12",
    "时间": "2026-03-04 10:30:00",
    "信息": "【第1次提交】新增功能（用户模块）：初始化用户信息查询接口",
    "修改文件": ["routers/user.py"],
    "分支": "fix/user-info"
  },
  {
    "序号": 2,
    "hash": "b82e1da",
    "时间": "2026-03-04 14:20:00",
    "信息": "【第2次提交】修复缺陷（用户接口）：修复空值导致的500错误",
    "修改文件": ["routers/user.py", "tests/test_user.py"],
    "分支": "fix/user-info"
  }
]
```
