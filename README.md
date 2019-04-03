# 环锂SDK集成文档
---
<br>
## 适用范围
HuanLiSDK适用于iOS10.0及以上操作系统<br>
<br>

# 集成准备
---
<br>
## 创建应用并获取AppKey和AppSercret
AppKey是环锂用来标示App的唯一标识符，集成SDK前需要在创建应用并获取相应的AppKey。
请开发者到官网注册自己的账号，创建应用用程序并获取对应的AppKey、AppSercret。
具体操作请查看如下网址：网址链接 
<br>

# 快速集成
---
可使用Cocoapods进行自动集成或手动下载集成。
<br>
## 自动集成（Cocoapods）
    target 'HuanLiDemo' do
    pod 'HLApi'
    end
 <br>   
## 手动集成
在官网下载最新版本SDK，包括HLApi.framework和HLImages.bundle 以及第三方分享SDK：QQ、微信、微博
<br>
### 导入SDK
将下载的SDK添加到工程
<br>
### 添加项目配置
target->Build Settings -> Other Linker Flags 加入-ObjC
<br>
### 加入依赖系统库
General->Linked Frameworks and Libraries 添加依赖库<br>
添加的是第三方分享SDK需要依赖的库<br>
    CFNetwork.framework
    Photos.framework
    CoreGraphics.framework
    Foundation.framework
    AVFoundation.framework
    CoreText.framework
    UIKit.framework
    Secrity.framework
    SystemConfiguration.framework
    ImageIO.framework
    QuartzCore.framework
    libc++.tbd
    libsqlite3.tbd
    libz.tbd
 <br>
### 配置SSO白名单
因应用使用了跳转到第三方分享功能，需要增加一个可跳转的白名单，即LSApplicationQueriesSchemes，否则将在SDK判断是否跳转时用到的canOpenURL时返回NO，进而只进行webview授权或授权/分享失败。在项目中的info.plist中加入应用白名单，右键info.plist选择source code打开(plist具体设置在Build Setting -> Packaging -> Info.plist File可获取plist路径)
<br>
```
<key>LSApplicationQueriesSchemes</key>
<array>
    <!-- 微信 URL Scheme 白名单-->
    <string>wechat</string>
    <string>weixin</string>
    <!-- 新浪微博 URL Scheme 白名单-->
    <string>sinaweibohd</string>
    <string>sinaweibo</string>
    <string>sinaweibosso</string>
    <string>weibosdk</string>
    <string>weibosdk2.5</string>
    <!-- QQ、Qzone URL Scheme 白名单-->
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
    <string>mqqopensdkapiV4</string>
    <string>mqzoneopensdk</string>
    <string>wtloginmqq</string>
    <string>wtloginmqq2</string>
    <string>mqqwpa</string>
    <string>mqzone</string>
    <string>mqzonev2</string>
    <string>mqzoneshare</string>
    <string>wtloginqzone</string>
    <string>mqzonewx</string>
    <string>mqzoneopensdkapiV2</string>
    <string>mqzoneopensdkapi19</string>
    <string>mqzoneopensdkapi</string>
    <string>mqqbrowser</string>
    <string>mttbrowser</string>
    <string>tim</string>
    <string>timapi</string>
    <string>timopensdkfriend</string>
    <string>timwpa</string>
    <string>timgamebindinggroup</string>
    <string>timapiwallet</string>
    <string>timOpensdkSSoLogin</string>
    <string>wtlogintim</string>
    <string>timopensdkgrouptribeshare</string>
    <string>timopensdkapiV4</string>
    <string>timgamebindinggroup</string>
    <string>timopensdkdataline</string>
    <string>wtlogintimV1</string>
    <string>timapiV1</string>
</array>
```
<br>
### 配置URL Scheme
* URL Scheme是通过系统找到并跳转对应app的一类设置，通过向项目中的info.plist文件中加入URL types可使用第三方平台所注册的appkey信息向系统注册你的app，当跳转到第三方应用授权或分享后，可直接跳转回你的app。
* targets->info->URL Types 添加 URL Types
* 配置第三方平台URL Scheme 
|平台|格式|举例|
|:---|:---|:---|
|微信|微信appkey|wxdc1e388c3822c80b|
|QQ|"tencent"+腾讯QQ互联应用appID|tencent100424468 |
|微博|“wb”+新浪appKey|wb3921700954|
<br>

# SDK 使用
---
<br>
## 初始化设置
应用启动后进行HuanLiSDK和第三方平台的初始化工作，实现sdk回调
<br>
```OC
#import <HLApi/HLApi.h>
@interface AppDelegate ()<HLSdkProtocol,HLShareResultProtocol>

@end

@implementation AppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // HLAPPKEY、HLAPPSECRET 是官网申请的appkey和appsecret
    // 指定delegate 接收SDK回调 (HLSdkProtocol)
    [HuanLiSDK registerHLApp:HLAPPKEY appSercret:HLAPPSECRET delegate:self];
    // 第三方平台初始化
    // 指定delegate接收分享结果回调 (HLShareResultProtocol)
    [HuanLiSDK registerQQApp:kThreeQQAppId WXApp:kThreeWeiXinAppId WBApp:kThreeSinaAppKey delegate:self];
    
    return YES;
}

// webview接收到交互消息 type 交互类型   object 附加参数
- (void)didReceiveWebViewMessageType:(HLScriptMessageActionType)type object:(NSString *)object
{
    
}

// 分享结果回调 statusCode分享结果   errMsg 失败信息   shareType 分享平台
- (void)shareResponseStatusCode:(HLShareResponseStatusCode)statusCode errMsg:(NSString *)errMsg shareType:(HLShareType)shareType
{
   
}
```
<br>
## 设置系统回调

```OC
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    return [HuanLiSDK handleOpenURL:url];
}
```
<br>

