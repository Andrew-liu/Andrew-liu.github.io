title: IOS之控制器数据传递和代理总结
date: 2014-12-06 17:35:06
tags: OC
---

---

>  copy: NSString/NSMutableArray
>  weak:代理/UI控件
>  strong: 其他OC对象


#1. Segue

storyboard上每根界面跳转的线, 都是`UIStoryboardSegue`对象

每个Segue对象都有三个属性:

- 唯一标识(`@property (nonatomic, readonly) NSString *identifier;`), 
- 来源控制器(`@property (nonatomic, readonly) id sourceViewController;`)
- 目标控制器(`@property (nonatomic, readonly) id destinationViewController;`)


##1.1. Segue的类型

- 自动型: 点击`某控件`后, 自动执行Segue,自动完成界面跳转(由控制拖出, 用于不需要做任何判断直接跳转界面的情景)
- 手动型: 需要手写代码才能完成界面的跳转(按住control, 由来源`控制器(不是按钮)`拖线到目标控制器, 需要`设置标识`)


<!--more-->
##1.2. performSegueWithIdentifier:sender:方法的完整执行过程

```c
/* Segue必须由来源控制器来执行，也就是说，这个perform方法必须由来源控制器来调用
self为源控制器
第一个参数为Segue的表示, 第二个参数为要传递的数据*/
[self performSegueWithIdentifier:@"login2contacts" sender:nil];

```


```c
//登陆过程
- (IBAction)login {
    //账号验证
    if ([self.accountField.text isEqualToString:@"mj"] && [self.pwdField.text isEqualToString:@"123"]) {
        //显示一个遮盖, 下面为一个延迟执行函数
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            //移除遮盖
            //登陆
            [self performSegueWithIdentifier:@"login2contact" sender:nil];
        });
        //执行login2contact这个Segue
    }
    else
    {
        //账户或者密码错误, 弹窗提示
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"警告" message:@"账号或者密码错误" delegate:nil cancelButtonTitle:@"好的" otherButtonTitles:nil, nil];
        [alert show];
    }
}

//执行Segue后, 跳转前会调用这个方法, 这个方法用来自己实现
//一般在这里给下一个控制器传递数据
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    //数据传递工作
    //1. 取得目标控制器
    UIViewController *contactVc = segue.destinationViewController;
    //2. 设置标题
    contactVc.title = [NSString stringWithFormat:@"%@的联系人列表", self.accountField.text];
}
```

1. 根据identifier去storyboard中找到对应的线
2. 新建UIStoryboardSegue对象
3. 设置Segue对象的sourceViewController（来源控制器）
4. 新建并且设置Segue对象的destinationViewController（目标控制器）
5. 调用sourceViewController的方法，做一些跳转前的准备工作并且传入创建好的Segue对象

```c
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender;
// 这个sender是当初performSegueWithIdentifier:sender:中传入的sender
```

```c
//执行跳转前调用这个方法做一些跳转准备
//在这个方法中目标控制器的 view 还没有被创建
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    //####每次跳转都会进入这个函数, 所以需要对跳转对象判断####
    //1. 获取目标控制器
    id vc = segue.destinationViewController;
    
    //判断其类型, 对不同类型的目标控制器执行不同的准备
    if([vc isKindOfClass:[AddViewController class]])
    {
        //如果跳转到添加联系人的控制器, 进行跳转, 获取添加控制器
        AddViewController *addVc = segue.destinationViewController;
        //设置代理, 拿到下一次控制器, 设置下一个控制器的代理为源控制器, 代理会在被触发时执行函数
        addVc.delegate = self;
    }
    else if ([vc isKindOfClass:[EditViewController class]])
    {
        //如果是编辑控制器, 进行跳转, 获得编辑控制器
        EditViewController *editVc = vc;
        //取得UITableView被点击选择的行号
        NSIndexPath *path = [self.tableView indexPathForSelectedRow];
        //把模型数据传过去, 在目标控制器已经定义了contact模型对象
        editVc.contact = self.contacts[path.row];
        //跳转之前给下一个控制器设置一个代理
        editVc.delegate = self;
    }
}
```


6. 调用Segue对象的- (void)perform;方法开始执行界面跳转操作
    - 取得sourceViewController所在的UINavigationController
    - 调用UINavigationController的push方法将destinationViewController压入栈中，完成跳转



##1.3. 数据后传

