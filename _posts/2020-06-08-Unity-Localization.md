---
layout: post
title:  "Unity实践—本地化框架实现"
description: 实现一个易于扩展和使用的本地化框架
date:   2020-06-08 02:30:07 +0800
categories: [Unity,C#]
tag: [实践,本地化,多语言,框架]

---

基于 Unity 现有文件管理系统实现的本地化框架，代码已上传 Git 项目  [GRUnityTools](https://github.com/Warl-G/GRUnityTools.git)，可直接下载源码或通过 UPM 使用  

现已实现 Resources、AssetBundle 和 Addressable 资源管理方式，另外可根据需求自由扩展  

提供逐行解析，csv 解析和 json 解析，三种本地化文本解析方式，也可根据需求自由扩展

本文其他地址：[简书](https://www.jianshu.com/p/dd979319374a)	[知乎](https://zhuanlan.zhihu.com/p/146556193)	[掘金](https://juejin.im/post/5edd2e6de51d4578787bb0a4)

## 实现目标   

1. 支持语言的可扩展性
2. 加载方式的可扩展性
3. 文本解析的可扩展性
4. 动态加载本地化资源    
5. 快速的资源更新方式

## 实现方式  

1. SystemLanguage

   为确保不同的项目和开发对象的本地化内容的规范统一，直接使用 `SystemLanguage` 枚举型作为各种语言的唯一ID  

2. ILocalizationLoader  

   通过自定义接口的形式，将加载方法开放给开发者实现，使不同项目实现最大自由地定制  

3. ILocalizationParser

   通过自定义接口的形式，将文本解析方法开放给开发者实现，使不同项目实现最大自由地定制   

4. Resources，AssetBundle 和 Addressable  

   使用 Unity 提供的三种动态加载资源方法实现`ILocalizationLoader`接口，提供三种预设本地化资源动态加载器  

5. LocalizationComponent  

   为进一步简化开发过程，提供该脚本实现切换语言时本地化控件的自动批量更新，支持 Text、TextMesh、Image 和 SpriteRender  

## 项目结构  

* LocalizationManager
  * ILocalizationLoader
    * LocalizationResourcesLoader
    * LocalizationAssetBundleLoader
    * LocalizationAddressableLoader
    * ...LocalizationCustomLoader
  * IlocalizationParser
    * LocalizationDefaultParser
    * ...LocalizationCustomParser
* LocalizationComponent
  * LocalizationComponentItem

## 实现过程   

为节省篇幅，部分示例代码有删改，具体代码可见项目 [GRUnityTools](https://github.com/Warl-G/GRUnityTools.git)，并包含使用示例

### LocalizationManager  

最初设计时，仅为实现本地化文本加载与切换的简单功能，因此所有的加载和解析功能均由 `LocalizationManager` 单例完成

首先思考的问题是如何使不同项目对拥有同一套语言表，由我个人补充难免会有缺漏，无法覆盖所有需求，最终决定使用系统提供的枚举型`SystemLanguage`，基本涵盖了所有应用度比较广的语言，同时确认以枚举名称作为本地化文本文件名称用于加载，可加入序号作为语言顺序  

构建文件数据结构，用于匹配语言加载文件  

```c#
public struct LocalizationFile
{
    public int Index { get; }
    public SystemLanguage Type { get; }
    public string Name  { get; }
    public string FileName  { get; }
		public LocalizationFile(string fileName = null)
    {   
        if (!string.IsNullOrEmpty(fileName))
        {
            Name = fileName;
            string[] lan = fileName.Split('.');
          	Name = lan[1];
            bool success;
          	int index;
          	SystemLanguage lanuageType;
          	success = Int32.TryParse(lan[0], out index);
            Index = success ? index : -1;
            success = Enum.TryParse(Name, true, out lanuageType);
            Type = success ? lanuageType : SystemLanguage.Unknown;
        }
    }
}
```

LocalizatioManager 单例初始化时，使用 Resources 获取规定目录下所有文本文件以获取支持的语言列表  

```c#
TextAsset[] res = Resources.LoadAll<TextAsset>("Localization");
List<LocalizationFile> fileList = new List<LocalizationFile>();

for (int i = 0; i < res.Length; i++)
{
    TextAsset asset = res[i];
    LocalizationFile data = new LocalizationFile(asset.name);
    fileList.Add(data);
    Resources.UnloadAsset(asset);
}
```

使用 `Resources` 提供的接口加载规定子目录下的其他本地化资源  

```c#
Resources.LoadAsync<T>("Localization/Assets/" + assetPath);
```

最初使用的自定义的文本规则的 txt 文件，每行一个键值对，等号分隔键和值，使用`Regex.Unescape`方法将文本中的转义符转义，LocalizationManager 持有解析后的字典数据，供外部控件使用  

```c#
public Dictionary<string, string> ParseTxt(string txt)
{
    if (string.IsNullOrEmpty(txt))
    {
        return null;
    }
    string[] lines = txt.Split('\n');
    Dictionary<string, string> localDict = new Dictionary<string, string>();
    foreach (string line in lines)
    {
        if (!string.IsNullOrEmpty(line))
        {
            string[] keyAndValue = line.Split(new[] {'='}, 2);
            if (keyAndValue.Length == 2)
            {
                string value = Regex.Unescape(keyAndValue[1]);
                localDict.Add(keyAndValue[0], value);
            }
        }
    }

    return localDict;
}
```

### ILocalizationLoader  

由于 `Resources`自身的弊端和限制，需要使用更加灵活的资源动态加载方法，因此又加入了对 AssetBundle 和 Addressable 的支持，有关这两种方式的信息可看文章[Unity学习—资源管理概览](https://warl-g.github.io/posts/Unity-Resource-Manage/)  

在设计这两种加载方式时意识到，提供现成的加载方法会有大量的约束限制，无法做到通用，因此决定抽离 LocalizationManager 所需的两个固定的方法作为接口，以实现外部自定义  

```c#
public interface ILocalizationLoader
{ 
  	//用于加载支持语言列表
    void LoadAllFileListAsync(Action<LocalizationFile[]> complete);
  	//用于加载本地化资源
    void LoadAssetAsync<T>(string localizationFileName, string assetPath, bool defaultAsset, Action<T> complete) where T : Object;
}
```

首先将之前的 `Resources` 加载方式改写为实现该接口的`LocalizationResourcesLoader`，并扩展了`LocalizationAssetBundleLoader`和`LocalizationAddressableLoader`两个加载器  

#### LocalizationAssetBundleLoader

将不同语言的 AssetBundle 打包到 Assets/StreamingAssets/Localization 目录下，并使用类似于`Resources`的资源命名方式命名，加载 Localization 的 manifest 获取支持语言的 Bundle 清单

```c#
string bundlePath = Path.Combine(Application.streamingAssetsPath, FilesPath, FilesPath);
//加载自动生成的 Localization Bundle
var loadrequest = AssetBundle.LoadFromFileAsync("Assets/StreamingAssets/Localization/Localization");
loadrequest.completed += operation =>
{
  	//获取 Localization Bundle 的 Manifest 文件信息
    AssetBundleManifest manifest =
        loadrequest.assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
    var files = manifest.GetAllAssetBundles();

    List<LocalizationFile> fileList = new List<LocalizationFile>();
    for (int i = 0; i < files.Length; i++)
    {
        if (!files[i].Equals(CommonAssetsPath.ToLower()))
        {
            LocalizationFile data = new LocalizationFile(files[i]);
            fileList.Add(data);
        }
    }

    loadrequest.assetBundle.Unload(manifest);
}
```

加载资源  

```c#
AssetBundleCreateRequest loadrequest = AssetBundle.LoadFromFileAsync(Path.Combine(Application.streamingAssetsPath, FilesPath,
                    localizationFileName));
loadrequest.completed += operation =>
{
    var request = loadrequest.assetBundle.LoadAssetAsync<T>(assetPath);
    request.completed += asyncOperation => 
    { 
      	//get request.asset
    };
};  
```

同时为更便捷打包本地化 AssetBundle 包，增加快捷方法  

```c#
string[] paths = Directory.GetDirectories(localizationFilePath);
AssetBundleBuild[] buildMap = new AssetBundleBuild[paths.Length];

for (int i = 0; i < paths.Length; i++)
{
    string bundleName = Path.GetFileName(paths[i]);
    buildMap[i].assetBundleName = bundleName;
    var files = Directory.GetFiles(paths[i], "*", SearchOption.AllDirectories);

    List<string> assets = new List<string>();
    foreach (var file in files)
    {
        if (!file.EndsWith(".meta"))
        {
            var filePath = file.Replace(Application.dataPath, "Assets");
            assets.Add(filePath);
        }
    }

    buildMap[i].assetNames = assets.ToArray();
}

var parent = Path.GetFileName(localizationFilePath);
BuildPipeline.BuildAssetBundles("Assets/StreamingAssets/" + parent, buildMap,
                    BuildAssetBundleOptions.ChunkBasedCompression, target);
```



#### LocalizationAddressableLoader  

由于`Addressable`的特性，无法通过固定的资源路径获取文件获取支持语言，因此使用 `Addressable` 的 `Label`功能，给每种语言的本地化文本文件打上`Localization`标签，实现统一获取，且需将文本文件 address 与对应`SystemLanguage`相同

```c#
Addressables.LoadResourceLocationsAsync(KLocalizeFolder).Completed += handle =>
{
    List<LocalizationFile> fileList = new List<LocalizationFile>();
    if (handle.Status == AsyncOperationStatus.Succeeded)
    {
        foreach (var file in handle.Result)
        {
            LocalizationFile data = new LocalizationFile(file.PrimaryKey);
            fileList.Add(data);
        }
    }
};
```

使用 Addressables 加载资源  

```c#
Addressables.LoadAssetAsync<T>(assetAddress).Completed += handle =>
{
    //get request.asset
};
```

### ILocalizationParser  

由于编写的文本解析格式过于个性化，并不通用，因此扩展支持了 csv 和 json 的解析，同时仿照`ILocalizationLoader`方式，抽出解析方法到`ILocalizationParser`，以增加扩展性  

```c#
public interface ILocalizationParser
{
    Dictionary<string, string> Parse(string text);
}
```

#### LocalizationDefaultParser  

项目将上述的 txt、csv 和 json 解析方法汇总到 `LocalizationDefaultParser`中，通过`LocalizationFileType`决定解析方式   

### LocalizationComponent

LocalizationComponent 是一套用于挂载在场景对象的脚本，用于自动更新本地化文本和资源，使用`LocalizationComponentItem`在 Inspector 中将 Component匹配本地化键

```c#
[Serializable]
public class LocalizationComponentItem
{
    [SerializeField] internal Component component;
    public string localizationKey;
    public string defaultValue;
    public bool setNativeSize;
    internal Vector2 originalImageSize = Vector2.zero;
}

public class LocalizationComponent : MonoBehaviour
{
        public LocalizationComponentItem[] items; 
  			private void LocalizationChanged(LocalizationFile localizationFile)
        {
            foreach (var item in items)
            {
                if (!string.IsNullOrEmpty(item.localizationKey) && item.component != null)
                {
                    string value = LocalizationManager.Singleton.GetLocalizedText(item.localizationKey);
                    if (value == null)
                    {
                        value = item.defaultValue;
                    }

                    if (item.component is Text text)
                    {
                        text.text = value;
                    }
                    else if (item.component is TextMesh mesh)
                    {
                        mesh.text = value;
                    }
                    else
                    {
                        Image image = null;
                        SpriteRenderer spriteRenderer = null;

                        if (item.component is Image imageComponent)
                        {
                            image = imageComponent;
                            if (item.originalImageSize == Vector2.zero)
                            {
                                item.originalImageSize = image.rectTransform.sizeDelta;
                            }
                        }

                        if (item.component is SpriteRenderer rendererComponent)
                        {
                            spriteRenderer = rendererComponent;
                        }

                        if (image != null || spriteRenderer != null)
                        {
                            //替换图片
                            LocalizationManager.Singleton.LoadLocalizationAssetAsync(value, item.defaultValue,
                                delegate(Sprite sprite)
                                {
                                    if (sprite != null)
                                    {
                                        if (image != null)
                                        {
                                            Image img = (Image) item.component;
                                            img.sprite = sprite;
                                            if (item.setNativeSize)
                                            {
                                                img.SetNativeSize();
                                            }
                                            else
                                            {
                                                image.rectTransform.sizeDelta = item.originalImageSize;
                                            }
                                        }

                                        if (spriteRenderer != null)
                                        {
                                            spriteRenderer.sprite = sprite;
                                        }
                                    }
                                });
                        }
                    }
                }
            }
        }
  
}
```