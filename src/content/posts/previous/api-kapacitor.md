---
title: kapacitor api
published: 2019-09-26
tags: [kapacitor]
---

## 定义任务

### 创建

`id` : `dutianze-t`

```http request
POST /kapacitor/v1/tasks
{
    "id" : "dutianze-t",
    "type" : "stream",
    "dbrps" : [{"db": "api", "rp" : "autogen"}],
    "script" : "var db = 'api'\n\nvar rp = 'autogen'\n\nvar measurement = 'account_login'\n\nvar groupBy = ['third_type']\n\nvar whereFilter = lambda: TRUE\n\nvar name = 'dutianze-t'\n\nvar idVar = name + '-{{.Group}}'\n\nvar message = ''\n\nvar idTag = 'alertID'\n\nvar levelTag = 'level'\n\nvar messageField = 'message'\n\nvar durationField = 'duration'\n\nvar outputDB = 'chronograf'\n\nvar outputRP = 'autogen'\n\nvar outputMeasurement = 'alerts'\n\nvar triggerType = 'relative'\n\nvar shift = 10m\n\nvar crit = 10\n\nvar data = stream\n    |from()\n        .database(db)\n        .retentionPolicy(rp)\n        .measurement(measurement)\n        .groupBy(groupBy)\n        .where(whereFilter)\n    |eval(lambda: \"value\")\n        .as('value')\n\nvar past = data\n    |shift(shift)\n\nvar current = data\n\nvar trigger = past\n    |join(current)\n        .as('past', 'current')\n    |eval(lambda: abs(float(\"current.value\" - \"past.value\")) / float(\"past.value\") * 100.0)\n        .keep()\n        .as('value')\n    |alert()\n        .crit(lambda: \"value\" > crit)\n        .stateChangesOnly()\n        .message(message)\n        .id(idVar)\n        .idTag(idTag)\n        .levelTag(levelTag)\n        .messageField(messageField)\n        .durationField(durationField)\n\ntrigger\n    |eval(lambda: float(\"value\"))\n        .as('value')\n        .keep()\n    |influxDBOut()\n        .create()\n        .database(outputDB)\n        .retentionPolicy(outputRP)\n        .measurement(outputMeasurement)\n        .tag('alertName', name)\n        .tag('triggerType', triggerType)\n\ntrigger\n    |httpOut('output')\n",
    "status" : "enabled"
}
```

```http response
{
    "link": {
        "rel": "self",
        "href": "/kapacitor/v1/tasks/dutianze-t"
    },
    "id": "dutianze-t",
    "template-id": "",
    "type": "stream",
    "dbrps": [
        {
            "db": "api",
            "rp": "autogen"
        }
    ],
    "script": "var db = 'api'\n\nvar rp = 'autogen'\n\nvar measurement = 'account_login'\n\nvar groupBy = ['third_type']\n\nvar whereFilter = lambda: TRUE\n\nvar name = 'dutianze-t'\n\nvar idVar = name + '-{{.Group}}'\n\nvar message = ''\n\nvar idTag = 'alertID'\n\nvar levelTag = 'level'\n\nvar messageField = 'message'\n\nvar durationField = 'duration'\n\nvar outputDB = 'chronograf'\n\nvar outputRP = 'autogen'\n\nvar outputMeasurement = 'alerts'\n\nvar triggerType = 'relative'\n\nvar shift = 10m\n\nvar crit = 10\n\nvar data = stream\n    |from()\n        .database(db)\n        .retentionPolicy(rp)\n        .measurement(measurement)\n        .groupBy(groupBy)\n        .where(whereFilter)\n    |eval(lambda: \"value\")\n        .as('value')\n\nvar past = data\n    |shift(shift)\n\nvar current = data\n\nvar trigger = past\n    |join(current)\n        .as('past', 'current')\n    |eval(lambda: abs(float(\"current.value\" - \"past.value\")) / float(\"past.value\") * 100.0)\n        .keep()\n        .as('value')\n    |alert()\n        .crit(lambda: \"value\" > crit)\n        .stateChangesOnly()\n        .message(message)\n        .id(idVar)\n        .idTag(idTag)\n        .levelTag(levelTag)\n        .messageField(messageField)\n        .durationField(durationField)\n\ntrigger\n    |eval(lambda: float(\"value\"))\n        .as('value')\n        .keep()\n    |influxDBOut()\n        .create()\n        .database(outputDB)\n        .retentionPolicy(outputRP)\n        .measurement(outputMeasurement)\n        .tag('alertName', name)\n        .tag('triggerType', triggerType)\n\ntrigger\n    |httpOut('output')\n",
    "vars": {},
    "dot": "digraph dutianze-t {\ngraph [throughput=\"0.00 points/s\"];\n\nstream0 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\nstream0 -> from1 [processed=\"0\"];\n\nfrom1 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\nfrom1 -> eval2 [processed=\"0\"];\n\neval2 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\neval2 -> shift3 [processed=\"0\"];\neval2 -> join5 [processed=\"0\"];\n\nshift3 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\nshift3 -> join5 [processed=\"0\"];\n\njoin5 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\njoin5 -> eval6 [processed=\"0\"];\n\neval6 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\neval6 -> alert7 [processed=\"0\"];\n\nalert7 [alerts_inhibited=\"0\" alerts_triggered=\"0\" avg_exec_time_ns=\"0s\" crits_triggered=\"0\" errors=\"0\" infos_triggered=\"0\" oks_triggered=\"0\" warns_triggered=\"0\" working_cardinality=\"0\" ];\nalert7 -> http_out10 [processed=\"0\"];\nalert7 -> eval8 [processed=\"0\"];\n\nhttp_out10 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\n\neval8 [avg_exec_time_ns=\"0s\" errors=\"0\" working_cardinality=\"0\" ];\neval8 -> influxdb_out9 [processed=\"0\"];\n\ninfluxdb_out9 [avg_exec_time_ns=\"0s\" errors=\"0\" points_written=\"0\" working_cardinality=\"0\" write_errors=\"0\" ];\n}",
    "status": "enabled",
    "executing": true,
    "error": "",
    "stats": {},
    "created": "2019-09-26T15:46:19.40868194+08:00",
    "modified": "2019-09-26T15:46:19.40868194+08:00",
    "last-enabled": "2019-09-26T15:46:19.40868194+08:00"
}
```

