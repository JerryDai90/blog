# JS 内存泄露排查

在后台在管理界面直接编辑 JS，类是于于下面这些

```json
{
  "data": ${data},
  "name": "xxxx",
  "testFun" : function(){}
}
```

`${data}` 就是动态的数据，到前端之前会做替换。然后每次 ajax 获取回来的时候构建一下然后赋值给前端控件。但是通 过 `new Function` 的方式实现对JS的编译。

```
var convertToFn = function (fun) {
    var f2 = (new Function("return " + fun))();
    return f2;
}
```

上线后，这种方式不出一天浏览器的内存已经被耗尽（10秒钟调用后台数据进行对控件局部刷新）。

![w300](http://img.lsof.fun/2018-04-20-15242356349254.jpg)

## 解决方案
不在支持后台一改动JS前端就可以识别，改动了JS需要刷新页面，但是依然支持编写JS脚本（计算和方法调用）。
1. 先将编写的JS输出到指定的方法里面（这里JSP的通过EL表达式输出到指定的位置，也可以通过其他动态的计算实现）
2. 外网有一个空的对象，每次只需要重新对此值进行重新赋值
3. 获取新的只需要调用getData 方法即可完成计算逻辑。

```javascript
//临时对象（通过动态赋予此对象来获得值更新）
var dynamicData = {};

//调用此方法来获取最新的值，而不用每次都将真个JS构建
var getData = function(){

    //将后台编辑的JS对象输出到界面
    return {
      "data": dynamicData,
      "name": "xxxx",
      "testFun" : function(){}
    }
}

//动态刷新数据
setInterval(function(){
    //赋值给此变量，后面调用的时候就可以拿到最新的了
    data = Math.random(1000000)*100;
    //这里就可以拿到最新的计算后的数据了
    getData();
}, 10);
```
通过改造后，内存已经稳定。不在增长。

