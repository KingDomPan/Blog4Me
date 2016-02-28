---
title: Nodejs进程模块
date: 2016-02-24 23:12:43
tags: [nodejs, child-porcess]
---

#### 创建子进程
- `spawn()`启动一个子进程来执行命令
- `exec()`启动一个子进程来执行命令, 带回调参数获知子进程的情况, 可指定进程运行的超时时间
- `execFile()`启动一个子进程来执行一个可执行文件, 可指定进程运行的超时时间
- `fork()` 与`spawn()`类似, 不同在于它创建的node子进程只需指定要执行的js文件模块即可
```javascript
// don't call this example code
var cp = require('child_process');
cp.spawn('node', ['work.js']);
cp.exec('node work.js', function(err, stdout, stderr) {
  // some code
});
cp.execFile('work.js', function(err, stdout, stderr) {
  // some code
});
cp.fork('./work.js');
```

<!-- more -->

#### 进程间通信
- IPC: window下使用Named Pipe实现, Linux下使用Unix Domain Socket实现
- 简单的API, 父进程取得n进行的句柄, 使用send方法向子进程发送消失, 同时监听message事件接收子进程发送过来的消息
- 过程
  - 父进程在创建子进程前创建IPC通道并监听, 用环境变量NODE_CHANNEL_FD告诉子进程的IPC的文件描述符
  - 子进程在启动的过程中连接IPC的FD
  - IPC实际上也是Stream的抽象, 在系统内核就完成了进程间的通信

#### 句柄传递
- 一个进程只能监听一个端口
- 子进程只能通过父进程进行请求转发(这样会浪费一倍的文件描述符[请求连接父进程])
- 解决办法: send发送数据的时候传送句柄 `send(message[, sendHandle])`
- 父进程接收到socket后直接将这个socket传递给子进程处理
  - 请求可以被父进程和子进程同时处理
  - 如果父进程的server关闭, 那么请求可以被子进程同事处理
  - 都基于绑定在同一个端口上的操作

#### 句柄的发送与还原
- send方法可以发送的对象包括如下集中
  - net.Socket对象: TCP套接字
  - net.Server对象: TCP服务器
  - net.Native: C++层面的TCP套接字和IPC管道
  - dgram.Socket: UDP套接字
  - dgram.Native: C++层面的UDP套接字

#### 端口服用
- 独立启动的TCP服务器拥有不同的文件描述符, 监听同一个端口失败
- 句柄传送中, 传送的是一个相同文件描述符的server, 子进程按照此文件描述符进行还原, 达到端口复用
- 文件描述符同一时间只能被一个端口使用, 这些进程是抢占式的服务
