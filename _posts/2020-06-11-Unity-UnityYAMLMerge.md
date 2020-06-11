---
layout: post
title:  "Unity工具—版本合并UnityYAMLMerge"
description: Unity 版本管理合并工具
date:   2020-06-11 23:30:07 +0800
categories: [Unity]
tag: [Unity工具,版本管理]
---

Unity 内部集成的版本合并工具，可用于第三方版本管理工具解决 Scene 和 Prefab 冲突问题  

本文译自[官方手册](https://docs.unity3d.com/Manual/SmartMerge.html)

本文其他地址：[简书](https://www.jianshu.com/p/afd4034fdfe2)	[知乎](https://zhuanlan.zhihu.com/p/147683577)	[掘金](https://juejin.im/post/5ee24ca8e51d45783f1108ec)  

### 前提  

使用第三方版本管理工具需先将 Unity 的 

1. Edit > Project Settings > Version Control 选项改为`Visible Meta Files`

2. Edit > Project Settings > Asset Serialization 选项改为`Force Text`

   该操作是将 Unity Asset 文件的内容由二进制格式改为 yaml 文本格式，以便于版本管理识别

### 工具位置   

Windows下，Unity 安装目录下

```
C:\Program Files\Unity\Editor\Data\Tools\UnityYAMLMerge.exe

or

C:\Program Files (x86)\Unity\Editor\Data\Tools\UnityYAMLMerge.exe
```

Mac OSX，右键显示包内容

```
/Applications/Unity/Unity.app/Contents/Tools/UnityYAMLMerge
```

### 使用方法  

#### P4V  

1. 配置 > 合并（Preference > Merge）  
2. 选择*其他应用* （Other application）
3. 点击*添加*（Add） 按钮
4. 在扩展区选择 `.unity` 
5. 在应用栏（Application）输入`UnityYAMLMerge`路径
6. 在参数栏（Arguments）输入 `merge -p %b %1 %2 %r`
7. 点击保存（Save）

#### Git  

配置 Git，添加以下内容到`.git`或`.gitconfig`文件  

```
[merge]
tool = unityyamlmerge

[mergetool "unityyamlmerge"]
trustExitCode = false
cmd = '<path to UnityYAMLMerge>' merge -p "$BASE" "$REMOTE" "$LOCAL" "$MERGED"
```

#### Mercurial  

添加以下内容到`.hgrc`文件  

```
[merge-patterns]
**.unity = unityyamlmerge
**.prefab = unityyamlmerge

[merge-tools]
unityyamlmerge.executable = <path to UnityYAMLMerge>
unityyamlmerge.args = merge -p --force $base $other $local $output
unityyamlmerge.checkprompt = True
unityyamlmerge.premerge = False
unityyamlmerge.binary = False
```

#### SVN  

添加以下内容到`~/.subversion/config`

```
[helpers]
merge-tool-cmd = <path to UnityYAMLMerge>
```

#### TortoiseGit

1. 打开 配置 > 差异查看器 > 合并工具 点击*高级* （Preferences > Diff Viewer > Merge Tool 点击 Advance）

2. 在扩展名下拉栏选择 `.unity`

3. 在扩展程序区输入

   ```
   <path to UnityYAMLMerge> merge -p %base %theirs %mine %merged
   ```

#### PlasticSCM

1. 配置 > 合并工具 点击*添加*按钮 （Preferences > Merge Tool 点击 Add）

2. 选择*外部*（External）合并工具  

3. 选择*使用下方路径文件*（Use with files that match the following pattern）

4. 添加`.unity`后缀

5. 输入指令 

   ```
   <path to UnityYAMLMerge> merge -p "@basefile" "@sourcefile"  "@destinationfile" "@output"
   ```

### SourceTree  

1. 工具 > 选项 > 差异（Tools > Options > Diff）
2. *合并工具*下拉栏选择*自定义* （Custom）
3. 在*合并指令*栏（Merge Command）输入UnityYAMLMerge的路径
4. 在参数栏（Arguments）输入`merge -p $BASE $REMOTE $LOCAL $MERGED`



