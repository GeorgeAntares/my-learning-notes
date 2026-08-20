# Maven 多模块项目管理 / Maven Multi-Module Project Management

## 一、什么是多模块项目 / What is a Multi-Module Project

Maven 多模块项目把一个大型项目拆分成多个子模块，每个模块负责不同的功能，模块之间可以互相依赖。

A Maven multi-module project splits a large project into multiple sub-modules, each responsible for different functionality. Modules can depend on each other.

```
<project-root>/（父项目 Parent）
├── pom.xml              → 父 POM，定义公共配置 / Parent POM, defines common config
├── ruoyi-admin/         → 启动模块，依赖所有其他模块 / Entry module, depends on all others
├── ruoyi-framework/     → 框架核心 / Framework core
├── ruoyi-system/        → 系统模块 / System module
├── ruoyi-common/        → 公共工具 / Common utilities
├── ruoyi-loop/          → 业务模块 / Business module
├── ruoyi-ui/            → 前端（非 Maven 模块）/ Frontend (not a Maven module)
└── sql/                 → SQL 脚本 / SQL scripts
```

## 二、模块依赖关系 / Module Dependencies

```
ruoyi-admin（启动模块 / Entry module）
    ├── ruoyi-framework
    ├── ruoyi-system
    └── ruoyi-loop
         ├── ruoyi-common
         └── ruoyi-system

依赖方向：admin → framework/system/loop → common
Dependency direction: admin → framework/system/loop → common
```

**规则 Rule**：下层模块不能依赖上层模块。`ruoyi-common` 不能依赖 `ruoyi-admin`。

Lower modules cannot depend on higher modules. `ruoyi-common` cannot depend on `ruoyi-admin`.

## 三、常用 Maven 命令 / Common Maven Commands

### 编译单个模块 / Compile a Single Module

```bash
# 只编译 ruoyi-admin 及其依赖的模块
# Compile ruoyi-admin and its dependencies
mvn -pl ruoyi-admin -am compile
```

| 参数 Parameter | 含义 Meaning |
|------|------|
| `-pl ruoyi-admin` | 只构建指定模块 / Build only the specified module |
| `-am` | 同时构建依赖的模块 / Also build dependencies |
| `compile` | 编译阶段 / Compile phase |
| `-q` | 安静模式，只输出错误 / Quiet mode, only errors |
| `-DskipTests` | 跳过测试 / Skip tests |
| `-Dmaven.test.skip=true` | 跳过测试编译和执行 / Skip test compile and execution |

### 全量编译 / Full Build

```bash
# 清理 + 安装到本地仓库（不跑测试）
# Clean + install to local repository (skip tests)
mvn clean install -Dmaven.test.skip=true
```

### 清理 / Clean

```bash
mvn clean          # 删除所有 target/ 目录 / Delete all target/ directories
mvn clean compile  # 清理后重新编译 / Clean then recompile
```

## 四、常见问题 / Common Issues

### 问题 1：找不到依赖的模块 / Cannot find dependent module

**症状 Symptom**：
```
Could not find the selected project in the reactor: ruoyi-admin
```

**原因 Cause**：不在项目根目录执行 Maven 命令 / Not running Maven from project root

**解决 Solution**：
```bash
cd <your-project-root>
mvn -pl ruoyi-admin -am compile
```

### 问题 2：编译报找不到类 / Compile error: class not found

**症状 Symptom**：
```
程序包 com.ruoyi.loop.onboarding.vo 不存在
Package com.ruoyi.loop.onboarding.vo does not exist
```

**原因 Cause**：同事漏提交了某个文件 / A colleague forgot to commit a file

**解决 Solution**：
1. 确认文件是否真的不存在 / Confirm the file truly doesn't exist
2. 通知同事补提交 / Notify colleague to commit the missing file
3. 临时方案：创建临时文件让编译通过 / Temp workaround: create temp file to pass compilation

### 问题 3：依赖下载失败 / Dependency download failed

**症状 Symptom**：
```
Could not resolve dependencies: Failed to collect dependencies
```

**原因 Cause**：Maven 中央仓库访问慢或失败 / Maven Central is slow or unreachable

**解决 Solution**：
1. 配置阿里云镜像 / Configure Aliyun mirror
2. 重新执行 `mvn clean install` / Re-run

### 问题 4：resource 文件未复制到 target / Resources not copied to target

**症状 Symptom**：`target/classes` 下缺少 `mybatis-config.xml`、`logback.xml` 等配置文件

**原因 Cause**：IDEA 的 Rebuild Project 不会复制资源文件 / IDEA's Rebuild doesn't copy resources

**解决 Solution**：
- 用 `Build Project`（Ctrl+F9）而不是 `Rebuild Project` / Use Build, not Rebuild
- 或用 Maven 命令 / Or use Maven: `mvn resources:resources`

## 五、IDEA 中的 Maven 配置 / Maven Configuration in IDEA

### Maven 路径 / Maven Path

```
<user-home>\.m2\wrapper\dists\apache-maven-3.9.16-bin\...\bin\mvn.cmd
```

### 关键设置 / Key Settings

| 设置 Setting | 推荐值 Recommended | 说明 Description |
|------|------|------|
| Delegate IDE build/run actions to Maven | ✅ 勾选 / Check | 避免编译器版本不一致 / Avoid compiler version mismatch |
| Maven Runner JRE | 17 | 与项目一致 / Match project |
| Maven auto-import | ✅ 开启 / Enable | pom.xml 修改后自动同步 / Auto-sync on pom.xml change |

### Maven 面板 / Maven Panel

IDEA 右侧边栏 → Maven 面板：
- 刷新按钮 Refresh：Reload All Maven Projects
- 生命周期 Lifecycle：clean → compile → package → install
- 模块列表 Module list：展开每个模块看依赖

## 六、pom.xml 基础 / pom.xml Basics

```xml
<project>
    <!-- 父项目 / Parent project -->
    <parent>
        <groupId>com.ruoyi</groupId>
        <artifactId>ruoyi</artifactId>
        <version>4.0.0</version>
    </parent>

    <!-- 当前模块信息 / Current module info -->
    <artifactId>ruoyi-admin</artifactId>
    <packaging>jar</packaging>

    <!-- 依赖 / Dependencies -->
    <dependencies>
        <dependency>
            <groupId>com.ruoyi</groupId>
            <artifactId>ruoyi-framework</artifactId>
        </dependency>
    </dependencies>
</project>
```

| 标签 Tag | 说明 Description |
|------|------|
| `<parent>` | 继承父 POM / Inherit parent POM |
| `<groupId>` | 组织 ID / Organization ID |
| `<artifactId>` | 模块 ID / Module ID |
| `<version>` | 版本号 / Version |
| `<packaging>` | 打包方式（jar/war/pom）/ Packaging type |
| `<dependencies>` | 依赖列表 / Dependency list |
