# Skills：提交记录追踪

## 适用场景

每次执行代码提交时，必须同时触发本 Skill，确保：
- Commit message **全部使用中文**
- 自动标注这是该仓库的**第几次提交**
- 将提交信息追加记录到 `commit_history.json`，形成可追溯的提交日志

---

## 执行步骤

### 步骤 1：查询当前提交序号

使用 `git rev-list --count HEAD` 查询仓库当前的总提交数，得到序号 N，本次提交将标注为「第 N+1 次提交」。

```bash
git rev-list --count HEAD
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

### 步骤 3：先更新 commit_history.json，再一起提交

**关键：先写历史记录，再统一提交，避免补录循环。**

在执行 `git commit` 之前，先将本次提交信息预写入 `commit_history.json`。
此时 hash 字段填写 `"pending"`（占位），待提交完成后原地替换为真实 hash。

具体流程：

1. **预写记录**：将本次提交记录追加到 `commit_history.json`，hash 填 `"pending"`
2. **暂存所有变更**：`git add` 业务文件 + `commit_history.json`
3. **执行提交**：`git commit` 一次性提交所有内容（包括历史记录文件）
4. **回填 hash**：提交成功后，用 `git rev-parse --short HEAD` 获取真实 hash，替换 `commit_history.json` 中的 `"pending"`
5. **修订提交**：`git commit --amend --no-edit` 将 hash 回填后的文件更新到同一次提交中

```bash
# 1. 预写记录（hash 占位为 pending）
# 2. 暂存所有文件
git add <业务文件> commit_history.json
# 3. 提交
git commit -m "【第N次提交】..."
# 4. 获取真实 hash 并回填
HASH=$(git rev-parse --short HEAD)
# 替换 commit_history.json 中的 "pending" 为真实 hash
# 5. 修订提交
git add commit_history.json
git commit --amend --no-edit
```

**注意**：这样整个过程只产生 **1 次提交**，不会出现补录循环。

### 步骤 4：输出提交摘要

提交并记录完成后，向用户输出提交摘要：

```
✅ 第 N 次提交完成
   Hash：xxxxxxx
   信息：【第N次提交】...
   修改文件：x 个
   记录已同步至：commit_history.json
```

---

## 注意事项

- **Commit message 必须全部使用中文**，包括类型、范围、描述、正文
- 序号从仓库第一次提交开始累计，不受分支影响（以仓库总提交数为准）
- `commit_history.json` 存放在仓库根目录，需纳入版本控制
- 若 `commit_history.json` 不存在，自动创建并初始化为空数组
- **`commit_history.json` 必须与业务文件在同一次提交中更新**，严禁单独为补录历史记录发起额外提交
- amend 回填 hash 后，commit 自身的 hash 会变化，因此 `commit_history.json` 中记录的 hash 与最终 `git log` 中的 hash 会有差异，这是预期行为，**不要为了对齐 hash 而反复 amend**，只需执行一次 amend 即可
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
