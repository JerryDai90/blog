# 微信返回按钮触发关闭（android 和 iOS 都支持）

## 场景
微信认证需要跳转多层页面，这个这个时候如果用户点击返回的话就会在这个几个页面中循环跳来跳去的（比如登陆成功后，我们去到首页，这个时候用户用完后点击了返回。就又去掉了登陆页面去了），so，我们需要用户返回到登陆页面的时候就认为用需要关闭页面。

## 问题
进过多次试验得出结果如下
android 的页面放回每次都是刷新页面的，iOS 的返回是缓存的，不刷新页面。但是我们想要的效果是返回到指定页面的时候就触发关闭操作


一轮搜索，都是介绍使用监听 `popstate` 方法来实现，在 android 上的

2次就会回到指定链接

进入2此之后，就触发不了popstate事件了（经过测试，如果这个时候的返回页面就是你在 history.pushState 的页面地址，也就是说会重新载入这个页面，也就没有 popstate 什么事了）
history.pushState(null, null, "index4menu.jsp?xxxx"); 还会缓存



window.addEventListener('pageshow', function () {
在点了2次以上的页面依旧会触发，但是却关闭不了页面

## 解决方案
### 思路
结合 `sessionStorage` 的特性（当前地址有页面打开的时候就存在，关闭就删除），我们使用一个变量存储在这个里面，一开始的时候就存储标记为，等到再次返回的时候触发到这个标记位后就关闭页面。

### 代码

先引入微信 js 

```html
<script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
```
示例代码

```javascript
var common = {};
/**
* （WeixinJSBridge）微信内置对象不是一开始就加载的，需要做回调，
*  jerry dai 20180917
*/
common.wechatJSBridgeReady : function (callback) {
    if (typeof WeixinJSBridge == "object" && typeof WeixinJSBridge.invoke == "function") {
        callback();
    } else {
        if (document.addEventListener) {
            document.addEventListener("WeixinJSBridgeReady", callback, false);
        } else if (document.attachEvent) {
            document.attachEvent("WeixinJSBridgeReady", callback);
            document.attachEvent("onWeixinJSBridgeReady", callback);
        }
    }
};
 /**
 * 触发微信返回的时候关闭当前页面函数（主要用于登陆成功后跳转，在点击退回的时候发现停留在了跳转页面中）
 * @param key 缓存的 key（主要用于多个页面跳转的时候不冲突使用，不传递默认是isClose ）
 * @param fnOnSetFlagCallback 设置关闭标记的时候回调（主要用户控制页面的逻辑初始化）
 * @param fnCloseCallback 关闭的时候回调
 * jerry dai 20180917
 */
common.goBackCloseWechatWindow : function (key, fnOnSetFlagCallback, fnCloseCallback) {
	key = key || "isClose";
    window.addEventListener('pageshow', function () {
        var isClose = sessionStorage.getItem(key);
        // alert(key+":"+isClose+", "+window.location.href)
        if( isClose == "true" ){
            (fnCloseCallback && fnCloseCallback)();
            common.wechatJSBridgeReady(function () {
                WeixinJSBridge.call('closeWindow');
            });
        }else{
            sessionStorage.setItem(key, "true");
			(fnOnSetFlagCallback && fnOnSetFlagCallback)();
        }
    });
};

```

## 参考资料
https://github.com/Tencent/weui/wiki/微信JSAPI



