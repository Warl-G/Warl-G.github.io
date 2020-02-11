---
layout: post
title:  "Unity学习—UGUI使用"
description: Unity UI使用方法
date:   2020-02-12 04:30:07 +0800
categories: [Unity]
tag: [学习笔记,UGUI]

---

关于Unity UGUI使用方面知识

[必读参考](http://www.arkaistudio.com/blog/2016/06/19/unity-ugui-%e5%8e%9f%e7%90%86%e7%af%87%e4%ba%94%ef%bc%9aauto-layout-%e8%87%aa%e5%8b%95%e4%bd%88%e5%b1%80/)    

本文其他地址：[简书](https://www.jianshu.com/p/b466775aa653)	[知乎](https://zhuanlan.zhihu.com/p/106366619) 	[掘金](https://juejin.im/post/5e430a53e51d4526f3639d77)  

### 画布 Canvas   

##### 渲染模式 Render Mode  

1. Screen Space - Overlay   
   
   全局覆盖，渲染在整个场景之上，自动随场景缩放，默认编辑展示大小为编辑像素值   
   
   Pixel Perfect 根据实际像素渲染，提高图像清晰  
   
   Sort Order 渲染层顺序  
   
2. Screen Space - Camera   

    仅对设定的 Render Camera 渲染，自适应填充相机视野，可将Canvas范围缩小至与场景同样大小   

    Sorting Layer 设定 Canvas 所在渲染层  

    Order in Layer 在渲染层中的顺序  

3. World Space  

     可自由调整Canvas大小位置   

     Event Camera 用于接收事件相机   

##### 缩放模式  Scale Mode  

1. Constant Pixel Size 固定像素尺寸  

   即实际屏幕像素尺寸

   Scale Factor 缩放因数控制该 Canvas 的 Rect Transform 的 Scale 属性

   Reference Pixels Per Unit 该 Canvas 映射的每Unity单位像素量，像素关系为 Sprite Pixels => Units => Canvas Pixels，普遍与 Sprite 一同设定为100像素每单位

2. Scale With Screen Size 屏幕尺寸缩放  

   Reference Resolution 设定参考分辨率，可设定为设计分辨率  

   Screen Match Mode 屏幕匹配模式   

   - Match Width or Height  依据设定权重对宽高的对数取插值得到缩放因数
   - Expand  匹配短边缩放（保持 Canvas 比例缩放至完全包含屏幕）
   - Shrink  匹配长边缩放（保持 Canvas 比例缩放至长边与屏幕相同）

3. Constant Physical Size 固定物理尺寸  

   即实际屏幕物理尺寸DPI

   Physical Unit 可选物理单位：Centimeters、Millimeters、Inches、Points、Picas

   FallBack Screen DPI 备用 DPI，无法获取屏幕信息时使用

   Default Sprite DPI 预设图片 DPI

### Rect Transform  

#### 支点 Pivot   

支点包含 x 和 y 两个0-1的值，表示支点位于2D控件的位置，控件的缩放和旋转都依据 Pivot 位置进行变化  

#### 锚点 Anchor   

控件上四个三角形，由 Min 和 Max 决定，范围由(0, 0)~(1, 1)，由父控件左下角到父控件右上角   

即锚点位置为父控件固定比例位置   

另有四个定位点确定的四边与锚点对应的四边的距离决定控件的大小与位置   

##### 锚点的使用    

以下内容详细效果可见[unity-ugui-原理篇三](http://www.arkaistudio.com/blog/2016/05/02/unity-ugui-原理篇三：recttransform/)   

锚点 Max 和 Min 的关系会改变Rect Transform的关系   

- Max X/Y 和 Min X/Y 都不同时为 Left、Top、Right、Bottom   
- 仅Max X 和 Min X 相同时为 Pos X、Top、Width、Bottom   
- 仅Max Y 和 Min Y 相同时 Left、Pos Y、Right、Height    
- Max X/Y 和 Min X/Y 都相同时为 Pos X、Pos Y、Width、Height    

Pos 为支点距离锚点的像素距离  

Width、Height 为控件宽高  

Left、Top、Right、Bottom 为控件四边与锚点四边像素距离   

可理解为相对锚点距离关系

- 四个锚点分散 上下左右**边距**约束   
- 左右锚点合并 上下**边距**约束，左右**位置**约束，宽度固定
- 上下锚点合并 左右**边距**约束，上下**位置**约束，高度固定   
- 四个锚点合并 上下左右**位置**约束，宽高固定

##### 预设锚点  Anchor Preset   

Rect Transform 左上角方形图标为预设好的的锚点布局，可设定对父层相对位置或边距

1. 锚点定位控件中心   

  ​	控件随父层大小相对位置移动  

2. 锚点定位控件四角   

  ​	大小随父层大小比例缩放  

3. 锚点定位父层四角    

  ​	控件距父层四边距离不变   

4. 锚点定位于一边   

  ​	则固定一边位置不随父层变化（如锚点固定上边，底边不随父层底边变化，另外三边保持与锚点边等间距） 

### 自动布局组件 Auto Layout Components  

Canvas 上仍可添加以下组件实现自动布局，但性能消耗相对较大，应避免使用

1. Layout Element   

  ​	自我控制最小宽高、偏好宽高、可调宽高、布局优先级  

2. Aspect Ratio Fitter   

  ​	自身比例约束宽高比或随父控件变化  

3. Layout Group   

  ​	父控件使用 Horizon/Vertical/Grid 做子空间布局   

4. Content Size Fitter   

  ​	父控件使用 随子控件大小变化  

### UI 优化
[参考](https://unity3d.com/how-to/unity-ui-optimization-tips)  

[官方](https://learn.unity.com/tutorial/optimizing-unity-ui#5c7f8528edbc2a002053b5a0)  

### 基本原理
Canvas 负责打包图形组件成批处理（batch），生成渲染命令到Unity图像系统，其过程通过C++实现，称为重批处理（rebatch）和批量构建（batch build）

当已Canvas有图形需要重组时则称该 Canvas 为脏画布（dirty），子 Canvas 将其子控件与父 Canvas 隔离，使父Canvas 不会受子 Canvas 控件脏化，反之亦然，但也有极端例子如父 Canvas 导致子 Canvas resize。Layout 组件与 Graphic 组件为独立两部分，但都依赖于*CanvasUpdateRegistry*，该类跟踪组件的Layout与Graphic并随willRenderCanvases 事件触发更新 Layout 与 Graphic 更新称为重建（rebuild），由Canvas通过透明队列绘制的图形，即与Alpha通道混合由后向前绘制，即**每个被光栅化图形像素都会被采样，无论是否被非透明图形遮挡**

### 工具
使用 Unity Profiler UI 板块查看 Batch 拆分原因（不同Material，不同Texture），合并不同Texture可使用 [精灵图集](https://www.jianshu.com/p/2640bdb29fdc)（Sprite atlases）位于同一图集内的图像会加入同一batch  

使用Unity Frame Debugger查看渲染顺序，决定调整相同材质层级，便于合并batch

### Tips
1. 使用Canvas动静分离，更新周期相近的的控件分为一Canvas，或长期不变的控件集中在一Canvas，新版本经过优化非极端条件不需要动静分离

2. 少用 Layout Element，减少 layout 层级计算时间，UI图像、文本和滚动矩形也是

   每个将其布局标记为脏化的UI元素至少会执行一次 GetComponents 调用。该调用会在 Layout Element 的父级中查找有效的布局组。如果找到，它会继续向上查找 Transform 层级，直到停止查找布局组或达到根层级（以先到为准）。因此，每个布局组会将一个 GetComponents 调用添加到每个子 Layout Element 的脏化过程，这导致嵌套不具足的性能极其低下。

3. Graphic Raycaster，关闭非交互控件的Raycast Target选项, 添加Raycast block阻挡下层事件检测
4. 避免使用 Camera.main，其方法本质为使用 Object.FindObjectWithTag，WorldSpace 需要分配 Event Camera，留空默认使用 Camera.main
5. 不要用户通过重定父级再禁用的方式集中 UI 对象，先禁用对象，然后将其父级重定到池中。  
6. 隐藏无用画布， disable Canvas Component，停止绘制，但保留批信息，不会调用 enable 方法，其他生命周期方法正常
7. 动画器每一帧都会脏化其元素，即使动画中的值不发生更改， 应对于很少发生更改或在响应事件时才会短时间发生更改的元素，请自行编写代码或补间系统
8. 禁用无效相机
9. 尽量减少 UI 层级，纹理整合入背景图像
10. 透明图像相互遮盖也会导致 batch 拆分、如文字透明区域遮盖图像，分离透明重合区域、改变视图层级同类分组
11. 使用 Sprite Atlas 精灵图集打包图像
12. 不要用 Best Fit 字体自适应
13. 在WorldSpace中 TextMeshPro 比 TextMeshProUGUI更高效

### 参考
[Sprite 与 Canvas unit 关系](https://blog.mutoo.im/2016/02/sprite-size-in-unity-gui/)  

[适配规则](https://www.jianshu.com/p/95cb4621206e)   

[ipx适配](https://zhuanlan.zhihu.com/p/35538663)   

[UI原理](http://www.arkaistudio.com/blog/2016/03/25/unity-ugui-%e5%8e%9f%e7%90%86%e7%af%87-%e4%b8%80%ef%bc%9acanvas/)  

[Canvas配置](https://www.jianshu.com/p/97286c9d588e)