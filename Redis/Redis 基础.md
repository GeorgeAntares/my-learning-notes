# Redis 基础 / Redis Fundamentals

> 从零理解 Redis 数据结构、持久化、缓存模式、常见问题，覆盖面试和项目实战。
> Understand Redis data structures, persistence, caching patterns, common issues from scratch, covering interviews and practice.

---

## 一、Redis 是什么 / What is Redis

Redis（Remote Dictionary Server）是基于内存的键值对数据库，读写速度极快。

Redis is an in-memory key-value database with extremely fast read/write speed.

**特点 Features：**
- 基于内存 In-memory：数据存在内存中，速度快 / Data in memory, fast
- 单线程 Single-threaded：6.0 前单线程，避免锁竞争 / Single-threaded before 6.0, no lock contention
- 丰富数据结构 Rich data types：不止 String，还有 List、Set、Hash、ZSet 等
- 可持久化 Persistence：内存数据可存到磁盘 / Can persist to disk
- 支持过期 Expire：可设置 key 自动过期 / Keys can auto-expire

**为什么快 Why it's fast：**
1. 纯内存操作 Pure in-memory operations
2. 单线程避免上下文切换 Single thread avoids context switching
3. I/O 多路复用 I/O multiplexing（epoll）
4. 高效数据结构 Efficient data structures

---

## 二、基本操作 / Basic Operations

### 1. 连接 Redis / Connecting

```bash
# 连接本地 Redis
redis-cli

# 连接远程 Redis
redis-cli -h <host> -p <port> -a <password>

# 选择数据库（默认 0-15）
SELECT 1
```

### 2. 通用命令 / General Commands

| 命令 | 作用 | 示例 |
|------|------|------|
| SET | 设置键值 | `SET name "Redis"` |
| GET | 获取值 | `GET name` |
| DEL | 删除 | `DEL name` |
| EXISTS | 判断是否存在 | `EXISTS name` |
| EXPIRE | 设置过期时间（秒）| `EXPIRE name 60` |
| TTL | 查看剩余过期时间 | `TTL name` |
| PERSIST | 移除过期 | `PERSIST name` |
| KEYS | 查看所有键 | `KEYS *`（生产慎用）|
| TYPE | 查看类型 | `TYPE name` |
| FLUSHDB | 清空当前库 | — |
| FLUSHALL | 清空所有库 | — |

> **注意 Note**：`KEYS *` 会阻塞 Redis，生产环境用 `SCAN` 替代。
> `KEYS *` blocks Redis. Use `SCAN` in production.

---

## 三、数据结构 / Data Structures

### 1. String（字符串）

最基本类型，可存字符串、数字、序列化数据。

The most basic type, can store strings, numbers, serialized data.

```bash
# 基本操作
SET user:1 "张三"
GET user:1                      # → "张三"

# 数字操作
SET counter 100
INCR counter                    # → 101（原子递增）
DECR counter                    # → 100
INCRBY counter 10               # → 110

# 设置过期时间
SET token "abc123" EX 3600      # 3600秒后过期 / Expires in 3600s

# 不存在时设置（分布式锁基础）
SET lock "locked" NX EX 30      # 只在不存在时设置

# 批量操作
MSET k1 "v1" k2 "v2"
MGET k1 k2
```

**应用场景 Use cases：**
- 缓存对象 JSON / Cache JSON objects
- 计数器 Counter（点赞、浏览量）
- 分布式锁 Distributed lock
- Session 存储

### 2. Hash（哈希表）

类似 Map，适合存储对象。

Like a Map, suitable for storing objects.

```bash
# 设置字段
HSET user:1 name "张三" age 25 email "zhangsan@example.com"

# 获取单个字段
HGET user:1 name                # → "张三"

# 获取所有字段
HGETALL user:1
# name  → "张三"
# age   → "25"
# email → "zhangsan@example.com"

# 获取所有字段名/值
HKEYS user:1
HVALS user:1

# 删除字段
HDEL user:1 email

# 数字递增
HINCRBY user:1 age 1            # age → 26
```

**应用场景 Use cases：**
- 存储用户信息 / Store user info
- 购物车 / Shopping cart（用户ID → {商品ID: 数量}）

**String vs Hash 存对象 / String vs Hash for objects：**

| | String（存 JSON） | Hash |
|--|----------------|------|
| 读取整体 | 一次 GET | 需要 HGETALL |
| 修改单字段 | 读出→改→写回（3次操作） | HSET 一次搞定 |
| 内存占用 | 较少 | 略多 |
| 适合 | 整体读写频繁 | 字段级修改频繁 |

