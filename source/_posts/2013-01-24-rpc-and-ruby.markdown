---
layout: post
title: "RPC 协议和他的 ruby 实现"
date: 2013-01-24 21:37
comments: true
categories: Ruby
---

## 什么是 RPC

RPC 是 Remote Procedure Call的缩写，翻译成中文就是远程过程调用。

在百度百科中，我们可以简单的得到精确的定义：

{% blockquote %}

RPC协议假定某些传输协议的存在，如TCP或UDP，为通信程序之间携带信息数据。在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。

RPC采用客户机/服务器模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。首先，调用进程发送一个有进程参数的调用信息到服务进程，然后等待应答信息。在服务器端，进程保持睡眠状态直到调用信息的到达为止。当一个调用信息到达，服务器获得进程参数，计算结果，发送答复信息，然后等待下一个调用信息，最后，客户端调用过程接收答复信息，获得进程结果，然后调用执行继续进行。

{% endblockquote %}

## Ruby and XML-RPC

XML-RPC 是一种基于 XML 传输的 RPC 实现方式。 而 ruby 里已经为我们内建了 XML-RPC 的类库，我们可以很方便的拿来编写程序。下面我们来看一个简单的例子。

``` ruby server.rb
require 'xmlrpc/server'

server = XMLRPC::Server.new 8080

server.add_handler 'user.yield' do |name|
  "Hi, #{name}! Welcome to the RPC world."
end

server.serve

```

``` ruby client.rb
require 'xmlrpc/client'

client = XMLRPC::Client.new 'localhost', '/', 8080

puts client.call 'user.yield', 'zhex'
```
