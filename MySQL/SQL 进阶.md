# SQL 进阶 / SQL Advanced

> JOIN、子查询、窗口函数、索引优化，覆盖面试和实际开发场景。
> JOIN, subqueries, window functions, index optimization, covering interview and practical scenarios.

---

## 一、JOIN 连接查询 / JOIN Queries

### 1. JOIN 类型一览 / JOIN Types Overview

```
表 A          表 B
┌─────────┐  ┌─────────┐
│  INNER  │  │         │
│  JOIN   │  │         │
├─────────┼──┼─────────┤
│  LEFT   │  │  RIGHT  │
│  JOIN   │  │  JOIN   │
├─────────┴──┴─────────┤
│     FULL  JOIN       │
└──────────────────────┘
```

| JOIN 类型 | 含义 | 图示 |
|-----------|------|------|
| INNER JOIN | 两表都有的交集 / Intersection | ⨂ |
| LEFT JOIN | 左表全部 + 右表匹配 / All left + matched right | ◐ |
| RIGHT JOIN | 右表全部 + 左表匹配 / All right + matched left | ◑ |
| FULL JOIN | 两表全部 / All from both | ◯ |

### 2. 示例表结构 / Sample Tables

```sql
-- 员工表 Employees
CREATE TABLE employees (
    emp_id   INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id  INT
);

-- 部门表 Departments
CREATE TABLE departments (
    dept_id   INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

-- 示例数据 Sample data
INSERT INTO employees VALUES
(1, '张三 Zhang San', 1),
(2, '李四 Li Si', 2),
(3, '王五 Wang Wu', NULL),   -- 没有部门 No department
(4, '赵六 Zhao Liu', 5);      -- 部门5不存在 Dept 5 doesn't exist

INSERT INTO departments VALUES
(1, '技术部 Tech'),
(2, '市场部 Marketing'),
(3, '财务部 Finance');        -- 没有员工 No employees
```

### 3. INNER JOIN（内连接）

只返回两表都匹配的行 / Returns only matching rows from both tables:

```sql
SELECT e.emp_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;

-- 结果 Result:
-- 张三  | 技术部
-- 李四  | 市场部
-- （王五没有部门，不出现 / Wang Wu has no dept, excluded）
-- （赵六的部门不存在，不出现 / Zhao Liu's dept doesn't exist, excluded）
```

### 4. LEFT JOIN（左连接）

返回左表全部行，右表不匹配的填 NULL / Returns all left rows, NULL for unmatched right:

```sql
SELECT e.emp_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;

-- 结果 Result:
-- 张三  | 技术部
-- 李四  | 市场部
-- 王五  | NULL      ← 左表全部保留 Left table fully kept
-- 赵六  | NULL
```

### 5. RIGHT JOIN（右连接）

返回右表全部行，左表不匹配的填 NULL / Returns all right rows:

```sql
SELECT e.emp_name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;

-- 结果 Result:
-- 张三  | 技术部
-- 李四  | 市场部
-- NULL  | 财务部    ← 右表全部保留 Right table fully kept
```

### 6. 多表 JOIN / Multiple JOINs

```sql
-- 查询员工、部门、薪资信息
SELECT e.emp_name, d.dept_name, s.salary
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN salaries s ON e.emp_id = s.emp_id
WHERE s.salary > 10000
ORDER BY s.salary DESC;
```

### 7. JOIN 性能要点 / JOIN Performance

| 要点 Point | 说明 |
|-----------|------|
| 小表驱动大表 Small drives large | 小表在前，大表在后 / Small table first |
| JOIN 字段建索引 Index JOIN columns | ON 条件字段加索引 / Add index on ON columns |
| 避免 SELECT * | 只查需要的列 / Select only needed columns |
| 限制 JOIN 数量 | 不超过 3-4 个表 / Max 3-4 tables |

---

## 二、子查询 / Subqueries

### 1. 子查询类型 / Subquery Types

