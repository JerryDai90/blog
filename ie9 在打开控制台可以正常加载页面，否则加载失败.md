# ie9 在打开控制台可以正常加载页面，否则加载失败

最近在调试个 bug，不开启控制台的话死活页面加载不出来。开启了控制台后页面就显示正常。
经过一顿搜索，得知在 ie9 下，如果不开启控制台的话，默认是没有 console 对象的，开启后则有。

汇总网上的解决方案，改造如下。新增 console.js，内容如下：

```javascript
try{
    console.log(11)
}catch (e){
    ;(function(g) {
        'use strict';
        var _console = g.console || {};
        var methods = ['assert', 'clear', 'count', 'debug', 'dir', 'dirxml', 'exception', 'error', 'group', 'groupCollapsed', 'groupEnd', 'info', 'log', 'profile', 'profileEnd', 'table', 'time', 'timeEnd', 'timeStamp', 'trace', 'warn'];

        var console = {version: '0.1.0'};
        var key;
        for(var i = 0, len = methods.length; i < len; i++) {
            key = methods[i];
            console[key] = function (key) {
                return function () {
                    if (typeof _console[key] === 'undefined') {
                        return 0;
                    }

                    Function.prototype.apply.call(_console[key], _console, arguments);
                };
            }(key);
        }

        g.console = console;
    }(window));
}
```

在具体页面引用的时候使用只有 ie9 才能识别的脚本

```html
<!--[if IE 9]>
	<script src="console.js" type="text/javascript"></script>
<![endif]-->

```

## 参考资料
[https://yanhaijing.com/javascript/2014/11/03/use-console.js/](https://yanhaijing.com/javascript/2014/11/03/use-console.js/)


