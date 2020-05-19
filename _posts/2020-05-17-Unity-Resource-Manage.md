---
layout: post
title:  "Unity学习—资源管理概览"
description: Unity 资源管理相关知识介绍
date:   2020-05-17 23:00:07 +0800
categories: [Unity]
tag: [学习笔记,资源管理,Resources,AssetBundle,Addressable]
---

本文介绍了 Unity 常用四种默认路径，以及 AssetDataBase、Resources、AssetBundle 和目前最新的 Addressable 四种资源管理方式

文中所有 API 均以版本 2019.3 为准

本文其他地址：[简书](https://www.jianshu.com/p/b77828d3f62f)	[知乎](https://zhuanlan.zhihu.com/p/141641436) 	[掘金](https://juejin.im/post/5ec14d28e51d454db11f8afc)  

## 资源路径

#### Application.dataPath

[官方文档](https://docs.unity3d.com/2020.2/Documentation/ScriptReference/Application-dataPath.html)

只读，Editor 可读写

游戏数据相对路径，即游戏安装路径，PC 上路径会使用 '/' 分割文件夹  

* **Unity Editor**: `<项目根路径>/Assets`  
* **Mac player**: `<App 包路径>/Contents`
* **iOS player**: `<App 包路径>/<AppName.app>/Data` 
* **Win/Linux player**: `<可执行数据文件夹路径>`
* **WebGL**: player data 文件的 绝对 url 地址 (不包含具体文件名)
* **Android**: 一般为 APK 路径， 若使用 split binary build, 则为 OBB 路径
* **Windows Store Apps**: player data 文件夹的绝对地址

#### Application.persistentDataPath

[官方文档](https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html)

可读写，用于持久化数据存储，在 iOS 和 Android 平台该路径指向设备的公共路径，该目录不会随 App 升级而删除，但可被用户直接删除

`persistentDataPath`的路径由`Bundle Identifier`生成的 GUID 组成，只要`Bundle Identifier`不变，路径不变  

iOS 会自动将 persistentDataPath 路径下的文件备份到 iCloud

* **Windows Store Apps**: `%userprofile%\AppData\Local\Packages\<productname>\LocalState`
* **iOS**: `/var/mobile/Containers/Data/Application/<guid>/Documents`
* **Android**: `/storage/emulated/0/Android/data/<packagename>/files` 该路径由  [android.content.Context.getExternalFilesDir](https://developer.android.com/reference/android/content/Context#getExternalFilesDir(java.lang.String).) 获得，部分机型该路径会指向 SD 卡  
* **Mac**: `~/Library/Application Support/<company name>/<product name>` ，旧版本还可能为 `~/Library/Caches` 或 `~/Library/Application Support/unity.company name.product name`，Unity 会查询并使用以上路径中最早的路径  

#### Application.streamingAssetsPath

[官方文档](https://docs.unity3d.com/ScriptReference/Application-streamingAssetsPath.html) [官方手册](https://docs.unity3d.com/Manual/StreamingAssets.html)

只读，Editor 可读写

流数据存储的相对路径，该目录下 Asset 在 Unity 编译时不会被 Unity 打包，使其在运行时可直接通过路径获取，可将资源放入 Assets 目录下任何名为 `StreamingAssets`文件夹

 `StreamingAssets`中资源可使用 I/O 读取，但 WebGL 和 Android 平台下该路径为 URL，不支持直接获取，因此需使用 `UnityWebRequest`获取。若其他平台使用 `UnityWebRequest` 获取，则需在路径前加上`"file://"` ，如 `"file://" + Application.streamingAssetsPath + "/file.mp4" `

* **Unity Editor, Windows, Linux players, PS4, Xbox One, Switch** : `Application.dataPath + "/StreamingAssets"`  
* **Mac**: `Application.dataPath + "/Resources/Data/StreamingAssets"`
* **iOS**: `Application.dataPath + "/Raw"`
* **Android**: `"jar:file://" + Application.dataPath + "!/assets"` (压缩后的 APK/JAR 文件)  

#### Application.temporaryCachePath

可读写，临时数据和缓存路径，应用更新或覆盖安装时不会被清除，手机空间不足时才可能会被系统清除

### 路径示例

| 路径                            | Editor                                                       | Windows                                                      | Mac OS                                                       | iOS                                                          | Android                                        |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------- |
| Application.dataPath            | 项目路径/Assets                                              | 安装路径/ProductName_Data                                    | /Applications/AppName.app/Contents                           | /var/mobile/Containers/Data/Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/AppName.app/Data | /data/app/package.name.apk                     |
| Application.persistentDataPath  | C:/Users/username/AppData/LocalLow/CompanyName/ProductName <br />或<br />/Users/username/Library/Application Support/CompanyName/ProductName | C:\Users\username\AppData\LocalLow\CompanyName\ProductName   | /Users/username/Library/Application Support/CompanyName/AppName | /var/mobile/Containers/Data/Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents | /data/data/package.name/files                  |
| Application.streamingAssetsPath | 项目路径/Assets/StreamingAssets                              | 安装路径/ProductName_Data/StreamingAssets                    | /Applications/AppName.app/Contents/Resources/Data/StreamingAssets | /var/containers/Bundle/Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/AppName.app/Data/Raw | jar:file:///data/app/package.name.apk/!/assets |
| Application.temporaryCachePath  | C:/Users/username/AppData/Local/Temp/CompanyName/ProductName<br />或<br />/var/folders/xx/xxxxxxxxxxxxxx/X/CompanyName/ProductName | C:\Users\username\AppData\Local\Temp\CompanyName\ProductName | /var/folders/xx/xxxxxxxxxxxxxx/X/CompanyName/ProductName     | /var/mobile/Containers/Data/Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Library/Caches | /data/data/package.name/cache                  |

### 读写权限说明

https://blog.csdn.net/BillCYJ/article/details/99712313

## 资源加载

[推荐官方教程](https://learn.unity.com/tutorial/assets-resources-and-assetbundles?_ga=2.122105355.182214086.1588732234-1900867135.1571822402#5c7f8528edbc2a002053b5a8)

### AssetDataBase

AssetDataBase 可在 Editor 环境下对项目 Asset 进行增删改查等操作（可实现与 Unity 编辑器顶部工具栏 Assets 选项下基本相同的功能），使用方法可参考[官方手册](https://docs.unity3d.com/Manual/AssetDatabase.html) [接口文档](https://docs.unity3d.com/ScriptReference/AssetDatabase.html)  

### Resources

[接口文档](https://docs.unity3d.com/ScriptReference/Resources.html)  

可在项目 Assets 目录下任意位置创建`Resources`文件夹，打包时 Unity 会整合所有位于`Resources`文件夹的 Asset 及其依赖，并生成一个只读的 `resources.assets` 资产文件，对于 Resources 目录中在游戏中被直接引用的资产，则会被另外打包到 `sharedassets0.assets` 中    

#### Resources 最佳实践

**官方的建议是不使用 Resources**，有以下几点原因：

1. Resources 文件夹会导致内存管理困难
2. 不适当使用 Resources 文件夹会加长应用启动和编译时间，Resources 文件夹越多，Asset 管理越困难
3. Resources 系统降低项目针对指定平台使用自定义内容的能力，并且无法实现增量更新（AssetBundle 是 Unity 针对不同设备提供特定内容的主要工具）

适合使用 Resources 的场景：

1. 因其简单快速的特性，适合用于快速原型和实验开发，但当正式开发时应当减少使用  
2. 适合以下条件都满足的状况  
   1. 该内容不会占用大量存储资源
   2. 该内容在整个生命周期都需要  
   3. 该内容几乎不需要修改
   4. 该内容在不同平台设备都一致  

#### Resources 序列化

项目编译时会将所有 Resources 目录下 Asset 和 Object 合并到一个序列化的 `resources.assets`文件，该文件中还包含了类似于 AssetBundle 的元数据（metadata）和索引信息，该信息包含了由对象名称转化得到的 GUID 和 Local ID 的查找树和对象位于序列化文件中的字节偏移量

对于大部分平台，查找树为时间复杂度为 O(n log(n)) 的平衡查找树，随着 Resources 中对象的增加，索引加载时间增长速度将超过线形增长速度  

Resources 系统在 Splash 展示时初始化，该过程不可跳过，经观察在低端设备上，10000 个 Asset 文件就会导致该过程长达数秒，哪怕很多对象在第一个场景没用到也会被加载

| 接口                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| FindObjectsOfTypeAll | 获取所有指定类型的对象                                       |
| Load                 | 加载 Resources 指定目录下的 Asset                            |
| LoadAll              | 加载 Resources 指定目录下的所有 Asset                        |
| LoadAsync            | 异步加载 Resources 指定目录下的 Asset                        |
| UnloadAsset          | 将 asset 从内存释放，重新加载 Asset 不会使之前的引用重新链接 |
| UnloadUnusedAssets   | 释放未使用的 Asset（包括仅在脚本堆栈使用，未在GameObject 使用） |

```c#
void Start()
{
    //Load a text file (Assets/Resources/Text/textFile01.txt)
    var textFile = Resources.Load<TextAsset>("Text/textFile01");

    //Load text from a JSON file (Assets/Resources/Text/jsonFile01.json)
    var jsonTextFile = Resources.Load<TextAsset>("Text/jsonFile01");
    //Then use JsonUtility.FromJson<T>() to deserialize jsonTextFile into an object

    //Load a Texture (Assets/Resources/Textures/texture01.png)
    var texture = Resources.Load<Texture2D>("Textures/texture01");

    //Load a Sprite (Assets/Resources/Sprites/sprite01.png)
    var sprite = Resources.Load<Sprite>("Sprites/sprite01");

    //Load an AudioClip (Assets/Resources/Audio/audioClip01.mp3)
    var audioClip = Resources.Load<AudioClip>("Audio/audioClip01");
}
```

### AssetBundle

[官方手册](https://docs.unity3d.com/Manual/AssetBundlesIntro.html) [接口文档](https://docs.unity3d.com/ScriptReference/AssetBundle.html)  

AssetBundle 是外部资产的集合，可独立于 Unity 构建过程外，是 Unity 更新非代码内容的主要工具，经常置于服务器上供用户终端动态获取；AssetBundle 使开发者可以提交更小的应用包，最小化运行时内存压力，使终端可以选择性加载优化内容  

该部分仅简单介绍 AssetBundle，更多信息可见 [Unity学习—AssetBundle](https://warl-g.github.io/posts/Unity-AssetBundle/)  

#### AssetBundle 构建

1. 首先分配资产对象所在 AssetBundle，在 Project 窗口选中需要打包的 Asset，在 Inspect 窗口底部可见如下图内容，底部 AssetBundle 后有两个输入选择框，第一个为该资源所在 AssetBundle 名称，第二个为 AssetBundle 变体名称

   除此之外，Unity 还提供了 `AssetImporter.assetBundleName`和`AssetImporter.assetBundleVariant`等接口将资源分配到 AssetBundle

   ![](https://q6gkza.dm.files.1drv.com/y4m3olKy-p_QVXh6lcomIsUpB7zEvardV-WSUcxoSAhYXlhkV6Zhds7oUOskF97y3PKQCqXPH2ujt5B69uwO7xuUk62vYe3I9wHaLfHMiY_ae-au61sCggd6mZwx7GX3OuP9-ZQ00P8CPRstIKfdT0e3KbgT6Lmg-yFpE6FJXpH_oGpzRCKvTukWlfbY--ORX5HMdJ-h21uiAS_tI6qk85J_A?width=216&height=256&cropmode=none) 

2. 然后即可构建 AssetBundle 了，使用`BuildPipeline.BuildAssetBundles()`即可构建 AssetBundle，其中可配置参数输出路径、构建选项、目标平台       

3. 或者可以使用 Unity 官方提供的工具管理 AssetBundle [AssetBundles-Browser](https://github.com/Unity-Technologies/AssetBundles-Browser) [官方手册](https://docs.unity3d.com/Manual/AssetBundles-Browser.html)    

```c#
[MenuItem("Build Asset Bundles/Normal")]
static void BuildABsNone()
{
    BuildPipeline.BuildAssetBundles("Assets/MyAssetBuilds", BuildAssetBundleOptions.None, BuildTarget.StandaloneOSX);
}
```



#### AssetBundle 构建选项

##### BuildAssetBundleOptions

* None
* UncompressedAssetBundle：不压缩
* DisableWriteTypeTree：不包含类型信息
* DeterministicAssetBundle：使用哈希值作为 Asset Id
* ForceRebuildAssetBundle：强制重建
* IgnoreTypeTreeChanges：增量构建检查时忽略类型树改动
* AppendHashToAssetBundleName：AssetBundle 名称后加哈希值
* ChunkBasedCompression：使用 LZ4 压缩
* StrictMode：构建过程中任务错误即构建失败
* DryRunBuild：试运行
* DisableLoadAssetByFileName：禁用名称查找资源，可降低运行时内存提高加载效率
* DisableLoadAssetByFileNameWithExtension：禁用带后缀名的名称查找资源，可降低运行时内存提高加载效率

#### AssetBundle 派发方式

根据实际情况选择 AssetBundle 时随项目打包，或后续通过网络下载，一般移动平台由于初始安装大小和下载限制，会选择安装后下载，而主机和电脑则随项目打包  

随项目打包有两个主要原因：

1. 减少项目构建时长，简化迭代开发，针对无需单独更新的 AssetBundle 可放在 StreamingAssets 目录下
2. 发布可更新的初始修正内容，用于节省用户初始安装后的时间和为后续修复做准备。但 StreamingAssets 不适用于该情况，若不考虑自定义下载和缓存系统，则可以使用 Unity 的缓存系统，从 StreamingAssets 下载初始缓存

一般推荐使用 `UnityWebRequest`下载 AssetBundle，若下载包为 LZMA 压缩，则缓存的为未压缩或使用 LZ4 重压缩的内容，若缓存已满，则 Unity 会删除最近最少使用的 AssetBundle

Unity 内置的 AssetBundle 缓存系统用于缓存 `UnityWebRequestAssetBundle.GetAssetBundle`下载的包，缓存仅以名称作为唯一标识。另外可通过重载方法可传入版本号（开发者自己管理版本号），缓存系统会比对版本号，选择匹配版本或下载新包   

缓存系统可通过 [Caching.expirationDelay](https://docs.unity3d.com/560/Documentation/ScriptReference/Caching-expirationDelay.html) 和 [Caching.maximumAvailableDiskSpace](https://docs.unity3d.com/560/Documentation/ScriptReference/Caching-maximumAvailableDiskSpace.html) 修改最小未使用过期时间和最大缓存空间，当缓存文件在过期时间内没被打开过即被删除，或缓存空间不足，则优先删除最近最少打开的缓存  

```c#
IEnumerator GetText()
{
    using (UnityWebRequest uwr = UnityWebRequestAssetBundle.GetAssetBundle("http://www.my-server.com/mybundle"))
    {
        yield return uwr.SendWebRequest();

        if (uwr.isNetworkError || uwr.isHttpError)
        {
            Debug.Log(uwr.error);
        }
        else
        {
            // Get downloaded asset bundle
            AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(uwr);
        }
    }
}
```



#### AssetBundle 加载

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

#### AssetBundle Asset 加载

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

#### AssetBundle 依赖

根据运行环境可以使用两个不同的 API 自动追踪 AssetBundle 之间的依赖。Editor 环境下，可使用`AssetDatabase`查询依赖，使用` AssetImporter`访问和修改 AssetBundle 的分配和依赖；运行时，可以通过基于 ScriptableObject 的 `AssetBundleManifest` API 加载在 AssetBundle 构建期间生成的依赖项信息  

当一个对象所在的 AssetBundle 被加载时，该对象就被分配了一个唯一的有效实例 ID，因此 AssetBundle 的加载顺序并不重要，重要的是在加载该对象本身之前，要优先把所有包含其依赖对象的 AssetBundle 加载好。

Unity 不会自动加载子 AssetBundle，具体可详见[手册](https://docs.unity3d.com/Manual/AssetBundles-Dependencies.html)，例：  

AssetBundle 1 中的 Material A 依赖于 AssetBundle 2 中的 Texture B，若要正常加载，与 AssetBundle 1 和 2 的加载顺序无关，但一定要保证加载 Material A 时，AssetBundle 2 已加载  

在构建 AssetBundle 时，Unity 创建一个包含每一个 AssetBundle 依赖信息的类型为  AssetBundleManifest 的序列化对象，该文件存在一个与其他 AssetBundle 在同一打包路径下的单独的 AssetBundle 中，且与父层文件夹名相同

有两种 API 查询依赖  

*  AssetBundleManifest.GetAllDependencies 获取 AssetBundle 的所有依赖层级
* AssetBundleManifest.GetDirectDependencies 获取 AssetBundle 直接依赖   

因该 API 会生成字符串数组，所以应尽量少用，且避免性能高峰时使用

**官方建议**，大部分场合下，在进入性能需求高的场景前，尽可能多地加载对象，尤其对于移动平台这种，访问本地存储慢，加载卸载对象引起内存流失会触发垃圾回收的平台    

#### Asset 分包策略

* 逻辑实体分包
* 对象类型分包
* 并发内容分包

##### 逻辑实体分包

依据资源在项目功能块的使用位置，如 UI、角色、环境和其他在生命周期中常出现的内容等分包

* 将所有 UI 的纹理和布局数据分包
* 将角色的模型和动画数据分包  
* 将多场景共用的纹理和模型分包

该分包方式适用于制作 DLC，可以只下载单个实体而无需下载无变化的资源，其关键点在于需要开发者清楚了解每个打包的资源所要用到的时机和位置

##### 对象类型分包

该方式适用于针对多平台分包，例如音频文件的压缩设置在 Windows 和 Mac OS 平台一样，另外由于纹理压缩格式和设置等改变频率远低于脚本和预设体，使用该分配方式可以使 AssetBundle 兼容更多的 Unity 版本  

##### 并发内容分包

并发内容分包可理解为以关卡为分组依据，将一个关卡内独有的角色、纹理、音乐等需要在同一时机加载的内容分为一包  

##### Tips

* 将常更新与不常更新内容分开  
* 将需要同时加载的对象分为一组，如一个模型，其所需的材质和动画分为一组  
* 若多个 AssetBundle 中的多个对象引用了其他 AssetBundle 中的单个 Asset，则将依赖项分离到单独的包中以减少重复  
* 确保两组完全不可能同时加载的对象不在用一包中，如低清和高清材质包  
* 若一个包中只有低于一半的对象被频繁加载，可将其拆分  
* 将一些同时加载的小包（资源少于5到10个）合并  
* 若一个包中的对象仅是版本不同，则可以使用 AssetBundle 变体  

#### Addressable

[Addressable](https://docs.unity3d.com/Packages/com.unity.addressables@1.8/manual/index.html) 系统为 Unity 新推出的资源管理系统，整合了 Unity 直接引用，Resources 和 AssetBundle 全部三种资源加载方式。通过可寻址资产的方式，便捷地实现了内容包的创建和部署。Addressable 系统使用异步加载的方式实现从任何位置加载任何依赖项，使得任何引用方式都更加便捷动态化

注意：需Unity 2018.3 及其以后版本

##### Addressable 优势

* 缩减迭代周期，无需修改代码优化内容  
* 自动依赖管理，将请求内容的依赖项一同加载  
* 自动内存管理，对管理资源自动引用计数
* 内容打包，负责构建和解析引用链，在将资源移动或重命名的情况下，依然可实现本地和远端部署  
* 配置文件，可配置多个配置文件，实现快速切换  

##### Addressable 概念

Addressable 由两个包组成，Addressable Assets package（主要功能） 和 Scriptable Build Pipeline package（依赖项）

* Address：资源的地址标记，用于运行时查询  
* AddressableAssetData：在项目 Assets 目录下用于存储 Addressable 资源的文件目录  
* Asset group：构建时处理的 Addressable 资产组  
* Asset group schema：数据构建时的配置  
* AssetReference：可根据需求延迟初始化的直接引用对象  
* Asynchronous loading：开发过程中无需更改代码也可修改资产位置和依赖项  
* Build script：打包资产，将 Address 和 Resources 映射  
* Label：为运行时加载相似项目提供额外的 Addressable Asset 标志    

##### Addressable 使用

###### Addressable 安装

1. 打开顶部工具栏 Window -> Package Manager，找到 Addressables，点击安装即可  

   ![打开Pacakage Manager](https://q6iqng.dm.files.1drv.com/y4m0DA6Rp509bhTJYzKhsP5B16iV6wGanCQLYB2LzgOR9vNxSoGjq6hYY_F5Y3oWPfMf27ghmfRyopVUwttXpJrkwAIG96SfwF2ep5eYMOI1j6ZdFD4tNyI8-wgwiANRjbcuGAtF_GkH6hpwNYfOdkvpsm0HHjXTWaOAFlWTxjf72sRGMXkyhfhMqUe0rVFbdPT9HiFmsFPRT6RGBvNcx-Otw?width=256&height=199&cropmode=none)

   ![安装 Addressable](https://q6j1kq.dm.files.1drv.com/y4mqiGo0Q7HbpiRxeTsKVm4r5c-RT3J8AlIkbgOgkwiU2WQ-CPCUbgszHM743nxI_yC0j-YTHuSGpKa9beVoFlysr2qByOaRJ_plHjJSQC5Qe-PuF21pyunZmUjgHLQqupmI8S2e9eI3VYZ--IB-sIKZHlrelGSEvY4Ct9xJOWejtedGmPMT05KssoBOj78nejBzP4sp1ruFw_T7uUSTvl9WQ?width=660&height=214&cropmode=none)

2. 安装完成后 Addressable 的主要功能都可在顶部工具栏 Window -> Asset Management -> Addressables 中找到   

   * Groups：Addressable 分组工具
   * Settings：Addressable 总体设置
   * Profiles：预设构建配置管理
   * EventViewer：监测引用计数工具  
   * Analyze：用于分析打包情况，检测重复等可自定义规则的分析工具
   * Hosting：模拟服务器

   ![打开 Addressables Group](https://q6hatq.dm.files.1drv.com/y4mDmHU-qIw34vCeDGBARgoCpiKXE5euG7IapacV9YD9GVMjJM9QanJBgb4fEprqVWjosKvO0A5nzHzM-ckQxzdWQC5apdmqNkGrJPheevtJGRZXHdfB8nRoCaKqSYADRlIVFkjAW_LTiknWrOvlrF2vbQ6f8SfaMxyQSdriBwvoNGlNm9_yhLS6L0eneZL7QcTLUgR1lAEl5trvvXqASm38g?width=636&height=660&cropmode=none)

3. 打开Group，创建新的配置，项目 Assets 目录下会自动创建一个 AddressableAssetData 目录，无需直接改动该目录  

   ![](https://q6iqqa.dm.files.1drv.com/y4mWgWzEQ0WeUQuaWPZev_oM4M5JDqoXccijo1kjLbZcW5YBicL43QMo6ofSl2wAHPBcAkRNugXkAznarx7Wt7p-kY_5EyXQyeCri8uyUHGsrX--v3lUfmAXihvoWQ-SgMXZS5XsC8oYmXcytLm-XX7N_isTwDIZ26soSNwDHmyT0ZWFRLfeAS1hcROJc7RezijQ8OYixMoTQnP0QFC8w3BhA?width=511&height=361&cropmode=none)

![](https://q6ijvw.dm.files.1drv.com/y4mRUSziz_ZzAXix6YyFaxACmNlCgs2Z_LnOmhdRXdIcMUqR3PuRaKX5Y4gp2IUhLn8TC4UeZX2pOQMXWskgX8AhQG_xm2Ows2otOpioRHtD7bSb44knt-qAbuhxz1ETzll-UrMdJI8J22F9sorIEQZ53_7v5jb2xf9hA2vs-r2dZ-KW8qwEua_ceyCsyWwym4NAxJC998DgAZu6l-6j7sttA?width=256&height=102&cropmode=none)

###### Addressable Group 配置

1. 在 Addressables Groups 窗口，右键或左上角 create 按钮即可创建 Group
2. 选中 Group，在 Inspector 中修改设置，Group 有三种配置项
   * Content Packing & Loading：打包的构建和加载路径，以及其他打包相关设置
   * Content Update Restriction：包更新限制
   * Resources and Built In Scenes：是否包含 Resources 或构建的场景
3. 添加 Asset 后，构建 Addressable Group  
   *  Addressables Groups 窗口，Build > New Build > Default Build Script
   * 或者使用 API `AddressableAssetSettings.BuildPlayerContent()`

###### Addressables Asset 配置和加载

1. 添加 Addressable Asset  

   - 将资源直接拖入 Addressables Groups 窗口下分组内即可  

   - 或者在 Project 中选择任何 Asset ，在 Inspector 下都可见名为 Addressable 的勾选框，勾选即可将该 Asset，可更改其 Addressable 名称，此时该 Asset 就被添加到默认的 Addressables Group 中了    

     ![](https://docs.unity3d.com/Packages/com.unity.addressables@1.8/manual/images/inspectorcheckbox.png)

2. (可选) 更改资源名，为资源添加 Label

3. 加载 Addressable Asset  

   * `Addressables.LoadAssetAsync<T>(string)` 异步加载资源对象
   * `Addressables.InstantiateAsync(string)` 场景中创建对象   
   * 或添加 `AssetReference`成员变量，在 Inspector 可选择 AssetReference 引用的资源对象

   ```c#
   public class AddressablesExample : MonoBehaviour {
       GameObject myGameObject;
           ...
           Addressables.LoadAssetAsync<GameObject>("AssetAddress").Completed += OnLoadDone;
       }
   
       private void OnLoadDone(UnityEngine.ResourceManagement.AsyncOperations.AsyncOperationHandle<GameObject> obj)
       {
           // In a production environment, you should add exception handling to catch scenarios such as a null result.
           myGameObject = obj.Result;
       }
   }
   ```


## 参考

[Unity读取内部、外部资源详解](https://gameinstitute.qq.com/community/detail/128044)  

[Unity资源管理](http://tonytang1990.github.io/2016/10/13/Unity资源/#Resources)

[The Addressable Asset System 正式版应用](https://zhuanlan.zhihu.com/p/77600079)