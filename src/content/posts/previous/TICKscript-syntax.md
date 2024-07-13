---
title:  TICKscript syntax
published:  2019-09-29
tags: [kapacitor]
---

# 概念
入门部分介绍了节点和管道的关键概念，节点代表流程调用单位，将数据进行批处理或逐点流处理，然后更改数据、存储数据或基于数据更改触发警报，管道是节点的逻辑组织链。
在Kapacitor中，TICKscript用于定义任务和模板任务，这些模板任务可作为生成新任务的模板。

TICKscript语法的借鉴了许多不同的语言，最多的是GO。

使用TICKscript时，将遇到几个语法子空间
* **InfluxQL** - 创建查询节点时，需要一个表示InfluxQL语句的字符串
* **Lambda expressions** - 其他节点和方法使用Lambda表达式

如入门中所述，管道是有向无环图，它包含有限数量的节点和边。每个边从一个节点指向另一个节点，通常从数据源节点开始，到数据集节点，再将其结果传递到数据处理节点。
<!--more-->

# TICKscript语法
TICKscript区分大小写，并使用Unicode。TICKscript解析器从上到下，从左到右扫描TICKscript代码，实例化变量和节点，然后在遇到它们时将它们链接成为管道。
加载TICKscript时，解析器检查在节点上调用的链接方法是否有效。如果遇到无效的链接方法，解析器将抛出错误消息**no method or property <identifier> on <node type>**。

## 代码表示
kong'he源文件使用UTF-8进行编码，脚本分为声明和表达式。只有一行的声明会创建变量；可以有多行的表达式会创建整个管道、管道链或管道分支。
* 空白 空格用来分隔变量名、运算符、变量，缩进用来表明方法调用的层次结构。
* 注释 通过在文本前使用一对正斜杠`//`在一行上创建

## 关键字
关键字是在语言中具有特殊含义的标记，因此不能用作函数或变量的标识符。

|字	| 说明 | 
|---|---|
|TRUE	| "true"| 
|FALSE	| "false"| 
|AND	| and | 
|OR	    | or | 
|lambda:| 后面的标志将被解释为lambda表达式。| 
|var	| 变量声明| 
|dbrp	| 数据库声明| 

## 操作符

标准操作符

| 操作符	| 说明	| 用例| 
| --- |  --- | ---| 
| +	| 加法或字符串连接 | 3 + 6，total + count ， 'foo' + 'bar'
| -	| 减法 | 10 - 1, total - errs
| *	| 乘法 | 	3 * 6，ratio * 100.0
| /	| 除法 | 	36 / 4, errs / total
| == | 等于 | 	1 == 1, date == today
| != | 不等 | 	result != 0, id != "testbed"
| <	| 小于| 	4 < 5, timestamp < today
| < =| 小于等于 | 	3 <= 6, flow <= mean
| >	| 大于| 	6 > 3.0, delta > sigma
| > = | 大于等于 | 9.0 >= 8.1, quantity >= threshold
| =~| 正则匹配 | tag =~ /^cz\d+/
| !~| 正则不匹配 | tag !~ /^sn\d+/
| !	| 逻辑不 | 	!TRUE, !(cpu_idle > 70)
| AND | 逻辑与	| rate < 20.0 AND rate >= 10
| OR | 	逻辑或	| status > warn OR delta > sigma

链操作符

| 操作符	| 说明	| 用例| 
| :--- |  :--- | :---| 
| &verbar; | 声明一个链方法调用，该调用将创建一个新节点的实例并链接到上方的节点 | stream  &verbar; from() |
| `.` | 声明属性方法调用，设置所属的节点或更改内部属性 | from()  .database(mydb) |
| @	  | 声明用户定义的函数(UDF)调用。本质上是一种将新的UDF节点添加到管道的链接方法 | from()    @MyFunc()|

## 变量
使用关键字`var`声明变量，变量是不可变的，也就是不能重新赋予新值，但是可以在其他声明中使用。
变量也在模板任务中充当占位符，以便在用模板创建新任务时填充。
命名变量时，必须以字母开头，跟任意数量的字母，数字和下划线。
变量的保存类型取决于为其分配的文字值

