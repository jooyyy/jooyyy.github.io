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

### Binder 归纳

> 在 Android 系统的 Binder 机制中，由一系列组件组成分别是 Client、Server、Service Manager和Binder驱动程序。组件间的关系如下：

![](/img/post-img-binder.png)

* Client、Server和Service Manager 实现在用户空间中，Binder驱动程序实现在内核空间中
* Binder驱动程序和Service Manager在Android平台中已经实现，开发者只需要在用户空间实现自己的Client 和Server
* Binder 驱动程序提供设备文件/dev/binder 与用户空间交互，Client、Server和Service Manager 通过open和ioctl文件操作函数与Binder驱动程序进行通信
* Client和Server之间的进程通信通过Binder驱动程序间接实现
* Service Manager 是一个守护进程，用来管理Server，并向Client提供查询Server接口的能力

#### Service Manager 是如何成为一个守护进程的

> Service Manager、Client和Server分别是运行在独立的进程当中，它们之间的通信也是采用Binder机制，Service Manager 在充当Binder机制的守护进程的角色的同时，也在充当Server的角色，一种特殊的Server。
>
> * 打开/dev/binder 文件：open("/dev/binder", O_RDWR);
> * 建立128K内存映射：mmap(NULL, mapsize, PROT_READ, MAP_PRIVATE, bs->fd, 0);
> * 通知Binder驱动程序它是守护进程：binder_become_context_manager(bs);
> * 进入循环等待请求的到来：binder_loop(bs, svcmgr_handler);

#### Server 和 Client 如何获得 Service Manager 

