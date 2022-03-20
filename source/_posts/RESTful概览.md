---
title: RESTful概览
date: 2022-02-07 22:15:41
tags: RESTful
---

Representational state transfer  表征性状态转移

<!-- more -->

#### 基本特点

- 无状态 

  一次调用就返回结果，不需要依赖上次连接

- 面向 “资源”

  链接里只有动词没有名词，api/v1/touristRoutes 可以，api/v1/GetTouristRoutes 有动词不行

- 使用 HTTP 动词表示操作

  | 动词   | 含义     | 例子                                  |
  | ------ | -------- | ------------------------------------- |
  | GET    | 查看     | HTTP GET api/v1/touristRoutes         |
  | POST   | 创建     | HTTP POST api/v1/touristRoutes        |
  | PUT    | 更新     | HTTP PUT api/v1/touristRoutes/{id}    |
  | PATCH  | 部分更新 | HTTP PATCH api/v1/touristRoutes/{id}  |
  | DELETE | 删除     | HTTP DELETE api/v1/touristRoutes/{id} |

- HATOAS 超媒体即应用状态引擎

#### 适用

面向对象（资源），增删改查这些比较好用

面向过程就不大好，比如说登录这种操作，路径包含 login 就不行了，也无法确定要用 get post 等哪个操作
