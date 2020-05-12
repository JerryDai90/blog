# window.postMessage 跨域调用方法

父子页面之间的通讯，如果同源的通过 window对象直接调用父(子)中的方法与对象（全局方法与对象）。如果出现跨域的，多系统情况下这种就被同源策略禁止掉了。因此需要使用其他方法解决。可以使用 window.postMessage 方法，附上技术规范 [https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)

> vue 等工程也同样适用

## 案例

由于 postMessage 是单向的调用，是没有回调概念的，下面的案例自行封装了一套代码，父页面发送请求后，子页面处理完成后即可回调函数，而不需要手动声明。

**parent.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>parent.html</title>
</head>
<body>
<span>parent.html</span>
<br>
<br>
<br>
<iframe src="./child.html"></iframe>
<br>
<br>
<button onclick="callChild()">调用子页面</button>
</body>

<script>

    let callChild = function () {
        window._postMessage("childFunction", "my is params", function (data) {
            alert("parent: callChild childFunction(), return "+data);
            console.log("p:" + data);
        })
    }

    /**
     * 支持直接对子页面进行调用，而且子页面处理完成会回调 _callback 方法
     * @param _method iframe中的函数
     * @param _params 参数
     * @param _callback 调用 _method 成功后返回的数据
     * @private
     */
    window._postMessage = function (_method, _params, _callback) {

        let messageBody = {
            method: _method,
            params: _params,
        }

        if (_callback != undefined) {
            let proxyMethod = "proxy" + parseInt(Math.random() * 1000);
            //用于回调当前页面的方法使用
            messageBody.returnMethod = proxyMethod;
            window[proxyMethod] = function (_p) {
                try {
                    _callback(_p);
                } finally {
                    delete window[proxyMethod];
                }
            }
        }

        //TODO 这里的对象需要按需求修改
        let iframeWin = document.getElementsByTagName("iframe")[0].contentWindow;
        iframeWin.postMessage(messageBody, '*');

    }

    /**
     * 监听其他地方调用的对象信息，由于回调
     */
    window.addEventListener("message", function (event) {
        let data = event.data;
        if (null == data || data.method == undefined) {
            return;
        }
        //判断是调用者还是被调用者的数据返回（存在的话就意味着是别人调用你，需要执行完成本地方法后执行回调）
        if (data.returnMethod != undefined) {
            let returnData = window._callActualMethod(data.method, data.params);
            //回调
            window._postMessage(data.returnMethod, returnData);
        } else {
            //这里是直接调用自身的方法（前面调用的时候已经将此方法丢到window里面了这里回调就可以调用到 window[proxyMethod] 方法）
            window._callActualMethod(data.method, data.params);
        }
    });

    window._callActualMethod = function (_method, _params) {
        // console.log(data.method);
        try {
            //这里可以去实现具体调用的方法是在 vue 作用域还是其他作用域中的函数
            return window[_method](_params);
        } catch (e) {
            console.warn(e);
        }
    }

    window.parentFunction = function () {
        return "parent";
    }


</script>
</html>

```

**child.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>child.html</title>
</head>
<body style="background-color: red">

<span>child.html</span>
<br>
<br>
<button onclick="callParent()">调用父页面</button>

</body>

<script>

    var callParent = function () {
        window._postMessage("parentFunction", "my is params", function (data) {
            alert("child: callParent parentFunction(), return "+data);
            console.log("c:" + data);
        })
    }

    //支持直接对子页面进行调用，而且子页面处理完成会回调 _callback 方法
    window._postMessage = function (_method, _params, _callback) {

        let messageBody = {
            method: _method,
            params: _params,
        }

        if (_callback != undefined) {
            let proxyMethod = "proxy" + parseInt(Math.random() * 1000);
            //用于回调当前页面的方法使用
            messageBody.returnMethod = proxyMethod;
            window[proxyMethod] = function (_p) {
                try {
                    _callback(_p);
                } finally {
                    delete window[proxyMethod];
                }
            }
        }

        //TODO 这里的对象需要按需求修改
        let iframeWin = window.parent;
        iframeWin.postMessage(messageBody, '*');

    }

    /**
     * 监听其他地方调用的对象信息，由于回调
     */
    window.addEventListener("message", function (event) {
        let data = event.data;
        if (null == data || data.method == undefined) {
            return;
        }
        //判断是调用者还是被调用者的数据返回（存在的话就意味着是别人调用你，需要执行完成本地方法后执行回调）
        if (data.returnMethod != undefined) {
            let returnData = window._callActualMethod(data.method, data.params);
            //回调
            window._postMessage(data.returnMethod, returnData);
        } else {
            //这里是直接调用自身的方法（前面调用的时候已经将此方法丢到window里面了这里回调就可以调用到 window[proxyMethod] 方法）
            window._callActualMethod(data.method, data.params);
        }
    });

    window._callActualMethod = function (_method, _params) {
        // console.log(data.method);
        try {
            //这里可以去实现具体调用的方法是在 vue 作用域还是其他作用域中的函数
            return window[_method](_params);
        } catch (e) {
            console.warn(e);
        }
    }


    window.childFunction = function () {
        return "child";
    }

</script>
</html>

```