| 类型 Type | 位置 Position | 特点 |
|----------|-------------|------|
| 标量子查询 Scalar | SELECT 子句 | 返回单个值 / Returns single value |
| 行子查询 Row | WHERE 子句 | 返回一行 / Returns one row |
| 列子查询 Column | WHERE 子句 | 返回一列 / Returns one column |
| 表子查询 Table | FROM 子句 | 返回多行多列 / Returns a table |
| EXISTS 子查询 | WHERE 子句 | 判断是否存在 / Check existence |

### 2. 标量子查询 / Scalar Subquery

```sql
-- 查询薪资高于平均值的员工
-- Find employees with above-average salary
SELECT emp_name, salary
FROM salaries
WHERE salary > (SELECT AVG(salary) FROM salaries);

-- 结果 Result:
-- 张三  | 15000    （平均 9000，高于平均）
-- 李四  | 12000
```

### 3. IN 子查询 / IN Subquery

```sql
-- 查询有薪水记录的员工
-- Find employees that have salary records
SELECT emp_name
FROM employees
WHERE emp_id IN (SELECT emp_id FROM salaries);

-- NOT IN：没有薪水记录的员工
SELECT emp_name
FROM employees
WHERE emp_id NOT IN (SELECT emp_id FROM salaries WHERE emp_id IS NOT NULL);
```

> **注意 Note**：`NOT IN` 子查询中有 NULL 值时会导致整个查询返回空。使用 `NOT EXISTS` 更安全。
> `NOT IN` with NULL values in subquery returns empty. Use `NOT EXISTS` instead.

### 4. EXISTS 子查询 / EXISTS Subquery

```sql
-- 查询有薪水记录的员工（EXISTS 版）
SELECT e.emp_name
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM salaries s
    WHERE s.emp_id = e.emp_id
);

-- 查询没有薪水记录的员工
SELECT e.emp_name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM salaries s
    WHERE s.emp_id = e.emp_id
);
```

**IN vs EXISTS：**

| | IN | EXISTS |
|--|----|----|
| 执行方式 | 先执行子查询，再匹配 | 外层查询逐行判断 |
| 适合 | 子查询结果小 / Small subquery | 外层查询小 / Small outer query |

### 5. 表子查询（派生表）/ Derived Table

```sql
-- 查询每个部门的平均薪资，并筛选高于 10000 的
SELECT dept_id, avg_salary
FROM (
    SELECT e.dept_id, AVG(s.salary) AS avg_salary
    FROM employees e
    JOIN salaries s ON e.emp_id = s.emp_id
    GROUP BY e.dept_id
) AS dept_avg
WHERE avg_salary > 10000;
```

---

## 三、聚合与分组 / Aggregation and Grouping

### 1. 常用聚合函数 / Common Aggregate Functions

| 函数 | 含义 | 示例 |
|------|------|------|
| COUNT() | 计数 | `COUNT(*)` — 行数；`COUNT(col)` — 非空个数 |
| SUM() | 求和 | `SUM(salary)` |
| AVG() | 平均值 | `AVG(salary)` |
| MAX() | 最大值 | `MAX(salary)` |
| MIN() | 最小值 | `MIN(salary)` |

### 2. GROUP BY / HAVING

```sql
-- 查询每个部门的员工数量和平均薪资
SELECT
    d.dept_name,
    COUNT(*) AS emp_count,
    AVG(s.salary) AS avg_salary
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
JOIN salaries s ON e.emp_id = s.emp_id
GROUP BY d.dept_name
HAVING AVG(s.salary) > 8000   -- 分组后筛选 Filter after grouping
ORDER BY avg_salary DESC;
```

**WHERE vs HAVING：**

| | WHERE | HAVING |
|--|-------|--------|
| 时机 | 分组前过滤 | 分组后过滤 |
| 能用聚合函数 | ❌ | ✅ |
| 位置 | GROUP BY 前 | GROUP BY 后 |

---

## 四、窗口函数 / Window Functions

窗口函数是 SQL 的高级特性，在面试中经常出现。

Window functions are advanced SQL features, frequently asked in interviews.

### 1. 基本语法 / Basic Syntax

```sql
函数名() OVER (
    PARTITION BY <分组列>   -- 类似 GROUP BY，但不合并行
    ORDER BY <排序列>       -- 组内排序
    ROWS BETWEEN ... AND ... -- 窗口范围
)
```

