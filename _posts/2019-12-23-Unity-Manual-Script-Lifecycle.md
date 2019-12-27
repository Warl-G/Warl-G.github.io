---
layout: post
title:  "Unity手册—脚本生命周期"
description: Unity脚本生命周期参考手册
date:   2019-12-23 12:33:07 +0800
categories: [Unity]
tag: [参考手册, 不定期更新]

---

翻译官方生命周期手册，用于开发参考，确认周期方法调用时间和调用频率，配合官网生命周期图使用风味更佳

本文其他地址： [简书](https://www.jianshu.com/p/995a7306caaf)   [知乎](https://zhuanlan.zhihu.com/p/98922753)  [掘金](https://juejin.im/post/5e007e6ef265da33ec7dbff1)

引用版本：[Unity 官方手册 ver. 2020.1](https://docs.unity3d.com/2020.1/Documentation/Manual/ExecutionOrder.html)

## 脚本周期
1. 初始化阶段   
   场景开始时
    1. Awake  
        场景创建时或prefab实例化时，若为inactive则在active时调用，仅执行一次 
    2. OnEnable   
        场景加载完，GameObject实例化时，对象enabled时
    3. OnLevelWasLoaded  
        场景全部加载完成后
    4. Reset 
        Editor级别，非playmode下脚本挂载时或主动调用
    5. Start  
        仅在场景中所有Awake 和 OnEnable执行后，所有场景中对象第一帧update之前，且仅执行一次  
1. 物理计算阶段   
    1. fixedupdate  
        每帧根据帧率高低可能多次调用也可能不调用，执行后立即开始物理计算和更新，做移动计算时无需使用Time.deltaTime
    2. 状态机周期  
        1. 状态机更新  
        2. OnStateMachineEnter   
                  挂载在一个动画图形上的状态机首次进入一个状态时的第一帧调用  
        4. OnStateMachineExit  
                 退出状态的最后一帧调用  
        4. 处理图形   
                评估所有动画图形  
        5. 触发动画事件  
                在该时间内当前帧和最后一帧之间，触发所有动画片段的动画事件
        6. 状态周期回调（OnStateEnter/OnStateUpdate/OnStateExit）
                一个状态层有最多三个活跃状态，current state, interrupted state, and next state，对应阶段周期方法顺序执行    
    3. 内部物理更新  
    4. 状态机周期  
        1. 处理动画  
        渲染动画结果图形
        2. IK动画周期  
        由workthread写入Transform
        7. 写入属性  
        由主线程写入场景其他动画属性
          4. 碰撞事件（Collision、Trigger）
          5. 协程（yield WaitForFixedUpdate）  
          在所有脚本的FixedUpdated执行后再执行
1. 输入事件   		
    1. OnMouseXXX 		
1. 游戏逻辑  
    1. update
    3. 协程  
        除WaitForFixedUpdate和WaitForEndOfFrame其他协程时机
    4. 状态机周期  
        同物理计算阶段过程，单无内部物理更新
    5. lateupdate
        	开始时update的计算都已完成，一普遍用法是相机跟随，update中对人物移动的计算已全部完成，此时在lateupdate中更新相机位置 
1. 场景渲染   
    1. OnWillRenderObject  
       若物体可见则每个相机调用一次  
   2. OnPreCull  
       相机剔除动作前
   5. OnBecameVisible/OnBecameInvisible  
       对任何相机可见/不可见时调用
    4. OnPreRender  
         相机渲染前
    5. OnRenderObject  
         所有常规场景渲染完成时，可在此绘制自定义几何图形
    6. OnPostRender  
         当一个相机渲染完场景后
    7. OnRenderImage  
         场景渲染后，可对Image做后期处理（滤镜）
8. Gizmo 渲染  
    1. OnDrawGizmos  
        仅在editor
9. GUI渲染
    1. OnGUI
        每帧多次调用 
10. 帧结束
    1. yield WaitForEndOfFrame  
11. 暂停  
    1. OnApplicationPause  
       一帧最后时调用，调用后会再触发一帧以刷新图像和切换暂停状态
12. 退出/销毁
    1. OnApplicationQuit  
    2. OnDisable  
    3. OnDestroy  
       所有帧刷新后  

![](https://docs.unity3d.com/2020.1/Documentation/uploads/Main/monobehaviour_flowchart.svg)