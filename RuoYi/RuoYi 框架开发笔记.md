# RuoYi 框架开发笔记

## 一、项目结构

RuoYi 是基于 Spring Boot 的后台管理框架，采用多模块 Maven 结构。

```
loop-serv/
├── ruoyi-admin/      → 启动模块（Controller 在这）
├── ruoyi-framework/  → 框架核心（Security、配置）
├── ruoyi-system/     → 系统模块（用户、菜单、配置）
├── ruoyi-common/     → 公共工具（SecurityUtils、AjaxResult）
├── ruoyi-loop/       → 业务模块（LOOP 业务逻辑）
├── ruoyi-ui/         → 前端 Vue 项目
└── sql/              → 数据库脚本
```

## 二、入口与启动

- 启动类：`ruoyi-admin` 模块下的 `RuoYiApplication.java`
- 后端端口：8080
- 前端端口：80（`npm run dev` 启动）
- Swagger 地址：`http://localhost:8080/swagger-ui.html`

### 启动顺序

1. 先启动后端：IDEA 中运行 `RuoYiApplication`
2. 再启动前端：`cd ruoyi-ui && npm run dev`
3. 前端通过代理把 `/dev-api` 请求转发到 8080

## 三、Controller 开发模式

### 运营端接口

路径前缀 `/api/v1/ops/`，需要登录 + 权限：

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

### 小程序端接口

路径前缀 `/api/v1/`，匿名可访问：

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

### 两端接口的区别

| | 运营端 | 小程序端 |
|--|--------|---------|
| 路径 | `/api/v1/ops/*` | `/api/v1/*` |
| 登录 | 需要登录 + 权限 | 匿名（@Anonymous） |
| 数据范围 | 全部数据 | 只返回公开数据 |
| Swagger 分组 | loop-v1 | loop-v1 |

## 四、返回格式

### AjaxResult（单个结果）

```java
return AjaxResult.success(data);        // 成功，带数据
return AjaxResult.success();            // 成功，无数据
return AjaxResult.error("错误信息");     // 失败
```

返回 JSON：
```json
{"code": 200, "msg": "操作成功", "data": {...}}
```

### TableDataInfo（分页结果）

```java
startPage();                            // 开启分页
List<T> list = service.selectList(query);
return getDataTable(list);
```

返回 JSON：
```json
{"code": 200, "msg": "查询成功", "rows": [...], "total": 13}
```

## 五、BaseController 分页

继承 `BaseController` 后可以使用分页方法：

| 方法 | 作用 |
|------|------|
| `startPage()` | 开启分页（PageHelper 拦截 SQL 自动加 LIMIT） |
| `getDataTable(list)` | 包装成分页结果（TableDataInfo） |
| `toAjax(rows)` | 把影响行数转成 AjaxResult（>0 成功） |

分页参数由前端传入：`pageNum`（页码）和 `pageSize`（每页条数）。

## 六、权限控制

### @PreAuthorize

```java
@PreAuthorize("@ss.hasPermi('loop:notice:add')")
```

- `@ss` 是 RuoYi 的权限验证器
- `hasPermi` 检查当前用户是否有指定权限
- 权限字符串格式：`模块:功能:操作`

### @Anonymous

```java
@Anonymous
@GetMapping("/notices")
```

标记接口为匿名访问，不需要登录。用于小程序端公开接口。

### 权限管理

权限通过 `sys_menu` 表的 `perms` 字段定义，角色通过 `sys_role_menu` 关联权限。

## 七、SecurityUtils

获取当前登录用户信息：

```java
SecurityUtils.getUsername();    // 当前用户名
SecurityUtils.getUserId();      // 当前用户ID
SecurityUtils.getLoginUser();   // 完整登录用户对象
```

## 八、Swagger 文档

### 注解

| 注解 | 作用位置 | 说明 |
|------|---------|------|
| `@Tag` | 类 | 分组标签 |
| `@Operation` | 方法 | 接口摘要 |
| `@Schema` | 字段 | 字段描述和示例值 |

### 分组配置

Swagger 通过 `GroupedOpenApi` 配置分组：

```java
GroupedOpenApi.builder()
    .group("loop-v1")
    .pathsToMatch("/api/v1/**")
    .build()
```

访问 Swagger 页面后，右上角下拉框切换分组。

## 九、数据库操作

### MyBatis 注解方式

```java
@Mapper
public interface SysConfigMapper {
    @Select("SELECT * FROM sys_config WHERE config_key LIKE CONCAT(#{prefix}, '%')")
    List<SysConfig> selectByPrefix(@Param("prefix") String prefix);
}
```

### Service 层

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

## 十、常见问题

### IDEA 编译报"不支持发行版本 22"

Project Structure（Ctrl+Alt+Shift+S）→ Project SDK 选 17 → Language Level 选 17

### 后端启动失败

1. 检查端口 8080 是否被占用
2. 检查数据库/Redis 连接
3. 执行 `mvn clean install -DskipTests` 重新编译

### 前端页面打不开

前端开发服务器（端口 80）和后端（端口 8080）是独立进程，重启后端不会自动启动前端：
```bash
cd ruoyi-ui
npm run dev
```
