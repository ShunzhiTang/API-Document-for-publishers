# 对接后检查清单

> 此处列出的所有对接项请自行阅读，并检查是否都已经符合要求，否则广告在投放过程中，将会出现流量被屏蔽，填充不高，收益较低等问题，请仔细核查检验

## 1. 客户端UA和IP地址上报

**请确认 `客户端UA` 和 `客户端IP地址` 通过 `http header` 在请求 ZPLAY Ads 平台时传递**

> 客户端UA、IP 如果不通过header传过来，将会取服务端的IP做为客户端地址, ZPLAY Ads平台 会将重复过多的IP当做作弊流量被屏蔽掉

## 2. 传递参数校验

* iOS请求中，请务必将设备的`IDFA`传递，否则无广告填充
* Android请求中，请务必传递`IMEI` `AdvertisingID` `AndroidID` `MAC`中任意一个参数，否则无广告填充

## 3. 关闭事件监听并关闭webview

* 请监听 ZPLAY Ads 的关闭事件 `close_playable_ads`，在监听到该事件后，关闭 Webview，返回游戏或者应用
### Android响应WebView的close事件示例
1. 允许WebView执行Javascript代码
```
mWebView.getSettings().setJavaScriptEnabled(true);
```
2. 添加Javascript接口
```
mWebView.addJavascriptInterface(new ZPLAYAdsJavascriptInterface(), "ZPLAYAds");
```
类ZPLAYAdsJavascriptInterface.java实现如下
```
private class ZPLAYAdsJavascriptInterface {

        @JavascriptInterface
        public void onCloseSelected() {
            // 可玩广告点击关闭按钮时，触发该方法
        }

        @JavascriptInterface
        public void onInstallSelected() {
            // 当点击"安装"按钮时，触发该方法
        }
    }
```
3. ZPLAY Ads关闭事件会调用```ZPLAYAdsJavascriptInterface.onCloseSelected()```方法，请在此回调中处理关闭事务。

### iOS监听WKWebView的关闭事件

1.WKWebView添加```zplayads```脚本消息处理程序

```objective-c
- (WKWebView *)previewAdWebView {
    if (!_previewAdWebView) {
        WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
        [config.userContentController addScriptMessageHandler:self name:@"zplayads"];
        config.allowsInlineMediaPlayback = YES;
        CGRect frame =
            CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height);
        _previewAdWebView = [[WKWebView alloc] initWithFrame:frame configuration:config];
        _previewAdWebView.scrollView.bounces = NO;
        _previewAdWebView.navigationDelegate = self;
        _previewAdWebView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
      
        if (@available(iOS 11.0, *)) {
            [_previewAdWebView.scrollView setContentInsetAdjustmentBehavior:UIScrollViewContentInsetAdjustmentNever];
        } 
    }
    return _previewAdWebView;
}
```

2.监听```WKScriptMessageHandler```的代理方法

```objective-c
#pragma mark - WKScriptMessageHandler
- (void)userContentController:(WKUserContentController *)userContentController
      didReceiveScriptMessage:(WKScriptMessage *)message {
    if ([message.name isEqualToString:@"zplayads"]) {
        [self handlePlayablePageMessage:message.body];
    }
}
// handle message
- (void)handlePlayablePageMessage:(NSString *)msg {
   if ([msg isEqualToString:@"user_did_tap_install"]) {
        NSLog(@"user_did_tap_install");
    }else if ([msg isEqualToString:@"close_playable_ads"]) {
        NSLog(@"close zplayads...");
    } 
}
```

## 4. 点击事件监听并打开应用市场

* 请监听 ZPLAY Ads 的点击事件  `user_did_tap_install`，在监听到该事件后，在应用内打开 APP Store 或者 Google Play，并打开跳转链接。
### Android响应WebView的install事件示例
参考 **Android响应WebView的close事件示例** ，点击事件会回调```ZPLAYAdsJavascriptInterface.onInstallSelected()```方法，请在此方法中处理打开应用市场事务。

### iOS监听WKWebView的安装事件

参考 **iOS监听WKWebView的关闭事件**，点击事件会返回```user_did_tap_install```，在```handlePlayablePageMessage:```处理您的跳转逻辑。

打开内置AppStore代码示例：[点击此处]()