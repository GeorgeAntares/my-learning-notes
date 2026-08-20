# MySQL 数据库基础 / MySQL Database Basics

## 一、连接数据库 / Connecting to Database

项目使用 MySQL，通过 Navicat 等工具连接。

The project uses MySQL, connected via tools like Navicat.

> **注意 Note**：以下为配置模板，请替换为实际值。
> The following is a config template. Replace with actual values.

| 参数 Parameter | 值 Value | 说明 Description |
|------|------|------|
| Host | `<db-host>` | 数据库服务器地址 / Database server address |
| Port | `<db-port>` | 端口 / Port |
| Database | `<db-name>` | 数据库名 / Database name |
| Username | `<username>` | 用户名 / Username |
| Password | `<password>` | 密码 / Password |

## 二、表结构文件位置 / Table Schema File Location

项目用 Flyway 管理表结构，SQL 脚本在 `sql/` 目录下，按版本号递增：

The project uses Flyway for schema management. SQL scripts are in the `sql/` directory, ordered by version number:

```
sql/
├── ry_20260417.sql              → RuoYi 基础表 / RuoYi base tables
└── loop/
    ├── V001__init.sql           → 第1版 / Version 1
    ├── V002__venue.sql          → 场馆表 / Venue tables
    └── V016__content_lifecycle.sql → 内容生命周期表 / Content lifecycle tables
```

## 三、常用表 / Common Tables

### sys_config（系统配置表 / System Config Table）

| 字段 Field | 类型 Type | 说明 Description |
|------|------|------|
| config_id | int | 配置ID（主键）/ Config ID (PK) |
| config_name | varchar(100) | 配置名称 / Config name |
| config_key | varchar(100) | 配置键（项目前缀开头）/ Config key (starts with project prefix) |
| config_value | varchar(500) | 配置值 / Config value |
| config_type | char(1) | Y=内置 N=自定义 / Y=built-in N=custom |
| create_by / create_time | — | 创建审计 / Create audit |
| update_by / update_time | — | 更新审计 / Update audit |
| remark | varchar(500) | 备注 / Remark |

### sys_notice（通知公告表 / Notice Table）

| 字段 Field | 类型 Type | 说明 Description |
|------|------|------|
| notice_id | int | 公告ID（主键）/ Notice ID (PK) |
| notice_title | varchar(50) | 标题 / Title |
| notice_type | char(1) | 1=通知 2=公告 / 1=notification 2=announcement |
| notice_content | longblob | 内容（富文本HTML）/ Content (rich text HTML) |
| status | char(1) | 0=正常 1=关闭 / 0=active 1=closed |
| create_by / create_time | — | 创建审计 / Create audit |

### sys_menu（菜单权限表 / Menu Permission Table）

| 字段 Field | 类型 Type | 说明 Description |
|------|------|------|
| menu_id | bigint | 菜单ID（主键）/ Menu ID (PK) |
| menu_name | varchar(50) | 菜单名称 / Menu name |
| parent_id | bigint | 父菜单ID / Parent menu ID |
| order_num | int | 显示顺序 / Display order |
| path | varchar(200) | 路由地址 / Route address |
| component | varchar(255) | 组件路径 / Component path |
| menu_type | char(1) | M=目录 C=菜单 F=按钮 / M=dir C=menu F=button |
| visible | char(1) | 0=显示 1=隐藏 / 0=visible 1=hidden |
| perms | varchar(100) | 权限标识 / Permission identifier |
| icon | varchar(100) | 菜单图标 / Menu icon |

## 四、常用 SQL / Common SQL

### 查询 / Query

```sql
-- 查询所有项目前缀开头的配置 / Query all configs starting with project prefix
SELECT * FROM sys_config WHERE config_key LIKE '<prefix>.%';

-- 查询已发布的公告 / Query published notices
SELECT * FROM sys_notice WHERE status = '0' ORDER BY create_time DESC;

-- 查询某个菜单的子菜单 / Query child menus
SELECT * FROM sys_menu WHERE parent_id = 2250 ORDER BY order_num;
```

### 新增 / Insert

```sql
-- 新增配置 / Insert config
INSERT INTO sys_config (config_name, config_key, config_value, config_type, create_by, create_time)
VALUES ('客服电话', '<prefix>.customer.service.phone', '400-xxx-xxxx', 'N', 'admin', NOW());
```

### 更新 / Update

```sql
-- 修改配置值 / Update config value
UPDATE sys_config SET config_value = '400-xxx-xxxx', update_by = 'admin', update_time = NOW()
WHERE config_key = '<prefix>.customer.service.phone';
```

### 删除 / Delete

```sql
-- 删除配置 / Delete config
DELETE FROM sys_config WHERE config_key = '<prefix>.customer.service.phone';

-- 批量删除公告 / Batch delete notices
DELETE FROM sys_notice WHERE notice_id IN (1, 2, 3);
```

## 五、表设计要点 / Table Design Key Points

### 审计字段 / Audit Fields

几乎所有表都有这 4 个审计字段：

Almost all tables have these 4 audit fields:

| 字段 Field | 说明 Description |
|------|------|
| create_by | 创建人 / Created by |
| create_time | 创建时间 / Creation time |
| update_by | 更新人 / Updated by |
| update_time | 更新时间 / Update time |

### 状态字段约定 / Status Field Convention

| 值 Value | 含义 Meaning |
|------|------|
| 0 | 正常/启用 / Normal/Active |
| 1 | 关闭/禁用 / Closed/Disabled |

### 配置键命名空间 / Config Key Namespace

配置键使用 `.` 分隔的命名空间：

Config keys use `.` separated namespace:

```
<project>.            → 项目前缀 / Project prefix
<project>.customer.*   → 客服相关 / Customer service related
<project>.app.*        → 应用相关 / App related
<project>.order.*      → 订单相关 / Order related
```

## 六、Navicat 使用技巧 / Navicat Tips

| 操作 Operation | 方法 Method |
|------|------|
| 查看表结构 View table schema | 双击表 → 设计表 / Double-click → Design Table |
| 执行 SQL Execute SQL | 点击查询 → 新建查询 / Click Query → New Query |
| 导出表数据 Export data | 右键表 → 导出向导 / Right-click → Export Wizard |
| 搜索数据 Search data | 工具 → 数据搜索 / Tools → Data Search |
