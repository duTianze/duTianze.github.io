---
title: TICKscript BatchNode
published: 2019-10-08
tags: [kapacitor]
---

# 批节点

`batch`节点处理子查询节点的创建，每次`query`调用都会创建一个子批处理节点，可以进一步对其进行配置。批处理任务中的`batch`变量是`BatchNode`的一个实例。

> 使用`BatchNode`时需要`QueryNode`。它定义了批处理数据的来源和时间表，并且应在任何其他链接方法之前使用。

构造器

| 链方法 | 描述       |
| :----- | :--------- |
| batch  | 无构造方法 |

属性方法

| setter  | 描述                               |
| :------ | :--------------------------------- |
| quiet() | 禁止来自此节点的所有错误记录事件。 |

<!--more-->

## 链接方法

### Deadman

帮助程序功能，用于在吞吐量较低时创建警报。

```javascript
    var data = batch
        |query()...
    // 如果吞吐量每10秒下降到100点以下并每10秒检查一次，则触发严重警报。
    data
        |deadman(100.0, 10s)
    //Do normal processing of data
    data...
```

加上 lambda 表达式

```javascript
    var data = batch
        |query()...
    // 如果吞吐量每10秒下降到100点以下并每10秒检查一次，则触发严重警报。
    // 仅在一天中的上午8点至下午5点之间触发警报。
    data
        |deadman(100.0, 10s, lambda: hour("time") >= 8 AND hour("time") <= 17)
    //Do normal processing of data
    data...
```

Returns: `AlertNode`

### Query

要执行的查询，不能在 WHERE 子句中包含时间条件，也不能包含`GROUP BY`子句。时间条件根据时间段，偏移量和时间表动态添加。
`GROUP BY`子句是根据传递给 groupBy 方法的尺寸动态添加的。

```javascript
batch
    |query(q string)
```

```javascript
batch|deadman(threshold float64, interval time.Duration, expr ...ast.LambdaNode)
```

Returns: `QueryNode`

### Stats

创建一个包含节点内部统计信息的新数据流。间隔表示基于实时发出统计信息的频率。这意味着间隔时间与源节点正在接收的数据点的时间无关。

```javascript
batch
    |stats(interval time.Duration)
```

Returns: `StatsNode`
