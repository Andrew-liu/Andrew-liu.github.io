title: IOS之UITabBarController
date: 2014-12-19 17:03:13
tags: OC
---

#1. UITabBarController简单使用
---

1. 初始化`UITabBarController`
2. 设置UIWindow的`rootViewController`为UITabBarController
3. 根据具体情况, 通过addChildViewController方法添加对应个数的子控制器

`UITabBarController添加控制器的方式有2种` :
```c
//添加单个子控制器
- (void)addChildViewController:(UIViewController *)childController;

//设置子控制器数组
@property(nonatomic,copy) NSArray *viewControllers;
```

<!--more-->

##1.1. UITabBarButton中UITabBarItem的属性

> UITabBarController的高度为49

```c
//标题文字
@property(nonatomic,copy) NSString *title;
//选中时的图标
@property(nonatomic,retain) UIImage *selectedImage;
//提醒数字
@property(nonatomic,copy) NSString *badgeValue;
self.tabBarItem.badageValue = @"10";
//图标
@property(nonatomic,retain) UIImage *image;
```

> 当在某个界面不想显示TabBar工具条时, 在控制器属性中设置> `hide bottom bar (隐藏底部的工具栏)`


##1.2. 主流UI框架



#2. 控制器声明周期
---

```
//view加载完毕调用
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    NSLog(@"didload");
}
//view即将显示到window上
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    NSLog(@"willapper");
}
//view显示完毕(已经实现到窗口)
-(void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    NSLog(@"didappear");
}
//view即将从window上移除(即将看不见)
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    NSLog(@"willids");
}
//view从window上完全移除
- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    NSLog(@"diddisappear");
}
```

#3. Modal
---

> show之外, 另一种控制器的切换方式`Modal`

- 从底向上弹出, 并且将所覆盖的控制器弹开(暂时, 根控制器始终不变)
- 控制器向下离开时, 会将弹开的控制器放回
- `通过会在Modal外包装一层导航控制器`

> 界面之间联系紧密一般用Show, 联系不紧密用Modal


##3.1. Modal的显示与销毁

```c
//以Modal的形式展示控制器
- (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion
//关闭当初Modal出来的控制器, 自我销毁
- (void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^)(void))completion;

//点击按钮弹出新的控制器
- (IBAction)jump {
    //展示TwoViewController
    TwoViewController *two = [[TwoViewController alloc] init];
    //在弹窗控制器外包装一层导航控制器
    UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:two];
    [self presentViewController:nav animated:YES completion:^{
        NSLog(@"展示动画完毕"); // 动画执行完调用
    }];
}

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view from its nib.
    self.title = @"第二个控制器";  //设置导航控制器的标题
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem  alloc] initWithTitle:@"取消" style:UIBarButtonItemStyleDone target:self action:@selector(cancel)];  //设置导航控制器的Item
};
//点击导航控制器的Item按钮销毁控制器
- (IBAction)cancel {
    [self dismissViewControllerAnimated:YES completion:^{
        NSLog(@"关闭控制器, 自我销毁");
    }];
}
```

##3.2. Modal的数据传递

同样分为两种形式, 数据的顺传和逆传 :

- 数据传递顺传(导航控制器Segue使用这个类似)
- 数据逆传使用`代理`

```c
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    NSLog(@"%@", segue.destinationViewController);
    UINavigationController *nav = segue.destinationViewController;
    NSLog(@"%@", nav.topViewController); //拿到栈顶控制器
    TwoViewController *two = (TwoViewController *)nav.topViewController;
    two.name = @"我这条数据将会传送给下一个控制器";  //在two控制器中头文件定义name属性
    
}

- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"%@", self.name);
}
```


#4. strong和weak
---

- `strong` 修饰称为强指针/强引用(`默认情况下, 任何指针都是强指针`)
-  `weak` 修饰弱指针/弱引用(`使用__weak修饰的指针`)

> ARC 判断准则: 只要没有任何强指针指向对象, 对象就被销毁
> 写一个@proerty 会自动生成一个_变量


`@property属性的用法`:

- weak(assign): 代理/UI控件
- strong(retain): 其他对象(除了代理/UI控件/字符串以外的对象)
- copy: 字符串
- assign: 非对象类型(基本数据类型int/float/BOOL/枚举/结构体)

