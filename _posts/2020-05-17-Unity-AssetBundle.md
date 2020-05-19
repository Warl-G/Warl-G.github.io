---
layout: post
title:  "Unity学习—AssetBundle"
description: Unity AssetBundle 机制与最佳实践
date:   2020-05-17 23:20:07 +0800
categories: [Unity]
tag: [学习笔记,资源管理,AssetBundle]


---

本篇文章主要内容来自于官方教程 [Assets, Resources and AssetBundles](https://learn.unity.com/tutorial/assets-resources-and-assetbundles)，介绍了 AssetBundle 的各类机制，使用方式和适用场景等

有关其他 Unity 资源管理的内容可见[Unity学习—资源管理概览](https://warl-g.github.io/posts/Unity-Resource-Manage/)  

文中所有 API 均以版本 2019.3 为准

本文其他地址：[简书](https://www.jianshu.com/p/9fb5e206df81)	[知乎](https://zhuanlan.zhihu.com/p/141646264) 	[掘金](https://juejin.im/post/5ec152a86fb9a0436153de90)  

## AssetBundle 作用

AssetBundle 是外部资产的集合，可独立于 Unity 构建过程外，是 Unity 更新非代码内容的主要工具，经常置于服务器上供用户终端动态获取；AssetBundle 使开发者可以提交更小的应用包，最小化运行时内存压力，使终端可以选择性加载优化内容  

## AssetBundle 组成

AssetBundle 主要由两部分组成：文件头和数据段

文件头包含了id、压缩类型、索引清单，该索引清单是与 Resources 相同的记录了序列化文件中的字节偏移量的查找表。对于大部分平台该表为平衡搜索树，对 Windows 和 OSX 系列（包括 iOS）则为红黑树，随着 AssetBundle 中对象的增加，构造清单所需时间的增长速度将超过线形增长速度   

数据段包含了 Asset 经过序列化的原始数据，数据还可选择是否压缩，若使用 LZMA 压缩，则将所有 Asset 的字节数组整体压缩；若使用 LZ4 压缩，则将每个 Asset 单独压缩；若不压缩，则数据保持原始字节流

## AssetBundle 加载

有四种不同的 API  用于加载 AssetBundle，但每个 API 的行为随压缩算法和平台而不同  

* AssetBundle.LoadFromMemoryAsync  

  ```c#
  IEnumerator LoadFromMemoryAsync(string path)
  {
      AssetBundleCreateRequest createRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(path));
      yield return createRequest;
      AssetBundle bundle = createRequest.assetBundle;
      var prefab = bundle.LoadAsset<GameObject>("MyObject");
      Instantiate(prefab);
  }
  ```

* AssetBundle.LoadFromFile    

  该方法可高效地从硬盘加载未压缩或 LZ4 压缩的 Assetbundle，加载 LZMA 压缩包时会先解压再加载到内存

  ```c#
  public class LoadFromFileExample : MonoBehaviour {
      function Start() {
          var myLoadedAssetBundle 
              = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetBundle"));
          
          if (myLoadedAssetBundle == null) {
              Debug.Log("Failed to load AssetBundle!");
              return;
          }
          var prefab = myLoadedAssetBundle.LoadAsset.<GameObject>("MyObject");
          Instantiate(prefab);
      }
  }
  ```

* WWW.LoadfromCacheOrDownload（5.6 及以前版本） 

  旧方法，已抛弃

* UnityWebRequestAssetBundle （5.3 及以后版本） 

  先使用`UnityWebRequest.GetAssetBundle`创建请求，再将请求传入`DownloadHandlerAssetBundle.GetContent(UnityWebRequest)`，下载完成后可像`AssetBundle.LoadFromFile` 一样，直接使用 assetBundle 对象   

  该方法使开发者更灵活处理下载数据，选择临时存储或长期缓存，避免不必要的内存使用。同时，由于是原生代码，没有托管堆栈扩展风险，DownloadHandler 也不会保留下载数据，进一步减少了内存开销

  LZMA 压缩包会在下载时解压，并以 LZ4 重新压缩缓存，可调用 [Caching.CompressionEnabled](https://docs.unity3d.com/ScriptReference/Caching-compressionEnabled.html?_ga=2.77657364.1457724009.1589157197-1900867135.1571822402) 修改

  ```c#
  IEnumerator InstantiateObject()
  {
      string uri = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName; 
      UnityEngine.Networking.UnityWebRequest request 
          = UnityEngine.Networking.UnityWebRequest.GetAssetBundle(uri, 0);
      yield return request.Send();
      AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
      GameObject cube = bundle.LoadAsset<GameObject>("Cube");
      GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
      Instantiate(cube);
      Instantiate(sprite);
  }
  ```

**官方推荐**尽量使用 `AssetBundle.LoadFromFile`，该 API 在速度、磁盘使用和运行时内存使用方面都最高效；需要下载则使用 *UnityWebRequest*

## AssetBundle Asset 加载

同步异步加载 Asset 一共有六种 API 可使用，同步方法一定比对应的异步方法快至少一帧

* LoadAsset (LoadAssetAsync)  
* LoadAllAssets (LoadAllAssetsAsync)  
* LoadAssetWithSubAssets (LoadAssetWithSubAssetsAsync)

`LoadAllAssets`适合加载包中大部分或所有独立 Unity 对象时使用，相较于多次重复调用另外两种 API，`LoadAllAssets`速度要稍快一点。因此当 Asset 数量巨大且一次性需要加载的 Asset 少于 2/3 的时候，建议将 AssetBundle 拆分成多个小包体，再使用`LoadAllAssets`加载  

`LoadAssetWithSubAssets`适合需要加载的对象内嵌了其他对象的情况，若加载对象均来自于一个 Asset 且包中有许多其他无关对象，则使用该 API  

其他情况均用`LoadAsset (LoadAssetAsync)`    

Unity 对象加载时在主线程执行，对象数据是在工作线程 worker thread，任何线程不敏感的操作都在工作线程执行

异步加载时会根据时间片限制每帧加载多个对象，自 Unity 5.3 后，对象加载就并行化了。多个对象在工作线程被反序列化、处理和集成，当对象加载完成，则触发 Awake 回调

同步加载方法 `AssetBundle.Load`会暂停主线程知道加载完成，它们还将加载过程进行时间切片，以使对象集成所占用的帧时间不超过特定的毫秒数，该值可通过`Application.backgroundLoadingPriority`设定  

* ThreadPriority.High: 最大 50 毫秒每帧
* ThreadPriority.Normal: 最大 10 毫秒每帧
* ThreadPriority.BelowNormal: 最大 4 毫秒每帧
* ThreadPriority.Low: 最大 2 毫秒每帧  

在其他因素相同的情况下，异步加载方法的调用到加载对象可用之间最小有一帧延迟，导致异步加载方法比同步方法执行所需时间更长  

## AssetBundle 依赖

根据运行环境可以使用两个不同的 API 自动追踪 AssetBundle 之间的依赖。Editor 环境下，可使用`AssetDatabase`查询依赖，使用` AssetImporter`访问和修改 AssetBundle 的分配和依赖；运行时，可以通过基于 ScriptableObject 的 `AssetBundleManifest` API 加载在 AssetBundle 构建期间生成的依赖项信息  

当一个对象所在的 AssetBundle 被加载时，该对象就被分配了一个唯一的有效实例 ID，因此 AssetBundle 的加载顺序并不重要，重要的是在加载该对象本身之前，要优先把所有包含其依赖对象的 AssetBundle 加载好。Unity 不会自动加载子 AssetBundle，具体可详见[手册](https://docs.unity3d.com/Manual/AssetBundles-Dependencies.html)，例：  

AssetBundle 1 中的 Material A 依赖于 AssetBundle 2 中的 Texture B，若要正常加载，与 AssetBundle 1 和 2 的加载顺序无关，但一定要保证加载 Material A 时，AssetBundle 2 已加载  

在构建 AssetBundle 时，Unity 创建一个包含每一个 AssetBundle 依赖信息的类型为  AssetBundleManifest 的序列化对象，该文件存在一个与其他 AssetBundle 在同一打包路径下的单独的 AssetBundle 中，且与父层文件夹名相同

有两种 API 查询依赖  

*  AssetBundleManifest.GetAllDependencies 获取 AssetBundle 的所有依赖层级
* AssetBundleManifest.GetDirectDependencies 获取 AssetBundle 直接依赖   

因该 API 会生成字符串数组，所以应尽量少用，且避免性能高峰时使用

**官方建议**，大部分场合下，在进入性能需求高的场景前，尽可能多地加载对象，尤其对于移动平台这种，访问本地存储慢，加载卸载对象引起内存流失会触发垃圾回收的平台  

##   AssetBundle 使用

不适当地卸载 AssetBundle 会导致对象缺失或者在内存中重复 

调用 `AssetBundle.Unload`可卸载 AssetBundle 的头信息，传入参数 true 或 false 决定是否同时卸载该包下所有已加载对象。由此诞生一个问题，当卸载了 AssetBundle 未卸载已加载对象时，此时这些对象与 AssetBundle 便失去关联了，重新加载 AssetBundle 并重新加载同一对象时，只会产生一个新的关联对象，而旧对象则无法使用`AssetBundle.Unload`卸载了，这就导致了内存中同时存在两个一样的对象  

![img](https://connect-prd-cdn.unity.com/20190130/e1119799-f21d-4d2f-a56a-550302752728_ab2d.jpg)

为避免该现象发生，有两种通用处理方法：

1. 最便捷常用的方法是在应用生命周期特定点，如关卡切换或加载界面时，卸载 AssetBundle  
2. 对每个对象引用计数，仅在所有 AssetBundle 对象未使用时卸载  

针对遗留的未卸载对象：

1. 销毁场景和代码中所有该对象的引用后，调用`Resources.UnloadUnusedAssets`
2. 切换到一个非叠加型的新场景会销毁当前场景所有对象并自动调用`Resources.UnloadUnusedAssets`  

对于场景资源统一的项目，可将每个带有资源的场景分别打包，在展示加载界面时，卸载旧场景所在的 AssetBundle 及其对象同时加载新场景所在的 AssetBundle  

可按加载时机区分将对象打包分组，如人物形象、 UI、 模型和纹理、等长期存在的内容，可分为一包并在开始时加载，其他内容依据所需时机分组  

另外还可能出现的问题是，在 AssetBundle 卸载之后加载 AssetBundle 中的对象时，会出现对象缺失的问题。出现该问题大部分原因为 Unity 丢失又重新获得对图形上下文的控制，如移动设备 App 挂起，PC 锁定等场景

## AssetBundle 发布

根据实际情况选择 AssetBundle 时随项目打包，或后续通过网络下载，一般移动平台由于初始安装大小和下载限制，会选择后续下载，而主机和电脑则随项目打包  

随项目打包有两个主要原因：

1. 减少项目构建时长，简化迭代开发，针对无需单独更新的 AssetBundle 可放在 StreamingAssets 目录下
2. 发布可更新的初始修正内容，用于节省用户初始安装后的时间和为后续修复做准备。但 StreamingAssets 不适用于该情况，若不考虑自定义下载和缓存系统，则可以使用 Unity 的缓存系统，从 StreamingAssets 下载初始缓存

对于 Android 平台，若 APK 经过压缩，`AssetBundle.LoadFromFile()`将需要更多时间读取 StreamingAssets，且不同版本的 Unity 所使用的存储算法可能也不一样。对于经过压缩的 APK 可使用` UnityWebRequest.GetAssetBundle` 将 AssetBundle 解压并缓存，但该操作会占用额外的缓存空间；或者可以导出 Gradle 项目并修改 build 文件添加无压缩选项，随后即可使用`AssetBundle.LoadFromFile()`并省去解压过程  

一般推荐使用 `UnityWebRequest`下载 AssetBundle，若下载包为 LZMA 压缩，则缓存的为未压缩或使用 LZ4 重压缩的内容，若缓存已满，则 Unity 会删除最近最少使用的 AssetBundle。

建议仅当现有 API 的内存消耗、缓存行为、性能不能满足或必须执行特定平台的代码时才使用自定义下载系统，如：

* 需要控制缓存细粒度时
* 需要自定义缓存策略时
* 需要执行平台特定代码，如 iOS 的后台下载  
* 需要在不支持 SSL 的品台上使用 SSL 下载  

Unity 内置的 AssetBundle 缓存系统用于缓存 `UnityWebRequestAssetBundle.GetAssetBundle`下载的包，缓存仅以名称作为唯一标识。另外可通过重载方法可传入版本号（开发者自己管理版本号），缓存系统会比对版本号，选择匹配版本或下载新包   

缓存系统可通过 [Caching.expirationDelay](https://docs.unity3d.com/560/Documentation/ScriptReference/Caching-expirationDelay.html) 和 [Caching.maximumAvailableDiskSpace](https://docs.unity3d.com/560/Documentation/ScriptReference/Caching-maximumAvailableDiskSpace.html) 修改最小未使用过期时间和最大缓存空间，当缓存文件在过期时间内没被打开过即被删除，或缓存空间不足，则优先删除最近最少打开的缓存  

### 自定义下载器

自定义下载器需考虑四点：

* 下载机制
* 存储位置
* 压缩类型
* 补丁 

可使用如下三种方式快速实现：

* C# 提供的 HttpWebRequest 和 WebClient 类  
* 自定义原生插件  
* 资源商店包  

若应用不需要支持 HTTPS/SSL，那么 WebClient 提供了最简单的下载机制，可实现异步直接下载到本地位置而不用额外的内存分配。

若需要更多参数选项控制下载器，则可使用 HttpWebRequest：

* 通过 HttpWebResponse.GetResponseStream 获取字节流  
* 在栈上分配固定大小的字节缓冲区 
* 读取请求结果放入缓冲区  
* 使用 IO 将缓冲区数据写入磁盘  

## Asset 分包策略

* 逻辑实体分包
* 对象类型分包
* 并发内容分包

### 逻辑实体分包

依据资源在项目功能块的使用位置，如 UI、角色、环境和其他在生命周期中常出现的内容等分包

* 将所有 UI 的纹理和布局数据分包
* 将角色的模型和动画数据分包  
* 将多场景共用的纹理和模型分包

该分包方式适用于制作 DLC，可以只下载单个实体而无需下载无变化的资源，其关键点在于需要开发者清楚了解每个打包的资源所要用到的时机和位置

### 对象类型分包

该方式适用于针对多平台分包，例如音频文件的压缩设置在 Windows 和 Mac OS 平台一样，另外由于纹理压缩格式和设置等改变频率远低于脚本和预设体，使用该分配方式可以使 AssetBundle 兼容更多的 Unity 版本  

### 并发内容分包

并发内容分包可理解为以关卡为分组依据，将一个关卡内独有的角色、纹理、音乐等需要在同一时机加载的内容分为一包  

### Tips

* 将常更新与不常更新内容分开  
* 将需要同时加载的对象分为一组，如一个模型，其所需的材质和动画分为一组  
* 若多个 AssetBundle 中的多个对象引用了其他 AssetBundle 中的单个 Asset，则将依赖项分离到单独的包中以减少重复  
* 确保两组完全不可能同时加载的对象不在用一包中，如低清和高清材质包  
* 若一个包中只有低于一半的对象被频繁加载，可将其拆分  
* 将一些同时加载的小包（资源少于5到10个）合并  
* 若一个包中的对象仅是版本不同，则可以使用 AssetBundle 变体  

## 常见问题

### 资产重复

若有一个未分配资产被多个不同 AssetBundle 中的已分配资产引用，则在构建 AssetBundle 时，该引用对象会被拷贝到每个 AssetBundle 中，造成空间和内存浪费

可使用 `AssetDatabase.GetDependencies`定位所有指定对象的依赖，使用`AssetImporter`查询分配了任何特定对象的AssetBundle

几种解决方案：

* 确保不同包中的对象没有共用依赖，或有共用依赖的对象分到一个包中，但对部分项目该方法会使 AssetBundle 过大不便于重复构建和下载
* 将有共同依赖项的 AssetBundle 分段，使其不会同时加载  
* 使所有依赖项被分配到 AssetBundle，但会增加应用追踪依赖的复杂度  

### 精灵图集重复

任何自动生成的图集会被分配到其包含精灵所在的 AssetBundle，若精灵对象被分配到多个包，则图集会被复制，因此需确保同一图集的精灵对象被分配到同一 AssetBundle 中  

### Android 纹理

由于 Android 生态的碎片化，经常需要使用不同格式压缩纹理。所有 Android 设备都支持 ETC1 压缩格式，但该格式不支持透明通道。如果应用不需要支持 OpenGL ES 2，则解决问题的最简单方法是使用 ETC2，所有支持 OpenGL ES 3 的 Android 设备都支持该方法。如果必须分为两种压缩格式，则可使用 AssetBundle Variants。

要使用 AssetBundle 变体，所有不适用 ETC1 的纹理必须独立到一个只有纹理的 AssetBundle 中，并创建该包对应不同压缩格式的变体，如 DXT5, PVRTC 和 ATITC，更改变体中纹理的 TextureImporter 设置到对应的压缩格式。运行时，使用 [SystemInfo.SupportsTextureFormat](http://docs.unity3d.com/ScriptReference/SystemInfo.SupportsTextureFormat.html) 检测设备支持的压缩格式，并选择对应变体

## AssetBundle 变体

AssetBundle Variants 的主要作用在于使 AssetBundle 随运行时环境调整其内容配置，AssetBundle Variants 使不同 AssetBundle 中的不同对对象在加载时公用一个实例 ID，使其看起来为同一个对象。

有两个经典案例：

* 变体简化了 AssetBundle 对特定平台的加载过程，例如高清和低清平台使用不同的变体，代码上只用同一个对象名称即可加载合适的资源  
*   变体使应用可以针对同一平台的不同硬件加载不同内容，如移动平台不同机型之间的显示纹理不同  

AssetBundle Variants 也有一定缺陷，主要是不同 Asset 会组成不同的变体，哪怕两组 Asset 之间仅仅是导入设置不同。该缺陷增加了管理大型项目资源的复杂程度，当改变一个资产时，所有的变体都需要更新。可使用规范的命名规则确认变体作用或使用代码在构建 AssetBundle 时更改导入设置    

## 其他

### 是否压缩

* 未压缩的 AssetBundle 加载速度比压缩过的快很多  
* 在构建 AssetBundle 时压缩很耗时  
* 压缩包占用空间小  
* 未压缩或 LZ4 占用更少内存  
* 包体过大或平台可用网络带宽过小建议压缩，否则不压缩  
* 主要由使用 Crunch 压缩算法的 DXT 压缩纹理组成的包应不压缩  

### WebGL

WebGL 仅支持主线程解压和加载 AssetBundle 且仅支持未压缩和 LZ4 压缩格式，为避免造成性能问题，建议减小包体，也可考虑使用  gzip/brotli 压缩 AssetBundle

## 参考  

[Unity资源管理](http://tonytang1990.github.io/2016/10/13/Unity资源/#Resources)