更新说明:

版本号               | 需求              | 修改内容                                   |
--------------------|------------------|-------------------------------------------|
1.0.1               | 使用不包含支付模块的微信SDK避免审核被拒   |pod 'WechatOpenSDK'<br> 修改为<br> pod 'WechatOpenSDK_NoPay'  |
1.0.2               | 新增判断用户是否同意用户协议和隐私条款的接口说明   | 新增 isUserAgreesTerms 使用说明  |
1.0.3              | 新增马甲包集成代码   | 添加 pod 'CLTGameSDK' 与 pod 'CLTGameSDKFG'  |
1.0.4              | 新增登录返回参数，添加游客登录接口   | 用于检验的参数LTUidToken,游客登录接口touristLogin:(loginUserBlock)block|
1.0.5              | 优化登出接口   | userLogoutToLoginUI(清除数据，弹出登录提示框) userLogout(清除数据，不弹出登录框)|
2.0.0              | UI风格变化，游客按钮的显示控制，LT平台注册新增参数   | showLoginManagerUI方法新增isShowGuestButton参数， registLTPlatformAppID方法新增withUIViewController参数 |

##第一步,添加库
>
>1，cocopods安装三方库 
>>1集成所有登录平台（Facebook、Google、QQ、微信）使用如下命令集成
>>
>```
>use_frameworks!
pod 'GoogleSignIn', '~> 4.4.0'
pod 'AFNetworking'
pod 'FacebookCore'
pod 'FacebookLogin'
pod 'FBSDKCoreKit', '~> 4.38.0'
pod 'FBSDKLoginKit', '~> 4.38.0'
pod 'WechatOpenSDK_NoPay', '~> 1.8.0'
pod 'LTQQOpenAPI', :git => 'https://github.com/zhubinfeng/LTQQOpenAPI'
# 韩文包与英文包两者选其一使用
# 韩文包使用如下命令集成
pod 'LTGameSDK', :git => 'https://github.com/zhubinfeng/LTGameSDK'
# 英文包使用如下命令集成
pod 'LTGameSDKEN', :git => 'https://github.com/zhubinfeng/LTGameSDKEN'
>```
>
>>2只集成Facebook、Google登录平台使用如下命令集成
>>
>```
>use_frameworks!
>pod 'GoogleSignIn', '~> 4.4.0'
pod 'AFNetworking'
pod 'FacebookCore'
pod 'FacebookLogin'
pod 'FBSDKCoreKit', '~> 4.38.0'
pod 'FBSDKLoginKit', '~> 4.38.0'
# 韩文包与英文包两者选其一使用
# 韩文包使用如下命令集成
pod 'LTGameSDKFG', :git => 'https://github.com/zhubinfeng/LTGameSDKFG'
# 英文包使用如下命令集成
pod 'LTGameSDKFGEN', :git => 'https://github.com/zhubinfeng/LTGameSDKFGEN'
>```
>
>2，info.plist添加如下字段
>
>```	
><key>NSAppTransportSecurity</key>
	<dict>
		<key>NSAllowsArbitraryLoads</key>
		<true/>
	</dict>
>```
>补充说明:
>>1,文档中的xxx代表对应平台申请的应用ID，使用时可以根据接口参数说明进行设置<br>
>>

##第二步，初始化LT应用

>1，在APPdelegate.h文件中导入头文件

>```
#import <LTGameSDK/LTSDKLoginManager.h>
```
>2,初始化应用(传入AppID和AppKey)<br>
```
[[LTSDKLoginManager sharedInstance] registLTPlatformAppID:@"xxxx" withAppkey:@"xxxx" withUIViewController:viewController];
```

##第三步，集成对应的第三平台

>URL Schemes 的添加方法