### 任务的变量声明
```javascript
var my_var = 'foo'
var MY_VAR = 'BAR'
var my_float = 2.71
var my_int = 1
var my_node = stream
```
### 模板任务的变量声明
```javascript
var measurement string
var frame duration
var warn = float
var period = 12h
var critical = 3.0
```

## 文字值
TICKscript识别五个类型标识符，这些标识符可用于模板任务中。否则，文字的类型将从其声明中进行解释。

类型标识符

| 标识 | 用例 | 
| :--- | :--- |
| string |	在模板任务中，将变量声明为`string` |
| duration | 在模板任务中，将变量声明为`duration` |
| int |	在模板任务中，将变量声明为字`int64` |
| float | 在模板任务中，将变量声明为`float64`。 |
| lambda | 在模板任务中，将变量声明为Lambda表达式类型 |

### 布尔值

布尔值是使用布尔关键字`TRUE`和`FALSE`生成的
```javascript
var true_bool = TRUE
...
   |flatten()
       .on('host','port')
       .dropOriginalFieldName(FALSE)
...
```

### 数字值

整数解释为`int64`，小数解释为`float64`，以`0`开头的整数解释为八进制。
```javascript
var my_int = 6
var my_float = 2.71828
var my_octal = 0400
```

### 持续时间值

持续时间由两部分组成：整数和持续时间单位。
```javascript
var span = 10s
var frequency = 10s
...
var views = batch
    |query('SELECT sum(value) FROM "pages"."default".views')
        .period(1h)
        .every(1h)
        .groupBy(time(1m), *)
        .fill(0)
```

### 时间单位

| 单位 | 说明 |
| :--- | :--- |
| u or µ | 微秒（百万分之一秒） |
| ms | 毫秒（千分之一秒） |
| s	 | 秒 |
| m | 分钟 |
| h	 | 小时 |
| d | 天 |
| w | 周 |

### 字符串

字符串以一个或三个单引号包起来：`'` 、`'''`，可以使用加法`+`来连接字符串。单引号中使用单引号要转义，三引号不用转义。这里面使用双引号都不用转义。
为了使长而复杂的字符串更易读，在字符串中允许使用换行符。
```javascript
var region1 = 'EMEA'
var old_standby = 'foo' + 'bar'
var query1 = 'SELECT 100 - mean(usage_idle) AS stat FROM "telegraf"."autogen"."cpu" WHERE cpu = \'cpu-total\' '
var query2 = '''SELECT 100 - mean(usage_idle) AS stat FROM "telegraf"."autogen"."cpu" WHERE cpu = 'cpu-total' '''

batch
   |query('''SELECT 100 - mean(usage_idle) AS stat FROM "telegraf"."autogen"."cpu" WHERE cpu = 'cpu-total' ''')

// 多行字符串
batch
   |query('SELECT 100 - mean(usage_idle)
           AS stat
           FROM "telegraf"."autogen"."cpu"
           WHERE cpu = \'cpu-total\'
           ')
```

### 字符串模板
字符串模板允许将节点属性，标签和字段添加到字符串中，需要将其包装在双花括号内。
```javascript
|alert()
  .id('{{ index .Tags "host"}}/mem_used')
  .message('{{ .ID }}:{{ index .Fields "stat" }}')
```
字符串模板还可以包括流程语句，例如`if ... else`以及对内部格式化方法的调用。
```javascript
.message('{{ .ID }} is {{ if eq .Level "OK" }}alive{{ else }}dead{{ end }}: {{ index .Fields "emitted" | printf "%0.3f" }} points/10s.')
```

### 字符串列表
字符串列表是在两个方括号之间声明的字符串的集合，可以使用文字，其他变量的标识符或星号通配符`*`来声明它们。
```javascript
var foo = 'foo'
var bar = 'bar'
var foobar_list = [foo, bar]
var cpu_groups = [ 'host', 'cpu' ]
...
stream
   |from()
      .measurement('cpu')
      .groupBy(cpu_groups)
...
```
模板中字符串列表的应用-`groups`
```javascript
dbrp "telegaf"."not_autogen"

var measurement string
var where_filter = lambda: TRUE
var groups = [*]
var field string
var warn lambda
var crit lambda
var window = 5m
var slack_channel = '#alerts'

stream
    |from()
        .measurement(measurement)
        .where(where_filter)
        .groupBy(groups)
    |window()
        .period(window)
        .every(window)
    |mean(field)
    |alert()
         .warn(warn)
         .crit(crit)
         .slack()
         .channel(slack_channel)
```

