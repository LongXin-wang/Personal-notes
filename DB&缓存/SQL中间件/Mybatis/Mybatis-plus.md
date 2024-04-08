- [Wrappers](#wrappers)
- [条件构造器](#条件构造器)
  - [功能详解](#功能详解)
    - [allEq](#alleq)
    - [eq](#eq)
    - [ne](#ne)
    - [inSql / notInSql](#insql--notinsql)
    - [orderby](#orderby)
    - [func](#func)
    - [nested](#nested)
    - [apply](#apply)
    - [lambda](#lambda)
    - [其它](#其它)
- [流式查询](#流式查询)

# Wrappers

MyBatis-Plus 提供了 Wrappers 类，它是一个静态工厂类，用于快速创建 QueryWrapper、UpdateWrapper、LambdaQueryWrapper 和 LambdaUpdateWrapper 的实例。使用 Wrappers 可以减少代码量，提高开发效率。

```Java
// 创建 QueryWrapper
QueryWrapper<User> queryWrapper = Wrappers.query();
queryWrapper.eq("name", "张三");

// 创建 LambdaQueryWrapper
LambdaQueryWrapper<User> lambdaQueryWrapper = Wrappers.lambdaQuery();
lambdaQueryWrapper.eq(User::getName, "张三");

// 创建 UpdateWrapper
UpdateWrapper<User> updateWrapper = Wrappers.update();
updateWrapper.set("name", "李四");

// 创建 LambdaUpdateWrapper
LambdaUpdateWrapper<User> lambdaUpdateWrapper = Wrappers.lambdaUpdate();
lambdaUpdateWrapper.set(User::getName, "李四");
```

# 条件构造器

在 MyBatis-Plus 中，Wrapper 类是构建查询和更新条件的核心工具。以下是主要的 Wrapper 类及其功能：

AbstractWrapper：这是一个抽象基类，提供了所有 Wrapper 类共有的方法和属性。它定义了条件构造的基本逻辑，包括字段（column）、值（value）、操作符（condition）等。所有的 QueryWrapper、UpdateWrapper、LambdaQueryWrapper 和 LambdaUpdateWrapper 都继承自 AbstractWrapper。

QueryWrapper：专门用于构造查询条件，支持基本的等于、不等于、大于、小于等各种常见操作。它允许你以链式调用的方式添加多个查询条件，并且可以组合使用 and 和 or 逻辑。

UpdateWrapper：用于构造更新条件，可以在更新数据时指定条件。与 QueryWrapper 类似，它也支持链式调用和逻辑组合。使用 UpdateWrapper 可以在不创建实体对象的情况下，直接设置更新字段和条件。

LambdaQueryWrapper：这是一个基于 Lambda 表达式的查询条件构造器，它通过 Lambda 表达式来引用实体类的属性，从而避免了硬编码字段名。这种方式提高了代码的可读性和可维护性，尤其是在字段名可能发生变化的情况下。

LambdaUpdateWrapper：类似于 LambdaQueryWrapper，LambdaUpdateWrapper 是基于 Lambda 表达式的更新条件构造器。它允许你使用 Lambda 表达式来指定更新字段和条件，同样避免了硬编码字段名的问题。

## 功能详解

Wrapper 方法通常接受一个 boolean 类型的参数，用于决定是否将该条件加入到最终的 SQL 中

```Java
queryWrapper.like(StringUtils.isNotBlank(name), Entity::getName, name)
            .eq(age != null && age >= 0, Entity::getAge, age);
```

### allEq

- params：一个 Map，其中 key 是数据库字段名，value 是对应的字段值。
- null2IsNull：如果设置为 true，当 Map 中的 value 为 null 时，会调用 isNull 方法；如果设置为 false，则会忽略 value 为 null 的键值对。
- filter：一个 BiPredicate，用于过滤哪些字段应该被包含在查询条件中。
condition：一个布尔值，用于控制是否应用这些条件。

```Java
// 设置所有字段的相等条件，如果字段值为null，则根据null2IsNull参数决定是否设置为IS NULL
allEq(Map<String, Object> params)
allEq(Map<String, Object> params, boolean null2IsNull)
allEq(boolean condition, Map<String, Object> params, boolean null2IsNull)

// 设置所有字段的相等条件，通过filter过滤器决定哪些字段应该被包含，如果字段值为null，则根据null2IsNull参数决定是否设置为IS NULL
allEq(BiPredicate<String, Object> filter, Map<String, Object> params)
allEq(BiPredicate<String, Object> filter, Map<String, Object> params, boolean null2IsNull)
allEq(boolean condition, BiPredicate<String, Object> filter, Map<String, Object> params, boolean null2IsNull)
```

```Java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.allEq(Map.of("id", 1, "name", "老王", "age", null));

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.allEq(Map.of("id", 1, "name", "老王", "age", null));

// 字段或者字段名包含a的才加入
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.allEq((field, value) -> field.contains("a"), Map.of("id", 1, "name", "老王", "age", null));

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.allEq((field, value) -> field.contains("a"), Map.of("id", 1, "name", "老王", "age", null));

-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE id = 1 AND name = '老王' AND age IS NULL

-- 带过滤器的普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE name = '老王' AND age IS NULL
```

### eq

- column：数据库字段名或使用 Lambda 表达式的字段名。
- val：与字段名对应的值。
- condition：一个布尔值，用于控制是否应用这个相等条件。

```java
// 设置指定字段的相等条件
eq(R column, Object val)

// 根据条件设置指定字段的相等条件
eq(boolean condition, R column, Object val)
```

```Java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("name", "老王");

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.eq(User::getName, "老王");

-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE name = '老王'
```

### ne

```java
// 设置指定字段的不相等条件
ne(R column, Object val)

// 根据条件设置指定字段的不相等条件
ne(boolean condition, R column, Object val)
```

```Java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.ne("name", "老王");

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.ne(User::getName, "老王");


-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE name <> '老王'
```

### inSql / notInSql

```Java

// 设置指定字段的 IN 条件，使用 SQL 语句   sqlValue：一个字符串，包含用于生成 IN 子句中值集合的 SQL 语句。
inSql(R column, String sqlValue)
inSql(boolean condition, R column, String sqlValue)
```

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.inSql("age", "1,2,3,4,5,6");

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.inSql(User::getAge, "1,2,3,4,5,6");

-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE age IN (1, 2, 3, 4, 5, 6)

QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.inSql("id", "select id from other_table where id < 3");

SELECT * FROM user WHERE id IN (select id from other_table where id < 3)
```

### orderby

```Java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.orderBy(true, true, "id", "name");

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.orderBy(true, true, User::getId, User::getName);

-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user ORDER BY id ASC, name ASC
```

### func

```Java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.func(i -> {
    if (true) {
        i.eq("id", 1);
    } else {
        i.ne("id", 1);
    }
});

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.func(i -> {
    if (true) {
        i.eq(User::getId, 1);
    } else {
        i.ne(User::getId, 1);
    }
});

-- 根据条件生成的 SQL 会有所不同
-- 如果条件为 true，则生成的 SQL 为：
SELECT * FROM user WHERE id = 1

-- 如果条件为 false，则生成的 SQL 为：
SELECT * FROM user WHERE id != 1
```

### nested

nested 方法是 MyBatis-Plus 中用于构建查询条件的高级方法之一，它用于创建一个独立的查询条件块，不带默认的 AND 或 OR 逻辑。通过调用 nested 方法，可以在查询条件中添加一个嵌套的子句，该子句可以包含多个查询条件，并且可以被外部查询条件通过 AND 或 OR 连接。

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.nested(i -> i.eq("name", "李白").ne("status", "活着"));

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.nested(i -> i.eq(User::getName, "李白").ne(User::getStatus, "活着"));

-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE (name = '李白' AND status <> '活着')
```

### apply

apply 方法是 MyBatis-Plus 中用于构建查询条件的高级方法之一，它允许你直接拼接 SQL 片段到查询条件中。这个方法特别适用于需要使用数据库函数或其他复杂 SQL 构造的场景。

```java
// 拼接 SQL 片段
apply(String applySql, Object... params)    //params：一个可变参数列表，包含 SQL 片段中占位符的替换值。
apply(boolean condition, String applySql, Object... params)
```

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.apply("id = 1");

LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.apply("date_format(dateColumn, '%Y-%m-%d') = '2008-08-08'");

QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.apply("date_format(dateColumn, '%Y-%m-%d') = {0}", "2008-08-08");

-- 普通 Wrapper 生成的 SQL
SELECT * FROM user WHERE id = 1

-- Lambda Wrapper 生成的 SQL
SELECT * FROM user WHERE date_format(dateColumn, '%Y-%m-%d') = '2008-08-08'

-- 使用参数占位符生成的 SQL
SELECT * FROM user WHERE date_format(dateColumn, '%Y-%m-%d') = '2008-08-08'
```

### lambda

从 QueryWrapper 获取 LambdaQueryWrapper：

从 UpdateWrapper 获取 LambdaUpdateWrapper：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
LambdaQueryWrapper<User> lambdaQueryWrapper = queryWrapper.lambda();
// 使用 Lambda 表达式构建查询条件
lambdaQueryWrapper.eq(User::getName, "张三");

UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
LambdaUpdateWrapper<User> lambdaUpdateWrapper = updateWrapper.lambda();
// 使用 Lambda 表达式构建更新条件
lambdaUpdateWrapper.set(User::getName, "李四");
```

### 其它

- gt 大于
- ge 大于等于
- lt 小于
- le 小于等于
- between 在某个范围内
- notBetween 不在某个范围内
- like 模糊查询
- notLike //NOT LIKE '%王%'
- likeLeft 左模糊查询  //name LIKE '%王'
- likeRight 右模糊查询 //name LIKE '王%'
- isNull 是否为 null
- in //id IN (1, 2, 3)
- notIn //id NOT IN (1, 2, 3)
- eqSql     // SELECT * FROM user WHERE id = (select MAX(id) from table)
- gtSql     // SELECT * FROM user WHERE id > (select MAX(id) from table)
- geSql     // SELECT * FROM user WHERE id >= (select MAX(id) from table)
- ltSql     // SELECT * FROM user WHERE id < (select MAX(id) from table)
- leSql     // SELECT * FROM user WHERE id <= (select MAX(id) from table)
- groupBy 分组 //SELECT id, COUNT(*) FROM user GROUP BY id
- orderbyAsc 升序 //lambdaQueryWrapper.orderByAsc(User::getId, User::getName);
- orderbyDesc 降序 //SELECT * FROM user ORDER BY id DESC
- having //lambdaQueryWrapper.groupBy(User::getAge).having("sum(age) > {0}", 10);
- or //lambdaQueryWrapper.eq(User::getId, 1).or().eq(User::getName, "老王");
- and //lambdaQueryWrapper.eq(User::getId, 1).and().eq(User::getName, "老王");
- last //lambdaQueryWrapper.orderByDesc(User::getId).last("limit 1");
- select //lambdaQueryWrapper.select(User::getId, User::getName);
- set //lambdaUpdateWrapper.set(User::getName, "老王");
- setSql //lambdaUpdateWrapper.setSql("name = '老王'");

# 流式查询

通过 ResultHandler 接口实现结果集的流式查询。这种查询方式适用于数据跑批或处理大数据的业务场景。

在 BaseMapper 中，新增了多个重载方法，包括 selectList, selectByMap, selectBatchIds, selectMaps, selectObjs，这些方法可以与流式查询结合使用

