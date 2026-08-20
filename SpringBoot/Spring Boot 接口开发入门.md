# Spring Boot 接口开发入门 / Getting Started with Spring Boot API Development

## 一、什么是 Controller / What is a Controller

Controller 是 Spring Boot 中处理 HTTP 请求的类。前端发请求过来，Controller 接收请求、调用 Service 处理业务逻辑、返回结果。

A Controller is a class in Spring Boot that handles HTTP requests. When the frontend sends a request, the Controller receives it, calls the Service layer to process business logic, and returns the result.

```
前端 ──HTTP请求──► Controller ──调用──► Service ──SQL──► 数据库
Frontend ──HTTP Request──► Controller ──calls──► Service ──SQL──► Database
                    ↑ You write code here
```

## 二、核心注解 / Core Annotations

### 1. @RestController

标记一个类为接口控制器，告诉 Spring："这个类是处理 HTTP 请求的"。

Marks a class as a REST controller, telling Spring: "This class handles HTTP requests."

```java
@RestController
@RequestMapping("/api/v1/ops/notice")
public class LoopNoticeController extends BaseController {
}
```

- `@RestController` = `@Controller` + `@ResponseBody`
- 所有方法的返回值自动转成 JSON 返回给前端
- All method return values are automatically converted to JSON and sent to the frontend

### 2. @RequestMapping

定义接口的路径前缀，所有该类下的接口都会加上这个前缀。

Defines the path prefix for the controller. All endpoints in this class will be prefixed with this path.

```java
@RequestMapping("/api/v1/ops/notice")  // All endpoints start with /api/v1/ops/notice
```

### 3. HTTP 方法注解 / HTTP Method Annotations

| 注解 Annotation | HTTP 方法 Method | 用途 Purpose | 示例 Example |
|------|----------|------|------|
| `@GetMapping` | GET | 查询数据 Query data | 获取公告列表 Get notice list |
| `@PostMapping` | POST | 新增数据 Create data | 创建新公告 Create new notice |
| `@PutMapping` | PUT | 修改数据 Update data | 编辑公告 Edit notice |
| `@DeleteMapping` | DELETE | 删除数据 Delete data | 删除公告 Delete notice |

## 三、参数接收 / Parameter Receiving

### 1. @PathVariable — 从 URL 路径取参数 / Extract from URL path

```java
@GetMapping("/{noticeId}")
public AjaxResult getInfo(@PathVariable Long noticeId) {
    // 前端访问 /api/v1/ops/notice/21
    // Frontend accesses /api/v1/ops/notice/21
    // noticeId 的值就是 21 / noticeId value is 21
    return AjaxResult.success(noticeService.selectNoticeById(noticeId));
}
```

### 2. @RequestBody — 从请求体取 JSON 数据 / Extract JSON from request body

```java
@PostMapping
public AjaxResult add(@Valid @RequestBody NoticeCreate r) {
    // 前端传: {"title":"系统维护通知", "type":"2", "content":"<p>内容</p>"}
    // Frontend sends: {"title":"...", "type":"...", "content":"..."}
    // Spring 自动把 JSON 映射到 NoticeCreate 对象
    // Spring automatically maps JSON to NoticeCreate object
    SysNotice notice = new SysNotice();
    notice.setNoticeTitle(r.title());
    return toAjax(noticeService.insertNotice(notice));
}
```

### 3. 直接参数绑定 — 查询参数自动映射 / Direct parameter binding — auto-map query params

```java
@GetMapping
public TableDataInfo list(SysNotice notice) {
    // 前端访问 /api/v1/ops/notice?noticeTitle=维护&noticeType=2
    // Frontend accesses /api/v1/ops/notice?noticeTitle=维护&noticeType=2
    // Spring 自动把查询参数映射到 SysNotice 对象的对应字段
    // Spring auto-maps query parameters to SysNotice fields
    startPage();
    List<SysNotice> list = noticeService.selectNoticeList(notice);
    return getDataTable(list);
}
```

## 四、参数校验 @Valid / Parameter Validation

在入参对象上使用 JSR 303 校验注解，Spring 自动验证参数合法性。

Use JSR 303 validation annotations on input objects. Spring automatically validates parameter legality.

```java
public record NoticeCreate(
    @NotBlank(message = "标题不能为空 / Title cannot be empty")
    @Size(max = 50, message = "标题不能超过50个字符 / Title cannot exceed 50 characters")
    @Schema(description = "公告标题 / Notice title", example = "系统维护通知")
    String title,

    @NotBlank(message = "类型不能为空 / Type cannot be empty")
    @Schema(description = "公告类型（1=通知 2=公告）/ Notice type (1=notification 2=announcement)", example = "2")
    String type,

    @Schema(description = "公告内容（富文本HTML）/ Content (rich text HTML)", example = "<p>维护内容</p>")
    String content
) {}
```

