# Spring Boot 接口开发入门

## 一、什么是 Controller

Controller 是 Spring Boot 中处理 HTTP 请求的类。前端发请求过来，Controller 接收请求、调用 Service 处理业务逻辑、返回结果。

```
前端 ──HTTP请求──► Controller ──调用──► Service ──SQL──► 数据库
                    ↑ 你写的在这
```

## 二、核心注解

### 1. @RestController

标记一个类为接口控制器，告诉 Spring："这个类是处理 HTTP 请求的"。

```java
@RestController
@RequestMapping("/api/v1/ops/notice")
public class LoopNoticeController extends BaseController {
}
```

- `@RestController` = `@Controller` + `@ResponseBody`
- 所有方法的返回值自动转成 JSON 返回给前端

### 2. @RequestMapping

定义接口的路径前缀，所有该类下的接口都会加上这个前缀。

```java
@RequestMapping("/api/v1/ops/notice")  // 该类下所有接口都以 /api/v1/ops/notice 开头
```

### 3. HTTP 方法注解

| 注解 | HTTP 方法 | 用途 | 示例 |
|------|----------|------|------|
| `@GetMapping` | GET | 查询数据 | 获取公告列表 |
| `@PostMapping` | POST | 新增数据 | 创建新公告 |
| `@PutMapping` | PUT | 修改数据 | 编辑公告 |
| `@DeleteMapping` | DELETE | 删除数据 | 删除公告 |

## 三、参数接收

### 1. @PathVariable — 从 URL 路径取参数

```java
@GetMapping("/{noticeId}")
public AjaxResult getInfo(@PathVariable Long noticeId) {
    // 前端访问 /api/v1/ops/notice/21
    // noticeId 的值就是 21
    return AjaxResult.success(noticeService.selectNoticeById(noticeId));
}
```

### 2. @RequestBody — 从请求体取 JSON 数据

```java
@PostMapping
public AjaxResult add(@Valid @RequestBody NoticeCreate r) {
    // 前端传: {"title":"系统维护通知", "type":"2", "content":"<p>内容</p>"}
    // Spring 自动把 JSON 映射到 NoticeCreate 对象
    SysNotice notice = new SysNotice();
    notice.setNoticeTitle(r.title());
    return toAjax(noticeService.insertNotice(notice));
}
```

### 3. 直接参数绑定 — 查询参数自动映射

```java
@GetMapping
public TableDataInfo list(SysNotice notice) {
    // 前端访问 /api/v1/ops/notice?noticeTitle=维护&noticeType=2
    // Spring 自动把查询参数映射到 SysNotice 对象的对应字段
    startPage();
    List<SysNotice> list = noticeService.selectNoticeList(notice);
    return getDataTable(list);
}
```

## 四、参数校验 @Valid

在入参对象上使用 JSR 303 校验注解，Spring 自动验证参数合法性。

```java
public record NoticeCreate(
    @NotBlank(message = "标题不能为空")
    @Size(max = 50, message = "标题不能超过50个字符")
    @Schema(description = "公告标题", example = "系统维护通知")
    String title,

    @NotBlank(message = "类型不能为空")
    @Schema(description = "公告类型（1=通知 2=公告）", example = "2")
    String type,

    @Schema(description = "公告内容（富文本HTML）", example = "<p>维护内容</p>")
    String content
) {}
```

常用校验注解：

| 注解 | 作用 |
|------|------|
| `@NotBlank` | 不能为 null 或空字符串 |
| `@Size(max=50)` | 最大长度限制 |
| `@Email` | 必须是邮箱格式 |
| `@Min` / `@Max` | 数值范围限制 |

## 五、权限控制

### 1. @PreAuthorize — 需要登录 + 权限

```java
@PreAuthorize("@ss.hasPermi('loop:notice:add')")
@PostMapping
public AjaxResult add(@Valid @RequestBody NoticeCreate r) {
    // 用户必须登录且拥有 loop:notice:add 权限才能调用
}
```

### 2. @Anonymous — 匿名访问（不需要登录）

```java
@Anonymous
@GetMapping
public TableDataInfo list() {
    // 任何人都能访问，不需要登录
}
```

## 六、返回格式

### AjaxResult（单个结果）

```java
return AjaxResult.success(data);  // {"code":200, "msg":"操作成功", "data":{...}}
return AjaxResult.error("公告不存在");  // {"code":500, "msg":"公告不存在"}
```

### TableDataInfo（分页结果）

```java
startPage();  // 开启分页
List<SysNotice> list = noticeService.selectNoticeList(notice);
return getDataTable(list);
// 返回: {"code":200, "rows":[...], "total":13}
```

## 七、完整接口示例

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

## 八、record 语法（Java 16+）

record 是 Java 16 引入的语法糖，用于定义不可变的数据载体。

```java
// record 写法
public record NoticeCreate(String title, String type, String content) {}

// 等价于传统写法
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

## 九、接口文档 @Schema / @Operation

Swagger（OpenAPI）注解，自动生成接口文档。

```java
@Tag(name = "平台公告", description = "运营端公告管理")
@RestController
public class LoopNoticeController {

    @Operation(summary = "新增公告", description = "运营端创建新公告")
    @PostMapping
    public AjaxResult add(@Valid @RequestBody NoticeCreate r) {}
}
```

- `@Tag`：Controller 级别的分组标签
- `@Operation`：接口级别的摘要说明
- `@Schema`：字段级别的描述和示例值

访问 `http://localhost:8080/swagger-ui.html` 即可看到自动生成的文档。
