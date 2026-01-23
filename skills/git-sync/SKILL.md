---
name: git-sync
description: 自动提交当前分支变更，合并到目标分支，并切回原分支。适用于开发完成后的快速同步。
---

# Git Sync Skill

当用户请求执行该 skill 时，请按照以下步骤操作。该 skill 接收两个参数：
1. **commit_message**: 本次提交的描述信息。
2. **target_branch**: 需要合并到的目标分支（如 `test` 或 `master`）。

### 执行流程：
**注意** 如果出现任何不符合预期，都直接退出不要强行继续，并提示用户

1. **环境确认**:
   - 确保当前处于 `process` 目录或 Git 仓库根目录。
   - 获取当前分支名称（记为 `current_branch`）。

2. **代码提交**:
   - 执行 `git add .`。
   - 执行 `git commit -m "<commit_message>"`。
   - 执行 `git pull`。
   - 执行 `git push`。
   - *注意*: 如果没有代码变更，直接退出。

3. **分支切换与合并**:
   - 切换到目标分支：`git checkout <target_branch>`。
   - 拉取最新代码：`git pull`。
   - 合并开发分支：`git merge <current_branch>`。
   - 拉取最新代码：`git pull`。
   - 提交代码：`git push`。
   - **安全性要求**: 如果合并过程中出现冲突，请立即停止并通知用户手动解决，不要强行继续。

4. **切回原分支**:
   - 执行 `git checkout <current_branch>`。

5. **完成反馈**:
   - 告知用户同步已完成，并列出执行的操作概要。

### 安全约束：
- **严禁**在合并失败的情况下强行提交或切换分支。
- 所有 Git 命令执行前请确认当前分支状态。
- 如果目标分支不存在，请提示用户。
