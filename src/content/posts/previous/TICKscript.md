---
title: TICKscript Introducing
published: 2019-09-27
tags: [kapacitor]
---

# TICKscript 入门

## 介绍 TICKscript 语言

Kapacitor 使用名为**TICKscript**的领域特定语言（DSL）来定义**任务(tasks)**，这些任务涉及数据的提取，转换和加载，此外还涉及跟踪任意更改和检测数据中的事件。
一项常见任务是定义警报，TICKscript 使用`.tick`文件来定义用于处理数据的**管道(pipelines)**。TICKscript 语言旨在将 **节点(node)** 中定义的数据处理操作的调用链接在一起。

每个脚本都有一个固定范围，并且该范围中的每个变量都可以引用文字值(例如字符串，整数或浮点值)，或者 node 实例的可以被调用的方法。

node 有两种形式的方法

-   **属性方法(Property methods)** 修改 node 的内部属性，并且返回此节点的引用，用点`.`来表明这是属性方法
-   **链方法(Chaining methods)** 创建新的**子节点(child node)**，返回该子节点的引用，用管`|`来表明这是属性方法
<!--more-->

## 节点

在 TICKscript 中，基本类型是**节点(Node)**，节点有**属性(properties)**和**链方法(chaining methods)**。节点可以被**父节点(parent node)**或**兄弟节点(sibling node)**的链方法创建。
无论是父节点创建的节点还是兄弟节点创建的节点，每种类型的节点都有一样的**方法签名(method signature)**[1]，链方法可以接受 0 或多个参数来初始化节点。
常见的节点类型有`batch`，`query`，`stream`，`from`，`eval` 和 `alert`。

顶级节点`stream`和`batch`，它们用来声明将要定义的任务的处理类型。复杂属性的节点可以通过属性方法来创建。

节点**想要(wants)**的数据，是**流模式**或者**批模式**的数据，有些节点可以全部处理。节点**提供(provides)**的数据，是流数据或批数据，有些可以都提供。

因此，根据 wants/provides 模式，可以定义四种通用的节点用例。

-   **batch -> stream** 计算平均值、最小值、最大值
-   **batch -> batch** 在一批数据中识别异常值
-   **stream -> batch** 把相似数据组合在一起
-   **stream -> stream** 应用数学函数，比如对点值求对数

[1] **方法签名**: 是方法名称、参数的数量、类型和顺序

## 管道

每个 TICKscript 有一个或多个**管道(Pipeline)**。管道是节点逻辑上的链接，并且无法循环到较早的节点。管道可以分配给变量，然后可以将不同管道合并，例如，`join`或者`union`节点，部分管道也可以分解为功能单元。
管道的初始节点是 kapacitor 的任务处理类型，可以是`stream`或者`batch`，这两种类型的管道无法合并。

## stream 还是 batch

使用哪种取决于系统资源和计算的方式：
批处理适合长时间范围大量数据的计算，因为批处理将数据保留在硬盘上，所以查询时会导致数据库的高负载。
流处理适合短时间范围的计算，因为不必将大量放在内存上，所以查询负载很小。

| stream                                                       | batch                                |
| ------------------------------------------------------------ | ------------------------------------ |
| 逐数据点读取数据，kapacitor 向 influxDB 订阅他所感兴趣的内容 | 从数据库中读取一帧历史数据           |
| 在数据存储到 influxDB 前处理数据                             | 只能处理已经处于 influxDB 中的数据。 |
| 在短时间范围内计算用流处理                                   | 在长时间范围内计算用批处理           |

## 像图一样的管道

Kapacitor 中的管道是有向无环图，也就是说每条边都是数据流向的方向，并且管道中不能有环。边也可以被认为是父节点与子节点的数据流关系。
任何管道的开始出都将声明两个基本边之一，好处是确定了任务的处理类型，后面的节点在其自身及其子节点之间建立边类型。

-   **stream -> from()** 一次传输单个数据点的边
-   **batch -> query()** 一次传输一块数据的边

## 管道的有效性

当串起节点并创建 kapacitor 任务，kapacitor 将检查 TICKscript 语法结构是否正确，以及边是否匹配节点。但是，直到运行时才会验证管道是否起有效，错误信息会出现在 kapacitor 的日志中。
例一是由于管道缺少字段值而引发的运行时错误，因为没有设置`eval`节点的属性`keep()`。kapacitor 无法预测任务在运行时的数据形式，只会记录错误。

例一

```bash
[cpu_alert:alert4] 2017/10/24 14:42:59 E! error evaluating expression for level CRITICAL: left reference value "usage_idle" is missing value
[cpu_alert:alert4] 2017/10/24 14:42:59 E! error evaluating expression for level CRITICAL: left reference value "usage_idle" is missing value
```

## 示例

### stream -> from()管道

```javascript
var db = "telegraf";

var rp = "autogen";

var measurement = "cpu";

stream |
    from().database(db).retentionPolicy(rp).measurement(measurement) |
    httpOut("dump");
```

使用变量 db 表示数据库、rp 表示保留策略、measurement 表示数据库中的表，最后会把 cpu 数据以 json 形式缓存到 HTTP REST 端口。

这个例子有三个节点

-   基础节点`stream`
-   必要节点`from()`，定义数据流
-   处理节点`httpOut()`，将接收到的数据缓存到 Kapacitor 的 REST 服务。

包含两条边

-   **stream->from()** 设置任务和数据流的类型
-   **from()->httpOut()** 将数据流传递到 HTTP 输出端口

`from()`节点包含三个属性方法

-   database(db) - 设置数据库
-   retentionPolicy(rp) - 设置保留策略
-   measurement(measurement) - 设置测量指标

### batch -> query()管道

```javascript
batch
    |query('SELECT * FROM "telegraf"."autogen".cpu WHERE time > now() - 10s')
        .period(10s)
        .every(10s)
    |httpOut('dump')
```

将把最近 10 秒钟活动的批测量中的最后 cpu 数据点缓存到 HTTP REST 端点

这个例子有三个节点

-   基础节点`batch`
-   必要节点`query()`，定义数据流
-   处理节点`httpOut()`，将接收到的数据缓存到 Kapacitor 的 REST 服务。

包含两条边

-   **stream->query()** 设置任务和数据流的类型
-   **query()->httpOut()** 将数据流传递到 HTTP 输出端口

`query()`节点包含两个属性方法

-   period() - 设置覆盖的时间段。
-   every() - 设置处理的频率

## 更多

有关使用 TICKscript 的基本示例，请参见[GitHub](https://github.com/influxdata/kapacitor/tree/master/examples)代码库中的最新示例。
有关中级到高级用例的 TICKscript 解决方案，请参阅[《指南》](https://docs.influxdata.com/kapacitor/v1.5/guides/)文档。
