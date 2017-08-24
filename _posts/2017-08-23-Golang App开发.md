---
layout:     post
title:      Golang App开发
subtitle:   
date:       2017-08-23
author:     Joy
header-img: img/page-bg-people.png
catalog: true
tags:
    - golang 
---
### Golang App开发

##### Golang环境搭建

* Mac 

  * brew install xxxx

* Windows 

  * ​暂时没配过

* 已经封装好的搭建工具

  ```
  curl http://kmgtools.qiniudn.com/v1/installKmg.bash?v=$RANDOM | bash
  ```

  * kmg install golang 
  * kmg install kmggopath



##### kmg 工具介绍

* 软件安装， 环境搭建

  * 安装 go、gomobile ，配置环境

    > kmg install golang 
    >
    > kmg install kmgGoPath

  * 安装软件

    > kmg install redis
    >
    > kmg install mysql
    >
    > kmg install tmux 
    >
    > kmg install git
    >
    > …...

* 客户端方法库生成

  * cgo
    * iOS 首先通过遍历函数名，找到需要加入到库的函数体以及它的引用，通过cgo生成对应的 \*.go 、\*.h 、\*.m文件，通过go build命令生成静态.a文件，然后用xcode的命令行工具生成.framework文件
    * 注意的点：
      * .xcodeproj 文件和framework在同一级目录下面
      * framework里面的文件名要和framework的文件名一致，否则无法import
      * 检查IPHONEOS_DEPLOYMENT_TARGET，参数为空会报错
      * 关闭bitcode配置

  * jni 

  * RPC（远程过程调用）

    >  允许一台计算机程序调用另一台计算机的子程序。
    >
    >  从面向对象的角度来看，就是从一个地方调用另一个地方的方法。距离是相对的，服务器之间可以叫RPC，同一台电脑不同的进程之间也可以叫RPC。

    * 运用到客户端到服务端的接口调用上，对网络请求和错误处理进行封装之后，调用者可以像调用一个对象上的方法一样方便地调用服务端API

* 管理后台快速搭建

  * kmgBootstrap Demo搭建
  * kmg make viewDemo

* 数据库的方法封装

  * kmgRedis 
  * kmgMysql

* VPN相关实现

  * 服务端
  * 客户端