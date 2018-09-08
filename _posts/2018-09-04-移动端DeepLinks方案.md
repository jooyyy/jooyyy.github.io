## 应用内打开方案

Android 和 iOS 都先后支持了 URL Scheme 和 Deep Links 的方案

#### URL Scheme

URL Scheme 

格式类似 schema://new/list?key1=value1&key2=value2这种形式，App自己在内部对url进行解析和路由。微信内部不支持

* iOS 的配置

  ```
  // Info.plist文件添加配置
  
  <key>CFBundleURLTypes</key>
  	<array>
  		<dict>
  			<key>CFBundleURLName</key>
  			<string>com.dalong.wallet</string>
  			<key>CFBundleURLSchemes</key>
  			<array>
  				<string>wx95f0e21524161ca9</string>
  				<string>tencent1107702313</string>
  				<string>tencent1107702313.content</string>
  				<string>QQ42063229</string>
  				<string>wb3893335307</string>
  			</array>
  		</dict>
  	</array>
  	
  // AppDelegate 添加
  - (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url NS_DEPRECATED_IOS(2_0, 9_0, "Please use application:openURL:options:") __TVOS_PROHIBITED;
  
  - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(nullable NSString *)sourceApplication annotation:(id)annotation NS_DEPRECATED_IOS(4_2, 9_0, "Please use application:openURL:options:") __TVOS_PROHIBITED;
  
  // no equiv. notification. return NO if the application can't open for some reason
  - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options NS_AVAILABLE_IOS(9_0); 
  ```

* iOS配置

  ```
  <activity
              android:name=".Activities.AirdropDetailActivity"
              android:label="@string/app_name"
              android:theme="@style/AppTheme.NoActionBar">
              <intent-filter
                  android:autoVerify="true"
                  tools:targetApi="m">
                  <action android:name="android.intent.action.VIEW" />
  
                  <category android:name="android.intent.category.DEFAULT" />
                  <category android:name="android.intent.category.BROWSABLE" />
  	
  				/* 前两个 data 是属于  App links 的配置  */
                  <data
                      android:host="dalong.com"
                      android:pathPrefix="/client"
                      android:scheme="http" />
                  <data
                      android:host="dalong.com"
                      android:pathPrefix="/client"
                      android:scheme="https" />
                  <data
                      android:host="dalong.com"
                      android:pathPrefix="/client"
                      android:scheme="dalong" />
              </intent-filter>
          </activity>
  ```

  

#### Deep link

* *deep link* 指 app 在处理特定的 URI 的时候可以直接跳转到对应的页面或者触发特定的逻辑，而不仅仅是启动 app 

* *deffered deep link* 在未安装应用的情况下，希望用户安装 app 以后可以deep link 到对应的内容

  

#### 实现 deep link 

*  Universal Links （iOS9+，目前 iOS 版本是11.4+；Android M + ）

  直接用 http/https 协议的 link 打开应用，web 的兼容性更好



#### deep link 配置步骤

###### iOS 配置步骤

* 应用 Capabilities 添加 Associated Domains，必须用 applinks: 前置，例子【applinks:dalong.com】

  配置好这个域名之后，应用第一次打开会从 https://domain.com/apple-app-site-association下载这个文件

* 上传 apple-app-site-association

  内容形如：

  ```
  {
    "applinks": {
      "apps": [],
      "details": {
        "TBEJCS6FFP.com.domain.App": {
          "paths":[ "/share/*" ]
        }
      }
    }
  }
  ```

  TBEJCS6FFP是 Apple 开发者账号的TeamID，Developer Center 可查；com.domain.app 是 bundle ID。

* 在 App 里面实现处理逻辑

  在 AppDelegate 中实现 URL 处理

  ```
  
  // 处理 Universal Link
  - (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler {
      if ([userActivity.activityType isEqualToString:NSUserActivityTypeBrowsingWeb]){
          NSURL *url = userActivity.webpageURL;
          NSURLComponents *urlComponents = [NSURLComponents componentsWithURL:url resolvingAgainstBaseURL:NO];
          NSMutableDictionary *parameter = [NSMutableDictionary dictionary];
          for (NSURLQueryItem *item in urlComponents.queryItems) {
              [parameter setValue:item.value forKey:item.name];
          }
          NSString *info = [parameter valueForKey:@"info"];
          //  rest logic ...
      }
      return YES;
  }
  ```



###### Android 配置

* Manifest文件配置

* 服务端能同构 https 访问一个校验 json 文件

  * https://developer.android.com/training/app-links/#testing

  * 响应只能是“application/json”类型的Content-type，其他类型都不支持！

  * 校验不支持重定向，所以不要配置链接重定向。

  * 生成sha256指纹证书java命令

    >  keytool -list -v -keystore my-release-key.keystore

###### web 兼容性问题

* 微信内

  - iOS9以上
  - iOS9以下
    - 方案一：schema 方案，微信屏蔽，需要引导
    - 方案二：跳应用宝，应用宝跳转

* 微信外

  没什么兼容性问题

* 未安装应用处理

  正常链接 Universal link 或者 schema 地址，然后采用 timeout 延迟跳转到下载页面，js 里面做

  ```
  $('a').click(function() {
      location.href = '自定义 URL scheme 或者 universal link 地址';
      setTimeout(function() {
          location.href = '下载中间页面页';
      }, 250);
      setTimeout(function() {
          location.reload();
      }, 1000);
  }
  ```

  

###### deffered Deep Linking 方案

在用户下载之前拿到用户的设备信息，上传到服务端，用户下载第一次打开应用，同样上传用户设备信息，拿到服务端保存的页面信息，客户端跳转。

web 上能拿哪些数据来做为标识？

* 屏幕尺寸
* 设备操作系统
* 设备 IP
* 访问时间

以上这些信息，单个作为判断条件的话容易重复，但是如果作为一个集合，时间限制在5-10min 以内，那么出现重复的可能性就降低了不少。

！！iOS9 以上，还可通过 SFSafariViewController 与 safari 共享沙盒，这样 cookie 的 sessionid 作为标识就更准确率。

