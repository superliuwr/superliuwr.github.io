---
title: Ruby on Rails
date: 2018-06-23 22:56:16
categories:
- RoR
tags:
- Ruby
- Rails
---

## Development Environment

Follow [this](https://ruby-china.org/wiki/install_ruby_guide) guide.

## Philosohpy
### Don't repeat yourself
### Convention over Configuration

## MVC

* Model: 操作数据库
* Controller: 把每个 http request 分发到对应的 Action( method) 来处理
* View: 显示页面

### Router

config/routes.rb: `resources :users`

```
GET  /users         index     显示 user的列表页
GET  /users/new     new       显示 user的新建页面。
GET  /users/3       show      显示id是3的user
GET  /users/3/edit  edit      显示 user(id =3)的编辑页面。
PUT  /users/3       update    对id = 3的user进行修改 （后面还会紧跟一大串的参数)
POST /users         create    对users进行创建（后面也有一大堆参数)
DELETE  /users/3    destroy   对 id=3的 user 进行删除操作。
```

可以使用`$ rake routes`查看当前项目中所有的路由

### View

### Model

### Controller

## Database Migration