```c
- (IBAction)login {
    //账号验证
    if ([self.accountField.text isEqualToString:@"mj"] && [self.pwdField.text isEqualToString:@"123"]) {
        //显示一个遮盖, 下面为一个延迟执行函数
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            //移除遮盖
            //登陆
            [self performSegueWithIdentifier:@"login2contact" sender:nil];
        });
        //执行login2contact这个Segue
    }
    else
    {
        //账户或者密码错误, 弹窗提示
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"警告" message:@"账号或者密码错误" delegate:nil cancelButtonTitle:@"好的" otherButtonTitles:nil, nil];
        [alert show];
    }
}

//执行Segue后, 跳转前会调用这个方法, 这个方法用来自己实现
//一般在这里给下一个控制器传递数据
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    //数据传递工作
    //1. 取得目标控制器
    UIViewController *contactVc = segue.destinationViewController;
    //2. 设置标题
    contactVc.title = [NSString stringWithFormat:@"%@的联系人列表", self.accountField.text];
}

```


##1.4. 数据回传(重要)

> 使用`代理`

**代理的实现过程**

1. 在目标控制器的头文件定义代理和代理对应的方法, 并定义代理属性

```c
#import <UIKit/UIKit.h>
//分别为联系人控制器, 联系人信息模型, 添加联系人控制器
@class ContactViewController, Contact, AddViewController;
//设置代理
@protocol AddViewControllerDelegate <NSObject>
@optional
//声明代理的方法, 定义这个方法在源控制器(也是就代理使用者)
- (void)addViewController:(AddViewController *)addVc DidAddContact: (Contact *)contact;
@end

@interface AddViewController : UIViewController
@property (nonatomic, strong) ContactViewController *contacts;//模型
@property (nonatomic, weak) id<AddViewControllerDelegate> delegate; //定义代理属性
@end
```

2. 当某个事件(`比如点击事件后需要传递数据给代理`)引发代理事件时, 通知代理, 如果代理有某个方法, 就调用这个方法通知道理

```c
//最好将所有参数封装成模型, 直接给代理传递模型
- (IBAction)add {
    //1. 关闭当前控制器(出栈)
    [self.navigationController popViewControllerAnimated:YES];
    //2. 传递数据给上一级控制器(contactViewController)
    //2.1. 通知代理 (调用代理方法)
    if ([self.delegate respondsToSelector:@selector(addViewController:DidAddContact:)]) {
        //初始化模型并复制
        Contact *contact = [[Contact alloc] init];
        contact.name = self.nameField.text;
        contact.phone = self.phoneField.text;
        //给代理传入模型
        [self.delegate addViewController:self DidAddContact:contact];
    }
}
```

3. 代理遵守协议, 代理接到消息后, 执行代理方法

```c
@interface ContactViewController () <AddViewControllerDelegate>
- (IBAction)logout:(id)sender;
@property (nonatomic, strong) NSMutableArray *contacts;
@end


//add控制器的代理方法
- (void)addViewController:(AddViewController *)addVc DidAddContact:(Contact *)contact
{
    //1. 添加模型数据
    [self.contacts addObject:contact];
    //2. 刷新tableView表格
    [self.tableView reloadData];
}
```

