<!-- beta -->
# 账户API
账户API 分三部分：
1. 初始化帐号系统
2. 登录
3. 获取账户信息 

账户的主要类是MHAccount，  
### 初始化帐号系统
SDK需要登陆后才能操作设备，所以APP开发必须申请API和取得appid，以及redirectURL。当申请得到appid和redirectUrl后，在AppDelegate的application:didFinishLaunchingWithOptions: 方法中初始化账户

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	_account = [[MHAccount alloc] initWithAppId:@"申请的appid" redirectUrl:@"http://xiaomi.com"];
    return YES;
}
```

### 登录
登录需要注册相关的通知事件，然后调用login 方法，权限为数组，更多的权限见[小米账户权限](https://dev.mi.com/docs/passport/scopes/)
```objc
//注册相关通知
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLogin:) name:MH_Account_Login_Sucess object:nil];

[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLogout:) name:MH_Account_Logout_Sucess object:nil];

[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLogCancel:) name:MH_Account_Login_Cancel object:nil];
    
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLoginFailure:) name:MH_Account_Login_Failure object:nil];

//登录
[_account login:@[@1,@3,@6000]];

```

### 获取当前登录账户信息
当登录成功后， 可以调用fetchAccountProfile 方法获取登录账户的相关信息。MHAccountProfile
```objc
-(void)accountLogin:(NSNotification*)notify {
    //保存登录信息
    BOOL saveResult = [UIAppDelegate.account save]; 
    [_account fetchAccountProfile:^(MHAccountProfile *profile, NSError *error) {
        _profile = profile;
	    NSLog(@"_profile.userId = %@",_profile.userId);
    }];
}

```



### 注意事项

MHAccount的login是一个异步请求，所以请在登录成功的回调中去拉取账户信息或者设备信息，而不是调用login方法后直接调用相关的方法。那样会因为账户没有登录成功而操作失败。代码类似如下
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	_account = [[MHAccount alloc] initWithAppId:@"申请的appid" redirectUrl:@"http://xiaomi.com"];
    [self registerLoginNotifcation];
    [_account login:@[@1,@3,@6000]];
    return YES;
}

- (void)registerLoginNotifcation {
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLogin:) name:MH_Account_Login_Sucess object:nil];

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLogout:) name:MH_Account_Logout_Sucess object:nil];

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLogCancel:) name:MH_Account_Login_Cancel object:nil];
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(accountLoginFailure:) name:MH_Account_Login_Failure object:nil];
}

-(void)accountLogin:(NSNotification*)notify {
    //保存登录信息。这个时候才表示登录成功，可以调用拉取设备列表，获取用户信息的API。 
    BOOL saveResult = [UIAppDelegate.account save]; 
    [_account fetchAccountProfile:^(MHAccountProfile *profile, NSError *error) {
        _profile = profile;
	    NSLog(@"_profile.userId = %@",_profile.userId);
    }];
}
```