### 2. 排名函数 / Ranking Functions

```sql
-- 按部门分组，薪资降序排名
SELECT
    e.emp_name,
    d.dept_name,
    s.salary,
    ROW_NUMBER() OVER (PARTITION BY e.dept_id ORDER BY s.salary DESC) AS row_num,
    RANK()       OVER (PARTITION BY e.dept_id ORDER BY s.salary DESC) AS rank_val,
    DENSE_RANK() OVER (PARTITION BY e.dept_id ORDER BY s.salary DESC) AS dense_rank_val
FROM employees e
JOIN salaries s ON e.emp_id = s.emp_id
JOIN departments d ON e.dept_id = d.dept_id;
```

**三个排名函数的区别 Ranking differences：**

| 薪资 Salary | ROW_NUMBER | RANK | DENSE_RANK |
|------------|-----------|------|------------|
| 15000 | 1 | 1 | 1 |
| 12000 | 2 | 2 | 2 |
| 12000 | 3 | 2 | 2 |  ← 并列后 RANK 跳号，DENSE_RANK 不跳
| 10000 | 4 | 4 | 3 |  ← RANK 跳到4，DENSE_RANK 只+1

### 3. 聚合窗口函数 / Aggregate Window Functions

```sql
-- 累计求和 Running total
SELECT
    emp_name,
    salary,
    SUM(salary) OVER (ORDER BY emp_id) AS running_total,
    AVG(salary) OVER (ORDER BY emp_id) AS running_avg
FROM salaries;

-- 每行占总和的百分比 Percentage of total
SELECT
    emp_name,
    salary,
    salary * 100.0 / SUM(salary) OVER () AS pct_of_total
FROM salaries;
```

### 4. 偏移函数 / Offset Functions

```sql
-- LAG：取前一行 / Previous row
-- LEAD：取后一行 / Next row
SELECT
    emp_name,
    salary,
    LAG(salary, 1) OVER (ORDER BY salary)  AS prev_salary,
    salary - LAG(salary, 1) OVER (ORDER BY salary) AS diff,
    LEAD(salary, 1) OVER (ORDER BY salary) AS next_salary
FROM salaries;

-- 结果示例 Result:
-- 张三  | 8000  | NULL  | NULL  | 9000
-- 李四  | 9000  | 8000  | 1000  | 10000
-- 王五  | 10000 | 9000  | 1000  | 15000
```

### 5. FIRST_VALUE / LAST_VALUE

```sql
-- 每个部门薪资最高和最低的人
SELECT
    emp_name,
    dept_name,
    salary,
    FIRST_VALUE(emp_name) OVER (PARTITION BY dept_id ORDER BY salary DESC) AS highest_paid,
    LAST_VALUE(emp_name) OVER (
        PARTITION BY dept_id ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid
FROM employees e
JOIN salaries s ON e.emp_id = s.emp_id;
```

### 6. 常见面试题 / Common Interview Questions

**Q1: 查询每个部门薪资最高的前 2 名员工**

```sql
SELECT * FROM (
    SELECT
        e.emp_name,
        d.dept_name,
        s.salary,
        ROW_NUMBER() OVER (PARTITION BY e.dept_id ORDER BY s.salary DESC) AS rn
    FROM employees e
    JOIN salaries s ON e.emp_id = s.emp_id
    JOIN departments d ON e.dept_id = d.dept_id
) ranked
WHERE rn <= 2;
```

**Q2: 查询薪资连续上升的员工**

```sql
SELECT emp_name FROM (
    SELECT
        emp_name,
        salary,
        salary - LAG(salary) OVER (ORDER BY emp_id) AS diff
    FROM salaries
) t
WHERE diff > 0;
```

---

## 五、索引 / Indexes

### 1. 索引类型 / Index Types

| 类型 | 特点 | 适用场景 |
|------|------|---------|
| 主键索引 PRIMARY | 唯一 + 非空 | 每张表必须有 |
| 唯一索引 UNIQUE | 唯一 | 邮箱、手机号 |
| 普通索引 INDEX | 加速查询 | WHERE / JOIN 条件 |
| 联合索引 Composite | 多列组合 | 多条件查询 |
| 全文索引 FULLTEXT | 文本搜索 | 搜索功能 |

