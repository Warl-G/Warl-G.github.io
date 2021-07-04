---
layout: post
title:  "Unity实践—Unity 内置资源独立打包"
description: Unity 自定义 Addressables 打包脚本实现内质资源打包
date:   2021-07-05 00:00:07 +0800
categories: [Unity]
tag: [实践,Addressables]
---

针对内置资源重复打包冗余的问题，编写 Addressables Build 脚本将内置资源独立打包

## 什么是内置资源  

Unity 提供了一些内置资源，可在编辑器中找到内置资源包`unity_builtin_extra`  

* Windows: `~/Editor/Data/Resources/unity_builtin_extra`
* MacOS:`~/Unity.app/Contents/Resources/unity_builtin_extra` 

`unity_builtin_extra` 中包含了一系列默认 Shader 和贴图等资源，可在编辑器中直接选择

![](https://dm2305files.storage.live.com/y4mmT4GmOESf-Yj4xXbLEeku6KcZpswW0VChnprWCTy4fv92n0KWoMpTcycYJocT3XvU0C3BH4jiVqz01lKg_bbR44ZaTFS3WuEYCItP2qXjN5F-cRs4jpVVxV4GV05yyqwWji_XJdLdq3T5XUoKTOJFoBvDkR06T4OJ3xqredPuTSBWeKHdqZYx08B-91pNqJr?width=356&height=332&cropmode=none)![](https://dm2305files.storage.live.com/y4mulECCyBOMqZKTaMg97CwDbjw3XScDvbjHG40tWsBjmeppaL_gi9cvQGbhngUdVMUAVN5yUYaaON2HYzv7-ksm7JvEs01lo_vP2AQi5AzW14TfErvT0MA1A6tnKETA2OSUu3gDUZAR38ygv-A3QybREveeD9KyjirO2ERbnXbiPubVNoZt6hSWdk5tAXMNSQz?width=360&height=603&cropmode=none)

由上图可见内置贴图资源路径为 `Resources/unity_builtin_extra`，在代码中可使用`AssetDatabase.GetAssetPath` 得到同样的路径

但无法通过该路径读取资源，编辑器下可用接口`AssetDatabase.GetBuiltinExtraResource`加载内置资源，以下为内置贴图路径

```c#
"UI/Skin/UISprite.psd"
"UI/Skin/Background.psd"
"UI/Skin/InputFieldBackground.psd"
"UI/Skin/Knob.psd"
"UI/Skin/Checkmark.psd"
"UI/Skin/DropdownArrow.psd"
"UI/Skin/UIMask.psd"
```

另外还有`Runtime`还有接口`Resources.GetBuiltinResource`，但目前没有明确用法

## 为什么要将内置资源打包   

若制作多个使用了同样内置资源的 Prefab 且被分到了不同的 Bundle 中，`Addressables` 的 `Default Build Script`是不会统计这些引用而单独分包的，会导致内置资源被重复打进不同的 Bundle 中   

可通过创建使用`Knob`和`UISprite`的 Image Prefab 各两个，并分别打成四个 Bundle

![](https://dm2305files.storage.live.com/y4m5CCk3VqWJP4vJJct_BCJjkZpBsZ1iCPE8udzn-3X6ov3Xd8cHALXRL2G1POHRJc-f8IGFXnl67an2WRGTPy0LugkwEHDbkx2WD9JgR3MHsD6tnqjd-94T-sWHEs1jChFG2N1dEBF-CvTcupJuR5sU2mPoCsgcGn1LOlutWq8V5H7OFFQoQs_nRZSEkDXdVn5?width=660&height=97&cropmode=none)![](https://dm2305files.storage.live.com/y4mEPfEJk-woS8KbQUtfs_bja5fFfsFnxy9jNJgDMu8ONXWrPQ7E7pVfH-Sn7o1xRRsVBpFl1B3iP757U02svle-eLdMqU7xhf4vIujrXmBLefrSAZu8EHk7492Tu3Kquc_avm3qm3dBdDgBesDsgBm156-wJr71fH7VTwZEax-mFLUgseapNqj7P3VU3qRc-7R?width=250&height=94&cropmode=none)

通过对四个 Bundle 解包可看到使用相同资源的 Bundle 都有类似如下的内容（`Knob` 或 `UISprite`），data 部分就是资源实际的数据内容，被重复打进了两个包

```yaml
ID: 5424255917358561739 (ClassID: 213) Sprite
	m_Name "Knob" (string)
	m_Rect  (Rectf)
		x 12 (float)
		y 12 (float)
		width 40 (float)
		height 40 (float)
	m_Offset (0 0) (Vector2f)
	m_Border (0 0 0 0) (Vector4f)
	m_PixelsToUnits 200 (float)
	m_Pivot (0.5 0.5) (Vector2f)
	m_Extrude 1 (unsigned int)
	m_IsPolygon 0 (bool)
	m_RenderDataKey  (pair)
		first 0000000000000000f000000000000000 (GUID)
		second 10913 (SInt64)
	m_AtlasTags  (vector)
		size 0 (int)
...............................
...............................
			size 184 (int)
			data (UInt8) #0: 205 204 204 61 205 204 76 61 0 0 0 0 92 143 194 61 205 204 204 189 0 0 0 0 205
			data (UInt8) #25: 204 204 61 123 20 174 189 0 0 0 0 123 20 174 61 205 204 204 61 0 0 0 0 205 204
			data (UInt8) #50: 76 189 205 204 204 61 0 0 0 0 10 215 163 189 205 204 204 189 0 0 0 0 92 143 194
			data (UInt8) #75: 189 41 92 143 61 0 0 0 0 205 204 204 189 143 194 245 60 0 0 0 0 205 204 204 189
			data (UInt8) #100: 174 71 97 189 0 0 0 0 0 0 0 0 62 62 62 143 62 62 62 143 62 62 62 143 62
			data (UInt8) #125: 62 62 143 62 62 62 143 73 73 73 137 116 116 116 135 146 146 146 94 255 255 255 0 255 255
			data (UInt8) #150: 255 0 146 146 146 50 117 117 117 134 55 55 55 143 62 62 62 143 62 62 62 143 62 62 62
			data (UInt8) #175: 143 62 62 62 143 62 62 62 143
		m_Bindpose  (vector)
			size 0 (int)

...............................
...............................
```

此时一个 Bundle 的大小约为 8 KB  

![](https://dm2305files.storage.live.com/y4mw4QLA-S-gsZfqty-t1N9Ogqlz5WSt1vHKSwuSQ4lCtU0Zmg41YydeuRUuBTyf9s3CAzP4LCKnWHvM_0R0XaBzRIm0AJABLawCMhY97SBmENonuMv0R7AHma9Dz6BGHSOQolxJ4JYHDDJH9jjolhHe6ow7tjg5ERB3pW8n3svGUPJ-bYJpAKm1o9IgzJ7kSe0?width=328&height=288&cropmode=none)

若内置资源使用范围比较广泛且分包较多，也是有可能造成一定的空间浪费，因此可重写`Addressables`打包脚本，将使用的内质资源独立打包

## 编写 Addressables 打包脚本   

### 默认打包脚本  

首先可以查看`Addressables`的默认打包流程，在`Packages/Addressables/Editor/Build/DataBuilders`下可找到`Addressables`提供的几种预设打包模式脚本，其中`BuildScriptPackedMode.cs`即为`Default Build Script`   

```c#
static IList<IBuildTask> RuntimeDataBuildTasks(string builtinShaderBundleName)
{
    var buildTasks = new List<IBuildTask>();

    // Setup
    buildTasks.Add(new SwitchToBuildPlatform());
    buildTasks.Add(new RebuildSpriteAtlasCache());

    // Player Scripts
    if (!s_SkipCompilePlayerScripts)
        buildTasks.Add(new BuildPlayerScripts());
    buildTasks.Add(new PostScriptsCallback());

    // Dependency
    buildTasks.Add(new CalculateSceneDependencyData());
    buildTasks.Add(new CalculateAssetDependencyData());
    buildTasks.Add(new AddHashToBundleNameTask());
    buildTasks.Add(new StripUnusedSpriteSources());
    buildTasks.Add(new CreateBuiltInShadersBundle(builtinShaderBundleName));
    buildTasks.Add(new PostDependencyCallback());

    // Packing
    buildTasks.Add(new GenerateBundlePacking());
    buildTasks.Add(new UpdateBundleObjectLayout());
    buildTasks.Add(new GenerateBundleCommands());
    buildTasks.Add(new GenerateSubAssetPathMaps());
    buildTasks.Add(new GenerateBundleMaps());
    buildTasks.Add(new PostPackingCallback());

    // Writing
    buildTasks.Add(new WriteSerializedFiles());
    buildTasks.Add(new ArchiveAndCompressBundles());
    buildTasks.Add(new GenerateLocationListsTask());
    buildTasks.Add(new PostWritingCallback());

    return buildTasks;
}

protected virtual TResult DoBuild<TResult>(AddressablesDataBuilderInput builderInput, AddressableAssetsBuildContext aaContext) where TResult : IDataBuilderResult
{
  //////////////////////
  //////////////////////
  var builtinShaderBundleName = Hash128.Compute(GetProjectName()) + "_unitybuiltinshaders.bundle";
  var buildTasks = RuntimeDataBuildTasks(builtinShaderBundleName);
  buildTasks.Add(extractData);
  
  IBundleBuildResults results;
	using (m_Log.ScopedStep(LogLevel.Info, "ContentPipeline.BuildAssetBundles"))
	using (new SBPSettingsOverwriterScope(ProjectConfigData.generateBuildLayout)) // build layout generation requires full SBP write results
	{
	    var exitCode = ContentPipeline.BuildAssetBundles(buildParams, new BundleBuildContent(m_AllBundleInputDefs), out results, buildTasks, aaContext, m_Log);

	    if (exitCode < ReturnCode.Success)
	        return AddressableAssetBuildResult.CreateResult<TResult>(null, 0, "SBP Error" + exitCode);
   }
  //////////////////////
  //////////////////////
}
```

抛弃代码中对资源的预分析和配置过程，如上代码为开始构建的核心部分，在`DoBuild`方法中创建构建任务队列，使用`ContentPipeline.BuildAssetBundles`开始构建打包  

`RuntimeDataBuildTasks`任务队列中有一个任务`CreateBuiltInShadersBundle`的功能是找到打包资源中使用到的内置 Shader 并独立打包，分析其中核心方法

```c#
public ReturnCode Run()
{
  //获取所有依赖资源中的内置资源，内置资源的GUID都统一为 0000000000000000f000000000000000
    HashSet<ObjectIdentifier> buildInObjects = new HashSet<ObjectIdentifier>();
    foreach (AssetLoadInfo dependencyInfo in m_DependencyData.AssetInfo.Values)
        buildInObjects.UnionWith(dependencyInfo.referencedObjects.Where(x => x.guid == k_BuiltInGuid));

    foreach (SceneDependencyInfo dependencyInfo in m_DependencyData.SceneInfo.Values)
        buildInObjects.UnionWith(dependencyInfo.referencedObjects.Where(x => x.guid == k_BuiltInGuid));

    ObjectIdentifier[] usedSet = buildInObjects.ToArray();
    Type[] usedTypes = BuildCacheUtility.GetTypeForObjects(usedSet);

    if (m_Layout == null)
        m_Layout = new BundleExplictObjectLayout();

  //从依赖的内置资源中找到所有的 Shader 资源，并记录在指定的 Bundle 名下
    Type shader = typeof(Shader);
    for (int i = 0; i < usedTypes.Length; i++)
    {
        if (usedTypes[i] != shader)
            continue;

        m_Layout.ExplicitObjectLocation.Add(usedSet[i], ShaderBundleName);
    }

    if (m_Layout.ExplicitObjectLocation.Count == 0)
        m_Layout = null;

    return ReturnCode.Success;
}
```

### 脚本改写  

由上述代码可见，默认的打包脚本已经帮助我们筛选出了所有的内置资源，只是额外添加了 Shader 单一类型的筛选，因此直接改造`CreateBuiltInShadersBundle`即可   

1. 创建一个新的实现`IBUildTask`的类 `CreateBuiltInBundle`，主要代码内容与`CreateBuiltInShadersBundle`保持一致，构造方法记录两个 Bundle 名ShaderBundleName 和 BundleName ，一个用于打包内置 Shader，一个用于打包其他内置资源，并对做出如下修改

```c#
public ReturnCode Run()
{
    HashSet<ObjectIdentifier> buildInObjects = new HashSet<ObjectIdentifier>();
    foreach (AssetLoadInfo dependencyInfo in m_DependencyData.AssetInfo.Values)
        buildInObjects.UnionWith(dependencyInfo.referencedObjects.Where(x => x.guid == k_BuiltInGuid));

    foreach (SceneDependencyInfo dependencyInfo in m_DependencyData.SceneInfo.Values)
        buildInObjects.UnionWith(dependencyInfo.referencedObjects.Where(x => x.guid == k_BuiltInGuid));

    ObjectIdentifier[] usedSet = buildInObjects.ToArray();
    Type[] usedTypes = ContentBuildInterface.GetTypeForObjects(usedSet);

    if (m_Layout == null)
        m_Layout = new BundleExplictObjectLayout();
    
  // 将 Shader 和非 Shader 资源分别记录到两个不同的 Bundle 中
    Type shader = typeof(Shader);
    for (int i = 0; i < usedTypes.Length; i++)
    {
        m_Layout.ExplicitObjectLocation.Add(usedSet[i], usedTypes[i] == shader ? ShaderBundleName : BundleName);
    }

    if (m_Layout.ExplicitObjectLocation.Count == 0)
        m_Layout = null;

    return ReturnCode.Success;
}
```

2. 创建一个新的 Build Script 继承自`BuildScriptBase`，所有代码和`BuildScriptPackedMode.cs`保持一致，菜单名称配置可自定义

   将`RuntimeDataBuildTasks`中`buildTasks.Add(new CreateBuiltInShadersBundle(builtinShaderBundleName));`替换为改造后的`CreateBuiltInBundle`，并在`DoBuild`方法中配置 Bundle 名称   

```c#
static IList<IBuildTask> RuntimeDataBuildTasks(string builtinShaderBundleName, string builtinBundleName)
{
    var buildTasks = new List<IBuildTask>();

    // Setup
    buildTasks.Add(new SwitchToBuildPlatform());
    buildTasks.Add(new RebuildSpriteAtlasCache());

    // Player Scripts
    if (!s_SkipCompilePlayerScripts)
        buildTasks.Add(new BuildPlayerScripts());
    buildTasks.Add(new PostScriptsCallback());

    // Dependency
    buildTasks.Add(new CalculateSceneDependencyData());
    buildTasks.Add(new CalculateAssetDependencyData());
    buildTasks.Add(new AddHashToBundleNameTask());
    buildTasks.Add(new StripUnusedSpriteSources());
    buildTasks.Add(new CreateBuiltInBundle(builtinShaderBundleName, builtinBundleName));
    buildTasks.Add(new PostDependencyCallback());

    // Packing
    buildTasks.Add(new GenerateBundlePacking());
    buildTasks.Add(new UpdateBundleObjectLayout());
    buildTasks.Add(new GenerateBundleCommands());
    buildTasks.Add(new GenerateSubAssetPathMaps());
    buildTasks.Add(new GenerateBundleMaps());
    buildTasks.Add(new PostPackingCallback());

    // Writing
    buildTasks.Add(new WriteSerializedFiles());
    buildTasks.Add(new ArchiveAndCompressBundles());
    buildTasks.Add(new GenerateLocationListsTask());
    buildTasks.Add(new PostWritingCallback());

    return buildTasks;
}

protected virtual TResult DoBuild<TResult>(AddressablesDataBuilderInput builderInput, AddressableAssetsBuildContext aaContext) where TResult : IDataBuilderResult
{
  //////////////////////
  //////////////////////
  var builtinBundleName = Hash128.Compute(GetProjectName()) + "_unitybuiltin.bundle";
  var builtinShadersBundleName = Hash128.Compute(GetProjectName()) + "_unitybuiltinshaders.bundle";
  var buildTasks = RuntimeDataBuildTasks(builtinShadersBundleName, builtinBundleName);
  buildTasks.Add(extractData);
  //////////////////////
  //////////////////////
}
```

### 修改效果  

构建 Bundle 后，多出一个大小为 7 KB 的`defaultlocalgroup_unitybuiltin.bundle`，通过解包可见其中只有之前重复打包的 Knob 和 UISprite 两个内置资源，而之前的四个 Bundle 已不再包含具体的资源数据，仅包含一段简单的引用数据，同时单个包体的大小由之前的 8 KB 减小为 4 KB

![](https://dm2305files.storage.live.com/y4m22aT96FBcaXS7Me9PNsVL4_lLHvZ_g_e5n5M5xMOM-3sNhGBVVjPCwpWcBwg89K_geOSR0VSjZkQ3ifM1XSctveiU1sGN_858IXyFO05UOmKbMB9U_XqvgNBa2COYNqCtJSvcMxn9-KSED_BaM-29RXZt6g1UlpBI63ocFWArwAolwQnouqLdXExFXivZtOV?width=460&height=249&cropmode=none)

|                | Builtin 打包前                                               | Builtin 打包后                                               |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Bundle 数量    | 4                                                            | 5                                                            |
| 总 Bundle 大小 | 32 KB                                                        | 22 KB                                                        |
| 单个包体大小   | ![](https://dm2305files.storage.live.com/y4mw4QLA-S-gsZfqty-t1N9Ogqlz5WSt1vHKSwuSQ4lCtU0Zmg41YydeuRUuBTyf9s3CAzP4LCKnWHvM_0R0XaBzRIm0AJABLawCMhY97SBmENonuMv0R7AHma9Dz6BGHSOQolxJ4JYHDDJH9jjolhHe6ow7tjg5ERB3pW8n3svGUPJ-bYJpAKm1o9IgzJ7kSe0?width=328&height=288&cropmode=none) | ![](https://dm2305files.storage.live.com/y4mNTKC7XIlZeqjt8z10j5DfGlT1Y5ge2x3W5Xu51btTjKZKExGnG_lwJKu49TXyU0QG8esWlQyxVAHeviOGA8N1RpokgeDQFT7VplM9TrkeM1z5MYOnYnYo2u8TdVSg9zMPFL1Lu2VX5UzbxZoELG82n5wgeMKSubQFvhxashY1lIVZ6QOcbJmU36GprjxxR0F?width=312&height=274&cropmode=none) |



源码链接：[GRTools.Addressables · Warl-G](https://github.com/Warl-G/GRUnityTools/tree/GRTools.Addressables/Editor/PackScrips)

## 参考  

[Unity内置资源如何打包避免冗余 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/372926245)