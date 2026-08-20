# Java 进阶语法 / Java Advanced Syntax

## 一、泛型 / Generics

### 什么是泛型 / What are Generics

泛型允许在编译时指定类型参数，避免类型转换和类型安全问题。

Generics allow specifying type parameters at compile time, avoiding casts and type safety issues.

```java
// 不用泛型（需要强制转换 / Without generics - needs cast）
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);  // 有类型安全风险 / Type safety risk

// 使用泛型 / With generics
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0);  // 不需要转换 / No cast needed
```

### 泛型方法 / Generic Methods

```java
// <T> 是类型参数声明，T 是类型变量
// <T> declares type parameter, T is the type variable
public <T> T getFirst(List<T> list) {
    if (list.isEmpty()) return null;
    return list.get(0);
}

// 使用 / Usage
List<String> names = List.of("Alice", "Bob");
String first = getFirst(names);  // T 推断为 String / T inferred as String

List<Integer> nums = List.of(1, 2, 3);
Integer firstNum = getFirst(nums);  // T 推断为 Integer / T inferred as Integer
```

### 泛型类 / Generic Classes

```java
public class Result<T> {
    private int code;
    private String msg;
    private T data;  // T 可以是任意类型 / T can be any type

    public Result(int code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    public T getData() { return data; }
}

// 使用 / Usage
Result<String> r1 = new Result<>(200, "success", "hello");
Result<List<Notice>> r2 = new Result<>(200, "success", noticeList);
```

### 通配符 / Wildcards

| 写法 Syntax | 含义 Meaning | 场景 Use Case |
|------|------|------|
| `<?>` | 任意类型 Any type | 只读不写 Read-only |
| `<? extends T>` | T 或 T 的子类 T or subclass | 读取 Read |
| `<? super T>` | T 或 T 的父类 T or superclass | 写入 Write |

```java
// 上界通配符 — 只能读 / Upper bound - read only
public double sum(List<? extends Number> nums) {
    double total = 0;
    for (Number n : nums) total += n.doubleValue();
    return total;
}

// 可以传入 List<Integer>, List<Double> 等
// Can accept List<Integer>, List<Double>, etc.
```

### 项目中的泛型示例 / Generics in Practice

```java
// RuoYi 的 AjaxResult 就是泛型 Map 的子类
// RuoYi's AjaxResult extends HashMap (implicitly generic)

// Service 接口也常用泛型 / Service interfaces often use generics
public interface BaseService<T> {
    List<T> selectList(T query);
    T selectById(Long id);
    int insert(T entity);
    int update(T entity);
    int deleteByIds(Long[] ids);
}
```

## 二、Stream API

### 什么是 Stream / What is Stream

Stream 是 Java 8 引入的集合操作 API，用声明式方式处理数据。

Stream is a collection processing API introduced in Java 8, processing data declaratively.

```java
// 传统写法 / Traditional
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.length() > 3) {
        result.add(s.toUpperCase());
    }
}

// Stream 写法 / Stream style
List<String> result = list.stream()
    .filter(s -> s.length() > 3)      // 过滤 / Filter
    .map(String::toUpperCase)          // 转换 / Map
    .collect(Collectors.toList());     // 收集 / Collect
```

### 常用操作 / Common Operations

#### 过滤 filter

```java
// 只保留长度大于3的字符串 / Keep strings longer than 3
list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());
```

#### 转换 map

```java
// 把字符串列表转为长度列表 / Convert strings to their lengths
list.stream()
    .map(String::length)           // 方法引用 / Method reference
    .collect(Collectors.toList());

// 等价写法 / Equivalent
list.stream()
    .map(s -> s.length())
    .collect(Collectors.toList());
```

#### 排序 sorted

```java
// 按长度排序 / Sort by length
list.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());

// 倒序 / Reverse order
list.stream()
    .sorted(Comparator.comparing(String::length).reversed())
    .collect(Collectors.toList());
```

#### 去重 distinct

```java
list.stream()
    .distinct()
    .collect(Collectors.toList());
```

#### 匹配 anyMatch / allMatch / noneMatch

```java
// 是否有任意元素匹配 / Any element matches
boolean hasLong = list.stream().anyMatch(s -> s.length() > 10);

// 是否全部匹配 / All elements match
boolean allLong = list.stream().allMatch(s -> s.length() > 3);

// 是否没有匹配 / No element matches
boolean noneLong = list.stream().noneMatch(s -> s.length() > 100);
```

#### 归约 reduce