### 正则表达式
正则表达式以正斜杠开始和结束：`/`，语法和其他语言一样。
```javascript
var cz_turbines = /^cz\d+/
var adr_senegal = /\.sn$/
var local_ips = /^192\.168\..*/
...
var locals = stream
   |from()
      .measurement('responses')
      .where(lambda: "node" =~ local_ips )

var south_afr = stream
   |from()
      .measurement('responses')
      .where(lambda: "dns_node" =~ /\.za$/ )
```
Lambda表达式作为文字，`lambda`开头，后跟冒号`:`。
```javascript
var my_lambda = lambda: 1 > 0
var lazy_lambda = lambda: "usage_idle" < 95
...
var data = stream
  |from()
...
var alert = data
  |eval(lambda: sigma("stat"))
    .as('sigma')
    .keep()
  |alert()
    .id('{{ index .Tags "host"}}/cpu_used')
    .message('{{ .ID }}:{{ index .Fields "stat" }}')
    .info(lambda: "stat" > 70 OR "sigma" > 2.5)
    .warn(lambda: "stat" > 80 OR "sigma" > 3.0)
    .crit(lambda: "stat" > 90 OR "sigma" > 3.5)
```

### 节点
```javascript
var data = stream
  |from()
    .database('telegraf')
    .retentionPolicy('autogen')
    .measurement('cpu')
    .groupBy('host')
    .where(lambda: "cpu" == 'cpu-total')
  |eval(lambda: 100.0 - "usage_idle")
    .as('used')
  |window()
    .period(span)
    .every(frequency)
  |mean('used')
    .as('stat')
...
var alert = data
  |eval(lambda: sigma("stat"))
    .as('sigma')
    .keep()
  |alert()
    .id('{{ index .Tags "host"}}/cpu_used')
...
```

## 使用标签，字段和变量
在任何脚本中，仅仅声明变量是不够的。它们保存的值也必须能被访问。在TICKscript中，还必须使用从InfluxDB中提取的标签和字段中保存的值。
```javascript
   var db = 'website'                       // 访问变量 声明字符串使用单引号
   ...
   var data = stream
    |from()
        .database(db)
   ...
    var data = stream
    |from()
       .database('telegraf')
       .retentionPolicy('autogen')
       .measurement('cpu')
       .groupBy('host')                      // 方法调用中: 访问tag和field，使用单引号。
       .where(lambda: "cpu" == 'cpu-total') // Lambda表达式中: 访问tag和field，使用双引号
    |eval(lambda: 100.0 - "usage_idle")
       .as('used')                          
    ...
```
命名lambda表达式结果，使用`as()`命名Lambda表达式结果并将其作为字段添加到数据集。`as()`就像InfluxQL中的`AS`关键字一样。
像数据标签和字段一样，lambda表达式结果可以在Lambda表达式中用双引号访问，也可以在方法中用单引号访问。
```javascript
  ...
      |window()
        .period(period)
        .every(every)
      |mean('used')
        .as('stat')

    // Thresholds
    var alert = data
      |eval(lambda: sigma("stat"))
        .as('sigma')
        .keep()
      |alert()
        .id('{{ index .Tags "host"}}/cpu_used')
        .message('{{ .ID }}:{{ index .Fields "stat" }}')
        .info(lambda: "stat" > info OR "sigma" > infoSig)
        .warn(lambda: "stat" > warn OR "sigma" > warnSig)
        .crit(lambda: "stat" > crit OR "sigma" > critSig)
```
使用InfluxQL节点进行现场访问，链方法`mean()`是节点类型InfluxQL的别名
```javascript
var data = stream
 |from()
   .database('telegraf')
   .retentionPolicy('autogen')
   .measurement('cpu')
   .groupBy('host')
   .where(lambda: "cpu" == 'cpu-total')
 |eval(lambda: 100.0 - "usage_idle")
   .as('used')
 |window()
   .period(period)
   .every(every)
 |mean('used')
   .as('stat')
```

