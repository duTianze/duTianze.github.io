---
title:  InfluxDB Introducing
date:  2019-12-09 16:00:00
tags: influxdb
---
# InfluxDB介绍
```bash
8888888           .d888 888                   8888888b.  888888b.
  888            d88P"  888                   888  "Y88b 888  "88b
  888            888    888                   888    888 888  .88P
  888   88888b.  888888 888 888  888 888  888 888    888 8888888K.
  888   888 "88b 888    888 888  888  Y8bd8P' 888    888 888  "Y88b
  888   888  888 888    888 888  888   X88K   888    888 888    888
  888   888  888 888    888 Y88b 888 .d8""8b. 888  .d88P 888   d88P
8888888 888  888 888    888  "Y88888 888  888 8888888P"  8888888P"
```

## 安装influxdb
### macOS
```bash
安装
$ brew update
$ brew install influxdb

启动服务端
$ influxd

打开新的命令行，进入CLI
$ influx
```

### docker
```bash
拉取镜像
$ docker pull influxdb

创建挂载目录
$ mkdir -p ~/tmp/influxdb

启动容器
$ docker run -d --rm -p 8086:8086 \
      --name=influxdb \
      -v ~/tmp/influxdb:/var/lib/influxdb \
      influxdb

进入CLI
$ docker exec -it influxdb influx
```
<!--more-->
## 操作
### 创建数据库
No news is good news!
```bash
> CREATE DATABASE mydb
```
### 显示数据库
```bash
> SHOW DATABASES
```
> 注意: _internal 是系统数据库

### 使用数据库
```bash
> USE mydb
```
这样之后，所有的命令都将仅在`mydb`上运行

### 写入数据
写入的格式
```bash
INSERT <measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```
例子
将`measurement`为cpu，`tag`为host和region的点，`value`为0.64，写入数据库，
```bash
> INSERT cpu,host=serverA,region=us_west value=0.64
```
---
### 查看数据
```bash
> SELECT "host", "region", "value" FROM "cpu"
name: cpu
time                host    region  value
----                ----    ------  -----
1575971743721439813 serverA us_west 0.64
>
```

### 练习
```bash
> INSERT temperature,machine=unit42,type=assembly external=25,internal=37

> SELECT * FROM "temperature"
name: temperature
time                external internal machine type
----                -------- -------- ------- ----
1575972117246720239 25       37       unit42  assembly
```
> 警告：在大的数据库上使用*不带LIMIT子句可能会导致性能问题。您可以使用Ctrl + C取消响应时间太长的查询。

[查看原始文档](https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/)
