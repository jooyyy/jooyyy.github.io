---
layout:     post
title:      Android Gradle 详解
subtitle:   
date:       2017-08-25
author:     Joy
header-img: img/page-bg-people.png
catalog: true
tags:
    - Gradle
---

#### Android Gradle 详解

##### Groovy介绍

Groovy是在java平台上的、具有像Python、Ruby和Smalltalk语言特性的灵活动态语言，Groovy保证了这些特性像Java语法一样被Java开发者使用。

##### Groovy部署

```
- curl -s get.sdkman.io | bash
- source "$HOME/.sdkman/bin/sdkman-init.sh"
- sdk install groovy
- groovy -version
```

关于Groovy语言的介绍有一篇不错的文章[Here](http://www.infoq.com/cn/articles/android-in-depth-gradle)

##### Gradle介绍

Gradle是一个编程框架，Android中用这个框架来完成App的编译打包

##### Gradle部署

```
sdk install gradle 4.1（前提是你已经部署了sdkman工具）
```

##### Gradle基本组件

每一个待编译的工程都叫一个Project，每一个Project在构建的时候都包含一系列的Task。在Android项目中，每一个App和每一个Library都是单独的Project，而在每一个Project的根目录下都需要有一个build.gradle，编译脚本，类似于Makefile。

主项目目录下也有一个build.gradle，用来配置其他的子Project。同时有一个settings.gradle，用来告诉Gradle主项目有多少个Project，方便Multi-Projects Build。

settings.gradle例子

```
include ':app'
include ':app/demoAar/aar/demoAar'
include 'CPosSystemSdk','CPosDeviceSdk'
```

另外，settings.gradle除了可以include外，还可以设置一些函数，这些函数会在gradle构建整个工程任务的时候执行，比如：

```
def init(){
  	println "=====> start init gradle"
  	// 这里可以来一波操作
}
inclue ':app'
...
```

* gradle projects 查看工程信息


* gradle tasks 查看任务信息

  > 对于Multi-project在根项目中，需要指定路径：gradle [project-path]:tasks，如果已经cd到那个目录了，就不需要指定路径

* gradle task-name 执行任务

  > gradle clean 是执行清理任务
  >
  > gradle properites 用来查看所有属性信息

##### Gradle工作流程

![](/img/post-gradle-01.png)

* 初始化阶段，前面说了在执行settings.gradle
* 配置阶段，解析每一个project中的build.gradle，内部Tasks会被添加到一个有向图中，用于解决执行过程中的依赖关系
* 执行阶段

##### Gradle API实例详解

Gradle主要有三种对象，与三种不同的脚本文件对应，在gradle执行的时候，会将脚本转换成对应的对象：

* Gradle 对象：在执行gradle xxx 的时候会从默认配置脚本中构造出一个Gradle对象，全局对象，一般很少配置这个脚本文件

* Project对象：每一个build.gradle都会转换成一个project对象

  * 对应具体工程，需要加载所需要的插件，调用aplly函数

    ```
    apply plugin: 'com.android.library'  		// 如果是编译Library，则加载此插件
    apply plugin: 'com.android.application'  	// 如果是编译Android App，则加载此插件
    ```

  * 加载其他的gradle文件

    **一个常见的场景是，常用的函数被封装到一个utils.gradle文件中，在其他的build.gradle中去加载这个工具文件**

    ```
    apply from: rootProject.getRootDir().getAbsolutePath() + "/utils.gradle"
    ```

  * 设置属性

  * Task介绍

  ​

* Settings对象：每一个settings.gradle都会转换成一个settings对象

```
//Library工程必须加载此插件。注意，加载了Android插件就不要加载Java插件了。因为Android  
//插件本身就是拓展了Java插件  
apply plugin: 'com.android.library'   
//android的编译，增加了一种新类型的ScriptBlock-->android  
android {  
       //你看，我在local.properties中设置的API版本号，就可以一次设置，多个Project使用了  
      //借助我特意设计的gradle.ext.api属性  
       compileSdkVersion =gradle.api  //这两个红色的参数必须设置  
       buildToolsVersion  = "22.0.1"  
       sourceSets{ //配置源码路径。这个sourceSets是Java插件引入的  
       main{ //main：Android也用了  
           manifest.srcFile 'AndroidManifest.xml' //这是一个函数，设置manifest.srcFile  
           aidl.srcDirs=['src'] //设置aidl文件的目录  
           java.srcDirs=['src'] //设置java文件的目录  
        }  
     }  
   dependencies {  //配置依赖关系  
      //compile表示编译和运行时候需要的jar包，fileTree是一个函数，  
     //dir:'libs'，表示搜索目录的名称是libs。include:['*.jar']，表示搜索目录下满足*.jar名字的jar  
     //包都作为依赖jar文件  
       compile fileTree(dir: 'libs', include: ['*.jar'])  
   }  
}  //android SB配置完了  
//clean是一个Task的名字，这个Task好像是Java插件（这里是Android插件）引入的。  
//dependsOn是一个函数，下面这句话的意思是 clean任务依赖cposCleanTask任务。所以  
//当你gradle clean以执行clean Task的时候，cposCleanTask也会执行  
clean.dependsOn 'cposCleanTask'  
//创建一个Task，  
task cposCleanTask() <<{  
    cleanOutput(true)  //cleanOutput是utils.gradle中通过extra属性设置的Closure  
}  
//前面说了，我要把jar包拷贝到指定的目录。对于Android编译，我一般指定gradle assemble  
//它默认编译debug和release两种输出。所以，下面这个段代码表示：  
//tasks代表一个Projects中的所有Task，是一个容器。getByName表示找到指定名称的任务。  
//我这里要找的assemble任务，然后我通过doLast添加了一个Action。这个Action就是copy  
//产出物到我设置的目标目录中去  
tasks.getByName("assemble"){  
   it.doLast{  
       println "$project.name: After assemble, jar libs are copied tolocal repository"  
        copyOutput(true)  
     }  
}  
/* 
  因为我的项目只提供最终的release编译出来的Jar包给其他人，所以不需要编译debug版的东西 
  当Project创建完所有任务的有向图后，我通过afterEvaluate函数设置一个回调Closure。在这个回调 
  Closure里，我disable了所有Debug的Task 
*/  
project.afterEvaluate{  
    disableDebugBuild()  
}  
```