# jQuery插件开发-面向对象

最近在做一个 OA 项目，需要重写整个前端页面。一开始整个工作并非是我牵头主导开发，缺少思考。导致后面扩展、开发困难。以至于牵一发动全身。总结一下经验。让童鞋开发的时候好注意一下。（因为这个锅现在是我背，我需要优化。。。）

我们如果做面向对象开发的时候，我们知道，一切皆对象。内部的封装细节我们都不需要知道，我们只要知道外部调用逻辑即可。基于这种思想我们也可以套用到前端的 JavaScript 中去。用过很多前端控件的童鞋都知道，在使用这些控件的时候提供很多切面方法给我们回调使用来实现定制需求。以下介绍也是围绕着此处展开。

## 插件实现想法
提供丰富的切面方法，如果是一个树形控件对象封装，基本要提供获一下方法

* 获取数据源回调方法（用于构建树形结构数据）
* 实例创建成功方法回调
* 点击选择事件回调
* 多选选中事件回调

。。。等。

以上只是列举了部分业务场景需要用的方法。详细需要自行结合业务需求。

这样的话使用此控件人只需要组合这些切面方法就可以使用该有的功能。其他细节不需要处理。按照功能性的不同提供差异化的接口处理方式。但是更加建议在一套系统中插件提供的切面处理是一致的（因为业务千变万化，实际操作中可能做不到。）

## 实现方式

### 常用实现方式

最常用的方法是在对象实例化的时候把相关切面方法带入到指定的参数中去，如下：

```javascript
$('#test').tree({
    on: {
        /**
         * 点击事件触发的回调方法（单选是点击节点触发，多选是勾选时触发）
         * @param self 当前对象
         * @param treeNode 点击的节点
         * @param clickItemObj 选中的对象信息
         */
        onClickItem: function (self, treeNode, clickItemObj) {
        },
        /**
         * 获取构建 ztree 的数据，数据结构和 ztree 要求的格式一致
         * @param callback 构建好的数据回调（必须）
         */
        onGetTreeData: function (callback) {
            callback();
        },
        onHide : function () {},
        onChange : function () {},
        onInitAfter : function () {}
    }
});
```
这种更多是用于和此对象非常结合很紧密的场景。需要拿到对象的实例化权利才可以做到。

### 基于 jQuery data 对象注册方式
适用场景：你无法掌控对象的创建（也就是没办法在初始化的时候就载入切面回调方法）。或者说你无法提供在初始化的时候个性化切面方法的需求就可以采用这个方式了。

```javascript
$('#test').data('callback', {
    on: {
        /**
         * 点击事件触发的回调方法（单选是点击节点触发，多选是勾选时触发）
         * @param self 当前对象
         * @param treeNode 点击的节点
         * @param clickItemObj 选中的对象信息
         */
        onClickItem: function (self, treeNode, clickItemObj) {
        },
        /**
         * 获取构建 ztree 的数据，数据结构和 ztree 要求的格式一致
         * @param callback 构建好的数据回调（必须）
         */
        onGetTreeData: function (callback) {
            callback();
        },
        onHide : function () {},
        onChange : function () {},
        onInitAfter : function () {}
    }
});
```
这里留有缺陷，就如果一个对象被多次调用以上方法的话。只会保留最后一个对象。（这里可以改动先判断是否有对象，有对象就变成一个 list，代码我懒得改了 -_-）。

在使用此方法注册切面方法的时候需要在编写插件的时候去获取 `data` 对象里面的方法进行回调

以上只是提供一种思路，具体还有很多实现方式的。

## 参考资料
jQuery data 对象介绍 [https://api.jquery.com/data/](https://api.jquery.com/data/)
jQuery 插件编写 [https://www.kancloud.cn/kancloud/javascript-standards-reference/46484](https://www.kancloud.cn/kancloud/javascript-standards-reference/46484)
JavaScript 模块化编程 [https://www.kancloud.cn/kancloud/javascript-standards-reference/46490](https://www.kancloud.cn/kancloud/javascript-standards-reference/46490)


