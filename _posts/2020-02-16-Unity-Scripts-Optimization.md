---
layout: post
title:  "Unity学习—脚本优化Tips"
description: Unity Scripts 优化
date:   2020-02-16 02:30:07 +0800
categories: [Unity]
tag: [学习笔记,脚本,优化,Tips]

---

脚本编写优化笔记

本文其他地址：[简书](https://www.jianshu.com/p/274b3af67a4d)	[知乎](https://zhuanlan.zhihu.com/p/107176069) 	[掘金](https://juejin.im/post/5e48304551882549236f72ca)  

[官方文档参考](https://unity3d.com/cn/learn/tutorials/topics/performance-optimization/optimizing-garbage-collection-unity-games?playlist=44069)

### 代码编译原理
Unity 首先将脚本编译为中间语言 CIL (Common Intermediate Language )，CIL可再被编译为不同的原生语言。然后对不同的平台 Unity 采用不同的编译方式，分为运行前 AOT(Ahead of Time)  和运行时 JIT(Just in Time) 编译


C# ==> CIL ==> AOT(build) / JIT(device)

源代码编译为托管代码（managed code），托管代码编译为原生代码，托管运行时（managed runtime）管理整合，负责内存自动管理、安全检查等  
程序执行时，CPU在引擎代码和托管代码之间进行安全检查，数据格式转换称为编组（marshalling），存在消耗但不是特别大

### 影响代码运行原因
- 糟糕代码结构
	方法多次无效调用  
- 调用其他不必要大开销代码
	引擎代码和原生代码的相互调用
- 不必要时机调用  
	过早进行物理判断、画面判断
- 代码本身占用过高
	大量计算

### 优化方式  

##### 避免大量调用Unity API

利用profile监测游戏调试  

1. SendMessage() and BroadcastMessage()消耗大，尽量直接对象调用或者使用Events或Delegates  

2. Find()  使用引用持有、Inspector面板引用、使用专用管理类  

3. Transform  修改对象transform属性会触发子对象OnTransformChanged，子对象越多消耗越大。可将需要频繁修改的transform属性存入Vector3对象进行计算，再将Vector赋值回transform
4. Transform.position每次调用会计算一次，Transform.localPosition调用则直接取值
5. Update() 空方法也会占用CPU调用资源，所以当Update为空时直接删除
6. Vector2 and Vector3 将向量计算替换为基本类型计算，例 Vector3.magnitude、 Vector3.sqrMagnitude
7. Camera.main 同Find()

##### 条件判断前置
```c#
void Update()
{
    for (int i = 0; i < myArray.Length; i++)
    {
        if (exampleBool)
        {
            ExampleFunction(myArray[i]);
        }
    }
}
```

```c#
void Update()
{
    if (exampleBool)
    {
        for (int i = 0; i < myArray.Length; i++)
        {
            ExampleFunction(myArray[i]);
        }
    }
}
```

##### 仅在展示变量变化时调用展示代码
```c#
private int score;

public void IncrementScore(int incrementBy)
{
    score += incrementBy;
}

void Update()
{
    DisplayScore(score);
}
```

```c#
private int score;

public void IncrementScore(int incrementBy)
{
    score += incrementBy;
    DisplayScore(score);
}
```

##### 控制代码调用帧数
```c#
void Update()
{
    ExampleExpensiveFunction();
}
```

```c#
private int interval = 3;

void Update()
{
    if (Time.frameCount % interval == 0)
    {
        ExampleExpensiveFunction();
    }
    else if (Time.frameCount % interval == 1)
    {
        AnotherExampleExpensiveFunction();
    }
}
```
##### 使用缓存
```c#
void Update()
{
    Renderer myRenderer = GetComponent<Renderer>();
    ExampleFunction(myRenderer);
}
```

```c#
private Renderer myRenderer;

void Start()
{
    myRenderer = GetComponent<Renderer>();
}

void Update()
{
    ExampleFunction(myRenderer);
}
```
##### 使用适当的数据结构  
[微软官方建议](https://docs.microsoft.com/en-us/dotnet/standard/collections/index)  

##### 减少垃圾收集
使用对象池，如敌人、子弹

##### 剔除 Culling  

```c#
void Update()
{
    UpdateTransformPosition();
    UpdateAnimations();
}
```

```c#
private Renderer myRenderer;

void Start()
{
    myRenderer = GetComponent<Renderer>();
}

void Update()
{
    UpdateTransformPosition();

    if (myRenderer.isVisible)
    {
        UpateAnimations();
    }
}

```
##### Level of detail
细节等级  

## 垃圾处理优化
### 简介
Unity 核心代码为手动内存管理，开发代码自动内存管理同栈区堆区  
栈区回收立即销毁、堆区回收会再在重新分配时清空  
垃圾收集器标记和销毁堆区内存，收集器定期清理堆  

### 垃圾收集器执行时机Garbage Collector
1. 堆区无足够控件分配
2. 自动周期执行
3. 手动执行

### 垃圾收集器执行步骤
1. 检查堆区对象
2. 查找所有引用
3. 标记空闲对象
4. 还原堆空间

### 问题  
1. 堆区对象越多执行时间越长
2. 错误执行时机，如 CPU高占用时调用
3. 堆碎片化，可分配空间不足，导致收集器频繁调用

### 方案  

##### 提前缓存  

```c#
void OnTriggerEnter(Collider other)
{
    Renderer[] allRenderers = FindObjectsOfType<Renderer>();
    ExampleFunction(allRenderers);
}
```

```c#
private Renderer[] allRenderers;

void Start()
{
    allRenderers = FindObjectsOfType<Renderer>();
}


void OnTriggerEnter(Collider other)
{
    ExampleFunction(allRenderers);
}
```
##### 不在频繁调用的方法里分配  
```c#
void Update()
{
    ExampleGarbageGeneratingFunction(transform.position.x);
}
```

改为条件执行  

```c#
private float previousTransformPositionX;

void Update()
{
    float transformPositionX = transform.position.x;
    if (transformPositionX != previousTransformPositionX)
    {
        ExampleGarbageGeneratingFunction(transformPositionX);
        previousTransformPositionX = transformPositionX;
    }
}
```

或延迟执行  

```c#
private float timeSinceLastCalled;

private float delay = 1f;

void Update()
{
    timeSinceLastCalled += Time.deltaTime;
    if (timeSinceLastCalled > delay)
    {
        ExampleGarbageGeneratingFunction();
        timeSinceLastCalled = 0f;
    }
}
```

或集合再分配

```c#
void Update()
{
    List myList = new List();
    PopulateList(myList);
}
```

```c#
private List myList = new List();
void Update()
{
    myList.Clear();
    PopulateList(myList);
}
```
使用对象池

### 常见原因
1. String 分配，没改变一次String就重新创建一个，删除原有String  
	* 使用String 缓存
	* text组件使用 拆分string变化和非变部分
	* System.Text.StringBuilder 不会分配空间
	* 移除Debug.Log()     
	
  ```c#
	public Text timerText;
	private float timer;
	
	void Update()
	{
	    timer += Time.deltaTime;
	    timerText.text = "TIME:" + timer.ToString();
	}
  ```


  ```c#
	public Text timerHeaderText;
	public Text timerValueText;
	private float timer;
	
	void Start()
	{
	    timerHeaderText.text = "TIME:";
	}
	
	void Update()
	{
	    timerValueText.text = timer.toString();
	}
  ```

  DEBUG log    

  ```
	#if … #endif
  ```


   ```c#
   public static class Logger {
   
	  [Conditional("ENABLE_LOGS")]

	     public static void Debug(string logMsg) {
	
	         UnityEngine.Debug.Log(logMsg);
	
	     }
	
	 }
   ```

2. Unity Function  

  方法调用时产生局部缓存   


  ```c#
  void ExampleFunction()
  {
      Vector3[] meshNormals = myMesh.normals;
      for (int i = 0; i < meshNormals.Length; i++)
      {
          Vector3 normal = meshNormals[i];
      }
  }
  ```

  GameObject.name or GameObject.tag  比较    


  ```c#
  private string playerTag = "Player";
  
  void OnTriggerEnter(Collider other)
  {
      bool isPlayer = other.gameObject.CompareTag(playerTag);
  }
  ```

  可使用 Input.GetTouch() 和 Input.touchCount 替代 Input.touches 或者用 Physics.SphereCastNonAlloc() 替代 Physics.SphereCastAll()

3. 值类型装箱  

4. Coroutines  

	```c#
	//装箱了0 产生内存垃圾
	yield return 0;
	yield return null;
	
	//每次循环产生一次垃圾
	while (!isComplete)
	{
	    yield return new WaitForSeconds(1f);
	}
	
	WaitForSeconds delay = new WaitForSeconds(1f);
	
	while (!isComplete)
	{
	    yield return delay;
	}
	```
	
5. foreach   
	
	```c#
	void ExampleFunction(List listOfInts)
	{
	    foreach (int currentInt in listOfInts)
	    {
	            DoSomething(currentInt);
	    }
	}
	```
	
	```c#
	void ExampleFunction(List listOfInts)
	{
	    for (int i = 0; i < listOfInts.Count; i ++)
	    {
	        int currentInt = listOfInts[i];
	        DoSomething(currentInt);
	    }
	}
	```
	
6. 函数引用

7. LINQ and Regular Expressions


### 代码结构  

##### 结构体  

结构体为值类型，包含引用类型变量会导致收集器检查真个结构体，占用更多时间  

```c#
public struct ItemData
{
    public string name;
    public int cost;
    public Vector3 position;
}
private ItemData[] itemData;
```

```c#
private string[] itemNames;
private int[] itemCosts;
private Vector3[] itemPositions;
```

##### 引用查找&角标查找  

​	return id更省

##### 手动回收  

```c#
System.GC.Collect();
```