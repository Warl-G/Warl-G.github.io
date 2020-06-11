---
layout: post
title:  "Unity工具—汇总"
description: Unity 工具汇总及说明
date:   2020-02-17 18:00:07 +0800
categories: [Unity]
tag: [Unity工具,汇总]
 
---

收集有用的 Unity 工具，使用方式，链接等

### 依赖管理  

1. [Play Services Resolver for Unity](https://warl-g.github.io/posts/play-services-resolver)  

   面向 Unity 管理 Android 和 iOS 原生依赖库的工具   

### 版本管理

1. [UnityYAMLMerge](https://warl-g.github.io/posts/Unity-UnityYAMLMerge)

   Unity 官方提供的 Unity 资源文件合并工具

### 多线程   

1. Loom   

   流传度很广的多线程异步与主线程同步工具，本人写的 TaskQueue 使用了该工具

2. [TaskQueue](https://github.com/Warl-G/GRUnityTools)  

   在 Unity 下的多线程任务队列工具，可实现串行、并发、同步、异步相互组合的任务队列，支持延迟执行和主线程同步

### 数据库   

1. [Mono.Data.Sqlite](https://warl-g.github.io/posts/unity-sqlite)

   可实现跨平台的 Sqlite 数据库  
   
2. [SqliteHelper](https://github.com/Warl-G/GRUnityTools)     

   使用 Mono.Data.Sqlite 和 TaskQueue 制作的数据库快捷操作和队列操作工具

### 本地化  

1. [I2 Localization]((https://assetstore.unity.com/packages/tools/localization/i2-localization-14884?locale=zh-CN))

   功能强大的本地化工具

2. [GRTools.Localization](https://github.com/Warl-G/GRUnityTools)

   可自定义资源存取规则和文本解析格式的本地化工具