> 对于普通的Server来说，Client 如果想要获得 Server 的远程接口，那么必须通过 Service Manager 远程接口提供的 getService 接口来获得，这本身就是一个使用Binder机制来进行进程间通信的过程。而对于Service Manager 这个Server来说， Client 如果想获得Service Manager 远程接口，却不必通过进程间通信机制来获得，因为Service Manager 远程接口是一个特殊的Binder引用，它的引用句柄一定是0



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

    interface IMyService {
      	int getPid();
      	void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString);
    }
    ```

  * 只需要将 .aidl 文件保存在项目的 src/ 目录内，SDK工具会在项目的 gen/ 目录中生成 IBinder 接口文件

* 实现接口

  * Android SDK 工具基于.aidl 文件，使用 Java 编程语言生成一个接口。此接口具有一个名为 Stub 的内部抽象类，用于扩展 Binder 类并实现 AIDL 接口中的方法。需要扩展 Stub 类并实现方法。

  * 实现 .aidl 生成的接口，需要扩展 Binder 接口，并实现从 .aidl 文件继承的方法

    ```
    private final IMyService.Stub mBinder = new IMyService.Stub() {
      	public int getPid() {
          	return Process.myPid();
      	}

    	public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) {
          	//  Do something.
    	}
    }
    ```

    基于AIDL生成的java文件，涉及到几个类：

    ![](/img/post-img-aidl.png)

    AIDL生成的就是IMyService这个接口，Stub 和 Proxy是这个接口的两个内部类，分别是 Binder和BinderProxy的子类，IMyService 继承自IInterface，IInterface 是一个用于表达 Service 提供的功能的一个契约，也就是说IInterface里面有的方法，Service都能提供。

    IMyService 为啥要分 Stub 和 Proxy 呢？这是为了要适用于本地调用和远程调用两种情况。如果 Service 运行在同一个进程，那就直接用 Stub，因为它直接实现了 Service 提供的功能，不需要上任何IPC过程。如果Service运行在其他进程，那客户端使用的就是 Proxy。

    请求用 Proxy 发出去了，Service 怎么接受请求并作出响应呢，Stub继承自Binder，Binder有一个onTransact (int code, android.os.Parcel data, android.os.Parcel reply, int flags)方法，参数分别对应了被调函数编号、参数包、响应包、flags。当 Proxy 发起了一个请求，服务端中响应线程就会通过 JNI 调用到 Stub 类，然后执行里面execTransact 方法，进而转到 onTransact 方法。

    ```
    @Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
      switch(code) {
      	case INTERFACE_TRANSACTION: {
          	reply.writeString(DESCRIPTOR);
        	return true;
      	}
        case TRANSACTION_increaseCounter: {
          	data.enforceInterface(DESCRIPTOR);
          	int _arg0;
          	_arg0 = data.readInt();
          	int _result = this.increaseCounter(_arg0);
          	reply.writeNoException();
          	reply.writeInt(_result);
          	return true;
        }	
      }
      	return super.onTransact(code, data, reply, flags);
    }
    ```

     生成代码十分清晰，就是根据传过来的数据包做相应的操作，然后把结果写回数据包，Binder 驱动来分发这些包。这段代码运行在 Service 本地进程中。

    Proxy 中生成了increaseCounter 具体实现

    ```
    @Override 
    public int increaseCounter(int increment) throws RemoteException {
      	Parcel _data = Parcel.obtain();
      	Parcel _reply = Parcel.obtain();
      	int _result;
      	try {
          _data.writeInterfaceToken(DESCRIPTOR);
          _data.writeInt(increment);
          mRemote.transact(Stub.TRANSACTION_increaseCounter, _data, _reply, 0);
          _reply.readException();
          _result = _reply.readInt();
      	}
      	finally {
          	_reply.recycle();
          	_data.recycle();
      	}
      	return _result;
    }
    ```

    那如何判断 Service 是运行在同一进程还是不同进程呢？

    ```
    public static IBackgroundService asInterface(IBinder obj) {
      	if ((obj == null)) {
          	return null;
      	}
      	IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      	if (((iin != null) && (iin instanceof IBackgroundService))) {
          	return ((IBackgroundService) iin);
      	}
      	return new IBackgroundService.Stub.Proxy(obj);
    }

    public IInterface queryLocalInterface(String descriptor) {
      	if (mDescriptor.equals(descriptor)) {
          	return mOwner;
      	}
      	return null;
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
      	
      	private final IMyService.Stub mBinder = new IRemoteService.Stub() {
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

### Messenger 方式进行进程间通信

* 服务实现一个 Handler，由其接收来自客户端的每个调用的回调

* Handler 用于创建 Messager 对象（对 Handler 的引用）

* Messenger 创建一个 IBinder，服务通过 onBind() 使其返回客户端

* 客户端使用 IBindler 将 Messenger（引用服务的 Handler） 实例化，然后使用后者将 Message 对象发送给服务

* 服务在其 Handler 中（具体讲，是在 handleMessage()方法中）接收每个Message

  ```
  public class MessengerService extends Service {
    	static final int MSG_SAY_HELLO = 1;
    	
    	class IncomingHandler extends Handler {
        	@Override 
        	public void handleMessage(Message msg) {
            	switch (msg.what) {
                	case MSG_SAY_HELLO:
                		
                		break;
            		default:
            			super.handleMessage(msg);
            	}
        	}
    	}
    	
    	final Messenger mMessenger = new Messenger(new IncomingHandler());
    	
    	@Override
    	public IBinder onBind(Intent intent) {
        	return mMessenger.getBinder();
    	}
  }
  ```

  ​

  ```
  public class ActivityMessenger extends Activity {
    	Messenger mService = null;
    	boolean mBound;
    	private ServiceConnection mConnection = new ServiceConnection() {
        	public void onServiceConnected(ComponentName className, IBinder service) {
            	mService = new Messenger(service);
            	mBound = true;
        	}
        	
        	public void onServiceDisconnected(ComponentName className) {
            	mService = null;
            	mBound = false;
        	}
    	}
    	
    	public void sayHello(View v){ 
    		if (!mBound) {
            	return;
    		}
    		Message msg = Message.obtain(null, MessengerService.MSG_SAY_HELLO, 0, 0);
    		try {
            	mService.send(msg);
    		}catch (RemoteException e) {
            	e.printStackTrace();
    		}
    	}
    	
    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
        	super.onCreate(savedInstanceState);
        	setContentView(R.layout.main);
    	}
    	
    	@Override
    	protected void onStart() {
        	super.onStart();
        	bindService(new Intent(this, MessengerService.class), mConnection, Context.BIND_AUTO_CREATE);
    	}
    	
    	@Override
    	protected void onStop() {
        	super.onStop();
        	if (mBound) {
            	unbindService(mCounnection);
            	mBound = false;
        	}
    	}
  }
  ```

  Messenger 两个