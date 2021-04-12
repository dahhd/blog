---
title: iOS网络请求类的简单封装
date: 2016-12-05 10:51:51

categories:
- 编程

tags:
- iOS
- OC

---

前言：

这篇文章主要讲解一下使用AFNetWorking(以下简称AFN)来自定义一个完整的网络请求类，来进行常用的网络请求后台数据的功能。 网上这样的例子很多，但是本文是基于AFNnetWorking的3.0以上版本的进行讲解的，同时也为支持ipv6协议（其实NSURLConnection也是支持ipv6的，博主模拟了ipv6运行环境，基于Connection的网络请求是没有任何问题的，也就是说AFN2.x以上，应该都没问题，这边提供一下 唐巧 的微信公众号放出来的文章：iOS应用支持 IPv6，就那点事儿），主要是针对NSURLConnection到NSURLSession的转变封装，至于网络请求的基类Session原理，后续文章会详细讲解。

下面直接看代码：

首先你的项目中应当已有AFN的第三方库的存在，可以使用pods来进行安装，至于还不会使用pods来管理第三方开源库的同学，请去自行谷歌，pods不在本篇的讨论范围之内。CocoaPods官网

首先我们新建一个继承自NSObject的自定义类，import引入AFN的头文件：

```
import <Foundation/Foundation.h>
import "AFNetworking.h
@interface HGHttpRequestManager : NSObject

```
实例化session网络请求管理者：
@property (nonatomic,strong)AFHTTPSessionManager *httpRequestManager;

设置这个类的单例方法，以便在实例化类的时候使用：

```
//BaseUrl，固定接口形式;yyy处填写不同模块需要拼接的请求短路径
const static NSString *BaseUrl = @"http://xxx.com/yyy";

static HGHttpRequestManager *manager = nil;

@implementation HGHttpRequestManager
//生成类对象单例，这都是知道的了（不知道留评论，我手把手教你，哈哈）
+ (HGHttpRequestManager *)sharedManager {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (!manager) {
            manager = [[super alloc] init];
        }
    });
    return manager;
}

```

在封装请求方法之前，这里先定义一个方法，用来拼接每次发起请求所需要的参数集合，返回一个Dictionary，这个字典的可变参数是可配置的，每次请求都有必传参数，首先在这个集合中定义完成，包括：签名、系统版本os、userToken、version、deviceID；这些都是每次请求必传参数列表：

```
//这个方法是封装一些基础参数，或者说每次请求API接口必传的参数，可根据自己的业务逻辑进行合理的封装
- (NSDictionary *)formatParameters:(NSMutableDictionary *)parameters{
    NSMutableDictionary *dictAllParams = [NSMutableDictionary dictionaryWithCapacity:0];
    //设置传入参数
    if (parameters) {
        [dictAllParams addEntriesFromDictionary:parameters];
    }
    [dictAllParams addEntriesFromDictionary:[self commonParameters]];
    //设置token
    if ([HGUserData sharedInstance].userToken) {
        [dictAllParams setValue:[HGUserData sharedInstance].userToken forKey:kUDKey_UserToken];
    }
    // 排序升序，签名value
    NSString *strSigned = [[HGSecurityManager sharedInstance] signParameters:dictAllParams withKey:HGX_KEY];
    [dictAllParams setValue:strSigned forKey:@"sign"];
    return dictAllParams.copy;
}
//生成字典，把结果return给上面的方法，以供GET或POST请求时获取
- (NSDictionary *)commonParameters{
    NSMutableDictionary *dict = [[NSMutableDictionary alloc]init];
    [dict safeSetValue:API_VER forKey:@"version"];
    [dict safeSetValue:[OpenUDID value] forKey:@"deviceId"];
    [dict safeSetValue:@"2" forKey:@"os"];
    return dict.copy;
}

```
封装GET方法：

