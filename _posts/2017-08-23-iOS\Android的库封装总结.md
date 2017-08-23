---
layout:     post
title:      Android和iOS的静态库生成
subtitle:   
date:       2017-08-23
author:     Joy
header-img: img/page-bg-people.png
catalog: true
tags:
    - golang iOS Android
---
### iOS封装生成.framework

##### 库的概念

* 静态库

  > 链接时完整地拷贝到执行文件中，使用多少次就有多少份冗余的拷贝
  >
  > iOS里面静态库的形式.a和.framework（我们自己创建的）
  >
  > .a+.h+资源文件 = .framework

* 动态库

  > 链接时不复制，程序运行的时候由系统动态加载到内存中，且只加载一次
  >
  > iOS里面动态库的形式.dylib和.framework（系统的）

##### .framework 生成

* go编译生成.a文件，不同的芯片类型生成不同的.a文件
* 对全部的.a文件进行merge
* 通过xcode工具生成framework文件
* 配置framework文件需要的.plist文件、Headers文件



#### Android 生成.aar和.jar文件

* aar是Android中独有的库类包，jar是Java中特有的库类包
* jar是class文件的归档文件，不包括资源文件；aar包含资源文件

**需要继续探索的点：iOS支持的框架类型：arm64   x86_64   armv7**

