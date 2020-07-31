# Vue 动态表单验证

核心操作：FormItem 的 :prop 需要与控件的 v-model 有一致的路径。
> 由于验证完成后需要判断值是否为空，那这个时候需要有个地方可以映射到这个值，此时:prop这个这个值对应的就是key

在固定表单需求中，通过指定 :prop，:rules 即可等到验证功能，但是对于动态生成的表单，业务逻辑来说，也就是需要动态生成 :prop 与 :rules。比如说：

```html
<Form :model="formItem" :rules="rules">
    <FormItem :prop="path1.path2.path3">
    </FormItem>
</Form>
```

验证的写法

```javascript
rules : {
    //一定要与 prop 值一致
    "path1.path2.path3" : [{required: true, message: 'xx不能为空', trigger: 'blur'}]
},
//这里的路径也是需要一致的，porp中的 点会分解调用的 formItem 里面的对象
formItem : {
    path1 : {
        path2 : {path3 : ""}
    }
}
```

以上的写法固定表单需求是可以触发验证的，但是如何需要动态增减表单对象以及验证规则，则需要编写 validator，手动验证。

### 源码分析
为什么 rules 中的 `"path1.path2.path3"` 可以映射到 formItem中的对象。可以打开 form-item.vue 文件

![](http://img.lsof.fun/2020-08-01-15962120140226.jpg)

也就是如果你是动态生产的表单，只需要按照这种规范生产`rules`以及在`formItem`里面存在相同路径的都是可以调用验证