```java
// 求和 / Sum
int sum = nums.stream().reduce(0, Integer::sum);

// 拼接字符串 / Concatenate
String joined = list.stream().reduce("", (a, b) -> a + b);
```

#### 收集 toMap

```java
// 把 List 转为 Map / Convert List to Map
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(
        s -> s,           // key
        String::length,   // value
        (a, b) -> a       // 冲突时保留前者 / Keep first on conflict
    ));
```

### 项目中的 Stream 示例 / Stream in Practice

```java
// 把配置列表转为 key-value Map / Convert config list to key-value map
Map<String, String> result = configs.stream()
    .filter(c -> !"Y".equals(c.getConfigType()))  // 过滤内置 / Filter built-in
    .collect(Collectors.toMap(
        SysConfig::getConfigKey,
        c -> Optional.ofNullable(c.getConfigValue()).orElse(""),
        (a, b) -> a
    ));
```

## 三、Optional

### 什么是 Optional / What is Optional

Optional 是一个容器对象，可能包含非空值也可能为空。用来避免 NullPointerException。

Optional is a container that may or may not contain a non-null value. Used to avoid NullPointerException.

```java
// 不用 Optional（可能空指针 / Without Optional - risk of NPE）
String value = config.getConfigValue();
int length = value.length();  // 如果 value 是 null，这里崩溃 / Crashes if null

// 使用 Optional / With Optional
int length = Optional.ofNullable(config.getConfigValue())
    .map(String::length)       // 如果非空，取长度 / If not null, get length
    .orElse(0);                // 如果为空，返回 0 / If null, return 0
```

### 创建 Optional / Creating Optional

```java
Optional<String> opt1 = Optional.of("hello");        // 不能为 null / Cannot be null
Optional<String> opt2 = Optional.ofNullable(maybeNull); // 可以为 null / Can be null
Optional<String> opt3 = Optional.empty();            // 空的 / Empty
```

### 常用方法 / Common Methods

| 方法 Method | 说明 Description |
|------|------|
| `isPresent()` | 是否有值 / Has value |
| `ifPresent(consumer)` | 有值则执行 / Execute if has value |
| `map(function)` | 转换值 / Transform value |
| `filter(predicate)` | 过滤值 / Filter value |
| `orElse(default)` | 为空时返回默认值 / Return default if empty |
| `orElseGet(supplier)` | 为空时返回 supplier 的值 / Return supplier result if empty |
| `orElseThrow(exception)` | 为空时抛异常 / Throw exception if empty |

### 使用场景 / Use Cases

```java
// 场景1：安全取值 / Scenario 1: Safe access
String value = Optional.ofNullable(map.get("key"))
    .orElse("default");

// 场景2：链式调用 / Scenario 2: Chained calls
int length = Optional.ofNullable(config)
    .map(SysConfig::getConfigValue)
    .map(String::length)
    .orElse(0);

// 场景3：条件执行 / Scenario 3: Conditional execution
Optional.ofNullable(notice)
    .filter(n -> "0".equals(n.getStatus()))
    .ifPresent(n -> System.out.println("已发布 / Published: " + n.getNoticeTitle()));
```

### 项目中的 Optional 示例 / Optional in Practice

```java
// ClientConfigController 中的用法 / Usage in ClientConfigController
result.put(
    c.getConfigKey(),
    Optional.ofNullable(c.getConfigValue()).orElse("")
);
// 如果 configValue 为 null，用空字符串代替 / Replace null with empty string
```

## 四、方法引用 / Method References

方法引用是 Lambda 的简写形式。

Method references are shorthand for lambdas.

| 类型 Type | Lambda | 方法引用 Method Reference |
|------|------|------|
| 静态方法 Static method | `s -> String.valueOf(s)` | `String::valueOf` |
| 实例方法 Instance method | `s -> s.length()` | `String::length` |
| 对象方法 Object method | `c -> c.getConfigKey()` | `SysConfig::getConfigKey` |
| 构造方法 Constructor | `() -> new ArrayList()` | `ArrayList::new` |

## 五、总结 / Summary

| 特性 Feature | 引入版本 Since | 核心价值 Core Value |
|------|------|------|
| 泛型 Generics | Java 5 | 类型安全，避免强制转换 / Type safety, avoid casts |
| Stream | Java 8 | 声明式集合操作 / Declarative collection processing |
| Optional | Java 8 | 避免 NullPointerException / Avoid NPE |
| record | Java 16 | 简洁的数据载体 / Concise data carriers |
| 方法引用 Method Reference | Java 8 | Lambda 简写 / Lambda shorthand |