<!--more-->

### 修改任务的 dbrps

```http request
PATCH /kapacitor/v1/tasks/dutianze-t
{
    "dbrps": [{"db": "NEW_DATABASE_NAME", "rp" : "NEW_RP_NAME"}]
}
```

### 启用任务

```http request
PATCH /kapacitor/v1/tasks/dutianze-t
{
    "status" : "enabled"
}
```

### 停用任务

```http request
PATCH /kapacitor/v1/tasks/dutianze-t
{
    "status" : "disabled"
}
```

### 定义任务时直接启用任务

```http request
POST /kapacitor/v1/tasks
{
    "id" : "dutianze-t",
    "type" : "stream",
    "dbrps" : [{"db": "DATABASE_NAME", "rp" : "RP_NAME"}],
    "script" : "stream\n    |from()\n        .measurement('cpu')\n",
    "status" : "enabled"
}
```

### 返回码

| Code | 说明                       |
| ---- | -------------------------- |
| 200  | 创建的任务, 并带有任务详情 |
| 404  | 任务不存在                 |

## 获取任务

```http request
GET /kapacitor/v1/tasks/dutianze-t
```

```http response
{
    "link" : {"rel": "self", "href": "/kapacitor/v1/tasks/TASK_ID"},
    "id" : "TASK_ID",
    "type" : "stream",
    "dbrps" : [{"db": "DATABASE_NAME", "rp" : "RP_NAME"}],
    "script" : "stream\n    |from()\n        .measurement('cpu')\n",
    "dot" : "digraph TASK_ID { ... }",
    "status" : "enabled",
    "executing" : true,
    "error" : "",
    "created": "2006-01-02T15:04:05Z07:00",
    "modified": "2006-01-02T15:04:05Z07:00",
    "last-enabled": "2006-01-03T15:04:05Z07:00",
    "stats" : {}
}
```

## 删除任务

```http request
DELETE /kapacitor/v1/tasks/TASK_ID
```

Response 204 —> Success

## 列出任务

### 获取所有任务

```http request
GET /kapacitor/v1/tasks
```

```http response
{
    "tasks" : [
        {
            "link" : {"rel":"self", "href":"/kapacitor/v1/tasks/TASK_ID"},
            "id" : "TASK_ID",
            "type" : "stream",
            "dbrps" : [{"db": "DATABASE_NAME", "rp" : "RP_NAME"}],
            "script" : "stream|from().measurement('cpu')",
            "dot" : "digraph TASK_ID { ... }",
            "status" : "enabled",
            "executing" : true,
            "error" : "",
            "stats" : {}
        },
        {
            "link" : {"rel":"self", "href":"/kapacitor/v1/tasks/ANOTHER_TASK_ID"},
            "id" : "ANOTHER_TASK_ID",
            "type" : "stream",
            "dbrps" : [{"db": "DATABASE_NAME", "rp" : "RP_NAME"}],
            "script" : "stream|from().measurement('cpu')",
            "dot" : "digraph ANOTHER_TASK_ID{ ... }",
            "status" : "disabled",
            "executing" : true,
            "error" : "",
            "stats" : {}
        }
    ]
}
```

### 获取匹配的任务(匹配 id)

```http request
GET /kapacitor/v1/tasks?pattern=TASK*
```

```http response
{
    "tasks" : [
        {
            "link" : {"rel":"self", "href":"/kapacitor/v1/tasks/TASK_ID"},
            "id" : "TASK_ID",
            "type" : "stream",
            "dbrps" : [{"db": "DATABASE_NAME", "rp" : "RP_NAME"}],
            "script" : "stream|from().measurement('cpu')",
            "dot" : "digraph TASK_ID { ... }",
            "status" : "enabled",
            "executing" : true,
            "error" : "",
            "stats" : {}
        }
    ]
}
```

如果该模式与任何任务都不匹配，将返回一个空列表，成功码 200

### 获取任务的指定属性

这里选择`status`、`executing`、`error`, 另外`id`和`link`永远会返回

```http request
GET /kapacitor/v1/tasks?fields=status&fields=executing&fields=error
```

```http response
{
    "tasks": [
        {
            "error": "",
            "executing": false,
            "id": "1",
            "link": {
                "rel": "self",
                "href": "/kapacitor/v1/tasks/1"
            },
            "status": "disabled"
        },
        {
            "error": "",
            "executing": false,
            "id": "111",
            "link": {
                "rel": "self",
                "href": "/kapacitor/v1/tasks/111"
            },
            "status": "disabled"
        }
    ]
}
```

response 200 mean success

## 其他

### Ping

可以通过这个接口 ping 服务, 来验证是否成功连接 kapacitor, 返回 204 代表成功

```http request
GET /kapacitor/v1/ping
```

return: `204 no content`

### 显示所有 api 接口

```http request
GET /kapacitor/v1/:routes
```

### 更多

更多 api 查看[kapacitor](https://docs.influxdata.com/kapacitor/v1.5/working/api/)官网
