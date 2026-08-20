# RuoYi 框架开发笔记 / RuoYi Framework Development Notes

## 一、项目结构 / Project Structure

RuoYi 是基于 Spring Boot 的后台管理框架，采用多模块 Maven 结构。

RuoYi is a backend management framework based on Spring Boot, using a multi-module Maven structure.

```
loop-serv/
├── ruoyi-admin/      → 启动模块（Controller 在这）/ Entry module (Controllers here)
├── ruoyi-framework/  → 框架核心（Security、配置）/ Framework core (Security, config)
├── ruoyi-system/     → 系统模块（用户、菜单、配置）/ System module (users, menus, config)
├── ruoyi-common/     → 公共工具（SecurityUtils、AjaxResult）/ Common utilities
├── ruoyi-loop/       → 业务模块（LOOP 业务逻辑）/ Business module
├── ruoyi-ui/         → 前端 Vue 项目 / Frontend Vue project
└── sql/              → 数据库脚本 / Database scripts
```

## 二、入口与启动 / Entry Point & Startup

- 启动类 Entry class：`ruoyi-admin` 模块下的 `RuoYiApplication.java`
- 后端端口 Backend port：8080
- 前端端口 Frontend port：80（`npm run dev` 启动）
- Swagger 地址 Swagger URL：`http://localhost:8080/swagger-ui.html`

### 启动顺序 / Startup Order

1. 先启动后端 Start backend first：IDEA 中运行 `RuoYiApplication` / Run in IDEA
2. 再启动前端 Then start frontend：`cd ruoyi-ui && npm run dev`
3. 前端通过代理把 `/dev-api` 请求转发到 8080 / Frontend proxies `/dev-api` requests to 8080

## 三、Controller 开发模式 / Controller Development Pattern

### 运营端接口 / Operations-side API

路径前缀 `/api/v1/ops/`，需要登录 + 权限：

Path prefix `/api/v1/ops/`, requires login + permission:

```java
@RestController
@RequestMapping("/api/v1/ops/notice")
public class LoopNoticeController extends BaseController {

    @PreAuthorize("@ss.hasPermi('loop:notice:list')")
    @GetMapping
    public TableDataInfo list(SysNotice notice) {
        startPage();
        List<SysNotice> list = noticeService.selectNoticeList(notice);
        return getDataTable(list);
    }
}
```

### 小程序端接口 / Client-side (Mini-program) API

路径前缀 `/api/v1/`，匿名可访问：

Path prefix `/api/v1/`, anonymous access:

```java
@RestController
@RequestMapping("/api/v1")
public class ClientNoticeController extends BaseController {

    @Anonymous
    @GetMapping("/notices")
    public TableDataInfo list() {
        startPage();
        return getDataTable(noticeService.selectNoticeList(query));
    }
}
```

### 两端接口的区别 / Differences Between Two Sides

| | 运营端 Operations | 小程序端 Client |
|--|--------|---------|
| 路径 Path | `/api/v1/ops/*` | `/api/v1/*` |
| 登录 Login | 需要登录 + 权限 Required + permission | 匿名 Anonymous (@Anonymous) |
| 数据范围 Data scope | 全部数据 All data | 只返回公开数据 Public data only |
| Swagger 分组 Swagger group | loop-v1 | loop-v1 |

## 四、返回格式 / Response Format

### AjaxResult（单个结果 / Single result）

```java
return AjaxResult.success(data);        // 成功，带数据 / Success with data
return AjaxResult.success();            // 成功，无数据 / Success, no data
return AjaxResult.error("错误信息");     // 失败 / Error
```

返回 JSON / Returns JSON:
```json
{"code": 200, "msg": "操作成功", "data": {...}}
```

### TableDataInfo（分页结果 / Paginated result）

```java
startPage();                            // 开启分页 / Enable pagination
List<T> list = service.selectList(query);
return getDataTable(list);
```

返回 JSON / Returns JSON:
```json
{"code": 200, "msg": "查询成功", "rows": [...], "total": 13}
```

## 五、BaseController 分页 / BaseController Pagination

继承 `BaseController` 后可以使用分页方法：

Extending `BaseController` gives access to pagination methods:

| 方法 Method | 作用 Purpose |
|------|------|
| `startPage()` | 开启分页（PageHelper 拦截 SQL 自动加 LIMIT）/ Enable pagination (PageHelper auto-adds LIMIT) |
| `getDataTable(list)` | 包装成分页结果（TableDataInfo）/ Wrap as paginated result |
| `toAjax(rows)` | 把影响行数转成 AjaxResult（>0 成功）/ Convert affected rows to AjaxResult (>0 = success) |

分页参数由前端传入：`pageNum`（页码）和 `pageSize`（每页条数）。

Pagination parameters are passed from frontend: `pageNum` (page number) and `pageSize` (items per page).

## 六、权限控制 / Access Control

### @PreAuthorize

```java
@PreAuthorize("@ss.hasPermi('loop:notice:add')")
```

- `@ss` 是 RuoYi 的权限验证器 / `@ss` is RuoYi's permission validator
- `hasPermi` 检查当前用户是否有指定权限 / `hasPermi` checks if current user has the specified permission
- 权限字符串格式 Permission string format：`模块:功能:操作 / module:feature:action`

### @Anonymous

```java
@Anonymous
@GetMapping("/notices")
```

标记接口为匿名访问，不需要登录。用于小程序端公开接口。

Marks the endpoint as anonymous access, no login required. Used for client-side public APIs.

### 权限管理 / Permission Management

权限通过 `sys_menu` 表的 `perms` 字段定义，角色通过 `sys_role_menu` 关联权限。

Permissions are defined in the `perms` field of the `sys_menu` table. Roles are linked to permissions via `sys_role_menu`.

## 七、SecurityUtils

获取当前登录用户信息 / Get current logged-in user info:

```java
SecurityUtils.getUsername();    // 当前用户名 / Current username
SecurityUtils.getUserId();      // 当前用户ID / Current user ID
SecurityUtils.getLoginUser();   // 完整登录用户对象 / Full login user object
```

## 八、Swagger 文档 / Swagger Documentation

### 注解 / Annotations

| 注解 Annotation | 作用位置 Applied to | 说明 Description |
|------|---------|------|
| `@Tag` | 类 Class | 分组标签 / Group tag |
| `@Operation` | 方法 Method | 接口摘要 / API summary |
| `@Schema` | 字段 Field | 字段描述和示例值 / Field description and example |

### 分组配置 / Group Configuration

Swagger 通过 `GroupedOpenApi` 配置分组：

Swagger configures groups via `GroupedOpenApi`:

```java
GroupedOpenApi.builder()
    .group("loop-v1")
    .pathsToMatch("/api/v1/**")
    .build()
```

访问 Swagger 页面后，右上角下拉框切换分组。

After opening the Swagger page, switch groups using the dropdown in the top right corner.

## 九、数据库操作 / Database Operations

### MyBatis 注解方式 / MyBatis Annotation Style

```java
@Mapper
public interface SysConfigMapper {
    @Select("SELECT * FROM sys_config WHERE config_key LIKE CONCAT(#{prefix}, '%')")
    List<SysConfig> selectByPrefix(@Param("prefix") String prefix);
}
```

### Service 层 / Service Layer

```java
public interface ISysConfigService {
    List<SysConfig> selectConfigList(SysConfig config);
    String selectConfigByKey(String configKey);
    int insertConfig(SysConfig config);
    int updateConfig(SysConfig config);
    void deleteConfigByIds(Long[] ids);
}
```

Service 层封装业务逻辑，Controller 调 Service，Service 调 Mapper。

The Service layer encapsulates business logic. Controller calls Service, Service calls Mapper.

## 十、常见问题 / Common Issues

### IDEA 编译报"不支持发行版本 22" / IDEA reports "unsupported release version 22"

Project Structure（Ctrl+Alt+Shift+S）→ Project SDK 选 17 → Language Level 选 17

Project Structure (Ctrl+Alt+Shift+S) → Set Project SDK to 17 → Set Language Level to 17

### 后端启动失败 / Backend fails to start

1. 检查端口 8080 是否被占用 / Check if port 8080 is occupied
2. 检查数据库/Redis 连接 / Check database/Redis connection
3. 执行 `mvn clean install -DskipTests` 重新编译 / Rebuild with Maven

### 前端页面打不开 / Frontend page won't load

前端开发服务器（端口 80）和后端（端口 8080）是独立进程，重启后端不会自动启动前端：

Frontend dev server (port 80) and backend (port 8080) are independent processes. Restarting the backend does not start the frontend:

```bash
cd ruoyi-ui
npm run dev
```