### 3. List（列表）

有序、可重复，支持从两端操作。

Ordered, allows duplicates, supports operations from both ends.

```bash
# 左/右推入
LPUSH queue "msg1" "msg2"       # → ["msg2", "msg1"]
RPUSH queue "msg3"              # → ["msg2", "msg1", "msg3"]

# 左/右弹出
LPOP queue                      # → "msg2"
RPOP queue                      # → "msg3"

# 查看范围
LRANGE queue 0 -1               # → ["msg1"]

# 长度
LLEN queue

# 阻塞弹出（消息队列）
BLPOP queue 30                  # 阻塞最多30秒 / Block up to 30s
```

**应用场景 Use cases：**
- 消息队列 Message queue
- 最新消息列表 Latest messages
- 操作日志 Operation log

### 4. Set（集合）

无序、不重复。

Unordered, no duplicates.

```bash
# 添加
SADD tags "java" "spring" "redis"

# 查看所有
SMEMBERS tags

# 判断是否存在
SISMEMBER tags "java"           # → 1

# 集合运算 Set operations
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"

SINTER set1 set2               # 交集 Intersection → {"b", "c"}
SUNION set1 set2               # 并集 Union → {"a", "b", "c", "d"}
SDIFF set1 set2                # 差集 Difference → {"a"}
```

**应用场景 Use cases：**
- 标签 Tags
- 共同好友 Mutual friends（交集）
- 去重 Deduplication

### 5. Sorted Set / ZSet（有序集合）

每个元素带一个分数（score），按分数排序。

Each element has a score, sorted by score.

```bash
# 添加（score + member）
ZADD leaderboard 100 "张三"
ZADD leaderboard 200 "李四"
ZADD leaderboard 150 "王五"

# 按分数升序排名（0 到 -1 表示全部）
ZRANGE leaderboard 0 -1 WITHSCORES
# 张三 → 100
# 王五 → 150
# 李四 → 200

# 降序排名
ZREVRANGE leaderboard 0 -1 WITHSCORES
# 李四 → 200
# 王五 → 150
# 张三 → 100

# 获取排名
ZRANK leaderboard "李四"        # → 2（升序第3名，0-indexed）
ZREVRANK leaderboard "李四"     # → 0（降序第1名）

# 分数范围查询
ZRANGEBYSCORE leaderboard 100 200

# 增加分数
ZINCRBY leaderboard 50 "张三"   # 张三 → 150
```

**应用场景 Use cases：**
- 排行榜 Leaderboard
- 延迟队列 Delay queue（score = 执行时间戳）
- 热搜排名 Hot search ranking

### 6. 数据结构总结 / Summary

| 类型 Type | 特点 | 典型场景 | 常用命令 |
|----------|------|---------|---------|
| String | 简单键值 | 缓存、计数器 | SET, GET, INCR |
| Hash | 字段-值 | 对象存储 | HSET, HGET, HGETALL |
| List | 有序可重复 | 消息队列 | LPUSH, RPOP, LRANGE |
| Set | 无序不重复 | 去重、交集 | SADD, SINTER, SMEMBERS |
| ZSet | 有序不重复 | 排行榜 | ZADD, ZRANGE, ZREVRANK |

---

## 四、过期策略 / Expiration Strategies

### 1. 设置过期 / Setting Expiration

```bash
# 设置时指定
SET token "abc" EX 3600         # 3600秒后过期

# 单独设置
EXPIRE key 60                   # 60秒
EXPIREAT key 1735689600         # Unix 时间戳

# 查看剩余时间
TTL key                         # -1=永不过期, -2=已过期/不存在
```

### 2. Redis 过期删除策略 / Expiration Deletion

| 策略 | 机制 | Redis 用法 |
|------|------|-----------|
| 定时删除 Timed | 每个key设定时器 | ❌ CPU消耗大 |
| 惰性删除 Lazy | 访问时检查是否过期 | ✅ 使用 |
| 定期删除 Periodic | 每隔一段时间随机抽查 | ✅ 使用 |

Redis 使用**惰性 + 定期**组合策略。

Redis uses **lazy + periodic** combination.

### 3. 内存淘汰策略 / Eviction Policies

当内存满了，Redis 如何选择删除哪个 key：

When memory is full, how Redis chooses which key to delete:

