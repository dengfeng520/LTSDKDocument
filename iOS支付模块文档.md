<center><H1>iOS支付模块文档</h1></center>



>###1、支付模块流程说明

 为保证支付安全性，不出现掉单、漏单的情况，`SDK`采用了苹果验证和`LT`服务器二次验证的方式，主要业务逻辑为：
 
 ![demo1](https://github.com/dengfeng520/LTSDKDocument/blob/master/demo1.png?raw=true)

 
(1)、客户端调用支付模块代码，由`SDK`向`LT`服务器发起一个订单请求；<br>
(2)、拿到服务器给的订单`ID`后，`SDK`发起苹果支付；<br>
(3)、苹果支付返回结果后，拿到支付`transaction`和`receipt_data`信息；<br>
(4)、`SDK`把苹果返回的`receiptData`信息发给服务端，`LT`服务器向苹果服务器校验；<br>
(5)、服务端对苹果服务器响应数据做处理和校验订单的合法性；<br>
(6)、通过服务器验证的结果来确定支付的结果；<br>

(7)、支付成功回调
支付成功回调的使用方法：需要CP服务器建立一个API供乐推服务调用回调地址所接受的参数格式

`METHOD: POST`

`HEADER:Content-Type:application/json`

`BODY:`

|参数 | 类型    |  说明|
| :-------------:|:-------------:| :-----:|
|`lt_order_id` | string     | 乐推订单号 |
|`pay_mode`  | int | 支付方式 |
|`custom`| array | dict|自定义数据|
|`app_name `|string |app名|
|`package_name`|string|包ID|
|`lt_gid`|int|乐推商品ID|
|`price `|double|金额|
|`abb`|string|币种缩写|
|`time `|int |支付时间戳|
|`token`|string|加密验证token|

CP方为了安全起见需要对token进行验证token的加密方法为md5({BODY的第二个参数}{APP_SECRET_KEY}{支付时间戳})BODY参数除了custom和token其他参数均为乱序，和上表顺序不一样，所以每次需要针对BODY取第二个参数APP_SECRET_KEY在乐推平台创建应用时生成支付时间戳为BODY中的time。



>###2、支付SDK简介

`SDK`支付模块目前给的有3个`.h`文件:<br>
(1)、`LTPayErrorCode`可以查看支付时的各种状态码或错误码；<br>
(2)、`GoodsModel`传人参数的`Model`文件，可以查看有哪些参数是必传的，哪些是非必传的；<br>
(3)、`LTPayManager`支付主类，用于完成支付、支付验证、支付回调等操作；

>###3、准备工作

###1、请在工程中设置`info.plist`中设置`Allow Arbitrary Loads`为YES！[设置方法请点这里](https://stackoverflow.com/questions/31254725/transport-security-has-blocked-a-cleartext-http)
###2、接入支付前请先[登录](https://dengfeng520.github.io/LTSDKDocument/LTGameSDK%E7%99%BB%E5%BD%95%E6%96%87%E6%A1%A3.html)！
###3、开启工程支付功能
![demo2](https://github.com/dengfeng520/LTSDKDocument/blob/master/demo2.jpeg?raw=true)
###4、其他更多细节请阅读[Apple官方文档：App内购买项目配置](https://help.apple.com/app-store-connect/#/devb57be10e7)

>###4、导入头文件

```
// 支付管理类
#import <LTGameSDK/LTPayManager.h>
// Model类
#import <LTGameSDK/GoodsModel.h>
```
>###5、初始化


```
@property (strong, nonatomic) LTPayManager *payManager;

```
**`SDK`提供了两套回调方法，通过`delegate`模式和`Blocks`方式；**

####方法一:使用自定义的初始化方法，通过Blocks回调

```
 //============================== 
 GoodsModel *goodsModel = [[GoodsModel alloc]init];
 // 
 goodsModel.gid = @"1";
 // 
 goodsModel.productId = @"com.gnetop.339Golds";
 goodsModel.custom = @{};
 //============================== 
 self.payManager = [[LTPayManager alloc]initPayWithViewModel:goodsModel SuccessBlocks:^(int code, NSDictionary * _Nonnull successInfoData, NSString * _Nonnull secuessInfoMessage) {
 } failureBlocks:^(int error, NSString * _Nonnull errorInfoMessage) {

 }];
//============================ 
 //实时监听当前的支付状态
 self.payManager.nowStatusBlocks = ^(int code, NSString * _Nonnull nowInfoMessage) {

 };

```
####方法二:使用alloc init的初始化方法，通过delegate回调

```
GoodsModel *goodsModel = [[GoodsModel alloc]init];
goodsModel.gid = @"1";
goodsModel.productId = @"com.gnetop.339Golds";
goodsModel.custom = @{};
//==============================
self.payManager = [[LTPayManager alloc]init];
self.payManager.goodsModel = goodsModel;
self.payManager.delegatePay = self;
```

如果使用了这种方式初始化，那么必须实现成功和失败代理：

```
-(void)paySuccessDelegate:(NSDictionary *)secuessInfoData secuessInfoMessage:(NSString *)secuessInfoMessage{
    NSLog(@"errorInfoMessage===========%@,%@",secuessInfoData,secuessInfoMessage);
}
// pay failure delegate
-(void)payFailureDelegate:(int)error errorInfoMessage:(NSString *)errorInfoMessage{
    NSLog(@"errorInfoMessage===========%d,%@",error,errorInfoMessage);
}

```

建议使用第一种方式初始化，使用`Blocks`回调。

>###6、回调返回参数

不论支付成功还是失败都返回状态码和对应的信息，
如支付成功后，返回的`code`为`1000`,同时返回了支付成功的键值对，举例：

```
code,secuessInfoData,secuessInfoMessage===========1000,{
    code = 200;
    msg = success;
    result = OK;
},==============Transaction is in queue, user has been charged.  Client should complete the transaction.
```

>###7、GoodsModel类

`GoodsModel`类中有四个参数，

```
// your APP Bundle Id
@property (nonatomic, copy) NSString *package_id;
//Must and cannot be null
@property (nonatomic, copy, nonnull) NSString *gid;
//商品ID 形如（com.test.1gold） Must and cannot be null
@property (nonatomic, copy, nonnull) NSString *productId
//预留的，数组或者字典Reserved, array or dictionary，Must and cannot be null
@property (nonatomic, copy, nonnull) id custom
```

>###8、错误码

在`LTPayErrorCode`类中返回了各种情况状态码：可根据返回的不同状态码来判断是哪里出错了。

```
    /// 购买成功
    TypePaySuccess = 1000,
    /// 请求订单时失败
    TypeGetOrderFailure = 1001,
    /// 没有商品 Product list is null
    TypeProductListIsNull = 10002,
    /// 商品ID和苹果给的不匹配Product ID and apple server do not match
    TypeProductIDNotMatch = 1003,
    /// 正在购买 buying
    TypeBuyingfromAppleServer = 1004,
    /// 已经收到Apple的购买成功通知 正在和服务器验证
    TypeMineServerVerifying = 1005,
    /// 购买失败
    TypePayFailureCode = 1006,
    /// 已经购买， Apple正在处理
    TypePaymentTransactionStateRestored = 1007,
    /// 正在购买中，最终状态还没确定
    TypePaymentTransactionStateDeferred = 1008,
    /// 服务器验证支付失败
    TypeServerPaymentVerificationFailed = 1009,  

```