### 2. 创建索引 / Creating Indexes

```sql
-- 普通索引
CREATE INDEX idx_emp_name ON employees(emp_name);

-- 唯一索引
CREATE UNIQUE INDEX idx_email ON users(email);

-- 联合索引
CREATE INDEX idx_dept_salary ON salaries(emp_id, salary);

-- 查看索引
SHOW INDEX FROM employees;

-- 删除索引
DROP INDEX idx_emp_name ON employees;
```

### 3. 联合索引最左前缀原则 / Leftmost Prefix Rule

```sql
-- 联合索引 (a, b, c)
-- ✅ 能用到索引 Can use index:
SELECT * FROM t WHERE a = 1;
SELECT * FROM t WHERE a = 1 AND b = 2;
SELECT * FROM t WHERE a = 1 AND b = 2 AND c = 3;

-- ❌ 不能用到索引 Cannot use index:
SELECT * FROM t WHERE b = 2;       -- 跳过了 a
SELECT * FROM t WHERE c = 3;       -- 跳过了 a 和 b
```

### 4. 索引优化原则 / Index Optimization

| 原则 Rule | 说明 |
|----------|------|
| 避免在索引列上使用函数 | `WHERE YEAR(date) = 2024` → 改用范围查询 |
| 避免 `!=` / `NOT IN` | 会导致全表扫描 |
| LIKE 左侧不要有 % | `LIKE '%abc'` 不能用索引 |
| 避免 SELECT * | 只查需要的列，利用覆盖索引 |
| 不要过度索引 | 索引越多，写入越慢 |

### 5. EXPLAIN 查看执行计划 / EXPLAIN

```sql
EXPLAIN SELECT * FROM employees WHERE emp_name = '张三';
```

| 列 Column | 关注点 Focus |
|----------|------------|
| type | 访问类型：ALL（全表扫描）→ index → range → ref → const |
| key | 实际使用的索引 |
| rows | 预估扫描行数 |
| Extra | Using index（好）/ Using filesort（差）/ Using temporary（差）|

---

## 六、事务 / Transactions

### 1. ACID 特性 / ACID Properties

| 特性 | 含义 | 说明 |
|------|------|------|
| 原子性 Atomicity | 事务要么全部成功，要么全部失败 | 转账：扣款+加款必须同时成功 |
| 一致性 Consistency | 事务前后数据一致 | 金额总和不变 |
| 隔离性 Isolation | 并发事务互不干扰 | 两人同时转账不冲突 |
| 持久性 Durability | 提交后永久保存 | 断电也不丢 |

### 2. 事务操作 / Transaction Operations

```sql
START TRANSACTION;   -- 开启事务 Begin

UPDATE accounts SET balance = balance - 500 WHERE id = 1;  -- 扣款
UPDATE accounts SET balance = balance + 500 WHERE id = 2;  -- 加款

-- 检查无误后提交
COMMIT;              -- 提交 Commit

-- 出错回滚
-- ROLLBACK;         -- 回滚 Rollback
```

### 3. 隔离级别 / Isolation Levels

| 隔离级别 | 脏读 Dirty Read | 不可重复读 Non-repeatable | 幻读 Phantom |
|---------|:---:|:---:|:---:|
| READ UNCOMMITTED | ⚠️ | ⚠️ | ⚠️ |
| READ COMMITTED | ✅ | ⚠️ | ⚠️ |
| REPEATABLE READ | ✅ | ✅ | ⚠️ |
| SERIALIZABLE | ✅ | ✅ | ✅ |

> MySQL 默认 REPEATABLE READ；PostgreSQL/Oracle 默认 READ COMMITTED。
> MySQL defaults to REPEATABLE READ; PostgreSQL/Oracle default to READ COMMITTED.

**三种读问题 Three read problems：**
- **脏读 Dirty Read**：读到别的事务未提交的数据 / Reading uncommitted data
- **不可重复读 Non-repeatable Read**：同一查询两次结果不同 / Same query returns different results
- **幻读 Phantom Read**：同一查询两次行数不同 / Same query returns different number of rows
