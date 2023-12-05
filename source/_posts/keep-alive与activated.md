---
title: keep-alive与activated
date: 2020-01-06 21:09:58
tags: vue
---
### keep-alive组件
当组件在进行切换时，你有时候想保持这些组件的状态，以避免反复重渲染导致的性能问题。
keep-alive组件可以缓存组件的内容，避免组件反复加载，影响效率

比如：我们缓存一个页面
在router.js或router/index.js中，在需要缓存的路由上添加meta:{ keepAlive:true }
```javascript
...
const routes = [
...
  {
    path: '/',
    name: 'home',
    component: Home,
    meta:{
      keepAlive:true
    }
  },
...
]
...
```
注意：meta不能改名，因为meta是路由对象专门定义，用来保存自定义属性值的配置项；但是keepAlive是自定义的属性名，可以改名。

在App.vue中

```html
...
<keep-alive>
    <!-- 需要缓存的路由 -->
   <router-view v-if="$route.meta.keepAlive" />
</keep-alive>
    <!-- 不需要缓存的路由 -->
<router-view v-if="!$route.meta.keepAlive" />
...
```
除此之外，你也可以用在动态组件上
```html
<!-- 失活的组件将会被缓存！-->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

### activated
当引入keep-alive 的时候，页面第一次进入，钩子的触发顺序：
created-> mounted-> activated，退出时触发deactivated。
当再次进入（前进或者后退）时，只触发activated。

比如在vue脚手架中path为“/”的 home组件
```javascript
...
export default{
    ...
    activated(){
        console.log('home组件激活')
    },
    deactivated(){
        console.log('home组件失活')
    }
    ...
}
...
```
在另一个path为“/about”的 about组件中
```javascript
...
export default{
    ...
    activated(){
        console.log('about组件激活')
    }
    ...
}
...
```
特别注意：
activated和deactivated钩子只能和keep-alive配合使用