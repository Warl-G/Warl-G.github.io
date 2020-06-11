---
layout: post
title:  "Unity工具—制作Package"
description: 制作 Unity 的本地和 Git 模块包
date:   2020-06-11 23:30:07 +0800
categories: [Unity]
tag: [Unity工具,Package,Git]

---

本文讲解如何制作 UPM 包，并通过本地和 Git 导入项目，参考[官方手册](https://docs.unity3d.com/Manual/CustomPackages.html)  

本文其他地址：[简书](https://www.jianshu.com/p/5ff2511fc824)	[知乎](https://zhuanlan.zhihu.com/p/112232175)	[掘金]()  

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

