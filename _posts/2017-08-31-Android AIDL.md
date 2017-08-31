---
layout:     post
title:      Android AIDL
subtitle:   
date:       2017-08-31
author:     Joy
header-img: img/page-bg-people.png
catalog: true
tags:
    - Android
---

### Adnroid 接口定义语言(AIDL)

可以利用它定义客户端与服务使用进程间通信（IPC）进行相互通信时头认可的编程接口。

> 注：只有允许不同应用的客户端用IPC方式访问服务，并且想要在服务中处理多线程时，才有必要使用AIDL。如果不需要执行跨越不同应用的并发IPC，就应该通过实现一个Binder 创建接口；或者如果想执行IPC，但根本不需要处理多线程，则使用 Messender 类来实现接口。

##### 定义 AIDL 接口

* 创建 .aidl 文件

  * 支持的数据类型：Java中的原语类型(int、long、char等)、String、CharSequence、List、Map

  * .aidl 文件中包括的所有代码注释都包含在生成的 IBinder 接口中

  * 只支持方法，不能公开 AIDL 中的静态字段

    ```
    package com.example.android;

    interface IRemoteService {
      	int getPid();
      	void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString);
    }
    ```

  * 只需要将 .aidl 文件保存在项目的 src/ 目录内，SDK工具会在项目的 gen/ 目录中生成 IBinder 接口文件

* 实现接口

  * Android SDK 工具基于.aidl 文件，使用 Java 编程语言生成一个接口。此接口具有一个名为 Stub 的内部抽象类，用于扩展 Binder 类并实现 AIDL 接口中的方法。需要扩展 Stub 类并实现方法。

  * 实现 .aidl 生成的接口，需要扩展 Binder 接口，并实现从 .aidl 文件继承的方法

    ```
    private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
      	public int getPid() {
          	return Process.myPid();
      	}

    	public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) {
          	//  Do something.
    	}
    }
    ```

    ​

* 向客户端公开该接口

  * 实现 Service 并重写 onBind() 以返回 Stub 类的实现

  * 一下是一个向客户端公开 IRemoteService 示例接口的服务示例

    ```
    public class RemoteService extends Service {
      	@Override
      	public void onCreate() {
          	super.onCreate();
      	}
      	
      	@Override
      	public IBinder onBind(Intent intent) {
          	return mBinder;
      	}
      	
      	private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
          	public int getPid() {
              	return Process.myPid();
          	}
          	public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) {
              	// Do something
          	}
      	}
    }
    ```

    现在，当客户端（如 Activity） 调用 bindService() 以连接此服务时，客户端的 onServiceConnected() 回调会接收服务的onBind() 方法返回的 mBinder 实例。

  * 当客户端在 onServiceConnected() 回调中收到 IBinder 时，它必须调用 YourServiceInterface.Stub.asInterface(service) 以将返回的参数转换成 YourServiceInterface 类型，例如：

    ```
    IRemoteService mIRemoteService;
    private ServiceConnection mConnection = new ServiceConnection() {
      	public void onServiceConnected(ComponentName className, IBinder service) {
          	mIRemoteService = IRemoteService.Stub.asInterface(service);
      	}
      	
      	public void onServiceDisconnected(ComponentName className) {
          	Log.e(TAG, "Servcie has unexpectedly disconnected");
          	mIRemoteService = null;
      	}
    }
    ```

#### 通过 IPC 传递对象

通过IPC接口把某个类从一个进程发送到另一个进程是可以实现的。不过必须确保该类的代码度 IPC 通道的另一端可用，并且该类必须支持 Parcelable 接口，Android 系统可通过它将对象分解成可编组到各进程的原语。

##### 创建支持Parcelabel 协议的类

* 让类实现 Parcelable 接口

* 实现 writeToParcel

* 为类添加一个名为 CREATOR 的静态字段，这个字段是一个实现 Parcelable.Creator 接口的对象

* 最后，创建一个声明可打包类的 .aidl 文件

  ```
  // Rect.aidl
  package android.graphics

  parcelabel Rect;
  ```

  ​

* 一下示例展示了 Rect 类如何实现 Parcelable 协议i

  ```
  public final class Rect implements Parcelable {
    	public int left;
    	public int top;
    	public int right;
    	public int bottom;
    	
    	public static final Parcelable.Creator<Rect> CREATOR = new Parcelable.Creator<Rect>() {
        	public Rect createFromParcel(Parcel in) {
            	return new Rect(in);
        	}
        	
        	public Rect[] newArray(int size) {
            	return new Rect[size];
        	}
        	
        	public Rect(){}
        	
        	private Rect(Parcel in) {
            	readFromParcel(in);
        	}
        	
        	public void writeToParcel(Parcel out) {
            	out.writeInt(left);
            	out.writeInt(top);
            	out.writeInt(right);
            	out.writeInt(bottom);
        	}
        	
        	public void readFromParcel(Parcel in) {
            	left = in.readInt();
            	top = in.readInt();
            	right = in.readInt();
            	bottom = in.readInt();
        	}
    	}
  }
  ```

  ​

#### Service