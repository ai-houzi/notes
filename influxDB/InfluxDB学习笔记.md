# InfluxDB学习笔记

## 官网

<https://www.influxdata.com/time-series-platform/influxdb/>

InfluxDB是InfluxData的核心产品。InfluxDB是一个开源分布式时序、时间和指标数据库，使用Go语言编写，无需外部依赖。其设计目标是实现分布式和水平伸缩扩展。

## 教程

<https://www.jianshu.com/p/48104975d60a>
<https://www.jianshu.com/p/a373784c0bf9>
<https://www.jianshu.com/p/b51ba7f88fb0>
[https://jkzhao.github.io/2017/12/15/时序数据库InfluxDB/](https://jkzhao.github.io/2017/12/15/%E6%97%B6%E5%BA%8F%E6%95%B0%E6%8D%AE%E5%BA%93InfluxDB/)

## 安装方式

<https://influxdata.com/downloads/>

### macOS

```
brew update
brew install influxdb
```

或
`https://dl.influxdata.com/influxdb/releases/influxdb-1.5.4_darwin_amd64.tar.gz tar zxvf influxdb-1.5.4_darwin_amd64.tar.gz`

### Docker

```
docker pull influxdb
```

### Ubuntu

```
wget https://dl.influxdata.com/influxdb/releases/influxdb_1.5.4_amd64.deb
sudo dpkg -i influxdb_1.5.4_amd64.deb
```

### CentOS

```
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.5.4.x86_64.rpm
sudo yum localinstall influxdb-1.5.4.x86_64.rpm
```

## 配置

### macOS安装

配置文件路径
`/usr/local/etc/influxdb.conf /etc/influxdb/influxdb.conf`

## 名词

## 创建数据库

```
influx

show databases;
create database testdb;
```

## 显示所有表

```
show measurements
```

## 新建表

无建表语句，第一次insert后自动创建
`insert weather,altitude=1000,area=北 temperature=11,humidity=-4`
`weather # 表名 altitude=1000,area=北 # tag temperature=11,humidity=-4 # field`

## 删除表

```
drop measurement weather
```

## series操作

series表示这个表里面的数据，可以在图表上画成几条线，series主要通过tags排列组合算出来。
`show series from weather`

## HTTP创建和删除数据库

curl -i -XPOST <http://localhost:8086/query> --data-urlencode "q=CREATE DATABASE mydb"
curl -POST <http://localhost:8086/query> --data-urlencode "q=DROP DATABASE mydb"

## HTTP添加数据

单条
`curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'`
多条
`curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server02 value=0.67 cpu_load_short,host=server02,region=us-west value=0.55 1422568543702900257 cpu_load_short,direction=in,host=server01,region=us-west value=2.0 1422568543702900257'`

## 使用HTTP查询数据

```
curl -GET 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=show measurements"
```

或直接在浏览器中:
`http://localhost:8086/query?pretty=true&db=mydb&q=show%20measurements`

查询多条：
`curl -GET 'http://localhost:8086/query?db=_internal' --data-urlencode "q=show databases;show measurements"`

### 时间格式

epoch=[h,m,s,ms,u,ns]
`curl -G 'http://localhost:8086/query' --data-urlencode "db=mydb" --data-urlencode "epoch=s" --data-urlencode "q=SELECT value FROM cpu_load_short WHERE region='us-west'"`

### 指定每次查询数据大小

chunk_size
`curl -G 'http://localhost:8086/query' --data-urlencode "db=mydb" --data-urlencode "chunk_size=200" --data-urlencode "q=SELECT value FROM cpu_load_short WHERE region='us-west'"`

## WEB控制台

1.3内置8086WEB管理已经移除，需要使用TICK工具栈中的Chronograf来进行管理。

下载安装说明：
<https://portal.influxdata.com/downloads>
`brew install chronograf`

使用:
[http://localhost:8888](http://localhost:8888/)