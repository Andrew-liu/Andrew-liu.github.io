title: IOS之多控制器管理
date: 2014-12-01 19:11:03
tags: OC
---

---


#1. 创建控制器的方式

> 1. 控制器首先应该设置自身的class
> 2. 设置所监控的View(控制器的view是延迟加载的, 用到时加载)
> 3. 控制器的view加载完毕后会调用viewDidLoad方法

##1.1. 直接创建

直接通过`alloc 和 init` 创建控制器

```c

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]]; //创建一个窗口
    self.window.backgroundColor = [UIColor whiteColor]; //设置窗口的背景色
    
    ViewController *one = [[ViewController alloc] init];  //此处ViewController为自定义的集成UIViewControl的类, 创建控制器
    one.view.backgroundColor = [UIColor blueColor]; //设置view的背景色
    self.window.rootViewController = one; //设置窗口的根控制器
    
    [self.window makeKeyAndVisible]; //窗口设置为主窗口并显示
    return YES;
}
```

<!--more-->

##1.2 通过storyboard创建

> 自己创建的stroyboard,需要设置一项Initial View Control
> 项目设置中设置了Main为Main interface, 所做的事情

1. 创建window
2. 加载storyboard, 并且创建初始化控制器
3. 显示窗口

```c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //1. 创建window
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    
    //2. 主动加载storyboard,在加载storyboard中的某个控制器
    //加载storyboard
    UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Main" bundle:nil];  //设置storyboard
    //UIViewController *vc = [storyboard instantiateInitialViewController];  //设置控制器, 箭头指向的控制器
    UIViewController *vc = [storyboard instantiateViewControllerWithIdentifier:@"red"]; //设置非主控制器, 也就是设置一个不是由箭头指向的控制器为控制器
    self.window.rootViewController = vc;  //设置窗口的根控制器
    3. 显示窗口
    [self.window makeKeyAndVisible];
    
    return YES;
}
```

##1.3. Xib创建方式

```c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.window.backgroundColor = [UIColor whiteColor];
    
    ThreeViewController *three = [[ThreeViewController alloc] initWithNibName:@"Three" bundle:nil];  //设置初始化的Xib的名称
    self.window.rootViewController = three;  //设置窗口的根控制器
    //view需要与control连线 设置First Owner的class viewcontrol 然后与View连线
    
    [self.window makeKeyAndVisible];
    return YES;
}
```

#2. ViewController的view创建(重要)