>![](https://github.com/zhubinfeng/SDKDemo/blob/master/URL_Schemes.png?raw=true)
>
>配置-ObjC方法 <br>
>Build Setting -> Other Linker Flags 添加-ObjC
>![](https://github.com/zhubinfeng/SDKDemo/blob/master/otherLinkerFlags.png?raw=true)

>**一 Facebook集成**
>>1,添加URL scheme fbxxxx
>>
>>2，在info.plist中dict节点下添加
>>
>>
         <key>FacebookAppID</key>
         <string>xxxx</string>
         <key>FacebookDisplayName</key>
         <string>应用的名称</string>
         <key>LSApplicationQueriesSchemes</key>
         <array>
         <string>fbapi</string>
         <string>fb-messenger-share-api</string>
         <string>fbauth2</string>
         <string>fbshareextension</string>
         </array>
>>
>>
>>3，配置-ObjC
>>
>>4,在APPdelegate.h文件中添加
>>
>>```
>>-(BOOL)application:(UIApplication *)app
           openURL:(NSURL *)url
           options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options{
    [[LTSDKLoginManager sharedInstance] application:app openURL:url options:options];
    return YES;
}
>>```
>>
>>5，登录调用
>>
>>```
>>[[LTSDKLoginManager sharedInstance] facebookLogin:^(LTUser * _Nonnull loginUser) {
        if (loginUser.stateCode == LTCodeTypeSuccess) {
            if (loginUser.userType == LTUserTypeTourist) {
                NSLog(@"游客登录成功");
            }else{
                NSString *lt_UID = loginUser.LT_UID;
                NSString *LTUidToken = loginUser.LTUidToken;
            }
        }else if (loginUser.stateCode == LTCodeTypeCancel){
            NSLog(@"用户取消了登录");
        }else if (loginUser.stateCode == LTCodeTypeFailed){
            NSLog(@"登录失败");
        }
    }];
>>```
>>

>**二 Google集成**
>>1,添加URL scheme com.googleusercontent.apps.xxxx
>>
>>2，配置-ObjC
>>
>>3，在要使用登录UIViewCcontroller中调用初始化
>>
>>```
>>[[LTSDKLoginManager sharedInstance] registGooglePlatform:@"xxxx.apps.googleusercontent.com"
>>withUIViewController:self];
>>```
>>
>>4，登录调用
>>
>>```
>>[[LTSDKLoginManager sharedInstance] googleLoginGetLTID:^(LTUser * _Nonnull loginUser) {
        if (loginUser.stateCode == LTCodeTypeSuccess) {
            if (loginUser.userType == LTUserTypeTourist) {
                NSLog(@"游客登录成功");
            }else{
                NSString *lt_UID = loginUser.LT_UID;
                NSString *LTUidToken = loginUser.LTUidToken;
            }
        }else if (loginUser.stateCode == LTCodeTypeCancel){
            NSLog(@"用户取消了登录");
        }else if (loginUser.stateCode == LTCodeTypeFailed){
            NSLog(@"登录失败");
        }    
    }];
>>```

>**三 微信集成**
>>1,添加URL schemes xxxx<br>
>>2,在APPdelegate.h文件中添加
>>
>>```
>>-(BOOL)application:(UIApplication *)app
           openURL:(NSURL *)url
           options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options{
    [[LTSDKLoginManager sharedInstance] application:app openURL:url options:options];
    return YES;
}
>>```
>>
>>3，在APPdelegate.h中初始化
>>
>>```
>>[[LTSDKLoginManager sharedInstance] registWeChatPlatform:@"xxx"];
>>```
>>
>>4，登录调用
>>
>>```
>>[[LTSDKLoginManager sharedInstance] weChatLogin:^(LTUser * _Nonnull loginUser) {
		 if (loginUser.stateCode == LTCodeTypeSuccess) {
            if (loginUser.userType == LTUserTypeTourist) {
                NSLog(@"游客登录成功");
            }else{
                NSString *lt_UID = loginUser.LT_UID;
                NSString *LTUidToken = loginUser.LTUidToken;
            }
        }else if (loginUser.stateCode == LTCodeTypeCancel){
            NSLog(@"用户取消了登录");
        }else if (loginUser.stateCode == LTCodeTypeFailed){
            NSLog(@"登录失败");
        }    
    }];
>>```


>**四 QQ集成**
>>
>>1，添加URL schemes tencentxxxx
>>
>>2,配置-fobjc-arc(与配置-ObjC方法相同)
>>
>>3,在info.plist中添加如下字段
>>
>>
         <key>LSApplicationQueriesSchemes</key>
         <string>mqqapi</string>
         <string>mqq</string>
         <string>mqqOpensdkSSoLogin</string>
         <string>mqqconnect</string>
         <string>mqqopensdkdataline</string>
         <string>mqqopensdkgrouptribeshare</string>
         <string>mqqopensdkfriend</string>
         <string>mqqopensdkapi</string>
         <string>mqqopensdkapiV2</string>
         <string>mqqopensdkapiV3</string>
         <string>mqzoneopensdk</string>
         <string>mqqopensdkapiV3</string>
         <string>mqqopensdkapiV3</string>
         <string>mqzone</string>
         <string>mqzonev2</string>
         <string>mqzoneshare</string>
         <string>wtloginqzone</string>
         <string>mqzonewx</string>
         <string>mqzoneopensdkapiV2</string>
         <string>mqzoneopensdkapi19</string>
         <string>mqzoneopensdkapi</string>
         <string>mqzoneopensdk</string>
>>
>>4,在APPdelegate.h文件中添加
>>
>>```
>>-(BOOL)application:(UIApplication *)app
           openURL:(NSURL *)url
           options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options{
    [[LTSDKLoginManager sharedInstance] application:app openURL:url options:options];
    return YES;
}
>>```
>>
>>5，在APPdelegate.h中初始化
>>
>>```
>>[[LTSDKLoginManager sharedInstance] registQQPlatform:@"xxxx"];
>>```
>>
>>6，登录处调用
>>
>>```
>>[[LTSDKLoginManager sharedInstance] QQLogin:^(LTUser * _Nonnull loginUser) {
		  if (loginUser.stateCode == LTCodeTypeSuccess) {
            if (loginUser.userType == LTUserTypeTourist) {
                NSLog(@"游客登录成功");
            }else{
                NSString *lt_UID = loginUser.LT_UID;
                NSString *LTUidToken = loginUser.LTUidToken;
            }
        }else if (loginUser.stateCode == LTCodeTypeCancel){
            NSLog(@"用户取消了登录");
        }else if (loginUser.stateCode == LTCodeTypeFailed){
            NSLog(@"登录失败");
        }    
     }];
>>```



##第四步，接口使用说明


>1，使用SDK的登录UI
>>1>设置用户协议和隐私条款连接地址
>>
>>```
>>[[LTSDKLoginManager sharedInstance] linkOfUserAgreement:@"https://github.com/" 
>>andPrivacyLine:@"https://www.baidu.com/"];
>>```
>
>>2>调用UI界面进行显示
>>
>```
>[[LTSDKLoginManager sharedInstance] showLoginManagerUI:self withBlock:^(LTUser * _Nonnull loginUser) {
>       if (loginUser.stateCode == LTCodeTypeSuccess) {
>           if (loginUser.userType == LTUserTypeTourist) {
                NSLog(@"游客登录成功");
            }else if(loginUser.userType != LTUserTypeUnLogin){
                NSString *lt_UID = loginUser.LT_UID;
                NSString *LTUidToken = loginUser.LTUidToken;
                NSString *messageStr = loginUser.message;
            }
>       }else if (loginUser.stateCode == LTCodeTypeCancel){
>           NSLog(@"用户取消了登录");
>       }else if (loginUser.stateCode == LTCodeTypeFailed){
>           NSLog(@"登录失败");
>       }
>   } isShowGuestButton:YES];
>```
>

>2,如何退出登录
>
>>1只清除数据，不弹出SDK登录界面
>
>```
>[[LTSDKLoginManager sharedInstance] userLogout];
>```
>
>>2清除数据并弹出SDK登录界面
>
>
>```
>    [[LTSDKLoginManager sharedInstance] userLogoutToLoginUI:self withBlock:^(LTUser * _Nonnull loginUser) {
        if (loginUser.stateCode == LTCodeTypeSuccess) {
            if (loginUser.userType == LTUserTypeTourist) {
                NSLog(@"游客登录成功");
            }else{
                NSString *lt_UID = loginUser.LT_UID;
                NSString *LTUidToken = loginUser.LTUidToken;
                NSString *messageStr = loginUser.message;
            }
        }else if (loginUser.stateCode == LTCodeTypeCancel){
            NSLog(@"用户取消了登录");
        }else if (loginUser.stateCode == LTCodeTypeFailed){
            NSLog(@"登录失败");
        }
        NSLog(@"Message = %@",loginUser.message);
        [self checkUserInfo];
    } isShowGuestButton:NO];
>```
>
>3,获取当前用户信息
>
>>1获取登录状态
>>
>```
>    [[LTSDKLoginManager sharedInstance] getUserLoginState:^(LTUser * _Nonnull loginUser) {
        if (loginUser.userType == LTUserTypeUnLogin) {
            NSLog(@"用户未登录");
        }else if (loginUser.userType == LTUserTypeTourist){
            NSLog(@"游客登录");
        }else{
            NSString *lt_UID = loginUser.LT_UID;
            NSString *LTUidToken = loginUser.LTUidToken;
            NSString *messageStr = loginUser.message;
        }
    }];
>```
>
>>2获取用户ID（获取其它信息可参考currentUser对象对应的LTUser.h文件的属性说明）
>>
>```
>NSString *lt_uid = [LTSDKLoginManager sharedInstance].currentUser.LT_UID;
>```
>
>>3获取此次登录附带信息（登录成功的附带提示信息、登录失败的错误提示等）
>>
>```
>NSString *messageStr = [LTSDKLoginManager sharedInstance].currentUser.message;
>```
>
>>4判断当前用户是否同意了用户协议和隐私条款
>>
>```
>BOOL isAgreeTerms = [[LTSDKLoginManager sharedInstance] isUserAgreesTerms];
>```
>
>>5游客登录
>>
>```
>    [[LTSDKLoginManager sharedInstance] touristLogin:^(LTUser * _Nonnull loginUser) {
        NSString *messageStr = loginUser.message;
    }];
>```
>