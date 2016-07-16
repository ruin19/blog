---
layout: post_layout
title: Qzone直播剥离IMSDK实践
author: ruin
time: 2016年07月16日 星期六
location: 汕头
pulished: true
excerpt_separator: "```"
---

2016年上半年，Qzone开始进军视频直播领域，依托腾讯云[互动直播](https://www.qcloud.com/doc/product/268/3158)的强大基础能力，迅速实现了从无到有的突破，并且在体验、性能方面不断优化，做到业界领先。

## Qzone直播剥离IMSDK实践
-----

2016年上半年，Qzone开始进军视频直播领域，依托腾讯云[互动直播](https://www.qcloud.com/doc/product/268/3158)的强大基础能力，迅速实现了从无到有的突破，并且在体验、性能方面不断优化，做到业界领先。  
接入互动直播主要包含QAVSDK和IMSDK。前者提供音视频上下行能力，包大小4MB，后者提供聊天和信令通道的能力，包大小8MB。Qzone出于减包压力，需要剥离IMSDK，采用替代方案实现原IMSDK提供的能力。  
互动直播对IMSDK的依赖分三大块：

1. 聊天
2. 业务通过IMSDK向音视频后台发送推流、录制命令
3. QAVSDK通过IMSDK向音视频后台请求接口机IP列表，上报等

我们针对这三大块采用对应方案解决

### 聊天
Qzone后台搭建消息服务，客户端轮询拉取。后台返回下次拉取时间，拉取频率后台可控。

### 推流和录制
[推流](https://www.qcloud.com/doc/product/268/3219)是业务请求音视频后台将上行流旁路出HLS或RTMP流，用于H5观看场景。  
[录制](https://www.qcloud.com/doc/product/268/3218)是业务请求音视频后台将上行流转码出MP4文件，用于直播结束后可以回看。  
带IMSDK的情况下，业务调用IMSDK的推流和录制函数，IMSDK负责打包推流和录制的请求，发给后台SSO，再转发给音视频后台。  
剥离IMSDK后，业务需要理解推流和录制的封包，再打到WNS请求包里，发给Qzone后台；Qzone后台需要解WNS包，重新按SSO协议打包，发给SSO，SSO转发给音视频后台。  
架构对比如下图：  
![去IMSDK之推流录制](http://o97en6hsv.bkt.clouddn.com/%E5%8E%BBIMSDK%E4%B9%8B%E6%8E%A8%E6%B5%81%E5%BD%95%E5%88%B6.png)

### 去除QAVSDK对IMSDK的依赖
QAVSDK本身也有依赖IMSDK发信令的地方，例如拉取接口机IP列表，上报等。  
剥离IMSDK的方案是：QAVSDK负责打解包，回调到Qzone，后面的逻辑就跟推流录制的方案一致，通过WNS转发Qzone后台--SSO--音视频后台。  
架构对比如下图：  
![去IMSDK之回调机制](http://o97en6hsv.bkt.clouddn.com/%E5%8E%BBIMSDK%E4%B9%8BQAVSDK%E5%9B%9E%E8%B0%83.png)
