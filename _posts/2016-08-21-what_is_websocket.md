---
layout: post_layout
title: websocket简介
author: ruin
time: 2016年08月21日 星期天
location: 深圳
pulished: true
excerpt_separator: "```"
---

### websocket是什么

websocket是随着HTML5系列规范而出现的一个新协议，在浏览器和服务器之间保持TCP长连接，不仅浏览器可以向服务器请求，服务器还可以主动向浏览器发送数据，主要解决了服务器向浏览器PUSH的问题。


### 为什么用websocket

由于HTTP的**被动性**特点，只能是浏览器向服务器发request，服务器向浏览器回response这样一来一回的模式，服务器不能主动向浏览器发消息。  
为了实现服务器有新东西能通知到浏览器，在websocket出现以前，人们一般是采用**polling**（轮询）和**long polling**（长轮询）的方式。


#### polling

polling最简单，浏览器每隔一段时间询问服务器有没有新的东西，服务器如果有，就把新东西返回给浏览器，如果没有，就返回告诉浏览器没有新东西。

```
浏览器：我有新的信件吗？
服务器：没有。
浏览器：我有新的信件吗？
服务器：没有。
浏览器：我有新的信件吗？
服务器：有了，给你。

```

#### long polling

long polling也是浏览器向服务器询问有没有新东西，服务器如果查到有，就返回新东西给浏览器，如果查不到，就不回包，直到服务器有新东西才返回。

```
浏览器：我有新的信件吗？
服务器：。。。（因为没有新的信件，沉默）
服务器：。。。（因为没有新的信件，沉默）
服务器：有了，给你。
```

#### websocket

由于HTTP的**被动性**，以上两种polling就是实现后台新东西通知到浏览器的常见方式了。  
为了实现真正意义上的PUSH，人们提出了websocket协议，这是一套基于TCP的应用层协议，和HTTP是完全不同的两套协议。  
其核心思想是在浏览器和服务器之间保持**TCP长连接**，并且应用层**全双工**，服务器可以主动向浏览器发送消息。

```
服务器：来来来，这里有新的信件给你
服务器：来来来，这里又有新的信件给你
```

### websocket的通信机制

浏览器和服务器之间是通过一次HTTP握手（请求和返回），协商切换到websocket协议的。我们来看一次HTTP握手的包头：  

#### 请求：

```
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```
这里关键在于Upgrade和Connection，浏览器请求切换到websocket协议

#### 返回：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```
HTTP返回码101表示协议切换，"Upgrade"到websocket协议。  
至此，后续浏览器和服务器之间就以websocket协议进行通信了。
