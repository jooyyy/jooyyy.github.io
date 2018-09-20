## iOS 归纳	

#### UIStatusBarStyle

* 颜色控制

  ```
  // < iOS 9
  [[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
  
  // >= iOS 9
  // 非 UINavigationController
  -(void)viewDidload {
      ...
      [self setNeedsStatusBarAppearanceUpdate];
  }
  
  -(UIStatusBarStyle)preferredStatusBarStyle {
      return UIStatusBarStyleDefault;
  }
  
  // UINavigationController 中
  方案一：
  // 黑色背景，让文字变成白色
  self.navigationController.navigationBar.barStyle = UIBarStyleBlack;
  // 背景色白色
  self.navigationController.navigationBar.barStyle = UIBarStyleDefault;
  
  方案二：
  // UINavigationController 中不调用原因是 NavigationController 覆盖了 ViewController 的方法，在 NavigationController中调用 VC 的 preferredStatusBarStyle方法
  
  Nav* nav = [[Nav alloc] initWithRootViewController:vc];
  self.window.rootViewController = nav;
  
  @implementation Nav
  
  - (UIStatusBarStyle)preferredStatusBarStyle{
  UIViewController* topVC = self.topViewController;
  return [topVC preferredStatusBarStyle];
  }
  ```




## CocoaReactive

iOS 中的事件包括：

* Target 
* Delegate
* KVO
* 通知
* 时钟
* 网络异步回调



## 代码注释规范

* 方法集体注释

  ```
  #pragma mark-UITableViewDatasource
  ```

*  其他都比较基础

* 注 onvcat 开发注释插件 (VVDocumenter)[https://github.com/onevcat/VVDocumenter-Xcode]已经被XCode 8 + 采用，方法前 option + command + / 