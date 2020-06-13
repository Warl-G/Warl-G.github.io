---
layout: post
title:  "Unity学习—JobSystem"
description: Unity 多线程 JobSystem 简述
date:   2020-06-14 01:00:07 +0800
categories: [Unity]
tag: [学习笔记,多线程,JobSystem]

---

译自[官方手册](https://docs.unity3d.com/Manual/JobSystem.html)，简述 Unity 另一个多线程接口，JobSystem，为 Unity ECS 系统的主要实现方式

本文其他地址：[简书](https://www.jianshu.com/p/6392a1bea139)	[知乎](https://zhuanlan.zhihu.com/p/148160780) 	[掘金](https://juejin.im/post/5ee503b6518825431437699e)  

## JobSystem

JobSystem 管理一组多核中的工作线程(Work Thread)，为避免上下文切换通常一个逻辑核配一个工作线程   

JobSystem 持有一个 Job 队列，工作线程从该队列中获取 Job 执行  

Job 是执行特定任务的小工作单元，Job 可以互相依赖   

### 线程安全  

JobSystem 执行时复制而非引用数据，避免了数据竞争，但 JobSystem 只能使用`memcpy`复制 [blittable types](https://docs.microsoft.com/en-us/dotnet/framework/interop/blittable-and-non-blittable-types) 数据。`Blittable types` 是 .Net 框架中的数据类型，该类型数据在托管代码与原生代码间传递无需转换  

### NativeContainer

复制数据来保证线程安全的弊端就是任务的结果也是独立的，因此使用`NativeContainer`将结果储存在公共内存中  

`NativeContainer`以相对安全的托管类型的方式指向一个非托管的内存地址，使Job 可以直接访问主线程数据而非复制  

Unity 自带 `NativeContainer`类型为 `NativeArray`，ECS 包又扩展了`NativeList`、`NativeHashMap`、`NativeMultiHashMap`和`NativeQueue`   

默认情况下，Job 同时拥有`NativeContainer`的读写权限，但 C# Job System 不允许多个 Job 同时拥有对一个`NativeContainer`的写权限，因此对不需要写权限的`NativeContainer`加上`[ReadOnly]`特性，以减少性能影响  

```c#
[ReadOnly]
public NativeArray<int> input;
```

JobSystem 支持多个 Job 同时读取同一数据  

#### NativeContainer Allocator

根据 Job 执行时长决定使用哪种 Allocator   

* Allocator.Temp  

  最快的分配方法，适用于一帧或几帧的生命时长，不能将该类型分配的数据传给 Job，在方法 Return 前执行`Dispose`

* Allocator.TempJob 

  分配速度比 Temp 慢比 Persistent 快，4帧的生命时长且线程安全。若四帧内没有调用`Dispose`，控制台会打印原生代码生成的警告。大部分小任务都使用该类型分配`NativeContainer`

* Allocator.Persistent  

  是对`malloc`的包装，能够维持尽可能地生命时长，性能不足的情况下不应使用

```c#
NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);
```

### 创建 Job

1. 声明实现 IJob 接口的结构体
2. 添加`blittable types`或 `NativeContainer`类型的成员变量
3. 实现`Execute`方法  

当 Job 执行时，`Execute`在一个核上执行一次  

```c#
public struct MyJob : IJob
{
    public float a;
    public float b;
    public NativeArray<float> result;

    public void Execute()
    {
        result[0] = a + b;
    }
}
```

### 调度 Job

1. 创建 Job
2. 填充 Job 数据 
3. 调用`Schedule`方法  

 只能在主线程调用`Schedule`方法，将 Job 放入队列等待执行，一旦 Job 被调度进队列旧无法中断  

```c#
NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);

// 填充数据
MyJob jobData = new MyJob();
jobData.a = 10;
jobData.b = 10;
jobData.result = result;

// 调度 Job
JobHandle handle = jobData.Schedule();

// 等待完成
handle.Complete();

// 所有 NativeArray 指向同一内存，可外部访问
float aPlusB = result[0];

// 释放 result array
result.Dispose();
```

### JobHandle 和 dependencies

JobHandle 是在调用`Schedule`返回的句柄，可使用该句柄作为参数传入另一个 Job 的`Schedule`作为依赖，使后者等待前者执行完成再执行

```c#
JobHandle firstJobHandle = firstJob.Schedule();
secondJob.Schedule(firstJobHandle);
```

对于多个依赖的 Job 可使用`JobHandle.CombineDependencies`组合这些 JobHandle  

```c#
NativeArray<JobHandle> handles = new NativeArray<JobHandle>(numJobs, Allocator.TempJob);
// 填充 handles
JobHandle jh = JobHandle.CombineDependencies(handles);
```

调用 JobHandle 的`Complete`方法可使主线程等待任务执行完成以安全访问该 Job 使用的 `NativeContainer`，该方法会从内存中刷新 Job 并开始执行然后将该 Job 中的`NativeContainer`持有权返回主线程     

若不需访问数据，但需要立即刷新执行 Job 缓存，则可以使用`JobHandle.ScheduleBatchedJobs`，但会影响性能

```c#
public struct MyJob : IJob
{
    public float a;
    public float b;
    public NativeArray<float> result;

    public void Execute()
    {
        result[0] = a + b;
    }
}

// Job adding one to a value
public struct AddOneJob : IJob
{
    public NativeArray<float> result;
    
    public void Execute()
    {
        result[0] = result[0] + 1;
    }
}

NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);

// Setup the data for job #1
MyJob jobData = new MyJob();
jobData.a = 10;
jobData.b = 10;
jobData.result = result;

// Schedule job #1
JobHandle firstHandle = jobData.Schedule();

// Setup the data for job #2
AddOneJob incJobData = new AddOneJob();
incJobData.result = result;

// Schedule job #2
JobHandle secondHandle = incJobData.Schedule(firstHandle);

// Wait for job #2 to complete
secondHandle.Complete();
// All copies of the NativeArray point to the same memory, you can access the result in "your" copy of the NativeArray
float aPlusB = result[0];

// Free the memory allocated by the result array
result.Dispose();
```

### IJobParallelFor

对于`IJob`，同一时间一个 Job 只能执行一个任务，若想同一时间执行多个相同的任务，则可以使用`IJobParallelFor`  

一种使用场景为 ParallelFor Job 在多核上同时对同一 NativeArray 进行操作，每核仅负责部分工作，ParallelFor Job 的`Execute`方法会传入index，用于访问数据源

```c#
struct IncrementByDeltaTimeJob: IJobParallelFor
{
    public NativeArray<float> values;
    public float deltaTime;

    public void Execute (int index)
    {
        float temp = values[index];
        temp += deltaTime;
        values[index] = temp;
    }
}
```

在调度 ParallelFor Job 时需规定调度任务总长度和每批次长度，C# Job System 会根据批次长度将任务总长分批，再放入 Unity Job 队列，每批同步执行，每批次任务内仅执行一个 Job  

当一个 Native Job 先完成时，它会“窃取”其他 Job 的一半批任务，既优化了性能，又保证了内存访问局部性  

批次数越低，线程间的任务分配越均匀，但也会带来额外开销，因此需要逐一测试出最佳性能的批次数

![](https://docs.unity3d.com/uploads/Main/jobsystem_parallelfor_job_batches.svg)



```c#
public struct MyParallelJob : IJobParallelFor
{
    [ReadOnly]
    public NativeArray<float> a;
    [ReadOnly]
    public NativeArray<float> b;
    public NativeArray<float> result;

    public void Execute(int i)
    {
        result[i] = a[i] + b[i];
    }
}

NativeArray<float> a = new NativeArray<float>(2, Allocator.TempJob);

NativeArray<float> b = new NativeArray<float>(2, Allocator.TempJob);

NativeArray<float> result = new NativeArray<float>(2, Allocator.TempJob);

a[0] = 1.1;
b[0] = 2.2;
a[1] = 3.3;
b[1] = 4.4;

MyParallelJob jobData = new MyParallelJob();
jobData.a = a;  
jobData.b = b;
jobData.result = result;

// Schedule the job with one Execute per index in the results array and only 1 item per processing batch
JobHandle handle = jobData.Schedule(result.Length, 1);

// Wait for the job to complete
handle.Complete();

// Free the memory allocated by the arrays
a.Dispose();
b.Dispose();
result.Dispose();
```

#### ParallelForTransform

专门用于操作 Transform 的 Parallel Job  

### 注意事项  

* 不要使用 Job 访问静态数据  

  从 Job 访问静态数据会绕开所有安全系统，可能会导致 Unity 崩溃

* 使用  [JobHandle.ScheduleBatchedJobs](https://docs.unity3d.com/ScriptReference/Unity.Jobs.JobHandle.ScheduleBatchedJobs.html) 方法立即执行已调度的 Job

  Job 在被调度后会被缓存不会立即执行，该方法可立即清空缓存队列中的 Job 并执行，但会影响性能，或调用  [JobHandle.Complete](https://docs.unity3d.com/ScriptReference/Unity.Jobs.JobHandle.Complete.html) 执行，ECS 系统已经隐式清空了缓存，以你无需主动调用 

* 不要更新 NativeContainer 内容  

  由于`ref returns`的缺陷，无法直接修改 NativeContainer 中的内容，需按如下方式 

  ```c#
  MyStruct temp = myNativeArray[i];
  temp.memberVariable = 0;
  myNativeArray[i] = temp;
  ```

* 调用 JobHandle.Complete 重获所有权  

  在主线程或新的 Job 使用前一 Job 占用的`NativeContainer`数据前，必须调用`JobHandle.Complete`重新获取其所有权，该方法会清空安全机制的状态，否则会导致内存泄漏（ 不能仅查看[JobHandle.IsCompleted](https://docs.unity3d.com/ScriptReference/Unity.Jobs.JobHandle.IsCompleted.html)状态）

* 只能在主线程调用 Schedule 和 Complete 方法  

  这两种方法只能在主线程调用，若一个 Job 依赖于另一个 Job，则在主线程使用 JobHandle  

* Schedule 和 Complete 的正确时机 

  准备数据完成时即可调用 Schedule，仅当需要结果时才调用 Complete，如在一帧的结尾与下一帧开始的空档中调度一个 Job  

* 使用 [ReadOnly] 标记 NativeContainer

  Job 同时用于对 NativeContainer 的读写权限，使用 [ReadOnly] 标记只读 Job 中的 NativeContainer 可提升效率  

* 检查数据依赖  

  在 Profiler 窗口中，主线程上的 `WaitForJobGroup` 标记表明 Unity 在等待一个工作线程的任务完成，该标记可能意味着需要解决的数据依赖，可通过查找`JobHandle.Complete`找到这些依赖  

* Debugging jobs  

  可以调用`Run`方法取代`Schedule`在主线程执行 Job

* 不要在 Job 中分配托管内存  

  在 Job 中分配托管内存会非常慢，且无法使用 Burst 编译提升效率