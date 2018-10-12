# vue不同组件（页面）间传值的方式
----------
标签（空格分隔）： this.$router.push
---

###  ---目录---
[TOC]

### 1. 路由传值
在跳转页面的时候，在js代码中的操作如下，在标签中使用<router-link>标签。
```
this.$router.push({
    name: 'routePage',
    query/params: {
        routeParams: params
    }
})
```
需要注意的是，实用params去传值的时候，在页面刷新时，参数会消失，用query则不会有这个问题。
这样使用起来很方便，但url会变得很长，而且如果不是使用路由跳转的界面无法使用。取值方式分别为：
```
this.$route.params.paramName和this.$route.query.paramName
```
    注：使用params传值，也可以做到页面刷新，参数不丢失，在命名路由时这样设置：
```
{
    path: '/OrderDetail/:orderId/:type',
    name: 'OrderDetail',
    component: OrderDetail,
    meta: {
        title: '订单详情',
        needAuth: true
    }
}
```
使用时：
```
this.$router.push({name: 'OrderDetail', params: {orderId: id, type: 'buy'}})
```
这种方式会把路由导航到 /OrderDetail/booking路径--([参考](http://router.vuejs.org/zh-cn/essentials/named-routes.html))
### 2. 通过parent,$chlidren等方法调取用层级关系的组件内的数据和方法
通过下面的方法调用：
```
this.$parent.$data.id  //获取父元素data中的id
this.$children.$data.id  //获取父元素data中的id
```
这样用起来比较灵活，但是容易造成代码耦合性太强，导致维护困难。
### 133. 通过eventBus传递数据
使用前可以在全局定义一个eventBus:
```
window.eventBus = new Vue();
```
在需要传递参数的组件中，定义一个emit发送需要传递的值，键名可以自己定义（可以为对象）:
```
eventBus.$emit('eventBusName', id);
```
在需要接受参数的组件重，用on接受该值（或对象）:
```
//val即为传递过来的值
eventBus.$on('eventBusName', function(val) {
    console.log(val)
})
```
最后记住要在beforeDestroy()中关闭这个eventBus:
```
eventBus.$off('eventBusName');
```
### 4. 通过localStorage或者sessionStorage来存储数据--([参考](http://blog.csdn.net/qq_32786873/article/details/70853819))
setItem存储value
用途：将value存储到key字段
用法：.setItem( key, value)
```
sessionStorage.setItem("key", "value"); 
localStorage.setItem("site", "js8.in");
```
getItem获取value
用途：获取指定key本地存储的值
用法：.getItem(key)
```
var value = sessionStorage.getItem("key"); 
var site = localStorage.getItem("site");
```
-----[链接](http://www.cnblogs.com/ygtq-island/p/6728137.html)-----



