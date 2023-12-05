---
title: vue中的绑定语法
date: 2019-11-10 21:51:45
tags: vue
---
# 一、绑定语法：
1. 什么是绑定语法: 就是在HTML中标记可能发生变化的位置的{{变量}}语法——学名: Interpolation 插值语法
2. 何时: 只要元素的内容中，某个位置可能随变量自动变化时，就要用绑定语法标记

# 二、指令:
1. v-bind:
```javascript
<div id="app">
    <h1>空气净化器</h1>
    <h2>{{pm25}}</h2>
    <h2>{{pm25<100?'img/1.png':
      pm25<200?'img/2.png':
      pm25<300?'img/3.png':
                'img/4.png'}}</h2>
     //属性名前有："就变成了{}
    <img :src="pm25<100?'img/1.png':
                pm25<200?'img/2.png':
                pm25<300?'img/3.png':
                         'img/4.png'">
  </div>
  <script>
    var vm=new Vue({
      el:"#app",
      data:{
        pm25:180
      }
    });
    setInterval(function(){
    //vm.data.pm25=
      vm.pm25=Math.random()*400
    },2000)
</script>
```
2. 控制显示隐藏:
   控制两个元素根据条件二选一显示隐藏
   <元素1  v-if="条件">
   <元素2  v-else>
    a. 结果: 每当new Vue扫描页面时，或者data中变量发生更改时，都会自动计算v-if=后的条件表达式的值。如果v-if="true"，则显示元素1，不显示元素2。否则如果v-if="false"，则不显示元素1，改为显示元素2
    b. 原理: 如果v-if="true"，则删除元素2，保留元素1
    - 如果v-if="false"，则删除元素1，保留元素2
    c. 强调:
    - v-else后不要写=和属性值！
    - v-if和v-else两个元素之间禁止插入其他元素！
