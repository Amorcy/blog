---
title: 通过JS获取网络类型,及当前所在系统类型
tags: [前端, JS, 移动端]
date: 2018-04-20
---

最近做移动端项目(Hybird)，PM要求前端通过判断网络类型以及是否在浏览器中判断用户在点击下载链接的时候，提醒用户是直接下载，还是切换网络之后下载。
于是写了以下方法判断网络类型,主要也是利用userAgent来做判断。
```JS
function getNetworkType() {  
  const ua = navigator.userAgent;  
  let networkStr = ua.match(/NetType\/\w+/) ? ua.match(/NetType\/\w+/[0]:'NetType/other';  
  networkStr = networkStr.toLowerCase().replace('nettype/', '');  
  let networkType;  
  switch (networkStr) {  
      case 'wifi':  
          networkType = 'wifi';  
          break;  
      case '4g':  
          networkType = '4g';  
          break;  
      case '3g':  
          networkType = '3g';  
          break;  
      case '3gnet':  
          networkType = '3g';  
          break;  
      case '2g':  
          networkType = '2g';  
          break;  
      default:  
          networkType = 'other'; 
  }  
  return networkType;  
}
```
<!-- more -->
接下来继续判断用户所在系统是Android还是IOS,且用户是否在浏览器中。
由于这个项目是嵌入在APP当中,所以如果要判断用户当前是否是在APP当中,则需要Android和IOS工程师在打壳的时候修改userAgent,加入我们想要的字段。
以下是Android打壳时候的代码:
```JAVA
private void initWebViewGlobalSettings() {
    String userAgent = webSettings.getUserAgentString();
    userAgent = userAgent + "xxx" + VersionUtils.getVersionName(this) + "$";
    webSettings.setUserAgentString(userAgent);
    mWebView.getSettings().setUserAgentString(userAgent);
}
```
端工程师帮我们修改完userAgent之后我们便可以来做判断了
```JS  
  export const isApp = /xxxx/i.test(window.navigator.userAgent)
  export const isAndroid = /android/i.test(window.navigator.userAgent)
  export const isIOS = /ios/i.test(window.navigator.userAgent)
  export const isWeixin = /micromessenger/i.test(navigator.userAgent)
  export const isWeChat = isWeixin
```
至此,用以上两个方法便可完成依据网络状况及系统类型的一些需求了。
