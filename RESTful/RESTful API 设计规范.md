# RESTful API 设计规范 / RESTful API Design Conventions

## 一、什么是 RESTful / What is RESTful

REST（Representational State Transfer）是一种 API 设计风格，用 HTTP 方法表示操作，用 URL 表示资源。

REST (Representational State Transfer) is an API design style that uses HTTP methods to represent operations and URLs to represent resources.

### 核心原则 / Core Principles

| 原则 Principle | 说明 Description |
|------|------|
| 资源导向 Resource-oriented | URL 表示资源，不是动作 / URL represents resource, not action |
| HTTP 方法表示操作 / HTTP method = operation | GET=查 POST=增 PUT=改 DELETE=删 |
| 无状态 Stateless | 每个请求包含所有信息 / Each request is self-contained |
| 统一接口 Uniform interface | 一致的 URL 和返回格式 / Consistent URLs and response format |

## 二、URL 设计 / URL Design

### 资源命名 / Resource Naming

| 规则 Rule | 正确 Correct | 错误 Wrong |
|------|------|------|
| 用名词复数 Use plural nouns | `/api/v1/notices` | `/api/v1/getNotice` |
| 不含动词 No verbs in URL | `/api/v1/ops/notice` | `/api/v1/ops/createNotice` |
| 层级表达关系 Hierarchy for relations | `/api/v1/ops/notice/{id}/read` | `/api/v1/ops/markNoticeRead?id=1` |
| 小写加连字符 Lowercase with hyphens | `/api/v1/user-profiles` | `/api/v1/UserProfiles` |

### 路径分层 / Path Hierarchy

```
/api/v1/                    → API 版本 + 小程序端 / API version + client side
/api/v1/ops/                → API 版本 + 运营端 / API version + ops side
/api/v1/ops/notice          → 运营端公告资源 / Ops notice resource
/api/v1/ops/notice/{id}     → 具体公告 / Specific notice
/api/v1/ops/notice/{id}/read → 公告的已读操作 / Notice read action
```

### 版本管理 / Versioning

```
/api/v1/    → 第一版 / Version 1
/api/v2/    → 第二版 / Version 2
```

在 URL 中包含版本号，方便后续升级不破坏旧接口。

Include version in URL for non-breaking upgrades.

## 三、HTTP 方法 / HTTP Methods

| 方法 Method | 操作 Operation | 幂等 Idempotent | 安全 Safe | 示例 Example |
|------|------|------|------|------|
| GET | 查询 Query | ✅ 是 Yes | ✅ 是 Yes | 获取公告列表 Get notice list |
| POST | 新增 Create | ❌ 否 No | ❌ 否 No | 创建新公告 Create notice |
| PUT | 全量修改 Full update | ✅ 是 Yes | ❌ 否 No | 更新公告 Update notice |
| PATCH | 部分修改 Partial update | ❌ 否 No | ❌ 否 No | 只改状态 Change status only |
| DELETE | 删除 Delete | ✅ 是 Yes | ❌ 否 No | 删除公告 Delete notice |

**幂等 Idempotent**：多次执行结果相同。GET 请求 100 次结果一样，POST 100 次可能创建 100 条数据。

Idempotent: Multiple executions produce the same result. GET 100 times = same result, POST 100 times = 100 records.

## 四、状态码 / Status Codes

### HTTP 状态码 / HTTP Status Codes

| 码 Code | 含义 Meaning | 场景 Scenario |
|------|------|------|
| 200 | 成功 OK | 请求正常处理 / Request processed |
| 201 | 已创建 Created | POST 新增成功 / POST create success |
| 400 | 请求错误 Bad Request | 参数校验失败 / Validation failed |
| 401 | 未授权 Unauthorized | 未登录 / Not logged in |
| 403 | 禁止访问 Forbidden | 无权限 / No permission |
| 404 | 未找到 Not Found | 资源不存在 / Resource not found |
| 500 | 服务器错误 Internal Error | 代码异常 / Code exception |

### 业务状态码 / Business Codes

RuoYi 框架使用自定义业务码在 HTTP 200 内返回：

RuoYi framework returns custom business codes within HTTP 200:

```json
{
  "code": 200,           // 200=成功 500=失败 / 200=success 500=fail
  "msg": "操作成功",      // 提示信息 / Message
  "data": { ... }         // 数据 / Data
}
```

| 码 Code | 含义 Meaning |
|------|------|
| 200 | 操作成功 / Success |
| 500 | 操作失败 / Fail |
| 401 | 未登录 / Not logged in |
| 403 | 无权限 / No permission |

## 五、请求与响应 / Request & Response

### GET 请求参数 / GET Parameters

通过 URL 查询参数传递：

Pass via URL query parameters:

```
GET /api/v1/ops/notice?noticeTitle=维护&noticeType=2&pageNum=1&pageSize=10
```