常用校验注解 / Common validation annotations:

| 注解 Annotation | 作用 Purpose |
|------|------|
| `@NotBlank` | 不能为 null 或空字符串 / Cannot be null or empty string |
| `@Size(max=50)` | 最大长度限制 / Max length limit |
| `@Email` | 必须是邮箱格式 / Must be email format |
| `@Min` / `@Max` | 数值范围限制 / Numeric range limit |

## 五、权限控制 / Access Control

### 1. @PreAuthorize — 需要登录 + 权限 / Requires login + permission

```java
@PreAuthorize("@ss.hasPermi('loop:notice:add')")
@PostMapping
public AjaxResult add(@Valid @RequestBody NoticeCreate r) {
    // 用户必须登录且拥有 loop:notice:add 权限才能调用
    // User must be logged in and have loop:notice:add permission
}
```

### 2. @Anonymous — 匿名访问（不需要登录）/ Anonymous access (no login required)

```java
@Anonymous
@GetMapping
public TableDataInfo list() {
    // 任何人都能访问，不需要登录
    // Anyone can access, no login required
}
```

## 六、返回格式 / Response Format

### AjaxResult（单个结果 / Single result）

```java
return AjaxResult.success(data);  // {"code":200, "msg":"操作成功", "data":{...}}
return AjaxResult.error("公告不存在");  // {"code":500, "msg":"公告不存在 / Notice not found"}
```

### TableDataInfo（分页结果 / Paginated result）

```java
startPage();  // 开启分页 / Enable pagination
List<SysNotice> list = noticeService.selectNoticeList(notice);
return getDataTable(list);
// 返回 / Returns: {"code":200, "rows":[...], "total":13}
```

## 七、完整接口示例 / Complete API Example

```java
@RestController
@RequestMapping("/api/v1/ops/notice")
public class LoopNoticeController extends BaseController {

    private final ISysNoticeService noticeService;

    @PreAuthorize("@ss.hasPermi('loop:notice:list')")
    @GetMapping
    public TableDataInfo list(SysNotice notice) {
        startPage();
        List<SysNotice> list = noticeService.selectNoticeList(notice);
        return getDataTable(list);
    }

    @PreAuthorize("@ss.hasPermi('loop:notice:add')")
    @PostMapping
    public AjaxResult add(@Valid @RequestBody NoticeCreate r) {
        SysNotice notice = new SysNotice();
        notice.setNoticeTitle(r.title());
        notice.setNoticeType(r.type());
        notice.setNoticeContent(r.content());
        notice.setCreateBy(SecurityUtils.getUsername());
        return toAjax(noticeService.insertNotice(notice));
    }
}
```

## 八、record 语法（Java 16+）/ record Syntax

record 是 Java 16 引入的语法糖，用于定义不可变的数据载体。

`record` is syntactic sugar introduced in Java 16 for defining immutable data carriers.

```java
// record 写法 / record syntax
public record NoticeCreate(String title, String type, String content) {}

// 等价于传统写法 / Equivalent traditional syntax
public class NoticeCreate {
    private final String title;
    private final String type;
    private final String content;

    public NoticeCreate(String title, String type, String content) {
        this.title = title;
        this.type = type;
        this.content = content;
    }

    public String title() { return title; }
    public String type() { return type; }
    public String content() { return content; }
}
```

record 自动生成：构造器、getter（名为字段名而非 getXxx）、equals、hashCode、toString。

`record` auto-generates: constructor, getter (named after the field, not getXxx), equals, hashCode, toString.

## 九、接口文档 @Schema / @Operation / API Documentation

Swagger（OpenAPI）注解，自动生成接口文档。

Swagger (OpenAPI) annotations that auto-generate API documentation.

```java
@Tag(name = "平台公告", description = "运营端公告管理 / Operations notice management")
@RestController
public class LoopNoticeController {

    @Operation(summary = "新增公告 / Add notice", description = "运营端创建新公告")
    @PostMapping
    public AjaxResult add(@Valid @RequestBody NoticeCreate r) {}
}
```

- `@Tag`：Controller 级别的分组标签 / Controller-level group tag
- `@Operation`：接口级别的摘要说明 / Endpoint-level summary
- `@Schema`：字段级别的描述和示例值 / Field-level description and example

访问 `http://localhost:8080/swagger-ui.html` 即可看到自动生成的文档。

Visit `http://localhost:8080/swagger-ui.html` to view the auto-generated documentation.
