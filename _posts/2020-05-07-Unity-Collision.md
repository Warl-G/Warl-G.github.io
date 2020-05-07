---
layout: post
title:  "Unity手册-碰撞检测相关汇总"
description: Unity 碰撞检测相关功能、API与设置汇总
date:   2020-05-07 22:30:07 +0800
categories: [Unity]
tag: [参考手册]

---

本文汇总了用于碰撞检测的方法和设置  

本文其他地址：[简书](https://www.jianshu.com/p/5802f2432d3b)	[知乎](https://zhuanlan.zhihu.com/p/138853762) 	[掘金](https://juejin.im/post/5eb41672e51d454dcd0bf6da)  

## 碰撞组件

### 刚体

两物体若要发生碰撞事件，则两物体必须都带有碰撞体 Collider 组件且运动的物体带有刚体 Rigidbody 组件  

以下为 Rigidbody 2D 或 Rigidbody Inspector 上的部分参数说明     

| Inspector 面板属性  | 属性值                 | 说明                                                         |
| ------------------- | ---------------------- | ------------------------------------------------------------ |
| BodyType            |                        | 刚体类型，决定刚体的移动旋转和碰撞方式，不要在运行时修改     |
|                     | Dynamic                | 动态类型，该类型物体依据物理模拟移动，与所有 Rigidbody2D 碰撞，最消耗性能，不要使用 Transform 设置刚体的 Position 和 Rotation |
|                     | Kinematic              | 运动学类型，该类型物体依据物理模拟移动但不受力的作用，仅与动态类型刚体碰撞，比动态类型节约性能，通过 Rigidbody2D.MovePosition 或 Rigidbody2D.MoveRotation 控制空间状态 |
|                     | Static                 | 静态类型，不依据物理模拟移动，仅与动态类型刚体碰撞           |
| Material            |                        | 物理材质                                                     |
| Simulated           |                        | 模拟选项                                                     |
| Collision Detection |                        | 碰撞器检测，3D Collider 仅 Sphere、Capsule 和 Box 支持连续检测 |
|                     | Discrete               | 离散检测，仅在每个 FixUpdate 时机检测                        |
|                     | Continuous             | 连续检测，持续检测碰撞，可避免碰撞体重叠或穿过，适用于被高速移动物体碰撞的物体，更消耗性能 |
|                     | Continuous Dynamic     | 连续动态检测，适用快速移动物体，更消耗性能                   |
|                     | Continuous Speculative | 连续推测检测，比以上两种连续检测更节约性能，且可用于 Kinematic 物体，处理角度运动更有优势，但依然有可能穿过高速移动物体 |

#### API  

Rigidbody2D 还有配合碰撞体使用的 API

| 接口             | 说明                                                 |
| ---------------- | ---------------------------------------------------- |
| Cast             | 以当前刚体上所有碰撞的体形状做投影，获取所有碰撞结果 |
| ClosestPoint     | 获取当前刚体上所有碰撞体周边距指定点最近的点         |
| Distance         | 获取当前刚体上所有碰撞体与指定碰撞体的最近距离       |
| GetContacts      | 获取当前刚体上所有碰撞体与其他碰撞体的接触点         |
| IsTouching       | 指定碰撞体是否与刚体上任一碰撞体接触                 |
| IsTouchingLayers | 指定 LayerMask 是否与刚体上任一碰撞体接触            |
| OverlapCollider  | 获取与刚体上碰撞体重叠的碰撞体                       |
| OverlapPoint     | 指定点是否与刚体上碰撞体重叠                         |



### 碰撞器和触发器

最常用的碰撞检测方式为 GameObject 添加碰撞器 Collider 组件或勾选 Collider 中的 `Is Trigger`选项将碰撞器设为触发器  

碰撞事件也会发送给禁用的 MonoBehavior 对象

#### [Collider2D](https://docs.unity3d.com/ScriptReference/Collider2D.html)

2D 碰撞器组件有以下六种   

* BoxCollider2D
* CapsuleCollider2D
* CircleCollider2D
* CompositeCollider2D
* EdgeCollider2D
* PolygonCollider2D
* TilemapCollider2D

##### 生命周期碰撞事件回调

* MonoBehaviour.OnCollisionEnter2D(Collider2D)
* MonoBehaviour.OnCollisionStay2D(Collider2D)
* MonoBehaviour.OnCollisionExit2D(Collider2D)  
* MonoBehaviour.OnTriggerEnter2D(Collider2D)
* MonoBehaviour.OnTriggerStay2D(Collider2D)
* MonoBehaviour.OnTriggerExit2D(Collider2D)  

##### 其他方法  

| 方法             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| Cast             | 依据当前 Collider2D 的形状向指定方向和距离做投影，返回投影路径上碰撞结果 |
| ClosestPoint     | 找到当前 Collider2D 周边距离指定点最近的位置                 |
| Distance         | 计算两 Collider2D 最短距离                                   |
| GetContacts      | 获取当前碰撞体所有接触点                                     |
| IsTouching       | 当前是否碰撞指定 Collider2D                                  |
| IsTouchingLayers | 当前是否碰撞指定 LayerMask                                   |
| OverlapCollider  | 获取当前 Collider2D 所有重叠的 Collider2D，可添加 LayerMask、Z 轴深度等筛选条件 |
| OverlapPoint     | 判断当前 Collider2D 是否与指定二维点重叠                     |
| Raycast          | 由当前Collider2D 位置发射出一条指定方向和距离的射线，返回射线碰撞结果（不包含自身），可添加 LayerMask、Z 轴深度等筛选条件 |

### [Collider](https://docs.unity3d.com/ScriptReference/Collider.html)

Collider 方法与 Collider2D 基本相同 

3D 碰撞器组件有以下几种：

* BoxCollider
* CapsuleCollider
* MeshCollider
* SphereCollider
* TerrainCollider
* WheelCollider  

##### 生命周期碰撞事件回调

* MonoBehaviour.OnCollisionEnter(Collider)
* MonoBehaviour.OnCollisionStay(Collider)
* MonoBehaviour.OnCollisionExit(Collider)  
* MonoBehaviour.OnTriggerEnter(Collider)
* MonoBehaviour.OnTriggerStay(Collider)
* MonoBehaviour.OnTriggerExit(Collider)  

## 物理材质  

物理材质 Physic Material 用于设置碰撞体摩擦力和弹力  

| 属性             | 值       | 说明                     |
| ---------------- | -------- | ------------------------ |
| Dynamic Friction | 0-1      | 动摩擦力                 |
| Static Friction  | 0-1      | 静摩擦力                 |
| Bounciness       | 0-1      | 弹性                     |
| Friction Combine |          | 碰撞摩擦力混合方式       |
|                  | Maximum  | 取两碰撞物体摩擦力最大值 |
|                  | Multiply | 取两碰撞物体摩擦力乘积   |
|                  | Minimum  | 取两碰撞物体摩擦力最小值 |
|                  | Average  | 取两碰撞物体摩擦力平均值 |
| Bounce Combine   |          | 同 Friction Combine      |



## 其他 API

代码做碰撞检测的方法有很多种，主要分为形状投影，射线投影，形状重叠

### Physics2D

[官方接口文档](https://docs.unity3d.com/ScriptReference/Physics2D.html)    

Physics2D 还提供许多静态接口用来判断物理碰撞

| 接口                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| BoxCast              | 另有`BoxCastAll` 和 `BoxCastNonAlloc`，投影一个盒形，返回一个或多个碰撞结果 |
| CapsuleCast          | 另有`CapsuleCastAll` 和 `CapsuleCastNonAlloc`，投影一个胶囊形，返回一个或多个碰撞结果 |
| CircleCast           | 另有`CircleCastAll` 和 `CircleCastNonAlloc`，投影一个圆形，返回一个或多个碰撞结果 |
| ClosestPoint         | 返回指定碰撞体周边距指定点最近的点                           |
| Distance             | 计算两碰撞体最近距离                                         |
| GetContacts          | 获取指定碰撞体所有接触的碰撞体                               |
| GetRayIntersection   | 另有`GetRayIntersectionAll` 和 `GetRayIntersectionNonAlloc`，投射一条 3D 射线，返回一个或多个碰撞结果 |
| IgnoreCollision      | 使物理系统忽略两碰撞体之间碰撞和触发效果                     |
| IgnoreLayerCollision | 使物理系统忽略两 Layer 之间碰撞效果                          |
| IsTouching           | 判断两个碰撞体是否接触                                       |
| IsTouchingLayers     | 判断指定碰撞体是否与指定层 LayerMask 的物体接触              |
| Linecast             | 另有`LinecastAll` 和 `LinecastNonAlloc`，投射一条线段，返回一个或多个碰撞结果 |
| OverlapArea          | 另有`OverlapAreaAll` 和 `OverlapAreaNonAlloc`，获取一个或多个与指定对角两点确定的矩形区域重叠的碰撞体 |
| OverlapBox           | 另有`OverlapBoxAll` 和 `OverlapBoxNonAlloc`，获取一个或多个与指定盒形区域重叠的碰撞体 |
| OverlapCapsule       | 另有`OverlapCapsuleAll` 和 `OverlapCapsuleNonAlloc`，获取一个或多个与指定胶囊形区域重叠的碰撞体 |
| OverlapCircle        | 另有`OverlapCircleAll` 和 `OverlapCircleNonAlloc`，获取一个或多个与指定圆形区域重叠的碰撞体 |
| OverlapCollider      | 获取与指定碰撞体重叠的碰撞体                                 |
| OverlapPoint         | 另有`OverlapPointAll` 和 `OverlapPointNonAlloc`，获取一个或多个与指定点重叠的碰撞体 |
| Raycast              | 另有 `RaycastAll`和`RaycastNonAlloc`，投射一条射线，返回一个或多个碰撞结果 |

### Physics

[官方接口文档](https://docs.unity3d.com/ScriptReference/Physics.html)   

Physics 接口与 Physics2D 接口大致相同，另有如下几个接口  

| 接口         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| CheckBox     | 是否有碰撞体与指定盒体区域与重叠                             |
| CheckCapsule | 是否有碰撞体与指定胶囊体区域与重叠                           |
| CheckSphere  | 是否有碰撞体与指定球体区域与重叠                             |
| ClosestPoint | 获取指定碰撞体上与指定坐标最近的点                           |
| SphereCast   | 另有 `SphereCastAll`和`SphereCastNonAlloc`，延指定射线投射一个指定半径的球体，返回一个或多个球体碰撞结果 |

### Ray

许多碰撞可用 Raycast 检测，创建射线的方法除 Ray 本身的构造方法以外，还可由相机产生

* Camera.ScreenPointToRay(Vector3)
* Camera.ViewportPointToRay(Vector3)

## Physics Setting

除经常使用的物理系统和 API 外，还可在`Editor > Project Settings > Physics(2D)`中设置一些 [3D](https://docs.unity3d.com/Manual/class-PhysicsManager.html)/[2D](https://docs.unity3d.com/Manual/class-Physics2DManager.html) 物理系统参数  

### Physics 2D Settings

| 设置                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Default Material           | 默认物理材质                                                 |
| Velocity Threshold         | 速度阈值，对相对速度低于该值的弹性碰撞会被处理为非弹性碰撞   |
| Baumgarte Scale            | Baumgarte 比例，决定 Unity 处理碰撞重叠的速度                |
| Default Contact Offset     | 触发接触距离的近似值，当碰撞体小于该值时，则触发碰撞，该值过小会减弱Unity计算多边形碰撞的能力，过大会造成假的顶点碰撞 |
| Queries Hit Triggers       | 使触发器响应物理检索系统，如 Linecasts、Raycasts 等，默认开启 |
| Queries Start In Colliders | 物理检索系统检索其开始的碰撞体                               |
| Layer Collision Matrix     | 碰撞矩阵，决定哪些 Layer 之间可碰撞                          |

### Physics Settings

除包含上述设置外，3D 还有如下碰撞相关设置

| 设置                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Bounce Threshold          | 速度阈值，对相对速度低于该值的弹性碰撞会被处理为非弹性碰撞   |
| Queries Hit Backfaces     | 使物理检索系统检测网格碰撞器的三角形背面，默认关闭           |
| Contacts Generation       | 接触产生方法，分 SAT(Legacy) 和 PCM 两种，Unity 5.5以前使用 SAT；PCM 则更有效率，产生稍有不同的反弹和减少了不必要的接触缓存，默认为 PCM |
| Contact Pairs Mode        | - Default Contact Pairs: 接收除 kinematic-kinematic 和 kinematic-static 外的所有碰撞器触发器事件  <br />- Enable Kinematic Kinematic Pairs: 接收 kinematic-kinematic 的碰撞和触发器事件<br />- Enable Kinematic Static Pairs: 接收 kinematic-static 的碰撞和触发器事件<br />- Enable All Contact Pairs: 接收所有碰撞器触发器事件 |
| Enable Unified Heightmaps | 使用与处理网格（Mesh）碰撞相同的方式处理地形（Terrain）碰撞  |
| Cloth Inter-Collision     | 布碰撞<br />-Distance: 定义每个布片的碰撞半径，避免过大导致抖动<br />-Stiffness: 定义布片间的刚性 |