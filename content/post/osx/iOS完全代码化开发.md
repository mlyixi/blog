---
title: "iOS完全代码化开发"
date: 2014-08-27T16:08:52+08:00
tags: ["iOS"]
categories: ["OSX"]
toc: true
---


## 视图
把自定义视图控制器赋给Delegation-window-rootViewController，在自定义视图控制器的loadView方法中初始化view.


NOIBDelegation.h

```objc
@class NOIBViewController;
 
@property (strong, nonatomic) NOIBViewController *viewController;
```


NOIBAppDelegate.m -- didFinishLaunchingWithOptions:

```objc
self.window.rootViewController = viewController;
```


NOIBViewController.h

```objc
@property (strong, nonatomic) UILabel *label;
```


NOIBViewController.m

```objc
- (void)loadView
{
    CGRect frame = CGRectMake(0, 0, 320, 480);
    self.view = [[UIView alloc] initWithFrame:frame];
    self.view.backgroundColor = [UIColor whiteColor];
    frame = CGRectMake(0, 0, 100, 50);
    label = [[UILabel alloc] initWithFrame:frame];
    label.text = @"hello world";
    [self.view addSubview:label];
}
```

# 事件响应


NOIBViewController.m

```objc
- (void)loadView
{
    frame= CGRectMake(20, 80, 280, 50);
    button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    button.frame=frame;
    [button setTitle:@"OK" forState:UIControlStateNormal];
    button.backgroundColor = [UIColor clearColor];
    [button addTarget:self action:@selector(btnClicked:) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:button];
}
 
-(IBAction)btnClicked:(id)sender
{
    Some codes
    //删除自身视图
    [self.view removeFromSuperview];
    //加载子视图
    [self.view addSubview:UserViewController.view];
}
```

# NavigationController
设置nav的initWithRootViewController为自定义视图控制器, 通过pushViewController方法加载其它视图控制器,一般popViewController自动实现--即为左侧按钮.　右侧按钮需要手动添加(`NavigationItem.rightBarButtonItem`).


appDelegate

```objc
nvc = [NumberViewController controllerWithNumber:1];
UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:nvc];
window.rootViewController = nav;
```


numberViewController

```objc
+ (id) controllerWithNumber: (int) number
{
    NumberViewController *viewController = [[NumberViewController alloc] init];
    viewController.number = number;
    viewController.textView.text = [NSString stringWithFormat:@"Level %d", number];
    return viewController;
}
 
 
- (void) pushController: (id) sender
{
    NumberViewController *nvc = [NumberViewController controllerWithNumber:number + 1];
    [self.navigationController pushViewController:nvc animated:YES];
}
 
- (void) viewDidAppear: (BOOL) animated
{
    self.navigationController.navigationBar.tintColor = COOKBOOK_PURPLE_COLOR;
     
    // match the title to the text view
    self.title = self.textView.text; 
    self.textView.frame = self.view.frame;
     
    // Add a right bar button that pushes a new view
    if (number < 6)
        self.navigationItem.rightBarButtonItem = 
        BARBUTTON(@"ush", @selector(pushController:));
}
```