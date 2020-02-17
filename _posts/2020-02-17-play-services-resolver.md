---
layout: post
title:  "Unity工具—Play Services Resolver 使用"
description: Unity 原生依赖管理工具 Play Services Resolver  
date:   2020-02-17 15:50:07 +0800
categories: [Unity]
tag: [Unity工具]
 
---

[unity-jar-resolver](https://github.com/googlesamples/unity-jar-resolver) 也叫 Play Services Resolver for Unity 是 Google 提供的面向 Unity 为解决 Android 与 iOS 原生项目依赖的工具库，目前Google Play services、FireBase、Facebook、Admob 等都使用了该库  

本文其他地址：[简书](https://www.jianshu.com/p/dc7489472156)	[知乎](https://zhuanlan.zhihu.com/p/107450061)	[掘金](https://juejin.im/post/5e4a633d518825490761e7b7)  

### Unity 原生依赖问题  

1. 将原生库整合入 Unity 项目过程复杂，且会导致 Plugin 负担过重
2. 使用 Plugin 可能需要处理依赖冲突
3. 为解决冲突可能需要删除部分文件而引出 bug  

### Play Services Resolver 功能

#### Android 依赖管理  
*Android Resolver* 组件能够**下载**和**整合**并解决 Android 库**依赖冲突**
如：多个代码对同一库的不同版本引用    

* 解决 plugin 之间的 Android 库冲突  
* 处理使用 Android 库（AARs, JARs）中的所有过程  
* （实验阶段）支持无需导包的 Java 组件最简化  

##### Android 解析过程  

默认 Android 解析策略为使用 Gradle 优先于直接整合入 Unity 工程，使用 Gradle 既可以用于内部构建也可用于导出Android 工程  

1. 删除之前解析结果以及位于 Plugins/Android 下标有 `gpsr` 的文件和目录  
2. 收集项目中所有 *Dependencies.xml 文件内的配置的安卓依赖项  
3. 运行 `download_artifacts.gradle` 解决冲突，成功则下载解析的 AAR 或 JAR  
4. 针对当前编译配置（内部或Gradle、导出或不导出）处理 AAR 和 JAR，该过程包含将每个引用的 applicationId 和工程的 bundle ID 打包到 AndroidManifest.xml   
5. 将处理好的 AAR 放入 Plugins/Android 目录下

#### iOS 依赖管理  
*iOS Resolver* 组件在对 iOS 平台进行构建时使用 CocoaPods 下载和整合 iOS 库解决版本兼容和重复问题

#### Unity Plugin 版本管理  
*Version Handler* 组件简化管理 Plugin 的传递依赖和升级过程  

	如：  
	PluginA 和 PluginB 分别引用了 LibC 的1.1和1.2版本  
	则 libC 的版本则取决于 PluginA 和 PluginB 的引入顺序
	先引入 PluginA 再引入 PluginB 则LibC为1.2 
	反之为1.1
方案  

列出一个包引用需求集合使得可以引用不同版本 Plugin
提供灵活 Plugin 最新版本选择逻辑

### Play Services Resolver 使用
若想将 unity-jar-resolver 集成到自定义 Plugin 中  

1. 需使用 Unity 命令行引入 play-services-resolver-*.unitypackage 并加入 -gvh_disable 选项
2. 导出自定 Plugin 时需加入   Assets/PlayServicesResolver 目录和 -gvh_disable 选项以使版本控制器正常  	

```shell
	Unity -gvh_disable \
      -batchmode \
      -importPackage play-services-resolver-1.2.46.0.unitypackage \
      -projectPath MyPluginProject \
      -exportPackage Assets MyPlugin.unitypackage \
      -quit
```
Version Handler 组件依靠推迟编辑器 DLLs 的加载以使自己第一个加载且能够确定启用最新版本的 plugin 组件。因 Play Services Resolver 在构建时已配置了资源的 metadata ，所以在编辑器组件被导入时无需首先启用。为确保在导入 Play Services Resolver.unitypackage 时保持该配置，需使用 ***-gvh_disable*** 选项避免 Version Handler 运行更改资源 metadata。

#### Android 使用
1. 插件包导入项目，若要将 Play Services Resolver 整合入自定义插件中，遵守上方使用说明
2. 复制```SampleDependencies.xml```到Editor目录下且以项目名重命名```*Dependencies.xml```  
3. 以上方使用说明导出 package   

若通过 Android SDK manager 安装组件：

```xml
<dependencies>
  <androidPackages>
    <androidPackage spec="com.google.android.gms:play-services-games:9.8.0">
      <androidSdkPackageIds>
        <androidSdkPackageId>extra-google-m2repository</androidSdkPackageId>
      </androidSdkPackageIds>
    </androidPackage>
  </androidPackages>
</dependencies>
```
通过 Maven central 安装组件：  

```xml
<dependencies>
  <androidPackages>
    <androidPackage spec="com.google.api-client:google-api-client-android:1.22.0" />
  </androidPackages>
</dependencies>
```

##### Android 其他功能  

可在顶部菜单栏 Assets > Play Services Resolver > Android Resolver 下找到

* Resolve 正常解析  
* Force Resolve 强制重新解析   
* Delete Resolved Libraries 删除已解析库   
* Display Libraries 显示库  
* Settings   
  * Use Gradle Daemon
  * Enable Auto-Resolution   在编辑器中自动下载和处理 Android 库
  * Enable Resolution On Build   在预编译阶段自动下载和处理 Android 库
  * Install Android Packages  
  * Explodes AARs 开关 Unity 下的 AAR 的自动解包（自动精简 ABI 和 Manifest 变量处理） 
  * Patch AndroidManifest.xml  将引用变量的 applicationid 和 bundle ID 一同打包到 Asset/Plugins/Android/AndroidManifest.xml 中
  * Patch mainTemplate.gradle  
  * Use Jetifier

#### iOS 使用  

使用方式同 Android ，但配置参数有所不同

```xaml
<dependencies>
  <iosPods>
    <iosPod name="Google-Mobile-Ads-SDK" version="~> 7.0" bitcodeEnabled="true"
            minTargetSdk="6.0" />
  </iosPods>
</dependencies>
```

使用脚本修改 Podfile  

```c#
using System.IO;
public class PostProcessIOS : MonoBehaviour {
[PostProcessBuildAttribute(45)]//must be between 40 and 50 to ensure that it's not overriden by Podfile generation (40) and that it's added before "pod install" (50)
private static void PostProcessBuild_iOS(BuildTarget target, string buildPath)
{
    if (target == BuildTarget.iOS)
    {

        using (StreamWriter sw = File.AppendText(buildPath + "/Podfile"))
        {   
            //in this example I'm adding an app extension
            sw.WriteLine("\ntarget 'NSExtension' do\n  pod 'Firebase/Messaging', '6.6.0'\nend");
        }
    }
}
```

##### iOS 其他功能  

可在顶部菜单栏 Assets > Play Services Resolver > iOS Resolver 下找到  

- Install Cocoapods  
- Settings  
  - Podfile Generation  生成 Podfile （需安装Cocoapods）
  - Cocoapods Intergration  整合策略，可选Xcode Workspace、Xcode Project、None
  - Use Shell To Execute Cocoapod Tool  
  - Auto Install Cocoapod Tools in Editor   

可配置参数  

* name  依赖项名称
* path 本地路径
* version 依赖版本
* bitcodeEnabled 默认 true
* minTargetSdk 最小支持 sdk
* configurations
* modular_headers 默认 false
* source
* subspecs  

#### Version Handler 使用  

1. 在想交由 Version Handler 管理的资源文件 (asset) 上添加 `gvh` 标签  
2. 在 `VERSION` 与当前 plugin 发布版本相同的资源文件上添加 `gvh_version-VERSION` 标签  
3. 可选：在编辑器的 DLL 上添加 `gvh_targets-editor` 标签并禁用 DLL 的 editor 目标平台，当导入 plugin 时，Version Handler 会自动启用最新版本 DLL  
4. 可选：若当前 plugin 包含其他 plugin，应给每个文件添加版本号并更改每个资源文件 GUID ，如此可使多个版本的 plugin 导入项目，Version Handler 会自动启用最新版本  
5. 创建一个名为 `MY_UNIQUE_PLUGIN_NAME_VERSION.txt` 的清单文件，该文件列出所有文件与工程根目录的相对路径，然后添加在该清单文件上添加 `gvh_manifest` 标签，表示该文件为 plugin 清单  
6. 使用自定义 plugin 重新分配 `Play Services Resolver`   

#### 源码构建  

由源码构建 plugin 首先需安装 Unity 并安装 iOS 和 Android 模块  

Linux / OSX 端可使用：  

```shell
./gradlew build
```

Windows：  

```shell
./gradlew.bat build
```

##### 发布  

1. 更新 `build.gradle` 的 `pluginVersion`   
2. 更新 `CHANGELOG.md`   
3. 使用 `./gradle release` 构建发布版本  
   - 更新 `play-services-resolver-*.unitypackage`  
   - 拷贝未解包的 plugin 到`exploded` 目录  
   - 更新`plugin` 目录下的 metadata 
4. 使用 `./gradle gitTagRelease`创建发布提交并打标签  
   - `git add -A` 记录文件的修改、创建、删除
   - `git commit --amend -a` 使用 change log 内容创建发布提交
   - `git tag -a RELEASE -m "version RELEASE"` 打标签 