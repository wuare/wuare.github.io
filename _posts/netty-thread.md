---
title: Netty线程
date: 2020-08-26 21:10:21
tags: Netty
---
Netty线程组NioEventLoopGroup，里面包含多个子线程，子线程实现为NioEventLoop。
<!-- more -->
每个NioEventLoop中有一个selector，用来注册连接。