```
- (void)getRequestWithPamameters:(NSMutableDictionary *)parameters
                          ToPath:(NSString *)path
                         success:(void (^)(NSDictionary *result))success
                         failure:(void (^)(NSError *error))failure
{
    //拼接参数
    NSDictionary *dictAllParams = [self formatParameters:parameters];
    //请求路径
    NSString *strUrl =  [NSString stringWithFormat:@"%@%@",BaseUrl,path];
    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    //设定可以接收的返回类型
    sessionManager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/html",@"text/json",@"text/javascript", nil];
    
    [sessionManager GET:strUrl parameters:dictAllParams progress:^(NSProgress * _Nonnull downloadProgress) {
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
    //成功或失败，用之前定义好的block把状态给回调出来
        if (success) {
            HGBaseModel *baseModel = [[HGBaseModel alloc]initWithDictionary:responseObject error:nil];
            //这里是一个code状态码，自己和后台约束设定的，code为99则表示用户还未登录，直接跳转到登录页面
            if (baseModel.code == 99) {
                [[HGUserData sharedInstance] setIsLogin:@NO];
                [self gotoLogin];
            }
            success(responseObject);
        }
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        if (failure) {
            failure(error);
        }
    }];
}

```

封装POST方法：

```
//参数列表使用封装的基本参数之后，会自动放到body之内
- (void)postRequestWithPamameters:(NSMutableDictionary *)parameters
                           ToPath:(NSString *)path
                          success:(void (^)(NSDictionary *result))success
                          failure:(void (^)(NSError *error))failure {
    //拼接参数
    NSDictionary *dictAllParams = [self formatParameters:parameters];

    //请求路径
    NSString *strUrl =  [NSString stringWithFormat:@"%@%@",BaseUrl,path];

    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    [sessionManager POST:strUrl parameters:dictAllParams progress:^(NSProgress * _Nonnull uploadProgress) {

    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        [self printSucceededResponseObject:responseObject withReqestUrl:strUrl andParams:dictAllParams];
        if (success) {
            success(responseObject);
        }
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        [self printError:error withReqestUrl:strUrl andParams:dictAllParams];
        if (failure) {
            failure(error);
        }
    }];
}

```

我们来看一下具体的使用方法，请求一个接口，返回一串json数据：


```
//首先，直接生成单例对象，调用封装好的GET方法：
//这里在请求时，加载一下loading转圈提示，提示用户在进行网络请求
[self showLoading];
[[HGHttpRequestManager sharedManager] getRequestWithPamameters:dictParams ToPath:kAPIPathOrderCancel success:^(NSDictionary *result) {
         //请求完成后，隐藏loading
        [self hideLoading];
        //model中封装的一个init方法，直接把请求结果result的dictionary格式数据转化成model存储
        OrderListModel *orderListModel = [[OrderListModel alloc]initWithDictionary:result error:nil];
        if (orderListModel.isValid) {

            [self showSucceededHud:orderListModel.msg];
//此时model有值，把model中的数组数据放到全局数组之中，这个数组就是数据源，然后调用自定义的一些方法，我这里是刷新订单列表OrderList
          [self.mArrayOrderList addObjectsFromArray:orderListModel.data];
          [self getOrderList];
          [self.listTableView reloadData];
        }else{
            [self showErrorHud:orderListModel.msg];
        }
    } failure:^(NSError *error) {
    //请求失败也要隐藏loading
        [self hideLoading];
    }];

```


这样一套完整的网络请求流程就完成了

这里有几点要说明下：
1、这里AFNetWorking使用的是3.1.0版本，3.0以下版本不适用于本文(如果需要，可以去看下源码，里面的请求类是RequestOperationManager)

2、Model封装方法就不展示了，model是基于JSONModel框架的，有兴趣的同学可以看下怎样继承，然后进行model解析

3、这里面的主要的一点是封装每次必传的请求参数，这是有些同学经常请求API写的很乱的一个原因所在。









