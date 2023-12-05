---
title: vue中的组件通信
date: 2019-12-01 21:49:02
tags: vue
---
vue中的组件通信无论是在做项目，还是笔试面试时都很重要

下面我们来说一说vue组件通信的三种情况

### 父->子
父子组件通信是利用自定义属性传值，该情形分为两步：
- 1.在父组件中，将子组件需要的成员绑定给一个自定义的属性。
- 2.在子组件中，通过props获得父组件放在子组件中的自定义属性值。

```javascript
//父组件
Vue.component('todo',{
    template:`
        <div>
            <h1>代办任务列表</h1>
            <todo-add></todo-add>
            <todo-list :msg='tasks'></todo-list>
        </div>
    `,
    data(){
        return {
            tasks:["吃饭","睡觉","打亮亮"]
        }
    },
    components:{
        todoAdd,
        todoList
    }
})
```
```javascript
//子组件
var todoList={
    template:`
        <ul >
            <todo-item v-for='(t,i) of tasks' :key='t' :t="t" :i="i"
            :tasks="tasks">
            </todo-item>
        </ul>
    `,
    props:['msg'],
    components:{
        todoItem
    }
}
```
### 子->父
子->父是自定义事件传值，分飞两步：
- 1.在父组件中，自定义一个事件，并绑定好事件处理函数
- 2.在子组件中，在自己的事件处理函数中通过$emit触发父组件的自定义事件，并传参

```html
<!--父组件-->
<template>
<div>
    <div>{{fa}}</div>
    <sun @show="showon"></sun>
</div>
</template>
<script>
import sun from "./child.vue"
export default {
    data(){
        return{
            fa:"这是父组件的值"
        }
    },
    methods:{
        showon(data){
            this.fa=data
        }
    },
    components:{
        sun
    }
}
</script>
```
```html
<!--子组件-->
<template>
    <div>
       <button @click="change">传递</button>
    </div>
</template>
<script>
export default {
    methods:{
        change(){
            this.$emit("show","这是子组件的内容")
        }
    }
}
</script>
```
### 兄弟之间传值
该情况有点特殊，分为三步：
- 1.在脚手架中创建一个公共的new Vue实例；用来保存事件，并关联到任何一个组件的处理函数上
- 2.数据的接受方组件，在自己加载完成后，向公共实例上添加一个自定义的事件，事件要绑定上自己的一个处理函数
- 3.发送方要在自己的事件处理函数中，找到公共实例对象，用$emit()触发别人提前在bus上绑定好的事件，并传参

1.单独创建一个bus.js
```javascript
import Vue from 'vue'
//一个公共的new Vue实例
export default new Vue();
```
2.main.js中
```javascript
import bus from './bus'
//放在原型对象上，任何一个组件都可以访问
Vue.prototype.bus=bus;
Vue.config.productionTip = false
new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```
3.数据的接受方,用$on向公共实例上添加一个自定义事件，并绑上自己的处理函数
```html
<template>
  <ul class="todo-list">
    <todo-item v-for="(task,i) of tasks" :key="i" :i="i">
      <template slot="task">{{task}}</template>
    </todo-item>
  </ul>
</template>
<script>
import todoItem from "./TodoItem"
export default {
  data(){
    return {
      tasks:['吃饭','睡觉',"打亮亮"]
    }
  },
  created(){
    //回调中的函数this指window，所以必须用bind永久绑定当前组件
    this.bus.$on("add_task",this.add.bind(this))
  },
  methods:{
    add(task){
      this.tasks.push(task);
    }
  },
  components:{
    todoItem
  }
}
</script>
```
4.数据的发送方,用$emit触发别人在公共实例上绑定好的事件，并传参

```html
<template>
  <div class="todo-add">
    <input type="text" v-model="task"><button @click="add">+</button>
  </div>
</template>
<script>
export default {
  data(){
    return {
      task:""
    }
  },
  methods:{
    add(){
      this.bus.$emit("add_task",this.task);
      this.task="";
    }
  }
}
</script>
```