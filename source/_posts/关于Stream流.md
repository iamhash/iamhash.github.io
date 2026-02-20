---
title: 关于Stream流
date: 2025-08-27 21:09:20
tags: [技术碎片]
---

## 背景
Stream 是 Java 8 引入的新特性，主要作用是用一种声明式、函数式的方式来处理集合、数组或 I/O 数据。

Stream 是 Java 8 之后最重要的特性之一，它让 Java 代码风格更现代、更简洁。Java 8同样引入 Lambda 表达式，Stream 与 Lambda 搭配使用，标志着 Java 从命令式编程向 函数式编程 迈进。
用一句话来总结也就是`集合注重数据存储，Stream 注重数据处理。`

<!-- more -->

## 核心思想

Stream流的思想：相当于流水线

使用步骤一般为：

1. 先得到一条**Stream流**（流水线），并把数据放上去。
2. 使用**中间方法**对流水线上的数据进行操作。
3. 使用**终结方法**对流水线上的数据进行操作。

### 如何获取Stream流

- 单列集合：直接使用Collections中的默认方法：.stream();
- 双列集合：一般不能直接使用。可用keySet()、entrySet()方法转换为单列以后再操作。
- 数组：使用Arrays工具类的静态方法：Arrays.stream(数组名)
- 一堆零散数据：使用Stream接口中的静态方法： Stream.of(T...values)

> 注意：基本数据类型的数组不能用Stream.of()，否则会把整个数组当作一个元素放到Stream中。因此参数数组只能是引用数据类型。

```
举例：list.stream()

Stream.of(1,2,3)

Arrays.stream(arr)
```

### stream的中间方法

**一些常用中间方法：**

**filter(参数为断定型接口)**：过滤

**limit(long maxSize)**：获取前面几个元素

**skip(long n)**：跳过前面几个元素

**distinct()**：元素去重（依赖hashcode和equals方法）底层中利用hashset去重

**concat(Stream a,Stream b)：合并a和b两个流为一个流**

**map(参数为函数型接口)**：转换流中的数据类型

```
举例：filter() → 过滤

map() → 映射

distinct() → 去重

sorted() → 排序

limit()/skip() → 截取
```

### stream的终结方法

forEach(参数为消费型接口)：遍历

count():统计

toArray():收集流中数据，放到数组中

collect(Collector collector):收集流中数据，放到集合中

> 收集方法collect：
>
> 1.收集到list(Set同理）：.collect(Collectors.toList());
>
> 2.收集到map中：
>
> 如果要收集到map中，键不能重复

```
举例：forEach() → 遍历

collect() → 收集结果，比如转成 List

reduce() → 规约（累积计算）

count() / max() / min()
```

## stream特点

**1.声明式编程**：不需要写 for 循环，类似 SQL 操作集合。

```
List<String> list = Arrays.asList("a", "b", "c", "a");
long count = list.stream().filter(s -> s.equals("a")).count();
```

相当于 “查询 list 中等于 a 的数量”。

**2.链式操作：**Stream 支持一连串操作（filter → map → reduce）。

**3.惰性求值：**中间操作不会立刻执行，只有遇到 终止操作 时才会真正计算。

**4.并行流：**通过 .parallelStream() 可以轻松实现并行计算，利用多核 CPU 提升性能。

## 总结

它在**集合处理**、**并行计算**等场景下几乎是首选工具。

它的地位类似于：在集合框架中的 SQL 查询语言。

Stream流是 Java 编程中的 现代风格代表。
