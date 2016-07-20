---
layout: post_layout
title: Qzone直播配置管理原理及优化
author: ruin
time: 2016年07月16日 星期六
location: 汕头
pulished: true
excerpt_separator: "```"
---

2016年上半年，Qzone开始进军视频直播领域，依托腾讯云[互动直播](https://www.qcloud.com/doc/product/268/3158)的强大基础能力，迅速实现了从无到有的突破，并且在体验、性能方面不断优化，做到业界领先。
本文主要讲述直播的配置下发原理及优化，要点：

1. 直播有哪些配置
2. 配置如何生效（默认方式）
3. 遇到的问题
4. 优化后的配置生效方式
5. 现在的配置发布过程

#### 直播有哪些配置
每个接入腾讯云互动直播的应用都有自己的一个配置管理页面，设置视频、音频、网络的一些参数。这个是Qzone的[配置管理页](http://console.qcloud.com/ilvb/trafficControl/1400007595)。
放个图感受一下
![直播配置](http://o97en6hsv.bkt.clouddn.com/%E7%9B%B4%E6%92%AD%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86%E7%AB%AF%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

#### 配置如何生效（默认方式）
默认情况下，在配置管理页面修改完后，sdk开始一个会话，调用`- (QAVResult)startContext:(ContextOperationBlock)block;`时，sdk内部会向配置服务器拉配置，这样配置就生效了。
这种方式是sdk最初提供的配置生效方式，也是目前除了Qzone以外，其他应用的配置生效方式

#### 遇到的问题
采用上述配置生效方式，Qzone遇到过两个问题：

1. `startContext`时，sdk内部拉配置有一定的失败率，失败后走默认配置，默认配置又不可针对应用定制。
2. `startContext`是app登录时执行一次，登录后如果后台配置有更新，需要等到用户退出后，下次登录才生效。

#### 优化后的配置生效方式
1. sdk增加一个外部传配置的模式，这种模式下，`startContext`不再从配置服务器拉配置
2. 配置通过Qzone成熟的wns配置系统下发
3. sdk支持会话期间可以更改配置，不用等下一次会话开始

