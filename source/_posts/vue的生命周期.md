---
title: vue的生命周期
date: 2019-10-27 21:48:51
tags: vue
---
- ### 1. 什么是组件的生命周期:
    每个组件在加载过程中所经历的阶段
- ### 2. 为什么使用组件的生命周期:
    我们经常需要在组件加载过程中自动执行一项操作，比如: 希望组件加载完，自动发送ajax请求获取数据
  但是在单页面中，因为只有一个index.html页面在首次加载，之后更换"页面", index.html是不换的，且DOM树也不需要整棵更新，所以DOMContentLoaded和window.onload都不能用，都不会反复触发！   
- ### 3.何时使用？
    今后，只要希望在组件加载的过程中自动执行一项任务时，都要在对应的生命周期中执行操作，而不是在页面的加载后执行。
- ### 4.包括以下几个阶段？
    (1). 创建阶段(create):
    a. 创建组件对象，同时创建组件中的data对象。此时已经可以发送ajax请求了
    b. 暂时还未创建虚拟DOM树，所以，在这个阶段不能执行DOM操作！
 注意：此阶段只有data对象，没有虚拟dom树
    (2). 挂载阶段(mount): 创建虚拟DOM树，将数据内容渲染到页面上显示。此时既可以发ajax请求，又可以执行DOM操作。
    *************首次加载到此结束*******************
    (3). 更新阶段(update): 只有data中的模型数据被更新时，才会触发更新阶段
    (4). 销毁阶段(destroy): 只有主动调用$destroy()函数删除一个组件时，才会触发销毁阶段。

- ### 5.钩子函数
    (1). 什么是钩子函数: 绑定在生命周期各个阶段，自动触发的特殊的回调函数。
    (2). 包括: 每个阶段，一前一后，都有两个钩子函数
```javascript
     beforeCreate(){ ... }
    a. create阶段
        created(){ ... ajax ... }
        beforeMount(){ ... }
    b. mount阶段
        mounted(){ ... ajax  ...  或  DOM操作 ...}
        beforeUpdate(){ ... }
    c. update阶段
        updated(){ ... }
        beforeDestroy(){ ... }
    d. destroy阶段
        destroyed(){ ... }
```
- ### 6. 如何使用钩子函数：
```javascript
组件对象{
    data(){
        return { ... }
    },
    methods:{ ... },
    watch:{ ... },
    computed:{ ... },
    components:{ ... },
    beforeCreate(){ ... },
    created(){ ... ajax ... },
    beforeMount(){ ... },
    mounted(){ ... ajax ... },
    beforeUpdate(){ ... },
    updated(){ ... },
    beforeDestroy(){ ... },
    destroyed(){ ... }
}
```