### 访问字符串模板中的值
```javascript
|alert()
  .id('{{ index .Tags "host"}}/mem_used') 
  .message('{{ .ID }}:{{ index .Fields "stat" }}')
```
### 类型转换
```javascript
   |eval(lambda: float("total_error_responses")/float("total_responses") * 100.0)
```
### 数值精度
```javascript
|alert()
  .id('{{ index .Tags "host"}}/mem_used')
  .message('{{ .ID }}:{{ index .Fields "stat" | printf "%0.2f" }}')
  
  stream
   // usage_idle值四舍五入到百分之一的百分比，然后用于警报节点的阈值方法中的比较。然后将其写入警报消息。
   |from()
      .measurement('cpu')
   |eval(lambda: floor("usage_idle" * 1000.0)/1000.0)
      .as('thousandths')
      .keep('usage_user','usage_idle','thousandths')
   |alert()
      .crit(lambda: "thousandths" <  95.000)
      .message('{{ index .Fields "thousandths" }}')
         // Whenever we get an alert write it to a file.
      .log('/tmp/alerts.log')
```
### 时间精度
```javascript
stream
    |from()
        .database('telegraf')
        .measurement('cpu')
        .groupBy(*)
    |window()
        .period(5m)
        .every(5m)
        .align()
    |mean('usage_idle')
        .as('usage_idle')
    |influxDBOut()
       .database('telegraf')
       .retentionPolicy('autogen')
       .measurement('mean_cpu_idle')
       .precision('s')
...
```

## 声明
TICKscript中有两种类型的语句：声明和表达式。声明可以声明TICKscript可以使用的变量或数据库。表达式表示方法调用的管道，这些方法创建处理节点并设置其属性。

### 声明
TICKscript使用两种类型的声明：数据库声明和变量声明。

数据库声明以关键字`dbrp`开头，后跟两个以句点分隔的字符串。第一个字符串声明将使用脚本的默认数据库。第二个字符串声明其保留策略。使用时，数据库声明语句应为TICKscript的第一个声明。
```javascript
dbrp "telegraf"."autogen"
```
变量声明以var关键字开头，后跟要声明的变量的标识符。赋值运算符后跟一个文字右侧值，该值将设置新变量的类型和值。
```javascript
var db = 'website'
var rp = 'autogen'
var measurement = 'responses'
var whereFilter = lambda: ("lb" == '17.99.99.71')
var name = 'test rule'
var idVar = name + ':{{.Group}}'
var data = stream
    |from()
        .database(db)
        .retentionPolicy(rp)
```
### 表达式
表达式以节点标识符或包含另一个表达式的变量标识符开头，然后它将链接方法，属性方法或用户定义的函数链接在一起。管道运算符`|`表示链接方法调用的开始，
将新节点返回到链中。点运算符`.`添加属性设置器。@运算符`@`引入了用户定义的功能。
```javascript
// Dataframe
var data = batch
  |query('''SELECT mean(used_percent) AS stat FROM "telegraf"."autogen"."mem" ''')
    .period(period)
    .every(every)
    .groupBy('host')

// Thresholds
var alert = data
  |eval(lambda: sigma("stat"))
    .as('sigma')
    .keep()
  |alert()
    .id('{{ index .Tags "host"}}/mem_used')
    .message('{{ .ID }}:{{ index .Fields "stat" }}')
    .info(lambda: "stat" > info OR "sigma" > infoSig)
    .warn(lambda: "stat" > warn OR "sigma" > warnSig)
    .crit(lambda: "stat" > crit OR "sigma" > critSig)

// Alert
alert
  .log('/tmp/mem_alert_log.txt')
```

### 创建节点
除`batch`和`stream`外，节点始终出现在管道表达式中，它们是通过链接方法创建的。通常使用节点类型名称标识链接方法，一个明显的例外是InfluxQL节点，它使用别名。

