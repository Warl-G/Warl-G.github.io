---
layout: post
title:  "Unity工具—汇总"
description: Unity 工具汇总及说明
date:   2020-02-17 18:00:07 +0800
categories: [Unity]
tag: [Unity工具]
 
---

收集有用的 Unity 工具，使用方式，链接等

### 依赖管理  

1. [Play Services Resolver for Unity](https://warl.top/posts/play-services-resolver)  

   面向 Unity 管理 Android 和 iOS 原生依赖库的工具  

### 多线程   

1. Loom   

   流传度很广的多线程异步与主线程同步工具，本人写的 TaskQueue 使用了该工具

2. [TaskQueue](https://github.com/Warl-G/GRUnityTools)  

   在 Unity 下的多线程任务队列工具，可实现串行、并发、同步、异步相互组合的任务队列，支持延迟执行和主线程同步

### 数据库   

1. Mono.Data.Sqlite  

   可实现跨平台的 Sqlite 数据库  
   
2. [SqliteHelper](https://github.com/Warl-G/GRUnityTools)     

   使用 Mono.Data.Sqlite 和 TaskQueue 制作的数据库快捷操作和队列操作工具