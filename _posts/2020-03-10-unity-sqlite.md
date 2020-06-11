---
layout: post
title:  "Unity工具—Mono.Data.Sqlite 使用"
description: Unity 平台 Sqlite 集成与使用方法
date:   2020-03-10 17:30:07 +0800
categories: [Unity,C#]
tag: [Unity工具,Sqlite,数据库]
---

为实现 Unity 下操作数据库，集成 Mono.Data.Sqlite， 并为实现快捷操作封装了 SqliteHelper  

本文其他地址：[简书](https://www.jianshu.com/p/5ff2511fc824)	[知乎](https://zhuanlan.zhihu.com/p/112232175)	[掘金](https://juejin.im/post/5e6750c66fb9a07c9f3feeba)

[源码与文档](https://github.com/mono/mono/tree/master/mcs/class/Mono.Data.Sqlite)  

[微软 Xamarin 官方 Mono.Data.Sqlite Android 示例](https://docs.microsoft.com/en-us/xamarin/android/data-cloud/data-access/using-adonet?tabs=macos)  

[微软 Xamarin 官方 Mono.Data.Sqlite iOS 示例](https://docs.microsoft.com/en-us/xamarin/ios/data-cloud/data/using-adonet?tabs=macos)

[Microsoft.Data.Sqlite](https://docs.microsoft.com/en-us/dotnet/standard/data/sqlite/?tabs=netcore-cli) 是微软 .NET 官方提供的 Sqlite 库，因为由同一作者编写，两者十分相似，所以 .NET [官方文档](https://docs.microsoft.com/en-us/dotnet/api/microsoft.data.sqlite?view=msdata-sqlite-3.1.0)可用于参考 

另附 C# API 查询工具：[HotExamples](https://csharpdoc.hotexamples.com/namespace/Mono.Data.Sqlite)

### 集成 Sqlite   

参考此非常全的集成[流程](https://stackoverflow.com/questions/50753569/setup-database-sqlite-for-unity)

#### 获取 Mono.Data.Sqlite.dll

想在 Unity 中使用 Sqlite 需要使用 `Mono.Data.Sqlite.dll`，该库为C# API，可直接从 Unity 客户端中获取

以 Mac 为例，大部分网上提供的目录为 `/Unity.app/Contents/MonoBleedingEdge/lib/mono/2.0-api/Mono.Data.Sqlite.dll` 但实际使用会发现该目录下的 dll 会出现版本无法兼容、提示加载失败等问题  

后发现 `/Unity.app/Contents/Mono/lib/mono/2.0/Mono.Data.Sqlite.dll` 更加稳定，经测试 2019.2.0f1、2018.4.16f1、2017.4.37f1 之间可互相使用，无异常  

另外其他博文有提到不同版本 Unity 会包含不同版本的  `Mono.Data.Sqlite.dll` ，会引发兼容性问题，使用时注意 PlayerSetting 中 `Api Compatibility Level` 需选择 `.NET Standard 2.0` ，但本文测试设置为 `.NET 4.x` 也能正常运行

#### iOS 与 macOS 环境配置  

无需额外配置，直接使用即可  

#### Android 环境配置  

Android 还需 `libsqlite3.so` ，该库为 sqlite3 由 C 编译得来，为适配不同的 Android 架构可下载  

[armeabi-v7a](https://github.com/Warl-G/GRUnityTools/blob/master/Assets/GRTools/Runtime/DataBase/Sqlite/Plugins/Android/libs/armeabi-v7a/libsqlite3.so) 放入 `Assets/Plugins/Android/libs/armeabi-v7a`

[arm64-v8a](https://github.com/Warl-G/GRUnityTools/raw/master/Assets/GRTools/Runtime/DataBase/Sqlite/Plugins/Android/libs/arm64-v8a/libsqlite3.so) 放入 `Assets/Plugins/Android/libs/arm64-v8a`  

[x86](https://github.com/Warl-G/GRUnityTools/blob/master/Assets/GRTools/Runtime/DataBase/Sqlite/Plugins/Android/libs/x86/libsqlite3.so) 放入 `Assets/Plugins/Android/libs/x86`  

对应 so 文件放入目录后选中，于 Inspector 的 platform 仅选择 Android，架构根据版本选择  

<img src="https://mh3ttq.dm.files.1drv.com/y4m3_5a1yIm5Z-uRT3NaCFkd2AazWIoV2-em71GU5DlDi7y5317_jBZ2xDzAzsBZZwfOq3QyDpVOSEq28gbY-CKfKvW2R-0p4n2WysFtyaf9pi-KVUDj-3anqjFRFpzItxSWZ7zEol-xTV9V57YZAlYfIDn7hKC4uUdG-6cufY9GbwOAf7fvlKOLn-vuIc36uQEm6b1yMQ88EqoVD1iUlBHjg?width=419&amp;height=362&amp;cropmode=none" style="zoom:80%;" />

#### Windows 环境配置  

Windows 需额外添加 sqlite3.dll 可从[官网下载](https://www.sqlite.org/download.html)  

Windows 分为 x86 和 x64 架构，根据需求添加于 Unity 项目中 `Asset/Plugins/x86` 和  `Asset/Plugins/x86_64`目录下，选中该dll，于 Inspector 中仅勾选 Standalone 平台和对应架构   

<img src="https://mh2kew.dm.files.1drv.com/y4mi7VlCgTQPnlzBCRi-qiszTBkbkuE1IFtxADkcviYNBtgJ6zpmSfcfKcLRVG1_RZQxUPvetNnam5qvOIzC6uh_9x3fEswluyn9Iu1ExHXTBZd_StWth5kWd-hmoWnwnVOVCavd2CS2dYXlslQDRBkzq1AI2kORpYN8gQIZREK_D_iUJG0nCPl7wEeW3kD3hlhIfLO4AaM9-8wmkKYdyQ95w?width=910&amp;height=842&amp;cropmode=none" style="zoom:40%;" />

#### 疑问  

1. `System.Data.dll` 没用到

   网上很多文章（甚至 Xamarin 官方 ）提到需将与 `Mono.Data.Sqlite`  同目录下的 `System.Data.dll` 一同添加到 Plugin 中（或全平台或仅 Android 平台）以保证版本同一，但实际测试发现，没有导入也可正常运行

2. `sqlite3.dll` 仅用于 Windows 平台

   另有说法，Android 需添加 `sqlite3.dll` 到 Plugin，但实际测试没添加也能正常运行

   

### Mono.Data.Sqlite 使用  

#### SqliteConnection   

1. 打开与关闭数据库  

   ```c#
   //需带'Data Source='前缀
   SqliteConnection connection = new SqliteConnection("Data Source=" + dbPath);
   //打开
   connection.Open();
   //关闭
   connection.Close();
   ```

#### SqliteCommand  

1. 构造操作指令  

   ```c#
   SqliteConnection connection = new SqliteConnection("Data Source=" + dbPath);
   //包括但不限于以下方法
   SqliteCommand command = new SqliteCommand("SELECT * FROM table");
   SqliteCommand command = new SqliteCommand("SELECT * FROM table", connection);
   SqliteCommand command = new SqliteCommand("SELECT * FROM table", connection, transaction);
   SqliteCommand command = connection.CreateCommand();
   command.CommandText = "SELECT * FROM table";
   
   
   ```

2. 执行与取消  

   ```c#
   SqliteCommand command = connection.CreateCommand();
   command.CommandText = "SELECT * FROM table";
   
   //执行并取得结果
   SqliteDataReader reader = command.ExecuteReader();
   //执行并返回执行行数 仅适用于INSERT、DELETE、UPDATE，其他语句返回0官方文档说返回-1
   int executeRowsCount = command.ExecuteNonQuery();
   //执行并返回结果的第一行第一列数据或 null
   var obj = command.ExecuteScalar();
   
   //取消
   command.Cancel();
   
   //异步执行
   command.ExecuteReaderAsync();
   command.ExecuteNonQueryAsync();
   command.ExecuteScalarAsync();
   
   ```

3. 添加参数  

   ```c#
   SqliteCommand command = connection.CreateCommand();
   //在SQL语句中使用key替代值，其中key 前需添加 :、@或$作为前缀
   command.CommandText = "INSERT INTO table (col1,col2,col3) VALUES ($key1,:key2,@key3);";
   command.Parameters.AddWithValue("$key1", 1);
   command.Parameters.AddWithValue(":key2", "value2");
   command.Parameters.Add(new SqliteParameter("@key3","value3"));
   //执行并取得结果
   SqliteDataReader reader = command.ExecuteReader();
   ```

4. 执行事务  

   ```c#
   using (var command = connection.CreateCommand())
   //开始事务
   using (var transaction = connection.BeginTransaction())
   {
     	try
       {
         	//执行数据库操作
         	command.CommandText = "INSERT INTO table (col1,col2,col3) VALUES ('value1','value2','value3')";
         	command.ExecuteNonQuery();
         	command.CommandText = "SELECT * FROM table";
         	SqliteDataReader reader = command.ExecuteReader();  
         	reader.Close();
         	//提交
     			transaction.Commit();
     	}
     	catch(SqliteException e)
       {
         	//回滚
           transaction.Rollback();
       }
   }
   ```

   

#### SqliteDataReader   

1. 读取结果  

   ```c#
   SqliteDataReader reader = command.ExecuteReader(); 
   //若一次执行多条 SQL 语句返回多组结果，使用 NextResult 到下一组
   While(reader.NextResult())
   {
     	//遍历该组结果每一行数据
   		While(reader.Read())
   		{
     			//获取该行列数
     			int count = reader.FieldCount;
     			//获取第一列数据
     			var obj = reader[0];
     			//获取指定列数据
     			var obj = reader["colum1"];
     			//获取第二列数据并以int类型返回
     			int result = reader.GetInt32(2);
   		}
   }
   
   //使用完需关闭reader，否则同一个 SqliteCommand 对象的下次查询会出问题
   reader.Close();
   //或使用 using 包裹 SqliteDataReader  
   using(SqliteDataReader reader = command.ExecuteReader()){
   
   }
   ```

   

#### 疑问  

1. Android Connection 要求"Data Source="替换为"URI=file:"  

   没换也正常  

   

### SqliteHelper  

SqliteHelper 为我封装的数据库操作工具，实现了部分基本的数据库快捷指令和数据库操作队列功能  

该工具已加入工具包 [GRUnityTool](https://github.com/Warl-G/GRUnityTools)，[文档](https://github.com/Warl-G/GRUnityTools/tree/master/Assets/GRTools/Documents/SqliteHelper.md)   

SqliteHelper 对 Mono.Data.Sqlite 中的 SqliteCommand 进行扩展，使 SqliteCommand 对象可直接快捷调用增删改查等操作  

SqliteHelperQueue 通过使用 [TaskQueue](https://github.com/Warl-G/GRUnityTools/blob/master/Assets/GRTools/Documents/ThreadQueue.md) 创建串行队列，将通过 SqliteHelperQueue 调用的数据库操作放入同一队列中，保证数据库的线程安全  