| 参数 Parameter | 类型 Type | 说明 Description |
|------|------|------|
| pageNum | int | 页码 / Page number |
| pageSize | int | 每页条数 / Items per page |
| (字段名) Field name | — | 过滤条件 / Filter condition |

### POST/PUT 请求体 / POST/PUT Body

通过 JSON 请求体传递：

Pass via JSON request body:

```json
POST /api/v1/ops/notice
Content-Type: application/json

{
  "title": "系统维护通知",
  "type": "2",
  "content": "<p>将于今晚22:00-次日02:00进行系统维护</p>",
  "status": "0"
}
```

### 统一响应格式 / Unified Response Format

#### 单个结果 Single Result

```json
{
  "code": 200,
  "msg": "操作成功 / Success",
  "data": {
    "noticeId": 21,
    "noticeTitle": "系统维护通知",
    "noticeType": "2",
    "noticeContent": "<p>...</p>",
    "status": "0",
    "createTime": "2026-08-18 10:30:00"
  }
}
```

#### 分页结果 Paginated Result

```json
{
  "code": 200,
  "msg": "查询成功 / Success",
  "rows": [
    { "noticeId": 21, "noticeTitle": "..." },
    { "noticeId": 22, "noticeTitle": "..." }
  ],
  "total": 13
}
```

#### 错误响应 Error Response

```json
{
  "code": 500,
  "msg": "公告不存在 / Notice not found"
}
```

## 六、权限与认证 / Authentication & Authorization

### Token 认证 / Token Authentication

```
# 需要登录的接口 / APIs requiring login
Authorization: Bearer <token>

# 匿名接口不需要 / Anonymous APIs don't need it
```

### 权限粒度 / Permission Granularity

| 层级 Level | 示例 Example | 说明 Description |
|------|------|------|
| 接口级 Endpoint | `@PreAuthorize("@ss.hasPermi('loop:notice:add')")` | 每个接口单独控制 / Per-endpoint control |
| 角色级 Role | `@PreAuthorize("@ss.hasRole('admin')")` | 按角色控制 / By role |
| 匿名 Anonymous | `@Anonymous` | 不需要登录 / No login required |

### 权限命名规范 / Permission Naming Convention

```
模块:功能:操作
module:feature:action

loop:notice:list    → 公告列表权限 / Notice list permission
loop:notice:add     → 公告新增权限 / Notice add permission
loop:notice:edit    → 公告编辑权限 / Notice edit permission
loop:notice:remove  → 公告删除权限 / Notice remove permission
```

## 七、接口设计实例 / API Design Examples

### 增删改查完整设计 / Complete CRUD Design

| 操作 Operation | 方法 Method | URL | 入参 Input | 出参 Output |
|------|------|------|------|------|
| 列表 List | GET | `/api/v1/ops/notice` | 查询参数 Query params | 分页 Paginated |
| 详情 Detail | GET | `/api/v1/ops/notice/{id}` | 路径参数 Path param | 单个 Single |
| 新增 Create | POST | `/api/v1/ops/notice` | JSON body | 成功/失败 Success/Fail |
| 修改 Update | PUT | `/api/v1/ops/notice` | JSON body (含id) | 成功/失败 Success/Fail |
| 删除 Delete | DELETE | `/api/v1/ops/notice/{ids}` | 路径参数 Path param | 成功/失败 Success/Fail |

### 两端分离设计 / Two-Side Separation Design

| 端 Side | 路径 Path | 认证 Auth | 数据范围 Data Scope |
|------|------|------|------|
| 运营端 Ops | `/api/v1/ops/notice` | 需登录+权限 Login+permission | 全部 All |
| 小程序端 Client | `/api/v1/notices` | 匿名 Anonymous | 仅公开 Public only |

## 八、Swagger 文档 / Swagger Documentation

### 注解使用 / Annotation Usage

```java
@Tag(name = "平台公告", description = "Notice management")
@RestController
@RequestMapping("/api/v1/ops/notice")
public class LoopNoticeController {

    @Operation(summary = "公告列表", description = "分页查询公告")
    @GetMapping
    public TableDataInfo list() { ... }

    @Operation(summary = "新增公告", description = "创建新公告")
    @PostMapping
    public AjaxResult add(@Valid @RequestBody NoticeCreate r) { ... }
}
```

### 文档要点 / Documentation Key Points

| 要素 Element | 注解 Annotation | 说明 Description |
|------|------|------|
| 分组 Group | `@Tag` | Controller 级别分类 / Controller-level grouping |
| 摘要 Summary | `@Operation(summary=)` | 一句话描述 / One-line description |
| 字段说明 Field desc | `@Schema(description=)` | 每个字段的含义 / Each field's meaning |
| 示例值 Example | `@Schema(example=)` | 示例输入值 / Example input value |
| 校验规则 Validation | `@NotBlank`, `@Size` | 参数限制 / Parameter constraints |