对于每种节点类型，创建该类型实例的方法使用相同的签名。因此如果一个`query`节点穿件一个`eval`节点并把它加入链，并且如果一个`from`节点穿件一个`eval`节点并把它加入链。
创建新的eval节点的链接方法将接受相同的参数，不管是哪个节点创建的。
```javascript
var data = stream
  |from()
    .database('telegraf')
    .retentionPolicy('autogen')
    .measurement('cpu')
    .groupBy('host')
    .where(lambda: "cpu" == 'cpu-total')
  |eval(lambda: 100.0 - "usage_idle")
    .as('used')
    .keep()
    
var data = batch
  |query('''SELECT 100 - mean(usage_idle) AS stat FROM "telegraf"."autogen"."cpu" WHERE cpu = 'cpu-total' ''')
    .period(period)
    .every(every)
    .groupBy('host')
  |eval(lambda: sigma("stat"))
    .as('sigma')
    .keep()
```
### 管道
重申一下，管道是由一个或多个表达式定义的节点的逻辑顺序链，管道可以以两个模式定义节点之一开始：批处理或流。
批处理管道的数据帧是在查询定义节点中定义的。用于流管道的数据流是在`from`定义节点中定义的。在定义节点之后，可以跟随任何其他类型的节点。
通过用管道`|`字符将节点添加到管道。可以使用`@`字符将用户定义的函数添加到管道中。
```javascript
// Dataframe
var data = batch
  |query('''SELECT 100 - mean(usage_idle) AS stat FROM "telegraf"."autogen"."cpu" WHERE cpu = 'cpu-total' ''')
    .period(period)
    .every(every)
    .groupBy('host')

// Thresholds
var alert = data
  |eval(lambda: sigma("stat"))
    .as('sigma')
    .keep()
  |alert()
    .id('{{ index .Tags "host"}}/cpu_used')
    .message('{{ .ID }}:{{ index .Fields "stat" }}')
    .info(lambda: "stat" > info OR "sigma" > infoSig)
    .warn(lambda: "stat" > warn OR "sigma" > warnSig)
    .crit(lambda: "stat" > crit OR "sigma" > critSig)

// Alert
alert
  .log('/tmp/cpu_alert_log.txt')
```
# 节点类型分类
## 特殊节点
这些节点是特殊的，因为可以使用其类型名称以外的标识符来创建和返回它们。可以使用代表其功能方面的别名。这可能适用于所有实例（如InfluxQL节点），或仅适用于一种实例（如Alert节点）。
* `alert` 可以作为`deadman`开关返回
* `influxQL` 直接在InfluxQL中调用函数，因此当调用使用InfluxQL方法名称的TICKScript链接方法时可以返回该函数。
    * `from()|mean()` 在from节点中定义的数据流上调用均值函数，并返回InfluxQL节点。
    * `query()|mode()` 在查询节点中定义的数据帧上调用模式函数，并返回一个InfluxQL节点。
## 数据源定义节点
TICKscript管道中的第一个节点是`batch`或`stream`。它们定义了用于处理数据的数据源。
* `batch` 声明中未使用链接方法调用语法。
* `stream` 声明中未使用链接方法调用语法。
## 数据定义节点
模式定义节点之后通常是节点，其目的是定义要由其他节点处理的数据的帧或数据流。
* `from` 有一个空的链接方法。只能跟随`stream`。使用属性方法进行配置。
* `query` 链接方法采用查询字符串。只能跟随`batch`
## 数据操作节点
可以使用操作节点来更改或生成数据集中的值。
* `default` 有一个空的链接方法。它的`field`和`tag`属性可用于设置数据系列中字段和标签的默认值。
* `sample` 接收int64或duration字符串。它根据计数或时间段提取数据样本。
* `shift` 接收一个持续时间字符串。它移动数据点时间戳。持续时间字符串可以以减号开头，以将标记及时向后移动。
* `where` 接收一个lambda节点。它可以与流管道一起使用，例如InfluxQL中的WHERE语句。
* `window` 有一个空的链接方法。使用属性方法进行配置。它通常在`stream`节点的管道中的`from`之后，以在移动时间范围内缓存数据。
## 处理节点
一旦定义了数据集，就可以将其传递到其他节点，这些节点将对其进行处理，对其进行转换或将基于其中的更改触发其他处理。
* 用于更改数据结构或将管道混合在一起的节点
    * `combine` 接收一个或多个lambda表达式的列表。它可以将来自单个节点的数据与其自身进行合并。
    * `eval` 接收一个或多个lambda表达式的列表。它在接收到的每个数据点上评估表达式，并使用其as属性，使结果可用于管道中的后续节点。
    * `groupBy` 接收一个或多个字符串的列表，这些字符串表示该系列的标签。它按标签对传入数据进行分组。
    * `join` 接收一个或多个引用管道表达式的变量的列表。它根据匹配的时间戳将来自任意数量管道的数据连接起来。
    * `union` 接收一个或多个引用管道表达式的变量的列表。它创建任意数量的管道的并集。
