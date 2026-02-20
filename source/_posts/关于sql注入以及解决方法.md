---
title: 关于SQL注入以及解决方法
date: 2025-08-24 20:00:56
tags: [技术碎片]
---

## 什么是SQL注入

SQL 注入是最常见的 Web 安全漏洞之一。

当攻击者通过输入框（比如登录表单、搜索框）中注入恶意的 SQL 语句，从而直接在后台拼接到数据库查询中，导致执行了开发者预期之外的 SQL。

<!-- more -->

举例：有一个登录验证的 SQL：

```
SELECT * FROM users WHERE username = 'admin' AND password = '123456';
```

如果代码直接拼接字符串：

```
sql = "SELECT * FROM users WHERE username = '" + userInput + "' AND password = '" + pwdInput + "'";
```

如果攻击者在用户名中输入`' OR '1'='1`，而密码随便写。就会变为：

```
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = 'xxx';
```

导致绕过验证，直接登录成功。

## SQL常见注入方式

1. 基于字符串拼接的注入（最常见）。

2. 布尔盲注：根据返回结果是否不同判断条件真假。

3. 时间盲注：利用 sleep() 或延迟函数判断真假。

4. 报错注入：利用数据库报错信息提取数据。

5. 联合查询注入：利用 UNION SELECT 拼接结果获取其他表数据。

## 解决方法

- 使用预编译语句（PreparedStatement）/参数化查询：

这是解决该问题最有效的方案，不拼接字符串，而是用参数绑定。

举例：

```java
String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, username);
ps.setString(2, password);
ResultSet rs = ps.executeQuery();

```

这样即使输入了 OR '1'='1 也会被当作普通字符串处理，不会执行注入。

- 使用ORM框架：MyBatis、Hibernate

ORM 框架一般都会内置参数化机制，避免拼接 SQL。

MyBatis是基于JDBC的持久层框架，本质上还是用JDBC来执行SQL。因此这里先讲JDBC的措施：

仍然是使用预编译：依旧是上面代码的例子，这里的 ? 是占位符，JDBC 会先把 **SQL 结构预编译**，再把**用户输入**作为 **参数绑定**，不会当作 SQL 语句拼接，所以能**有效防止注入**。

而MyBatis中：

如果使用 ${} 就容易字符串拼接：

```
<select id="getUser" resultType="User">
    SELECT * FROM users WHERE username = '${username}'
</select>
```

此时若传入 admin' OR '1'='1，就会拼接到 SQL 里，存在注入风险。

而采用 #{ } 来进行参数占位，就是安全的：

```
<select id="getUser" resultType="User">
    SELECT * FROM users WHERE username = #{username}
</select>
```

这里的 #{} 底层会用 PreparedStatement 的 ? 占位符处理，相当于参数化查询，能防止注入。

而这里就会引出：

## ${}和#{}的区别：

### #{} —— 预编译参数占位符

会被替换为 ?，底层用 PreparedStatement 来执行。此时SQL结构会被预编译，而用户输入会作为参数绑定，而不是直接拼接到SQL字符串中。【有效防止注入】

### ${} —— 字符串直接拼接

直接把参数值拼接到 SQL 里。类似于 JDBC 的 Statement，不会做预编译。【容易造成 SQL 注入】

### 实际应用场景

#{}：绝大多数场景都应该用 #{}，比如 **条件查询、插入、更新**。

${}：只有在 动态拼接 SQL 关键字时才用，比如**表名、列名**不能用 ? 占位，只能拼接：

```
<select id="getData" resultType="Map">
    SELECT ${column} FROM ${table} WHERE id = #{id}
</select>
```

但这种写法要小心，必须配合 白名单校验，确保 column/table 只能是你允许的范围，否则会被注入。

**总结**：

- #{} 会被替换为 ?，底层用 PreparedStatement，参数会被安全绑定，可以防止 SQL 注入，一般查询和更新都用它。
- ${} 是字符串拼接，类似于 Statement，会直接把参数拼进去，有 SQL 注入风险，通常只在动态拼接表名、列名等场景下用，而且需要白名单过滤。
