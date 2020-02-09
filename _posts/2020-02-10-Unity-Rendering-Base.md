---
layout: post
title:  "Unity学习—渲染知识基础"
description: Unity 渲染相关基础
date:   2020-02-10 03:30:07 +0800
categories: [Unity]
tag: [学习笔记,渲染]

---

关于Unity 渲染概念的基本知识，和部分技术名词解释

部分特性还未使用过，且网上资料很少，所以欠缺解释，后期补足

本文其他地址：[简书](https://www.jianshu.com/p/ee01f6d7b2f0)	[知乎](https://zhuanlan.zhihu.com/p/105978439) 	[掘金](https://juejin.im/post/5e405c22e51d45270218e8b1)

## 名词解释  

* 纹理 Texture  
  图片，U3D中纹理的类型分为8种，如法线贴图为2D图片模拟立体效果

* 贴图 Map  
  包含纹理和UV坐标，通过UV坐标将纹理映射到3D物体表面

* 材质 Material  
  包含贴图，主要是用来表现物体对光的交互。将输入的贴图或者颜色，加上对应的 Shader，以及对 Shader 的特定的参数设置，将这些打包在一起就是一个材质

* 着色器 Shader   
  Shader 负责将输入的 Mesh（网格）以指定的方式和输入的贴图或者颜色等组合作用，然后输出。绘图单元可以依据这个输出来将图像绘制到屏幕上 
  可理解为一套计算公式，用来计算一个对象对于光照会有怎样的反应

* 图元 Primitive  
  几何顶点被组合为图元（点，线段或多边形）

* 片元 Fragment  
  图元被合成片元，最后片元被转换为帧缓存中的像素数据

* 光栅化 Rasterization  
  光栅化是将一个图元转变为一个二维图象的过程

* 可见性测试  
  又叫测试混合，包括裁切测试、Alpha测试、模板测试和深度测试

* Cull  

  剔除无需渲染的对象

* Draw Call   

  CPU 调用图像编程接口，命令 GPU 进行渲染的操作，Draw Call 包含网格和渲染数据，将不同物体的数据合并为一个 Draw Call 称为批处理（Batching）

* Batch   

  GPU 渲染所需数据包，由 CPU 调用 Draw Call 时一同发送给 GPU

* Setpass Call     

  CPU 发送该指令通知 GPU 更改渲染状态（Render State）
  
  

## 渲染过程  

### 渲染阶段  

1. 应用阶段（CPU）  
   * CPU 负责决定哪些对象被渲染，哪些被剔除，从硬盘加载到内存再到显存
   * 设置对象渲染状态，即材质纹理着色器
   * 输出渲染图元，向 GPU 发送 Draw Call
2. 几何阶段（GPU）
   * 顶点着色器、曲面细分着色器、几何着色器分别对图元做不同处理
   * 裁剪多余图元
   * 映射三维坐标到屏幕坐标
3. 光栅化阶段（GPU）
   * 三角形设置： 计算屏幕坐标系顶点数据，获取光栅化网格数据
   * 三角形遍历： 对三角形网格覆盖像素生成片元，插值计算片元状态
   * 片元着色器 处理片元输出像素
   * 可见性测试 舍弃测试不通过的片元
   * 将像素缓冲区的像素颜色经后缓冲区绘制，再与前缓冲区交换并刷新显示在屏幕上  

### 渲染操作   

##### CPU

1. 检查场景，判断渲染对象，如：是否在相机视野内。不渲染物体剔除（Cull）  
2. 收集排序渲染物体的信息放入 Draw Call 指令
3. 为每一个 Draw Call 创建 Batch，Batch 也包含 Draw Call 以外的数据
   1. CPU发送命令（SetPass Call）GPU更改渲染相关变量，称为渲染状态（Render State），仅在需要更改时发送命令
   2. CPU发送 Draw Call 给GPU，Draw Call 指示 GPU 使用最近发送的 SetPass Call 提供的设置渲染网格
   3. 某些情况一个 Batch 需要多个 Pass，Pass 为 Shader 中的一段代码，一个新的 Pass 需要对 Render State 进行变更。对于 Batch 中的每一个 Pass，CPU 将发送一个新的SetPass call 并且再次发送 Draw Call  

##### GPU

1. 依据 CPU 指令顺序处理渲染
2. 若当前为 SetPass Call，则更新 Render State
3. 若为 Draw Call，GPU 依据 Shader 不同段落分步有序渲染网格

##### 线程

1. Main Thread 大部分 CPU 任务，一些渲染任务
2. Render Thread 向 GPU 发送命令
3. Worker Thread 每个线程用于独立任务如剔除操作
4. 并非所有平台都支持多线程渲染  
5. CPU 和 GPU 之间通过命令缓冲区来实现并行处理  


## 其他
### 网格 Mesh

[参考](https://zentia.github.io/2018/04/21/Unity-Mesh/)  
[参考](https://blog.csdn.net/liu_if_else/article/details/73294579)

#### Mesh组成

1. 顶点坐标 Vertices  
   顶点组成三角形，确定图形渲染位置，网格的基本组成部分  
2. 三角形序列 Triangle  
   三个顶点组成一个三角形，三个顶点一组，存入一个一维数组，顶点顺序决定z轴方向，顺指针表正面，逆时针表背面，Unity 默认只渲染正面  
3. 顶点法线 Normal  
   法线向量垂直于三角面，若一顶点有多个面，则有多个法线向量，用于计算光照等（利用光照方向向量与法线向量点积值确定光照渲染强度）  
4. 纹理坐标 uv  
   uv 数组数量必须与顶点数量一致  
   uv 为两个坐标轴，u轴水平左向右，v轴垂直上向下，一般是0-1，定义贴图每个点的位置信息
   GPU 对贴图采样，分为u*v个颜色，由各个顶点的纹理坐标决定使用贴图的哪块颜色，其余点采用插值决定  




### 参考   

[Unity基础原理 - 3D模型渲染](https://zhuanlan.zhihu.com/p/66691283)

[unity shader:渲染流程](https://blog.csdn.net/zjz520yy/article/details/77587238)

[Unity性能优化（三）-图形渲染优化](https://blog.csdn.net/qq_21397217/article/details/80401708)  

《Unity Shader入门精要》