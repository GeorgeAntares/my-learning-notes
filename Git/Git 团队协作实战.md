# Git 团队协作实战

## 一、日常协作流程

团队开发中，每天都有同事往远程仓库推代码，你需要拉取最新代码再开始工作。

### 标准拉取流程（stash 三步法）

当你本地有未提交的改动，远程也有新提交时：

```bash
# 1. 暂存本地改动（临时收起来，不丢）
git stash -u

# 2. 拉取远程代码
git pull

# 3. 恢复本地改动
git stash pop
```

- `git stash -u`：`-u` 表示连未跟踪的新文件一起暂存
- 执行后工作区变干净，可以安全拉取
- `git stash pop`：把暂存的改动放回来

### 为什么不能直接 git pull

如果本地改了文件 A，远程也改了文件 A，直接 `git pull` 会被拒绝：

```
error: Your local changes to the following files would be overwritten by merge
```

stash 三步法就是解决这个问题的。

## 二、提交代码

### 选择性提交

团队项目中，不是所有改动都该提交。用 IDEA 的 Commit 面板（Ctrl+K）可以只勾选需要提交的文件：

| 该提交 | 不该提交 |
|--------|---------|
| 业务代码（Controller、Service） | `.idea/*`（IDE 配置） |
| 前端页面（Vue、JS） | 调试用临时改动（SecurityConfig 放行） |
| SQL 脚本 | `package-lock.json`（锁文件） |
| 配置文件改动 | 临时文档、PDF |

### Commit Message 规范

```
简短描述（一行）

- 改动点1
- 改动点2
```

示例：
```
平台公告增删改查接口及小程序端读取接口

- LoopNoticeController: 运营端公告增删改查5个接口
- ClientNoticeController: 小程序端公告列表/详情/标记已读3个接口
```

## 三、推送被拒绝

### 原因

同事在你之前 push 了代码，远程比你的本地新：

```
! [rejected] main -> main (fetch first)
```

### 解决

```bash
# 先拉取再推送
git pull --rebase
git push
```

`--rebase` 的作用：把你的提交"挪到"同事的提交之上，保持线性历史。

## 四、冲突解决

### stash pop 冲突

```bash
git stash pop
# CONFLICT (content): Merge conflict in xxx.java
```

处理方式：
1. 打开冲突文件，找到三段标记：
   ```
   <<<<<<< Updated upstream
   同事的代码
   =======
   你的代码
   >>>>>>> Stashed changes
   ```
2. 决定保留哪部分（删掉另一部分和标记符号）
3. `git add <文件名>` 标记已解决

### 选择保留哪一方

| 命令 | 含义 |
|------|------|
| `git checkout --ours <file>` | 保留远程版本（丢弃本地改动） |
| `git checkout --theirs <file>` | 保留本地版本（丢弃远程改动） |

### .idea 文件冲突

`.idea/` 是 IDEA 自动生成的配置，冲突了直接丢弃本地改动：

```bash
git checkout -- .idea/
```

## 五、常用命令速查

| 场景 | 命令 |
|------|------|
| 暂存本地改动 | `git stash -u` |
| 查看暂存列表 | `git stash list` |
| 恢复暂存 | `git stash pop` |
| 丢弃暂存 | `git stash drop` |
| 拉取并变基 | `git pull --rebase` |
| 查看状态 | `git status` |
| 查看改动 | `git diff` |
| 撤销工作区改动 | `git checkout -- <file>` |
| 取消暂存 | `git restore --staged <file>` |

## 六、IDEA 中的 Git 操作

| 操作 | 快捷键 |
|------|--------|
| 提交面板 | Ctrl+K |
| 推送面板 | Ctrl+Shift+K |
| 查看文件 diff | 右键 → Git → Compare with Repository Version |

## 七、注意事项

1. **不要提交 `.idea/` 文件**：这是 IDE 配置，每个人环境不同
2. **不要提交调试用临时改动**：如 SecurityConfig 的 `permitAll()` 放行
3. **拉取前先 stash**：养成习惯，避免冲突
4. **提交前检查文件列表**：确认只提交该提交的文件
5. **push 被拒绝不要慌**：`git pull --rebase` 再 push 就行
