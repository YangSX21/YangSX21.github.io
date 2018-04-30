---
title: ios布局适配
date: 2018-04-30 17:30:16
tags: ios
---
> 在去年11月份，那时候iPhoneX刚出货，公司的应用也是提前就开始准备适配IPhoneX，还做梦一样想着能不能在app store获得优先适配的推荐位。这个适配是分给我的任务，当时完成之后准备码一篇博客，但是懒得很，后来发现内容也不多，放下之后过了快半年了。
上个月，因为一直有用户提意见，公司准备给iPad用户适配横屏，这个活也是加到了我身上，现在大致把活干完了。
正好这俩有相似之处，就把笔记理一下写篇文章。

## IPhoneX适配的准备工作
先说一点和布局无关的。iphoneX是和ios11一起发布的，同时Xcode也是在这个时候更新到版本号9。所以你如果想要适配IPhoneX，先要搞定这俩东西。

### Xcode9的适配
1. xib报错Illegal Configuration，是因为Xcode9开始不支持ios7以前的xib文件，所以必须更改xibd的build for的版本号，至少要ios7即以上。然后需要Clean然后重启Xcode，感觉很繁琐。解决方案在[这里](https://stackoverflow.com/questions/44429415/illegal-configuration-compiling-ib-documents-for-earlier-than-ios-7-is-no-longe)。
2. Ordered comparison between pointer and zero ('NSMutableArray *' and 'int')类似这种把Array或者char直接与0比较，大概是严格了，会报错。用length等代替吧。
3. C++ requires a type specifier for all declarations 好像是库变了，通过把check_Compile_time改成了__Check_Compile_Time解决。
主要遇到的问题就这三个，解决之后就能正常搬到Xcode9进行运行了。

### ios11的适配
ios11在页面上导致的主要差别在navigation和tableView。

1. navigation的层级结构变动，导致导航栏的文字和图标发生混乱。
在ios11情况下，强行下移状态栏高度,比较粗暴，但是有效。
2. tableView进行了更改，在ios11中，tableView默认使用了self-sizing。self-sizing，可以通过实现estimatedRowHeight相关的属性来展示动态的内容。而如果之前的项目没有设置estimatedRowHeight，则会导致出现问题。可以通过以下代码关闭该属性，或者可以对其进行正确的设置。
```
self.tableView.estimatedRowHeight = 0;
self.tableView.estimatedSectionHeaderHeight = 0;
self.tableView.estimatedSectionFooterHeight = 0;
```

完成上面的准备工作之后，接下来的IPhoneX的适配就是界面适配了，这点和iPad横屏适配我觉得差不多，就放在一起说好了。

## iphoneX页面布局
X和之前的IPhone有很多差别，但对于我而言，只要知道一些简单的点就行了。
iphoneX主要是安全区扩大，上方的状态栏增长到44pt，底部的工具栏或者tab栏垫高34pt。所以适配主要就是围绕这两点进行的。

### 启动图设置
IPhone会在启动时出现启动图，在这期间进行一些初始化操作，其中就有适配操作，它会根据图片来判断机型。这是IPhoneX适配的第一步。

|设备|屏幕尺寸|分辨率（pt）|	Reader	|分辨率（px）	|渲染后	|PPI|
|----|----|----|----|----|----|----|
|iPhone 3GS|	3.5吋|	320x480|	@1x	|320x480|		|163|
|iPhone 4/4s|	3.5吋|	320x480|	@2x|	640x960|		|330|
|iPhone 5/5s/5c|	4.0吋|	320x568|	@2x|	640x1136|		|326|
|iPhone 6	|4.7吋|	375x667|	@2x	|750x1334|		|326|
|iPhone 6Plus|	5.5吋|	414x736|	@3x|	1242x2208|	1080x1920	|401|
|iPhone 6s|	4.7吋|	375x667|	@2x	|750x1334|		|326|
|iPhone 6sPlus|	5.5吋|	414x736|	@3x|	1242x2208|	1080x1920	|401|
|iPhone 7|	4.7吋|	375x667|	@2x	|750x1334|		|326|
|iPhone 7Plus|	5.5吋|	414x736|	@3x|	1242x2208|	1080x1920|	401|
|IPhone8|4.7寸|375x667|@2x|750x1334||326| 
|IPhone8Plus|5.5寸|414x736|@3x|1242x2208|1080x1920|401|
|iphoneX|5.8寸 |375x812|@3x | 1125 x 2436|| 458|

比较清楚的，X的屏幕分辨率不同于以往任意一款IPhone，所以你需要重新制作一张启动图进行配置。在工程Targets-App Icons and Launch Images-Launch Image Source中设置启动图资源目录，在Assets.xcassets中的启动图资源文件夹中设置相应屏幕的启动图。这种方法兼容ios7,ios8。

### 官方控件适配
官方控件会自动进行拉长来进行适配，如果app中都是官方控件，并且按照约束方式，或者正确的绝对布局方式，那么基本不用更改。
会出现问题的只有使用了绝对布局，并且使用常数20代指状态栏高度，这样只要将其修改为按照[[UIApplication sharedApplication] statusBarFrame].size.height获取就OK了。

### 使用了非官方控件
这种情况下其实也就比官方控件多一步，就是需要加上IPhoneX的判断，来给你的底部控件垫高，拉长你的顶部控件。同时，如果你使用的不是约束布局，你还需要重新做一下布局，很简单的，不是吗？

### iPhoneX横屏适配
X的横屏简直是灾难，首先，你得给它左边留出44pt，然后给底部留出一定的宽度。这个宽度也不确定是多少，为了不影响使用，我觉得大概7pt就可以了。

当然上面说的点都是将可操作控件放到安全区而已，所以除非是状态栏，其他位置的展示内容，如果不带有操作性，就不需要预留。

## iPad页面适配
ios布局主要分成绝对布局和约束布局，绝对布局主要是应用简单，给定坐标长宽四个数字就能设置一个View。约束布局是一种相对布局，给定相对于其他控件的位置，然后由系统计算成绝对布局进行显示。如果不是因为特别急着进行开发的话，推荐使用约束，可以使用[masonry](https://github.com/SnapKit/Masonry)。

IPad横屏主要处理方式主要是由之前页面的布局方式所决定。
如果之前页面严格按照约束进行编写，那么适配时需要的功夫就很少，这个和约束布局的自动调用适配有关。下面详细说。
如果之前的页面按照绝对布局进行编写，那么适配时需要想办法拦住横竖屏切换的事件，在里面重新设置页面布局。
### 绝对布局
#### 通过layoutSubviews刷新
viewcontroller的viewWillLayoutSubviews和viewDidLayoutSubviews。

这两个方法顾名思义是在自己的[self.view layoutsubviews]时进行调用的。所以可以认为是在self.view的layoutsubviews方法的外部体现，因为没有办法直接去改self.view的layoutsubviews方法。
这两个方法首先会在viewcontroller初始化的后的viewWillAppear之后进行调用。首先在ViewController初始化的时候self.view是一个nil，但是在用到该View的时候会去使用loadView进行创建，这时候设置了frame，所以会调用layoutsubviews。

view的layoutsubviews方法。

- init初始化不会触发layoutSubviews，但是是用initWithFrame 进行初始化时，当rect的值 非CGRectZero时,也会触发。
- addSubview会触发layoutSubviews。如果添加的子控件没有Frame,不会调用;
- 设置view的Frame会触发layoutSubviews，前提是该View 已经被添加到父控件，当然前提是frame的值设置前后发生了变化, 此时View和其父控件的layoutSubviews都会调用;
- 滚动一个UIScrollView会触发layoutSubviews ,因滚动UIScrollView其子控件肯定对应会刷新,也就肯定会被调用;
= 旋转Screen会触发控制器对应UIView上的layoutSubviews事件
改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件

总结一下是，该View的frame发生变动，会调用本身的layoutSubviews，然后会传导上去触发父控件的layoutSubviews。不对，从外向里的，会先调用父控件的layoutSubviews，然后再调用本身的layoutsubviews。

##### layoutSubviews的强制刷新
通过setNeedsLayout能够设定一个标记，让这个View会在下一次runloop里面进行刷新，如果需要马上刷新，就调用layoutIfNeeded。接下来会直接调用layoutSubview进行刷新。

可以看到在进行旋转的时候会触发UIView上面的View的layoutSubViews方法，在Controller中viewDidLayoutSubviews或者View的layoutSubviews方法中都可以对页面或者View进行重新布局来适配横屏。但是这个比较有问题的是，这个方法的触发比较频繁且很难把握，如果重新布局进行了相当多的操作会影响性能。

#### 通过通知刷新
```
//开启横竖屏监听事件，不开启的话一直是unkown状态
if (![UIDevice currentDevice].generatesDeviceOrientationNotifications) {
    [[UIDevice currentDevice] beginGeneratingDeviceOrientationNotifications];
}
//进行监听
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(handleDeviceOrientationChange:) 
                                     name:UIDeviceOrientationDidChangeNotification object:nil];

//设备方向改变的处理
- (void)handleDeviceOrientationChange:(NSNotification *)notification{
}

//销毁 设备旋转 通知监听
[[NSNotificationCenter defaultCenter] removeObserver:self name:UIDeviceOrientationDidChangeNotification object:nil ];
// 结束 设备旋转通知
[[UIDevice currentDevice]endGeneratingDeviceOrientationNotifications];

```

监听横竖屏事件的方式非常直观，写起来也简单，在对单独页面进行处理的时候是非常好的方式，但是对于大量页面都进行通知的方式进行处理就显得非常繁琐了。

### 约束布局
约束布局在横竖屏适配上面有很大的优势，在进行布局之后系统会自动帮你进行横竖屏的适配。来探索一下。
首先，与上面绝对布局里面提到的方法一一对应，有viewcontroller中的updateViewConstraints方法，View中的updateConstraints方法。强制刷新的setNeedsUpdateConstraints和updateConstraintsIfNeeded。不过updateConstraints最后需要调用[super updateConstraints]，否则会崩。
然后，约束布局在本质上将计算frame的工作交给代码，所以最后是会转换成绝对布局的。所以在调用updateConstraints之后会调用layoutsubviews。尽量不要混用，会比较混乱。

在做iPad横屏过程中，明显感觉到约束布局的优势，所以之后在尽可能情况下基本都是使用约束布局进行开发了。