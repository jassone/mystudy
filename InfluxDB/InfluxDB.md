## InfluxDB

InfluxDB是领先的开源时间序列数据库（TSDB）。
InfluxDB使用Go语言编写，适用于各类时间序列数据的高效存储与检索。被广泛应用于监控系统，如cpu利用率，io，内存等指标；穿戴设备，如心率，体温；IoT实时数据等场景。

## 一、什么是时间序列数据

数据源每隔一段时间产生一条数据，除了时间戳和值不一样，其他都相同。比如cpu使用率，随着时间变化，它产生的数据就是时间序列数据。

## 二、InfluxDB 主要特性

- 部署简单；
- 极强的写能力以及高压缩率；
- InfluxQL：类SQL的查询语句；
- 具有一系列聚合函数；

## 三、InfluxDB 重要概念

- database：数据库

- measurement：数据库表

- point：一行数据，由时间戳（time）、标签（tags）和值（field）组成

  time：每条数据记录的时间，也是数据库自动生成的主索引

  tags：各种有索引的属性

  fields：各种记录的值

## 相关命令

### 1、基本命令

```sh
//查看默认配置

influxd config

//指定配置文件启动，启动服务端

influxd -config /etc/influxdb/influxdb.conf
```

 ### 2、数据库命令

```sh
#influxdb使用命令（默认8086 配置多实例时端口不同），启动客户端
influx
influx -port 8086
influx -host ip -port 8086

#创建数据库
create database test

#删除数据库
drop database test

#查看数据库
show databases

# 使用数据库 
use test

# 创建普通用户
create user 'username' with password 'password'

# 创建管理员用户
create user 'username' with password 'password' with all privileges

# 登录
auth

# 修改密码
influx user password -n 'username'
```

## 相关wiki

- InfluxDB入门教程 https://www.jianshu.com/p/09a68551ba86
- go 客户端demo https://github.com/influxdata/influxdb1-client