---
title: InfluxDB Concepts
published: 2019-12-11
tags: [influxdb]
---

# influxdb 概念

## InfluxDB 关键概念

### 例子

**measurement**:census

| time                 | butterflies | honeybees | location | scientist  |
| -------------------- | ----------- | --------- | -------- | ---------- |
| 2015-08-18T00:00:00Z | 12          | 23        | 1        | langstroth |
| 2015-08-18T00:00:00Z | 1           | 30        | 1        | perpetua   |
| 2015-08-18T00:06:00Z | 11          | 28        | 1        | langstroth |
| 2015-08-18T00:06:00Z | 3           | 28        | 1        | perpetua   |
| 2015-08-18T05:54:00Z | 2           | 11        | 2        | langstroth |
| 2015-08-18T06:00:00Z | 1           | 10        | 2        | langstroth |
| 2015-08-18T06:06:00Z | 8           | 23        | 2        | perpetua   |
| 2015-08-18T06:12:00Z | 7           | 22        | 2        | perpetua   |

数据是从 2015 年 8 月 18 日 00:00 到 2015 年 8 月 18 日 6:12，在两个位置（位置 1 和位置 2），两位科学家（langstroth 和 perpetua）计数的蝴蝶和蜜蜂的数量。

<!--more-->

### 解释

#### Field

本节将通过以上数据介绍 influxdb 的含义。在上方的数据中，influxDB 中的所有数据都有一列称为时间。**time**存储时间戳。接下来的两列 **butterflies** 和 **honeybees** ，是字段。字段由**key-value**组成，键**butterflies**对应值 **12 ~ 7**, 键 **honeybees**对应值**23 ~ 22**。

在上面的数据中，key-value 对的集合构成一个字段集。这是样本数据中的所有八个字段集：

-   butterflies = 12 honeybees = 23
-   butterflies = 1 honeybees = 30
-   butterflies = 11 honeybees = 28
-   butterflies = 3 honeybees = 28
-   butterflies = 2 honeybees = 11
-   butterflies = 1 honeybees = 10
-   butterflies = 8 honeybees = 23
-   butterflies = 7 honeybees = 22

**Field values**是 InfluxDB 数据结构必不可少的部分，如果没有字段，则 InfluxDB 中没有数据。注意，字段未使用索引，使用字段值查询必须扫描所有值。这些查询相对于`tag`查询的性能不高。通常字段不应作为常用查询的关键词。

#### tag

数据的最后两列 **location** 和 **scientist**是`tag`。`tag`由**tag-key**和**tag-value**组成,都存储为字符串。样本数据中的 tag-key 是**location**和**scientist**。标签 location 两个值：1 和 2。标签 scientist 有两个标签值：langstroth 和 perpetua。

在上面的数据中，标签集是所有标签 key-value 的不同组合。样本数据中的四个标记集是：

-   location = 1, scientist = langstroth
-   location = 2, scientist = langstroth
-   location = 1, scientist = perpetua
-   location = 2, scientist = perpetua

**tags**是可选的。无需在数据结构中包含 tag，但最好使用它们，因为标签被加了索引。这意味着对使用 tag 查询速度更快，并且 tag 非常适合存储常见查询的数据。

> 假设大多数查询都使用字段键 butterflies 和 honeybees：
> SELECT _ FROM "census" WHERE "butterflies" = 1
> SELECT _ FROM "census" WHERE "honeybees" = 23
>
> 由于未对该字段进行索引，因此 InfluxDB 在提供响应之前会先扫描每个值，这种行为可能会让查询效率很低。为了优化查询，重新安排架构，用 butterflies 和 honeybees 作为标签，scientist 和 location 作为字段，再使用 butterflies 和 honeybees 查询，不会扫描每个字段，查询就更快。

#### measurement

`measurement`是包含**tags**、**fields**和**time**列的容器，也是对存储的关联字段数据的描述。measurement 名称是字符串，在概念上都类似于表。

#### retention policy

`retention policy`描述 InfluxDB 保留数据多长时间(DURATION)，单个**measurement**可以属于不同的保留策略，以及该数据在集群中存储多少副本(REPLICATION)。在样本数据中，**census**中的所有内容都是**autogen**保留策略。InfluxDB 自动创建该保留策略；它具有无限的保留时间，并且复制因子设置为 1。

#### series

在 InfluxDB 中，`series`是 **retention policy**, **measurement**, 和 **tag** 的集合。上面的数据包括四个 series：

| Arbitrary series number | Retention policy | Measurement | Tag set                             |
| ----------------------- | ---------------- | ----------- | ----------------------------------- |
| series 1                | autogen          | census      | location = 1,scientist = langstroth |
| series 2                | autogen          | census      | location = 2,scientist = langstroth |
| series 3                | autogen          | census      | location = 1,scientist = perpetua   |
| series 4                | autogen          | census      | location = 2,scientist = perpetua   |

#### point

`point`是由四个部分组成的单个数据记录：**measurement**, **tag set**, **field set**, 和**timestamp**。**point**由**series**和**time**组成唯一标识。

例如，这里有个`point`:

```bash
name: census
-----------------
time                    butterflies honeybees   location    scientist
2015-08-18T00:00:00Z    1           30          1           perpetua
```

[ref](https://docs.influxdata.com/influxdb/v1.7/concepts/key_concepts/#sample-data)
