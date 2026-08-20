# IDEA 使用技巧 / IDEA Usage Tips

## 一、项目管理 / Project Management

### 打开项目 / Open Project

`File` → `Open` → 选择项目根目录 / Select project root directory

**注意 Note**：如果只打开了子模块（如 `ruoyi-admin`），会看不到 `sql/` 目录。要打开整个项目根目录。

If you only open a sub-module, you won't see directories like `sql/`. Always open the project root.

### 项目结构设置 / Project Structure

快捷键 Shortcut：`Ctrl+Alt+Shift+S`

| 设置项 Setting | 推荐值 Recommended |
|------|------|
| Project SDK | 17 |
| Project language level | 17 |
| Modules → Language level | 17（每个模块都要检查 / Check each module） |

### 编译器设置 / Compiler Settings

快捷键 Shortcut：`Ctrl+Alt+S` → `Build, Execution, Deployment` → `Compiler` → `Java Compiler`

| 设置项 Setting | 推荐值 Recommended |
|------|------|
| Project bytecode version | 17 |
| Target bytecode version | 17（或留空 / Or leave empty） |

## 二、快捷键 / Shortcuts

### 通用 / General

| 快捷键 Shortcut | 功能 Function |
|------|------|
| Ctrl+K | 提交面板 Commit panel |
| Ctrl+Shift+K | 推送面板 Push panel |
| Ctrl+Alt+Shift+S | 项目结构 Project Structure |
| Ctrl+Alt+S | 设置 Settings |
| Ctrl+F9 | Build Project（编译 / Compile） |
| Ctrl+Shift+F9 | Rebuild（重新编译 / Recompile） |
| Ctrl+Shift+F | 全局搜索 Global search |
| Ctrl+N | 查找类 Find class |
| Ctrl+Shift+N | 查找文件 Find file |
| Double Shift | 万能搜索 Search everywhere |

### 代码 / Code

| 快捷键 Shortcut | 功能 Function |
|------|------|
| Alt+Enter | 智能提示（修复导入等）/ Smart fix (import, etc.) |
| Ctrl+Alt+L | 格式化代码 Format code |
| Ctrl+Alt+O | 优化导入 Optimize imports |
| Ctrl+D | 复制行 Duplicate line |
| Ctrl+Y | 删除行 Delete line |
| Ctrl+/ | 注释/取消注释行 Toggle line comment |

### Git

| 快捷键 Shortcut | 功能 Function |
|------|------|
| Ctrl+K | 提交 Commit |
| Ctrl+Shift+K | 推送 Push |
| Ctrl+T | 拉取 Pull |
| Alt+` | Git 操作面板 Git operations panel |

## 三、常见问题解决 / Common Issues & Solutions

### 问题 1：代码没有报错但编译失败 / No error marks but compilation fails

**解决 Solution**：`File` → `Invalidate Caches...` → 勾选前两项 → `Invalidate and Restart`

### 问题 2：编译报"不支持发行版本 22" / "Unsupported release version 22"

**原因 Cause**：IDEA 编译器设置被重置 / IDEA compiler settings reset

**解决 Solution**：
1. Project Structure（Ctrl+Alt+Shift+S）→ Project SDK 选 17 → Language Level 选 17
2. Settings（Ctrl+Alt+S）→ Compiler → Java Compiler → bytecode version 改 17
3. 如果还报错：勾选 `Delegate IDE build/run actions to Maven`

### 问题 3：Maven 依赖找不到 / Maven dependency not found

**解决 Solution**：
1. 右侧 Maven 面板 → 点刷新按钮 Reload All Maven Projects
2. 或终端执行 `mvn clean install -DskipTests`

### 问题 4：资源文件没复制到 target / Resources not in target

**原因 Cause**：`Rebuild Project` 不复制资源文件 / Rebuild doesn't copy resources

**解决 Solution**：
- 用 `Build Project`（Ctrl+F9）代替 Rebuild / Use Build instead of Rebuild
- 或 Maven 命令 `mvn resources:resources`

### 问题 5：端口被占用 / Port already in use

**解决 Solution**：
```powershell
# 查看占用端口的进程 / Find process using the port
netstat -ano | findstr :8080

# 杀掉进程 / Kill process
taskkill /PID <进程号 PID> /F
```

## 四、Git 操作 / Git Operations in IDEA

### 查看代码改动 / View Changes

| 操作 Operation | 方法 Method |
|------|------|
| 查看未提交改动 View uncommitted changes | Ctrl+K 打开 Commit 面板，双击文件名 / Open Commit panel, double-click file |
| 和远程版本比较 Compare with remote | 右键文件 → Git → Compare with Repository Version |
| 查看文件历史 View file history | 右键文件 → Git → Show History |

### 文件颜色含义 / File Color Meaning

| 颜色 Color | 状态 Status | 含义 Meaning |
|------|------|------|
| 绿色 Green | Untracked | 新文件，未加入版本控制 / New file, not tracked |
| 蓝色 Blue | Modified | 已修改 / Modified |
| 红色 Red | Untracked (in Commit panel) | 新文件，在提交面板中 / New file in commit panel |
| 灰色 Gray | Ignored | 被忽略 / Ignored |
| 无色 Normal | Unchanged | 未改动 / No changes |

## 五、调试 / Debugging

### 启动调试 / Start Debug

点击代码行号左侧设置断点 → 点击虫子图标（Debug）启动

Click left of line number to set breakpoint → Click bug icon to start debug

### 调试面板 / Debug Panel

| 按钮 Button | 功能 Function |
|------|------|
| Step Over (F8) | 执行当前行，不进入方法 / Execute line, don't enter method |
| Step Into (F7) | 进入方法内部 / Step into method |
| Step Out (Shift+F8) | 跳出当前方法 / Step out of current method |
| Resume (F9) | 继续执行到下一个断点 / Continue to next breakpoint |