1. 联系人列表界面点击加号添加联系人(即将进行控制器的跳转), 调用`- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender`方法(拿到目标控制器,设置添加控制器的代理为self(也就是源控制器)
2. 进行控制器跳转
3. add方法(目标控制器触发代理事件)关闭当前控制器,判断代理联系人控制器有没有实现某个方法, 调用联系人列表的代理
4. 代理执行这个代理方法




#2. 两种弹窗

##2.1. 底部弹窗


```c
//遵守协议
@interface ContactViewController () <UIActionSheetDelegate>

//设置底部弹窗需要遵守UIActionSheetDelegate协议
- (IBAction)logout:(id)sender {
    UIActionSheet *sheet = [[UIActionSheet alloc] initWithTitle:@"确定要注销吗?" delegate:self cancelButtonTitle:@"取消" destructiveButtonTitle:@"确定" otherButtonTitles:nil, nil];
    [sheet showInView:self.view];//显示弹窗
}

//buttonIndex为被点击的按钮的索引, 由上到下从零编号
- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex
{
    if (buttonIndex != 0) {
        return;
    }
    if (buttonIndex == 0) {
        //点击了确定
        [self.navigationController popViewControllerAnimated:YES];
    }
}
```



##2.2. 直接弹窗

```c
- (IBAction)login {
    //账号:mj mima:123
    if ([self.accountField.text isEqualToString:@"mj"] && [self.pwdField.text isEqualToString:@"123"]) {
        //显示一个遮盖
        //模拟延迟
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            //移除遮盖
            //进行控制器的跳转
            [self performSegueWithIdentifier:@"login2contact" sender:nil];
        });
        //执行login2contact这条线
    }
    else
    {
        //账户或者密码错误
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"警告" message:@"账号或者密码错误" delegate:nil cancelButtonTitle:@"好的" otherButtonTitles:nil, nil];
        [alert show];
    }
}
```




#3. 代理总结

> 协议的定义并不实现任何方法(只做声明),当某个带向遵守协议, 这个对象必须实现协议中声明的方法


A是B的代理, B将发送消息(数据)

-  为B定义一个代理,并声明代理方法

```c
@protocol 代理协议名称 <NSObject> 
@optional  
声明代理方法  
@end
```

- B定义一个delegate变量

```c
@interface B : UIViewController
@property (nonatomic, weak) id<代理协议名称> delegate; 
@end
```

- 当事件发生后, 判断协议, 然后通知代理

```c
if ([self.delegate respondsToSelector:@selector(声明的方法) {
        //通知代理
        [self.delegate 调用的声明的方法];
```
- A遵守协议, 并B的代理为A

```c
@interface A () <代理协议名称>
...

B.delegate = self(A)
```

- A中定义协议中的方法


> 受[IOS之协议和委托](http://abbeychenxi.net/2014/12/05/IOS%E4%B9%8B%E5%8D%8F%E8%AE%AE%E5%92%8C%E5%A7%94%E6%89%98/)启发, 对代理理解更深入

#4. 消息中心监听

> `消息中心`就像一个广播基站，发送一条消息，在所有的添加监听的地方都能够收到此信息，并作出不同或者相同的动作

在前4个页面中都创建一个消息中心用来监听 一个object `@"change"`,我们在第五个页面，通过消息中心，发送一个`@"change"` 消息，这样前四个页面就可以收到这个消息，然后做出相应的动作 


* 一个文本输入框的文字发生改变时,文本输入框会发出一个`UITextFieldTextDidChangeNotification`通知
* 因此通过监听通知来监听文本输入框的文字改变

```c
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textChange) name:UITextFieldTextDidChangeNotification object:textField];
// textField文本输入框的文字改变了,就会调用self的textChange方法
```

```c
- (void)viewDidLoad {
    [super viewDidLoad];
    //文本框发生改变时会发生通知
    //监听者self控制器监 object要监听对象, selector为监听方法
    //添加消息中心监听(添加观察者，也能说成添加监听) [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(myNotification:)name:@“test1" object:nil]; 
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textChange) name:UITextFieldTextDidChangeNotification object:self.accountField];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textChange) name:UITextFieldTextDidChangeNotification object:self.pwdField];
    
    //后发消息object:发送的参数 [[NSNotificationCenter defaultCenter]postNotificationName:@“test1" object:arra]; 
}
//消息监听后必须移除监听
- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

//文本框发生改变后调用(监听方法)
- (void)textChange
{
    //控制登陆按钮的状态
    self.loginBtn.enabled = (self.accountField.text.length && self.pwdField.text.length);
}
```

[iOS设计模式之NSNotificationCenter 消息中心](http://www.bkjia.com/IOSjc/910686.html)


#5. UISwitch

* UISwitch继承自UIControl,因此也能像UIButton一样监听一些事件,比如状态改变事件
* UISwitch可以通过拖线监听状态改变
* UISwitch可以通过`addTarget:...`方法监听状态改变

```c
- (void)addTarget:(id)target action:(SEL)action forControlEvents:(UIControlEvents)controlEvents;
// 其中controlEvents参数传递的是:UIControlEventValueChanged(值改变事件)
```

```c
//rmbPwd和autoLogin分别为UISwitch按钮
//记住密码开关状态改变
- (IBAction)rmbPwdChange {
    if(self.rmbPwd.isOn == NO)
    {
        //self.autoLogin.on = NO;
        [self.autoLogin setOn:NO animated:YES];
    }
}
- (IBAction)autoLoginChange {
    if (self.autoLogin.isOn) {
        //self.rmbPwd.on = YES;
        [self.rmbPwd setOn:YES animated:YES];
    }
}
```