| 策略 | 含义 |
|------|------|
| noeviction | 不淘汰，写入报错（默认）|
| allkeys-lru | 所有key中淘汰最久未使用 |
| allkeys-lfu | 所有key中淘汰最少使用 |
| allkeys-random | 随机淘汰 |
| volatile-lru | 有过期时间的key中淘汰最久未使用 |
| volatile-lfu | 有过期时间的key中淘汰最少使用 |
| volatile-random | 有过期时间的key中随机淘汰 |
| volatile-ttl | 有过期时间的key中淘汰最快过期的 |

---

## 五、持久化 / Persistence

### 1. RDB（快照）

| 特性 | 说明 |
|------|------|
| 方式 | 定时把内存数据快照写入磁盘 / Periodic snapshot to disk |
| 优点 | 文件小，恢复快 / Small file, fast recovery |
| 缺点 | 可能丢失最后一次快照后的数据 / May lose data after last snapshot |
| 适用 | 容忍少量数据丢失的场景 / Tolerant of minor data loss |

### 2. AOF（追加日志）

| 特性 | 说明 |
|------|------|
| 方式 | 记录每条写命令到日志 / Log every write command |
| 优点 | 数据安全，最多丢1秒 / Safe, max 1s loss |
| 缺点 | 文件大，恢复慢 / Large file, slow recovery |
| 适用 | 数据安全要求高 / High data safety requirement |

### 3. RDB + AOF 混合

Redis 4.0+ 支持，兼顾恢复速度和数据安全。

Supported since Redis 4.0+, balancing recovery speed and data safety.

---

## 六、缓存模式 / Caching Patterns

### 1. Cache-Aside（旁路缓存）— 最常用

```
应用 Application
    │
    ├── 1. 先查缓存 Check cache → 命中 hit → 返回 return
    │
    └── 2. 未命中 miss → 查数据库 query DB → 写回缓存 write cache → 返回 return
```

```java
// 伪代码 Pseudocode
String value = redis.get(key);
if (value == null) {
    value = db.query(key);         // 查数据库 Query DB
    if (value != null) {
        redis.set(key, value, 300); // 写回缓存，设5分钟过期 Write cache
    }
}
return value;
```

### 2. Read/Write-Through

应用只与缓存交互，缓存负责同步到数据库。

App only interacts with cache; cache syncs to DB.

### 3. Write-Behind

异步写入数据库，性能最高但有数据丢失风险。

Asynchronous write to DB, highest performance but data loss risk.

---

## 七、缓存常见问题 / Common Cache Problems

### 1. 缓存穿透 Cache Penetration

**问题 Problem**：查询不存在的数据，缓存和数据库都没有，每次都查数据库。
Querying non-existent data, neither cache nor DB has it, every request hits DB.

**解决 Solution：**
```java
// 方案1：缓存空值 Cache null value
if (value == null) {
    value = db.query(key);
    if (value == null) {
        redis.set(key, "", 60);   // 缓存空值，短过期
        return null;
    }
    redis.set(key, value, 300);
}

// 方案2：布隆过滤器 Bloom filter（判断数据是否可能存在）
```

### 2. 缓存击穿 Cache Breakdown

**问题 Problem**：热点 key 过期瞬间，大量请求同时查数据库。
Hot key expires, massive requests hit DB simultaneously.

**解决 Solution：**
```java
// 方案1：加互斥锁 Mutex lock
if (redis.get(key) == null) {
    if (tryLock(key)) {            // 获取锁
        try {
            value = db.query(key);
            redis.set(key, value, 300);
        } finally {
            releaseLock(key);
        }
    } else {
        Thread.sleep(50);          // 等一下重试 Retry
        return get(key);
    }
}

// 方案2：热点 key 永不过期，后台异步更新
```

### 3. 缓存雪崩 Cache Avalanche

**问题 Problem**：大量 key 同时过期，或 Redis 宕机，所有请求打到数据库。
Massive keys expire simultaneously, or Redis crashes, all requests hit DB.

**解决 Solution：**
- 过期时间加随机值：`expire = 300 + random(60)` 避免同时过期
- 搭建 Redis 集群，避免单点故障
- 限流降级，保护数据库

### 4. 三者对比 / Comparison

| | 穿透 Penetration | 击穿 Breakdown | 雪崩 Avalanche |
|--|----------------|---------------|---------------|
| 原因 | 查不存在的数据 | 热点key过期 | 大量key同时过期/Redis宕机 |
| 量级 | 少量请求 | 大量请求 | 超大量请求 |
| 解决 | 缓存空值/布隆过滤器 | 互斥锁/永不过期 | 随机过期/集群/限流 |

