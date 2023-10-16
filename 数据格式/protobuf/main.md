## protobuf

## 一、继承知识

### 1、特点

protobuf之所以小且快，就是因为使用**变长的编码规则**，只保存有用的信息，节省了大量空间。

## 二、细节

### 1、创建

```protobuf
syntax = "proto3";
package user;
option go_package = "./"; // 必须要带上
```

## 相关wiki

- protobuf详解 https://zhuanlan.zhihu.com/p/432875529 
- protobuf简介(数据结构方面)http://t.zoukankan.com/hnxxcxg-p-9405192.html