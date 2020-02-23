---
layout: post
title:  "Unity实践—多线程任务队列实现"
description: 实现一个用于 Unity 的多线程任务队列
date:   2020-02-24 4:30:07 +0800
categories: [Unity,C#]
tag: [实践,Task,队列,多线程]
---

Unity 已可使用 Thread、Task 等处理多线程任务，但缺少成熟的多线程任务队列工具，所以在此实现一个，代码已上传 Git 项目  [GRUnityTools](https://github.com/Warl-G/GRUnityTools.git)，可直接下载源码或通过 UPM 使用  

本文其他地址：[简书](https://www.jianshu.com/p/6048924fcf16)	[知乎](https://zhuanlan.zhihu.com/p/108730944)	[掘金](https://juejin.im/post/5e52dcf5518825495b29958f)

### 实现目标    

1. 串行与并发队列  

   队列是首要实现目标，且需要串行与并发两种队列，以覆盖不同需求

2. 同步与异步执行 

   因任务队列过多可能阻塞主线程，所以除同步执行外还需要多线程异步操作  

3. 主线程同步  

   因为有多线程，但 Unity 部分操作只能在主线程执行，所以还需要线程同步到主线程

### 实现方式  

1. Task   

   Task 为当前 .Net 提供的实用性最高的多线程接口，可实现任务的监控与操纵  

2. TaskScheduler   

   Task 专用调度器，可更便捷地实现 Task 队列调度  

3. Loom  

   Loom 为网络上广为流传的 Unity 中调用主线程的工具类，目前找不到源码最原始地址，代码拷贝自[知乎](https://zhuanlan.zhihu.com/p/23986194)

### 实现过程   

#### 方案选择

最初即决定使用 Task 作为队列基本单位，但完全没有考虑 `TaskScheduler`。原计划手动实现一个调度器，负责保存传入的 Task 放入队列，可设置同步异步，根据设置实现对队列的不同操作。后来再研究微软官方文档时发现在其 Task 文档的示例中有一个 `LimitedConcurrencyLevelTaskScheduler` 的演示代码，直接通过 `TaskScheduler` 实现了可控并发数量的调度器，且当设置并发数为1时队列中的任务会逐一按顺序执行即产生了串行队列效果   

`TaskScheduler` 有两种使用方式  

方式一：为 `TaskFactory` 配置 TaskScheduler，通过 `TaksFactory` 使用配置的调度器启动 Task

```c#
//创建并发数32的调度器
LimitedConcurrencyLevelTaskScheduler scheduler = new LimitedConcurrencyLevelTaskScheduler(32); 
//方式1 
TaskFactory factory = new TaskFactory(scheduler);
factory.StartNew(()=>{
  //执行任务
});
```

方式二：直接使用 `Task.Start(TaskFactory)` 方法  

```c#
//创建并发数1的调度器（此时即为串行队列效果）
LimitedConcurrencyLevelTaskScheduler scheduler = new LimitedConcurrencyLevelTaskScheduler(1);
//声明一个 Task 对象
Task task = new Task(()=>{
  //任务
});
//启动 Task 指定调度器
task.Start(scheduler);
```

#### 编写源码  

创建名为 `TaskQueue` 的类，添加变量  

```c#
//根据需求设置默认并发数
private const int DefaultConcurrentCount = 32;
//线程锁
private static object _lock = new object();
//默认静态串行队列对象
private static TaskQueue _defaultSerial;
//默认静态并发队列对象
private static TaskQueue _defaultConcurrent;
//持有的调度器
private LimitedConcurrencyLevelTaskScheduler _scheduler;  

//提供默认串行队列
public static TaskQueue DefaultSerailQueue
{
    get
    {
        if (_defaultSerial == null)
        {
            lock (_lock)
            {
                if (_defaultSerial == null)
                {
                    _defaultSerial = new TaskQueue(1);
                }
            }
        }
        return _defaultSerial;
    }
}

//提供默认并发队列
public static TaskQueue DefaultConcurrentQueue
{
    get
    {
        if (_defaultConcurrent == null)
        {
            lock (_lock)
            {
                if (_defaultConcurrent == null)
                {
                    _defaultConcurrent = new TaskQueue(DefaultConcurrentCount);
                }
            }
        }
        return _defaultConcurrent;
    }
}
```

提供快捷构造方法  

```c#
//默认构造方法，因 Loom 为 UnityEngine.Monobehaviour对象，所以必须执行初始化方法将其加入场景中
public TaskQueue(int concurrentCount)
{
    _scheduler = new LimitedConcurrencyLevelTaskScheduler(concurrentCount);
    Loom.Initialize();
}
//创建串行队列
public static TaskQueue CreateSerialQueue()
{
    return new TaskQueue(1);
}
//创建并发队列
public static TaskQueue CreateConcurrentQueue()
{
    return new TaskQueue(DefaultConcurrentCount);
}
```

下面是各种同步、异步、主线程执行方法，方法会将执行的 Task 返回，以便在实际使用中需要对 Task 有其他操作   

需注意 `RunSyncOnMainThread` 和 `RunAsyncOnMainThread` 为独立的执行系统与队列无关，若在主线程执行方法直接在主线程同步执行即可

```c#
//异步执行，因Task本身会开启新线程所以直接调用即可
public Task RunAsync(Action action)
{
    Task t = new Task(action);
    t.Start(_scheduler);
    return t;
}

//同步执行，使用 Task 提供的 RunSynchronously 方法
public Task RunSync(Action action)
{
    Task t = new Task(action);
    t.RunSynchronously(_scheduler);
    return t;
}

//同步执行主线程方法
//为避免主线程调用该方法所以需要判断当前线程，若为主线程则直接执行，防止死锁  
//为保证线程同步，此处使用信号量，仅在主线程方法执行完成后才会释放信号
public static void RunSyncOnMainThread(Action action)
{
    if (Thread.CurrentThread.ManagedThreadId == 1)
    {
        action();
    }
    else
    {
        Semaphore sem = new Semaphore(0, 1);
        Loom.QueueOnMainThread((o => { 
            action();
            sem.Release();
        }), null);
        sem.WaitOne();
    }
}

//因 Loom 本身即为不会立即执行方法，所以直接调用即可
public static void RunAsyncOnMainThread(Action action)
{
    Loom.QueueOnMainThread((o => { action(); }), null);
}
```

扩展延迟执行方法，因延迟本身为异步操作，所以只提供异步执行方式  

```c#
// 此处使用async、await 关键字实现延迟操作， delay 为秒，Task.Delay 参数为毫秒
public Task RunAsync(Action action, float delay)
{
    Task t = Task.Run(async () =>
    {
        await Task.Delay((int) (delay * 1000));
        return RunAsync(action);
    });
    return t;
}
```

### 实现效果  

并发队列异步执行 			<img src="https://mh2dza.dm.files.1drv.com/y4mFq0X9VsFzEcvj5bA-Ig_Wy50kBp0-Ybssp6eKmCUuW9WvzBz4vabng8b9tGQkgxD45sMOjHIDjLVpyzxy9AWtz2kGDMbj6ngyOfpeBM7m85xhjV1s0kzNJXBWYTjRJaq7YQpEuEmq0IQGn49qxWI8VFU-pWOl7_PX_25Zo7qThcGkT28MzR_C3CfrgZpVY7Yk8v6jAcxbNMFndGAPVRy-g?width=648&amp;height=786&amp;cropmode=none" alt="并发异步" style="zoom:50%;" />

并发队列同步执行  	  	<img src="https://mh0cwa.dm.files.1drv.com/y4mjGr9g2ZbC4QAdHjnNeiMcsx-IPsIU5irknMT9FFITOLqr6YF_6lQkidXcuPaz3kBvb7tzADWh4kmlPTLdfWudWel5ONXBcmlHgfsWkE3it8j_ffaqnR08SDK9RotyPH1-MHNlV7q36TLWTd_GnFRn2vL3x18tAiJELTVXur9XOmcyDYfFesYybPs1c0TTgFb131kAVWYZNRWNrkGvKSwhg?width=672&amp;height=776&amp;cropmode=none" style="zoom:50%;" />  

串行队列异步执行			<img src="https://mh1ukg.dm.files.1drv.com/y4mUNcP0eEg0vOb75Q7hkn_kFs0G57qPDue_bZVuwgVNS3UGqH-guGzYQyUZNQKa9TOfDj8yLareYjCBW_U9fQoGrhAjGzpDRGekUB6h800DjrvLLyCfxsMBZcRQgd_BDhOXMT_Wt4BLsKyJ7SLxWEhrkjJJR8_voFW9oIxOhVYUkrtyf4oJKdguPKtoYXMF8QbAltwrW7kHD8HqN9RU_gU6g?width=678&amp;height=768&amp;cropmode=none" style="zoom:50%;" />  

串行队列同步执行			<img src="https://mh04qq.dm.files.1drv.com/y4m7S3A-SgUZPP8WqUiiCiTiZ-qkR4M5PhqO54jVZ4fk5sXZb3RP7ExyGvTFaNTbdalpKGDLwkBPW9v4H92_vJuD4TYJ3zeOfM3Wfy83MvNSizChlFBcjsrq3p_RAmzE1Tjh8noET9cjIp0LVpGMuBWzcpbkBjXztVbD4LITJFIPcveltI30GvmHz26uWwU9-X24DBL_v4wJoPyAoOyPY1e3w?width=744&amp;height=778&amp;cropmode=none" style="zoom:50%;" />  

并发队列延迟执行			<img src="https://mh1n4w.dm.files.1drv.com/y4mR1WPWwg5zE8JLD4LUSrA0e5KKrw34p_ABnm5LrKm9ANHsH0cAWgnE4EGOUooLFIXnAoUgW_B7azWJfmtIOllYyM2caXzFW2ohv_jUdTIWceMoAwxI-FX4_N3R4MHGd1Zn57N53VFffSCLdVN5iW5gcqUQkLNcKIp-D--iMbuQwLAcwKatjYNu9xxB9HpaPDXUCNOHBN-a02fnMo0KGr5RQ?width=782&amp;height=774&amp;cropmode=none" style="zoom:50%;" />   

子线程异步执行主线程	<img src="https://mh3mbg.dm.files.1drv.com/y4mwUsPnV21yuIiCziF8pa0-qMhjF2CYcZ7HtKN5VEQR83v0XUOcdZ5yKtxkJEUgiqgBGzm0YocP1vS2MgLlxghYLQYel9VBfTXD_SFjAdGtERtveZ2pvSPy6abIwi-EtfEbLsAyR_ibqOtKW550nPb3LD-NFyLquNNBBReNdzuhn9YGsgVlKZj4xtwCsqU33J4ppMADn9-sqS48t5z-Q4odA?width=890&amp;height=394&amp;cropmode=none" style="zoom:50%;" />  

子线程同步执行主线程	<img src="https://mh0xpg.dm.files.1drv.com/y4m3vNSOXCrotw6f7DUZCWgqA_-PBMUi0sQI84gZcZiC8Ir3AYZXYCOMQmVhbKNuQiI-qCFMWBidbeledKJRDCq8n_AYweTMyKAdnHn5V5EnkvIdk9QLyvhdhOOkRewIwPSKgU42AITfIf8VHpCBNhl8k-wuH2BbLJ_D6UEZ8D1vC6jQlPsjtChobW9KXvMIbcJkwH8YvtLaHvRyCSaFyuhNA?width=872&amp;height=396&amp;cropmode=none" style="zoom:50%;" />  





到此一个多线程任务队列工具就完成了，一般的需求基本可以满足，后续还可提供更多扩展功能，如传参、取消任务等  

另外我个人想尽力将这套工具脱离 UnityEngine.Monobehaviour，但目前还没找到除 Loom 外其他 Unity 获取主线程的方法，当然 Loom 本身仍然是一个很巧妙的工具  

若想了解 `LimitedConcurrencyLevelTaskScheduler` 和 `Loom` 可继续想下看

### 其他

#### LimitedConcurrencyLevelTaskScheduler

TaskScheduler 为抽象类，想自定义任务调度需继承该类，并复写部分内部调度方法

LimitedConcurrencyLevelTaskScheduler ，以下简称为 LCLTS，为[微软官方文档](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.tasks.taskscheduler?redirectedfrom=MSDN&view=netframework-4.8)提供的示例代码，用于调度任务，控制并发数    

##### LCLTS 工作流程  

1. 将 Task 入队放入链表
2. 判断当前已执行任务数量，若未达到最大值则通过 `ThreadPool` 分配工作线程执行，并计数+1  
3. 标记已分匹配的线程并死循环执行任务队列，将已执行的任务出队  
4. 任务队列为空时退出循环，并移除标记  
5. 若有任务想插队到线程执行，先检查当前线程标记，若无标记则无法执行插队操作，该操作为避免任务占用队列外繁忙线程  
6. 若插队成功则检查该 Task 是否已在队列中，若存在则出队执行，若不存在则直接执行   

##### 源码解释

```c#
public class LimitedConcurrencyLevelTaskScheduler : TaskScheduler
{
   //ThreadStatic 线程变量特性，表明是当前线程是否正在处理任务
   [ThreadStatic]
   private static bool _currentThreadIsProcessingItems;

  // 任务队列，使用链表比 List 和 Array 更方便执行插队出队操作（队列中不会出现空位）
   private readonly LinkedList<Task> _tasks = new LinkedList<Task>(); // 该队列由 lock(_tasks) 锁定

   // 最大并发数
   private readonly int _maxDegreeOfParallelism;

   // 当前已分配入队的任务数量 
   private int _delegatesQueuedOrRunning = 0;

   // 带并发数的构造方法
   public LimitedConcurrencyLevelTaskScheduler(int maxDegreeOfParallelism)
   {
       if (maxDegreeOfParallelism < 1) throw new ArgumentOutOfRangeException("maxDegreeOfParallelism");
       _maxDegreeOfParallelism = maxDegreeOfParallelism;
   }

   // 将 Task 放入调度队列
   protected sealed override void QueueTask(Task task)
   {
      //将任务放入列表，检查当前执行数是否达到最大值，若未达到则分配线程执行，并计数+1
       lock (_tasks)
       {
           _tasks.AddLast(task);
           if (_delegatesQueuedOrRunning < _maxDegreeOfParallelism)
           {
               ++_delegatesQueuedOrRunning;
               NotifyThreadPoolOfPendingWork();
           }
       }
   }

   // 使用 ThreadPool 将 Task 分配到工作线程
   private void NotifyThreadPoolOfPendingWork()
   {
       ThreadPool.UnsafeQueueUserWorkItem(_ =>
       {
         	 //标记当前线程正在执行任务，当有 Task 想插入此线程执行时会检查该状态
           _currentThreadIsProcessingItems = true;
           try
           {
               // 死循环处理所有队列中 Task
               while (true)
               {
                   Task item;
                   lock (_tasks)
                   {
                       // 任务队列执行完后退出循环，并将占用标记置为 false
                       if (_tasks.Count == 0)
                       {
                           --_delegatesQueuedOrRunning;
                           break;
                       }

                       // 若还有 Task 则获取第一个，并出队
                       item = _tasks.First.Value;
                       _tasks.RemoveFirst();
                   }

                   // 执行 Task
                   base.TryExecuteTask(item);
               }
           }
           // 线程占用标记置为 false
           finally { _currentThreadIsProcessingItems = false; }
       }, null);
   }

   // 尝试在当前线程执行指定任务
   protected sealed override bool TryExecuteTaskInline(Task task, bool taskWasPreviouslyQueued)
   {
       // 若当前线程没有在执行任务则无法执行插队操作
       if (!_currentThreadIsProcessingItems) return false;

       // 若该任务已在队列中，则出队
       if (taskWasPreviouslyQueued) 
          // 尝试执行 Task
          if (TryDequeue(task)) 
            return base.TryExecuteTask(task);
          else
             return false; 
       else 
          return base.TryExecuteTask(task);
   }

   // 尝试将已调度的 Task 移出调度队列
   protected sealed override bool TryDequeue(Task task)
   {
       lock (_tasks) return _tasks.Remove(task);
   }

   // 获取最大并发数
   public sealed override int MaximumConcurrencyLevel { get { return _maxDegreeOfParallelism; } }

   // 获取已调度任务队列迭代器
   protected sealed override IEnumerable<Task> GetScheduledTasks()
   {
       bool lockTaken = false;
       try
       {
         	 // Monitor.TryEnter 作用为线程锁，其语法糖为 lock (_tasks)
           Monitor.TryEnter(_tasks, ref lockTaken);
           if (lockTaken) return _tasks;
           else throw new NotSupportedException();
       }
       finally
       {
           if (lockTaken) Monitor.Exit(_tasks);
       }
   }
}
```

#### Loom    

Loom 通过继承 UnityEngine.MonoBehaviour，使用 Unity 主线程生命周期 `Update` 在主线程执行方法，同时 Loom 也支持简单的多线程异步执行

##### Loom 结构和流程  

Loom 包含两个队列有延迟方法队列和无延迟方法队列，两条队列方法都可执行传参方法   

1. 将 `Action` 和 `param` 以及延迟时间打包入*结构体*放入延迟或无延迟队列   
2. 若为延迟任务，则使用 `Time.time` 获取添加任务的时间加上延迟时间得到预定执行时间打包入*延迟任务结构体*并入队
3. 待一个 `Update` 周期执行，清空*执行队列*旧任务，取出*无延迟队列*所有对象，放入*执行队列*，清空*无延迟队列*，遍历执行*执行队列*任务  
4. 同一个 `Update` 周期，清空*延迟执行队列*旧任务，取出预计执行时间小于等于当前时间的任务，放入延迟执行队列，将取出的任务移出*延迟队列*，遍历执行*延迟执行队列*任务    

##### Loom 的使用

用户可将 Loom 脚本挂载在已有对象上，也可直接代码调用方法，Loom 会自动在场景中添加一个不会销毁的 Loom 单例对象

代码中使用 `QueueOnMainThread` 将延迟和无延迟方法加入主线程队列，  `RunAsync` 执行异步方法

```c#
public class Loom :MonoBehaviour
{
    public static int maxThreads = 8;
    static int numThreads;

    private static Loom _current;
    //private int _count;
    public static Loom Current
    {
        get
        {
            Initialize();
            return _current;
        }
    }

    void Awake()
    {
        _current = this;
        initialized = true;
    }

    static bool initialized;

    [RuntimeInitializeOnLoadMethod]
    public static void Initialize()
    {
        if (!initialized)
        {

            if (!Application.isPlaying)
                return;
            initialized = true;
            var g = new GameObject("Loom");
            _current = g.AddComponent<Loom>();
#if !ARTIST_BUILD
            UnityEngine.Object.DontDestroyOnLoad(g);
#endif
        }
    }
    public struct NoDelayedQueueItem
    {
        public Action<object> action;
        public object param;
    }

    private List<NoDelayedQueueItem> _actions = new List<NoDelayedQueueItem>();
    public struct DelayedQueueItem
    {
        public float time;
        public Action<object> action;
        public object param;
    }
    private List<DelayedQueueItem> _delayed = new List<DelayedQueueItem>();

    List<DelayedQueueItem> _currentDelayed = new List<DelayedQueueItem>();

    public static void QueueOnMainThread(Action<object> taction, object tparam)
    {
        QueueOnMainThread(taction, tparam, 0f);
    }
    public static void QueueOnMainThread(Action<object> taction, object tparam, float time)
    {
        if (time != 0)
        {
            lock (Current._delayed)
            {
                Current._delayed.Add(new DelayedQueueItem { time = Time.time + time, action = taction, param = tparam });
            }
        }
        else
        {
            lock (Current._actions)
            {
                Current._actions.Add(new NoDelayedQueueItem { action = taction, param = tparam });
            }
        }
    }

    public static Thread RunAsync(Action a)
    {
        Initialize();
        while (numThreads >= maxThreads)
        {
            Thread.Sleep(100);
        }
        Interlocked.Increment(ref numThreads);
        ThreadPool.QueueUserWorkItem(RunAction, a);
        return null;
    }

    private static void RunAction(object action)
    {
        try
        {
            ((Action)action)();
        }
        catch
        {
        }
        finally
        {
            Interlocked.Decrement(ref numThreads);
        }

    }


    void OnDisable()
    {
        if (_current == this)
        {

            _current = null;
        }
    }



    // Use this for initialization
    void Start()
    {

    }

    List<NoDelayedQueueItem> _currentActions = new List<NoDelayedQueueItem>();

    // Update is called once per frame
    void Update()
    {
        if (_actions.Count > 0)
        {
            lock (_actions)
            {
                _currentActions.Clear();
                _currentActions.AddRange(_actions);
                _actions.Clear();
            }
            for (int i = 0; i < _currentActions.Count; i++)
            {
                _currentActions[i].action(_currentActions[i].param);
            }
        }

        if (_delayed.Count > 0)
        {
            lock (_delayed)
            {
                _currentDelayed.Clear();
                _currentDelayed.AddRange(_delayed.Where(d => d.time <= Time.time));
                for (int i = 0; i < _currentDelayed.Count; i++)
                {
                    _delayed.Remove(_currentDelayed[i]);
                }
            }

            for (int i = 0; i < _currentDelayed.Count; i++)
            {
                _currentDelayed[i].action(_currentDelayed[i].param);
            }
        }
    }
}
```