- 实现`loadView()`后, 会忽略storyboard和xib, 会首先检测`loadView`, 这个方法用来自定义View,
- 控制器view是`延迟加载`的, 用到时再加载(控制器view加载完毕后调用`viewDidLoad`方法

> 一旦加载view马上调用loadView()->viewDidLoad()->返回view被使用


![View](http://picturebag.qiniudn.com/42.png)


---

> View的加载过程:


![View的加载过程](http://picturebag.qiniudn.com/47.png)

UIView

> 凡是继承自UIButton都可以直接使用addsubview监听事件

![结构一览](http://picturebag.qiniudn.com/43.png)


#3. 多控制器

- `用一个控制器管理其他多个控制器`
- 如果`控制器A`管理`控制器B,C,D`, 控制器A称为B, C, D的`父控制器`(B,C,D称为控制器A的`子控制器`)
- IOS提供了2个比较特殊的控制器
    - UINavigationController
    - UITabBarController


UINavigationController

> 状态栏高度20, 导航栏高亮为44, 一个导航控制器只有`一个导航栏`

```c
//正确的获得导航栏的位置
//程序获得了焦点, 所有的控件才能接收触发点击时间.
- (void)applicationDidBecomeActive:(UIApplication *)application {
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    UINavigationController  *nav = (UINavigationController *)self.window.rootViewController;
    NSLog(@"%@", NSStringFromCGRect(nav.navigationBar.frame));
}
```


![导航控制器](http://picturebag.qiniudn.com/44.png)

导航栏使用栈操作, 将子控制器push到栈(`@property(nonatomic,copy) NSArray *viewControllers;`)中(控制器创建view并显示出来)
**显示导航控制器上的永远是栈顶控制器**

---

- UINavigationController的使用步骤

1. 初始化UINavigationController
2. 设置UIWindow的rootViewController为UINavigationController
3. 根据具体情况，通过push方法添加对应个数的子控制器
4. 某些情况下移除控制器

```c
//将栈顶的控制器移除
- (UIViewController *)popViewControllerAnimated:(BOOL)animated;
//回到指定的子控制器
- (NSArray *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated;
//回到根控制器（栈底控制器）
- (NSArray *)popToRootViewControllerAnimated:(BOOL)animated;
```






```c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen]bounds]];
    self.window.backgroundColor = [UIColor whiteColor];
    
    //1. 创建导航控制器
    UINavigationController *nav = [[UINavigationController alloc] init];
    
    //2. 添加子控制器
    OneViewController *one = [[OneViewController alloc] init];
    [nav pushViewController:one animated:YES];
    UIViewController *vc1 = [[UIViewController alloc] init];
    vc1.view.backgroundColor = [UIColor redColor];
    [nav pushViewController:vc1 animated:YES];//nav.viewControllers = @[one];
    
    //3. 设置窗口的根控制器
    self.window.rootViewController = nav;
    
    [self.window makeKeyAndVisible];
    return YES;
}
```

##3.1. 修改导航栏

> **导航栏的内容由栈顶控制器的navigationItem属性决定** 


UINavigationItem有以下属性影响着导航栏的内容:

```c
//左上角的返回按钮
@property(nonatomic,retain) UIBarButtonItem *backBarButtonItem;
//中间的标题视图
@property(nonatomic,retain) UIView          *titleView;
//中间的标题文字
@property(nonatomic,copy)   NSString        *title;
//左上角的视图
@property(nonatomic,retain) UIBarButtonItem *leftBarButtonItem;
UIBarButtonItem *rightBarButtonItem  右上角的视图
@property(nonatomic,retain) UIBarButtonItem *rightBarButtonItem;
```


```c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view from its nib.
    self.navigationItem.title = @"第一个控制器";
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCamera target:nil action:nil];
    self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemEdit target:nil action:nil];
    //左上角的返回键不是由当前栈顶元素决定, 由上一个控制器决定
    self.navigationItem.backBarButtonItem = [[UIBarButtonItem alloc]initWithTitle:@"返回" style:UIBarButtonItemStyleDone target:nil action:nil];
}
```



##3.2. 界面一览

```c
- (void)applicationDidBecomeActive:(UIApplication *)application
{
    NSString *xml = [self digView:self.window];
    [xml writeToFile:@"/Users/aplle/Documents/window.xml" atomically:YES];
}
- (NSString *)digView:(UIView *)view
{
    if ([view isKindOfClass:[UITableViewCell class]]) return @"";
    // 1.初始化
    NSMutableString *xml = [NSMutableString string];
    
    // 2.标签开头
    [xml appendFormat:@"<%@ frame=\"%@\"", view.class, NSStringFromCGRect(view.frame)];
    if (!CGPointEqualToPoint(view.bounds.origin, CGPointZero)) {
        [xml appendFormat:@" bounds=\"%@\"", NSStringFromCGRect(view.bounds)];
    }
    if ([view isKindOfClass:[UIScrollView class]]) {
        UIScrollView *scroll = (UIScrollView *)view;
        if (!UIEdgeInsetsEqualToEdgeInsets(UIEdgeInsetsZero, scroll.contentInset)) {
            [xml appendFormat:@" contentInset=\"%@\"", NSStringFromUIEdgeInsets(scroll.contentInset)];
        }
    }
    // 3.判断是否要结束
    if (view.subviews.count == 0) {
        [xml appendString:@" />"];
        return xml;
    } else {
        [xml appendString:@">"];
    }
    
    // 4.遍历所有的子控件
    for (UIView *child in view.subviews) {
        NSString *childXml = [self digView:child];
        [xml appendString:childXml];
    }
    
    // 5.标签结尾
    [xml appendFormat:@"</%@>", view.class];
    
    return xml;
}
```

UIWindow(UINavigationController的View(栈顶控制器的View(状态栏和导航栏)))`不同于IOS6`

##3.3. 控制器的生命周期

循环的生命周期:

![循环的声明周期](http://picturebag.qiniudn.com/46.png)


```c
/**
 *  view加载完毕
 */
- (void)viewDidLoad
{
    [super viewDidLoad];
    NSLog(@"MJOneViewController-viewDidLoad");
}
/**
 *  view即将显示到window上
 *
 */
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    NSLog(@"MJOneViewController-viewWillAppear");
}
/**
 *  view显示完毕(已经显示到窗口)
 */
- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    NSLog(@"MJOneViewController-viewDidAppear");
}
/**
 *  view即将从window上移除(即将看不见)
 *
 */
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    NSLog(@"MJOneViewController-viewWillDisappear");
}
/**
 *  view从window上完全移除(完全看不见)
 *
 */
- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    NSLog(@"MJOneViewController-viewDidDisappear");
}
/**
 *  view即将销毁的时候调用
 */
- (void)viewWillUnload
{
    [super viewWillUnload];
}
/**
 *  view销毁完毕的时候调用
 */
- (void)viewDidUnload
{
    [super viewDidUnload];
    // 由于控制器的view已经不在了,需要显示在view上面的一些数据也不需要
    self.apps = nil;
    self.persons = nil;
}
/**
 *  当接收到内存警告的时候
 */
- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
}
```

内存警告处理过程:

![内存警告处理过程](http://picturebag.qiniudn.com/45.png)



#4. storyboard的创建导航控制器步骤

1. 删除已经创建的UIView Controller, 重新创建`UINavigationController`
2. 创建新的`UIView Controller`, 设置class为`自定义的ViewController类`
3. 在`UINavigationController`上双击,设置其rootViewcontroller为新创建的`UIView Controller`
4. 可以直接在可视化界面上修改导航栏的样式或者添加push.

#5. UIView和UIViewController区别

- UIViewController是视图控制器, 而UIView是视图, UIViewController用于控制UIView。

> 可以认为UIViewController就是一个相框, 而UIView就是一个相片, 相框可以随时随地的拿走这个相片而换另外一张相片, 或者在这张相片上加一个新的相片, 而相片却不能操纵相框的.

 
- UIView用于向用户展示表现的内容, 并接受用户的交互

- UIViewController相当于导演, 按照计划编排属下的UIView以何种形式展现.UIVewController的另一个功能是处理用户交互操作, UIViewController本身是不知道用户交互的，这就需要UIView将用户的交互操作(例如:touchesBegintouchesMoved)传递上来。一般常用的两种方法完成这种传递： 
    1. [self nextResponder] touchesBegin:touches…
    2. 使用Notification 

- 如果UIViewController得到了用户的输入, 那么它应该对UIView做些改变以响应用户的输入, 这里要注意UIViewController应该只改变UIView的表现, 而不应该处理其它事情, 更多的操作通过Delegate来进行

> `UIView是一个视图, UIViewController是一个控制器, 每一个viewController管理着一个view`

