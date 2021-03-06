
### iOS原生热更新

最终效果： 代码在保存之后，立马在模拟器上看到修改后的效果， 避免Command+R重新编译耗费时间的问题； 如果APP页面层级太深的话，传统调试要一步步点进到指定页面，使用该方案直接就能看到效果，所见即所得。

---



每次都被我们项目的编译速度整的快没脾气了，一直想着优化项目的编译速度。 想想之前做的RN项目的热部署效果真的很爽，不爽之余想到：他用个杂交品种能热部署，而我用苹果亲儿子没道理不行啊！能不能搞个runtime之类的跟新啊。
所以花了一个上午时间，终于找到了这个成吨减少工作量的方案。

超级简单，只有三步：
1、一个工具
2、选定项目目录
3、把一个文件放到项目中

无需其他任何配置，不对项目结构造成任何侵害。

----
### 1、工具下载 InjectionIII
InjectionIII 是我们需要用到个一个工具，不要因为要用一个工具而厌烦这个方案，它很简单。
它是免费的，app store 搜索：InjectionIII，Icon是 一个针筒。
也是开源的，
<br/>
GitHub链接： [https://github.com/johnno1962/injectionforxcode](https://github.com/johnno1962/injectionforxcode)
<br/>
App Store链接： [ https://itunes.apple.com/cn/app/injectioniii/id1380446739?mt=12]( https://itunes.apple.com/cn/app/injectioniii/id1380446739?mt=12)


### 2、配置路径
打开InjectionIII工具，选择Open Project，选择你的代码所在的路径，然后点击Select Project Directory保存。

![image.png](https://upload-images.jianshu.io/upload_images/2953683-861930b2a363de45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/2953683-7e7945bddb3cba56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意：InjectionIII 的File Watcher选项要保持选中状态。

### 3、导入配置文件
使用 CocoaPods 集成
```
pod '~> InjectionIIITools'

```
### 4、启动项目，修改验证
在Xcode Command+R运行项目 ，看到Injection connected 提示即表示配置成功。
![image.png](https://upload-images.jianshu.io/upload_images/2953683-0466996daaf1b816.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在需要修改的页面，修改控件UI，然后Command+S保存一下代码，立刻就在模拟器上显示修改的信息了。


工具使用中如有问题可以参考github上的过往经验，也欢迎留言我们一起讨论。
工具git地址：[https://github.com/johnno1962/injectionforxcode](https://github.com/johnno1962/injectionforxcode)

### 5、每个VC要使用的话，还需要去写injected，有点烦人。
用runtime 给每个VC加个方法**class_addMethod**

依托InjectionIII的iOS热部署配置文件，无侵害，导入即用。

```
@implementation InjectionIIIHelper

#if DEBUG
/**
InjectionIII 热部署会调用的一个方法，
runtime给VC绑定上之后，每次部署完就重新viewDidLoad
*/
void injected (id self, SEL _cmd) {
//重新加载view
[self loadView];
[self viewDidLoad];
[self viewWillLayoutSubviews];
[self viewWillAppear:NO];
}

+ (void)load
{
//注册项目启动监听
__block id observer =
[[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationDidFinishLaunchingNotification object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {
//更改bundlePath
[[NSBundle bundleWithPath:@"/Applications/InjectionIII.app/Contents/Resources/iOSInjection10.bundle"] load];
//[[NSBundle bundleWithPath:@"/Applications/InjectionIII.app/Contents/Resources/iOSInjection.bundle"] load];

[[NSNotificationCenter defaultCenter] removeObserver:observer];
}];

//给UIViewController 注册injected 方法
class_addMethod([UIViewController class], NSSelectorFromString(@"injected"), (IMP)injected, "v@:");

}
#endif
@end
```
。


