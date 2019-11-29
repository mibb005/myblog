---
title: node中使用redis
date: 2019-11-29 15:27:05
tags:
 - node
 - redis
---

# Redis简介

Redis是一个高性能的key-value数据库，Redis把数据存在内存中，并在磁盘中记录数据的变化。因为将数据存在内存中，所以数据操作非常快。

# 安装

以windows环境为例，先下载windows版本的redis，地址如下：3.2.100
下载完成后，解压，我这里解压到D:redis目录下

# 开启服务

打开一个 cmd 窗口，进入目录到 D:redis，运行 redis-server.exe redis.windows.conf。

出现上面界面，则redis已经在本机端口6379启动了服务，那么接下来，便可以用客户端连接到redis服务端了。

# 在node中使用redis

首先，安装驱动：`npm install redis`

`redis`支持多种数据类型，常用的有键/值对，哈希表，链表，集合等。

## 普通数据

我们先来看看如何存储和获取键/值对。

```
var redis = require('redis')

var client = redis.createClient(6379, '127.0.0.1')
client.on('error', function (err) {
  console.log('Error ' + err);
});

// 1 键值对
client.set('color', 'red', redis.print);
client.get('color', function(err, value) {
  if (err) throw err;
  console.log('Got: ' + value)
  client.quit();
})
```

## 哈希表

哈希表有点类似ES6中的Map。

```
client.hmset('kitty', {
  'age': '2-year-old',
  'sex': 'male'
}, redis.print);
client.hget('kitty', 'age', function(err, value) {
  if (err) throw err;
  console.log('kitty is ' + value);
});

client.hkeys('kitty', function(err, keys) {
  if (err) throw err;
  keys.forEach(function(key, i) {
    console.log(key, i);
  });
  client.quit();
});
```

## 链表

Redis链表类似JS数组，lpush向链表中添加值，lrange获取参数start和end范围内的链表元素, 参数end为-1，表明到链表中最后一个元素。
注意：随着链表长度的增长，数据获取也会逐渐变慢（大O表示法中的O(n)）

```
client.lpush('tasks', 'Paint the house red.', redis.print);
client.lpush('tasks', 'Paint the house green.', redis.print);
client.lrange('tasks', 0, -1, function(err, items) {
  if (err) throw err;
  items.forEach(function(item, i) {
    console.log(' ' + item);
  });
  client.quit();
});
```

## 集合

类似JS中的Set，集合中的元素必须是唯一的，其性能: 大O表示法中的O(1)

```
client.sadd('ip', '192.168.3.7', redis.print);
client.sadd('ip', '192.168.3.7', redis.print);
client.sadd('ip', '192.168.3.9', redis.print);
client.smembers('ip', function(err, members) {
  if (err) throw err;
  console.log(members);
  client.quit();
});
```

## 信道

Redis超越了数据存储的传统职责，它还提供了信道，信道是数据传递机制，提供了发布/预定功能。

```
var redis = require('redis')
var clientA = redis.createClient(6379, '127.0.0.1')
var clientB = redis.createClient(6379, '127.0.0.1')

clientA.on('message', function(channel, message) {
  console.log('Client A got message from channel %s: %s', channel, message);
});
clientA.on('subscribe', function(channel, count) {
  clientB.publish('main_chat_room', 'Hello world!');
});
clientA.subscribe('main_chat_room');
```

上面代码中，clientA订阅了main_chat_room，这时clientA捕获到订阅事件，执行回调函数，clientB向main_chat_room发送了一条信息Hello world!
clientA接受到信息后，在控制台打印出了相关信息。

