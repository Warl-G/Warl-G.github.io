---
layout: post
title:  "Unity工具—制作与使用Package"
description: 制作 Unity 的本地和 Git 模块包
date:   2020-06-12 22:30:07 +0800
categories: [Unity]
tag: [Unity工具,Package,Git]

---

本文讲解如何制作 UPM 包，并通过本地和 Git 导入项目，参考[官方手册](https://docs.unity3d.com/Manual/CustomPackages.html)，实际项目可参考本人制作的工具包 [GRUnityTools](https://github.com/Warl-G/GRUnityTools)

本文其他地址：[简书](https://www.jianshu.com/p/de117ff4c937)	[知乎](https://zhuanlan.zhihu.com/p/147975801)	[掘金](https://juejin.im/post/5ee3912de51d45783f110a93)  

## Unity Package Manager  

UPM 是 Unity 提供的包管理系统，可快速便捷地下载和更新 Unity 功能，和发现与共享用户自定义组件

1. Unity Editor 下可在 Window > Package Manager 打开 UPM 窗口并安装官方包或通过 URL 添加自定义包 
2. 在项目路径下 `./Packages/manifest.json`，显示了所有依赖包名和版本，可根据格式添加自己所需的包

## 制作 Package  

### 内容  

Package 可包含

* C# 脚本
* 程序集 Assembly  
* 原生插件 Plugin
* 模型、贴图、动画和音频等其他资产 

也可添加其他信息文件

* 修改日志 CHANGELOG.md
* 说明文件 README.md
* 版权声明 LICENSE.md
* 文档 Document~ （文件夹名后加~可使工程忽略该文件夹，文档仍是 md）

另外每个包还包含一个 Package 清单，`package.json`，用于声明包体信息，包括名称、版本、依赖和仓库地址等  

### 命名规范

* 包名起始必须为`com.<company-name>`，例如`com.unity.timeline`
* 若在 UI 显示则需低于50个字符，否则可低于214个字符
* 仅包含小写字母、数字、连字符`-`、下划线`_`和点`.`
* 为表明命名空间，可在命名空间后缀加点。如`com.unity.2d.animation`和`com.unity.2d.ik`

### 文件布局  

官方手册推荐布局样式  

```
<root>
  ├── package.json
  ├── README.md
  ├── CHANGELOG.md
  ├── LICENSE.md
  ├── Editor
  │   ├── Unity.[YourPackageName].Editor.asmdef
  │   └── EditorExample.cs
  ├── Runtime
  │   ├── Unity.[YourPackageName].asmdef
  │   └── RuntimeExample.cs
  ├── Tests
  │   ├── Editor
  │   │   ├── Unity.[YourPackageName].Editor.Tests.asmdef
  │   │   └── EditorExampleTest.cs
  │   └── Runtime
  │        ├── Unity.[YourPackageName].Tests.asmdef
  │        └── RuntimeExampleTest.cs
  └── Documentation~
       └── [YourPackageName].md
```

另外对于一个工程多个 Package 的情况，可用如下布局

```
<root>
  ├── Packages
   		├── YourPackageName 
   		│		├── package.json
  		│		├── README.md
  		│		├── CHANGELOG.md
  		│		├── LICENSE.md
   		│		├── Editor
   		│		│		├── Unity.[YourPackageName].Editor.asmdef
   		│		│		└── EditorExample.cs
   		│		├── Runtime
   		│		│		├── Unity.[YourPackageName].asmdef
   		│		│		└── RuntimeExample.cs
   		│		├── Tests
   		│		│		├── Editor
   		│		│		│		├── Unity.[YourPackageName].Editor.Tests.asmdef
   		│		│		│		└── EditorExampleTest.cs
   		│		│		└── Runtime
   		│		│				├── Unity.[YourPackageName].Tests.asmdef
   		│		│				└── RuntimeExampleTest.cs
   		│		└── Documentation~
      │ 			└── [YourPackageName].md
   		└── YourAnotherPackage
   				├── Editor
   				├── Runtime
   				├── Tests
   				└── Documentation~
```

### Package 配置清单  

每个 Package 需有一个`package.json`文件，用于配置 Package 信息，在 Package 导入项目后，这些信息会在 Package Manager 窗口展示出来    

#### 必要信息  

* name  

  包名，依据上述命名规范

* version  

  版本号，需遵守 主版本.次版本.补丁版本 规则即 `x.x.x` 

#### 建议信息  

下述信息并非强制，但建议添加以更易提供他人使用  

* displayName  

  在编辑器中的展示名称  

* description

  简介，展示与 Package Manager 窗口中，支持 UTF-8 编码字符

* unity

  支持 Unity 最小版本，主版本号.次版本号 如 2018.3

#### 其他信息  

* unityRelease  

  指明某些特定 Unity 发行版本  

* dependencies

  json中以键值对出现的依赖包名和版本，如`"com.unity.some-package": "1.0.0"`

* keywords  

  关键词数组

* type

  内部使用类型

* author

  作者信息，可包含name、email 和 url

```json
{
  "name": "com.unity.example",
  "version": "1.2.3",
  "displayName": "Package Example",
  "description": "This is an example package",
  "unity": "2019.1",
  "unityRelease": "0b5",
  "dependencies": {
    "com.unity.some-package": "1.0.0",
    "com.unity.other-package": "2.0.0"
 },
 "keywords": [
    "keyword1",
    "keyword2",
    "keyword3"
  ],
  "author": {
    "name": "Unity",
    "email": "unity@example.com",
    "url": "https://www.unity3d.com"
  }
}
```

### Assembly Definition 文件  

包中的`.cs`脚本文件必须与程序集定义文件 `.asmdef`关联（在同一目录下），`.asmdef`等同于 .Net 生态的 C# 工程

若一程序集内代码有对其他程序集的代码的引用，则必须在`.asmdef`的 Inspector 中`Assembly Definition References`选项下添加对该程序集文件的引用

同时对于不同平台的代码，根据代码作用在`.asmdef`文件的`Platforms`选项下选定程序集运行平台  

对`.asmdef`文件命名建议依据*公司.功能.平台*划分，示例  ：

* Editor 平台代码：`Editor/MyCompany.MyFeature.Editor.asmdef`
* Runtime 代码：`Runtime/MyCompany.MyFeature.Runtime.asmdef`
* 测试代码：`Tests/Editor/MyCompany.MyFeature.Editor.Tests.asmdef`和`Tests/Editor/MyCompany.MyFeature.Runtime.Tests.asmdef`



### 发布与使用 Package

* 提供压缩包
* 发布到 Git，通过Package Manager 导入
* 搭建 Package 注册服务器，使用 npm 发布

#### Git 发布 Package  

1. 将完整项目或单一 Package 目录上传 Git 
2. 将 Package 所在目录拆分到单独分支
3. 为该分支打标签（打版本号）
4. 提交远端  

##### 实际操作

假设一个项目有两个包
包目录: `Assets/Packages/FirstPackage` 和 `Assets/Packages/SecondPackage`

```shell
# 使用 Git 命令，将第一个包拆分到分支 FirstPackage 分支，分支名可任意命名，但为他人使用方便最好与包名相同
git subtree split --prefix=Assets/Packages/FirstPackage --branch FirstPackage
# 为该分支打版本 Tag，Tag 也可自由命名，同样为方便使用，打版本号
# 因举例该项目存在两个包因此版本前加分支名作为前缀，若仅单一包可只打版本号
# 注：版本号与package.json中统一，避免混乱
git tag FirstPackage@1.0.0 FirstPackage  
# 分支和 Tag 推送远端
git push origin FirstPackage --tags 
```

#### Git 导入  

1. 在需导入的项目路径下`./Packages/manifest.json`文件中`dependencies`下添加包名和Git URL 组成的键值对，URL 后需加 # 和 Package 的分支名、标签或提交，如

   ```json
   {
     "dependencies": {
       "com.company.firstpackage": "https://github.com/company/PackageSample.git#FirstPackage"
       }
   }
   ```

   

2. 或者使用 Package Manager 窗口中的 + 按钮，`Add package from git URL`，直接添加上述样式的的 Git 地址

3. 其他链接形式可参考[官方手册](https://docs.unity3d.com/Manual/upm-git.html)

#### 本地导入  

MacOS 绝对地址  

```json
{
  "dependencies": {
    "com.company.firstpackage": "file:/Users/company/Assets/Packages/FirstPackage"
  }
}
```

Windows 绝对地址

```json
{
  "dependencies": {
    "com.company.firstpackage": "file:C:/Users/company/Assets/Packages/FirstPackage"
  }
}
```

相对地址

```json
{
  "dependencies": {
    "com.company.secondpackage": "file:../Packages/secondpackage"
  }
}
```

## 参考  

https://www.jianshu.com/p/153841d65846