---

## 八、Redis 在项目中的实际应用 / Redis in Practice

### 1. Spring Boot 集成 / Spring Boot Integration

```java
@Service
public class ConfigService {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    // 读取配置 Read config
    public String getConfig(String key) {
        // 1. 先查缓存 Check cache first
        String value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            return value;
        }

        // 2. 缓存未命中，查数据库 Cache miss, query DB
        value = configMapper.selectValueByKey(key);
        if (value != null) {
            // 3. 写回缓存 Write to cache, 5 min TTL
            redisTemplate.opsForValue().set(key, value, 5, TimeUnit.MINUTES);
        }
        return value;
    }

    // 更新配置时刷新缓存 Refresh cache on update
    public void updateConfig(String key, String value) {
        configMapper.updateValueByKey(key, value);
        redisTemplate.opsForValue().set(key, value, 5, TimeUnit.MINUTES);
    }

    // 删除配置时清除缓存 Delete cache on remove
    public void deleteConfig(String key) {
        configMapper.deleteByKey(key);
        redisTemplate.delete(key);
    }
}
```

### 2. RuoYi 框架中的 Redis / Redis in RuoYi

RuoYi 使用 Redis 缓存系统配置：

RuoYi uses Redis to cache system configs:

```java
// 写入配置时同步刷新 Redis
// Refresh Redis when config is written
@CachePut   // 或手动 redisTemplate ops
public void insertConfig(SysConfig config) {
    configMapper.insert(config);
    redisCache.setCacheObject(
        "sys_config:" + config.getConfigKey(),
        config.getConfigValue()
    );
}

// 读取配置时优先从 Redis 获取
// Read config from Redis first
public String selectConfigByKey(String configKey) {
    String value = redisCache.getCacheObject("sys_config:" + configKey);
    if (value != null) return value;

    // Redis 没有，查数据库 If not in Redis, query DB
    SysConfig config = configMapper.selectConfigByKey(configKey);
    if (config != null) {
        redisCache.setCacheObject("sys_config:" + configKey, config.getConfigValue());
        return config.getConfigValue();
    }
    return null;
}
```

### 3. 常见应用场景总结 / Common Use Cases

| 场景 Scenario | 数据结构 | 过期策略 |
|--------------|---------|---------|
| 系统配置缓存 | String | 5-30 分钟 |
| 用户 Session | String/Hash | 30 分钟 |
| 登录验证码 | String | 5 分钟 |
| 排行榜 | ZSet | 不过期 |
| 购物车 | Hash | 不过期或7天 |
| 分布式锁 | String + NX | 30 秒 |
| 点赞计数 | String + INCR | 不过期 |
| 消息队列 | List | 不过期 |

---

## 九、面试高频问题 / Common Interview Questions

### Q1: Redis 为什么快 / Why is Redis fast?

1. 纯内存操作 / Pure in-memory
2. 单线程避免锁竞争和上下文切换 / Single thread avoids locks and context switching
3. I/O 多路复用（epoll）/ I/O multiplexing
4. 高效数据结构 / Efficient data structures

### Q2: Redis 是单线程的吗 / Is Redis single-threaded?

- Redis 6.0 前：核心命令处理是单线程 / Core command processing is single-threaded
- Redis 6.0 后：引入多线程处理网络 I/O，命令执行仍单线程 / Multi-threaded network I/O, still single-threaded command execution

### Q3: 如何实现分布式锁 / How to implement distributed lock?

```bash
# 加锁 Lock（原子操作 Atomic）
SET lock:order:123 "holder_id" NX EX 30

# 解锁 Unlock（Lua 脚本保证原子性 Lua script for atomicity）
# 先判断是不是自己的锁，再删除
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

### Q4: Redis 和 MySQL 如何保证数据一致 / How to ensure Redis-MySQL consistency?

| 方案 | 操作 | 缺点 |
|------|------|------|
| 先更新DB，再删缓存 | DB.update → Cache.delete | 删除可能失败 |
| 先删缓存，再更新DB | Cache.delete → DB.update | 并发时可能脏数据 |
| 延迟双删 | 删缓存→更新DB→延迟→再删缓存 | 复杂 |

最常用：**先更新 DB，再删缓存**，配合消息队列重试。

Most common: **Update DB first, then delete cache**, with MQ retry.