* 用于转换或处理数据集中的数据点的节点
    * `delete` 空链接方法。它依靠属性(`field`，`tag`)从数据点删除字段和标签。
    * `derivative` 接收一个字符串，该字符串表示要为其计算导数的字段。
    * `flatten` 空链接方法。它依靠属性在特定的维度上展开一组点。
    * `influxQL` 特殊节点，它提供对InfluxQL功能的访问。不能直接创建。
    * `stateCount` 链接方法采用lambda表达式。它计算处于给定状态的连续点的数量。
    * `stateDuration` 链接方法采用lambda表达式。它计算给定状态持续的时间。
    * `status` 链接方法采用持续时间表达式。它以给定的时间间隔发出有关另一个节点的内部统计信息。
* 触发事件，流程的节点
    * `alert` 空链接方法。它依靠许多属性来配置警报的发出。
    * `deadman` 一个辅助函数，它是警报的别名，当数据流低于指定的阈值时会触发该警报。
    * `httpOut` 链接方法需要一个字符串。它缓存接收到的每个组的最新数据，并使用字符串参数作为最终定位器上下文在Kapicator http服务器上将其提供。
    * `httpPost` 链接方法采用字符串数组。也可以是空的。它将数据发布到字符串数组中指定的HTTP端点。
    * `influxDBOut` 空链接方法–通过属性设置器配置。它在接收到数据时将其写入InfluxDB。
    * `k8sAutoscale` 空链接方法。它依赖于许多配置属性。它会在Kubernetes™ 资源上触发自动缩放。
    * `log` 空链接方法。它依赖于级别和前缀属性进行配置。它记录通过它的所有数据。
## 用户自定义节点
用户定义的功能是实现由用户程序或脚本定义的功能的节点，这些程序作为单独的进程运行，并通过套接字或标准系统数据流与Kapacitor进行通信

# InfluxQL in TICKscript
TICKscript中，InfluxQL主要应用在`query`节点，其链接方法采用InfluxQL查询字符串，几乎总是SELECT语句。
InfluxQL与SQL非常相似，在`query`节点一般只有三点是必须的：`SELECT`、`FROM`、`WHERE`
```sql
SELECT {<FIELD_KEY> | <TAG_KEY> | <FUNCTION>([<FIELD_KEY>|<TAG_KEY])} FROM <DATABASE>.<RETENTION_POLICY>.<MEASUREMENT> WHERE {<CONDITIONAL_EXPRESSION>}
```
一般查询语句
```javascript
batch
    |query('SELECT cpu, usage_idle FROM "telegraf"."autogen".cpu WHERE time > now() - 10s')
        .period(10s)
        .every(10s)
    |httpOut('dump')
```
使用变量的InfluxQL
```javascript
var my_field = 'usage_idle'
var my_tag = 'cpu'

batch
    |query('SELECT ' + my_tag + ', ' + my_field + ' FROM "telegraf"."autogen".cpu WHERE time > now() - 10s')
        .period(10s)
        .every(10s)
    |httpOut('dump')
```
使用函数调用的InfluxQL
```javascript
var data = batch
  |query('''SELECT 100 - mean(usage_idle) AS stat FROM "telegraf"."autogen"."cpu" WHERE cpu = 'cpu-total' ''')
    .period(period)
    .every(every)
    .groupBy('host')
```
