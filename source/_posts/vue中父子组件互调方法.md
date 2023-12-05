---
title: vue中父子组件互调方法
date: 2020-01-01 21:47:15
tags: vue
---
在做项目时，只记得vue组件可以传递值，却忽略了组件之间也可以传递方法

我们先说说ref这个特殊属性：
ref 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 $refs 对象上。如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例
[官方文档解释](https://cn.vuejs.org/v2/api/#ref)

### 父组件调用子组件的方法
当父调用子的方法时，要用到ref这个特殊属性
父组件：
```javascript
<template>
<div>
    <h1>这是父组件</h1>
    <button @click="go">调用子组件方法</button>
    <hr>
    <child ref="c"></child>
</div>
</template>
<script>
import child from '../components/child'
export default {
    components:{
        child
    },
    methods:{
        go(){
            //this.$refs.c指向child这个组件
            this.$refs.c.childMethod()
        }
    }
}
</script>
```
子组件：
```javascript
<template>
<div>
    <h3>这是子组件</h3>    
</div>
</template>
<script>
export default {
    methods:{
        childMethod(){
            console.log('这是子组件中的方法')
        }
    }
}
</script>
```
该结果是：当点击‘调用子组件方法’这个按钮时，则会在控制台输出‘这是子组件中的方法’

### 子组件调用父组件的方法时
当子调用父的方法时，分为以下三种情况

#### 1.子组件通过使用this.$parent.xxx
父组件
```javascript
<template>
<div>
    <h1>这是父组件</h1>
    <child></child>
</div>
</template>
<script>
import child from '../components/child'
export default {
    components:{
        child
    },
    methods:{
        fatherMethod(){
            console.log('这是父组件中的方法')
        }
    }
}
</script>
```
子组件
```javascript
<template>
<div>
    <h3>这是子组件</h3>    
    <button @click="goFa">调用父组件中的方法</button>
</div>
</template>
<script>
export default {
    methods:{
        goFa(){
            //this.$parent指向父组件
            this.$parent.fatherMethod()
        }
    }
}
</script>
```
该结果是：当在子组件中点击‘调用父组件中的方法’这个按钮时，则会在控制台输出‘这是父组件中的方法’

#### 2.子组件通过使用this.$emit('xxx')
跟子传值给父组件是一样的道理，通过触发父组件中的一个自定义事件，从而触发这个自定义事件的处理函数
父组件：
```javascript
<template>
<div>
    <h1>这是父组件</h1>
    <child @fatherMethod='fatherMethods'></child>
</div>
</template>
<script>
import child from '../components/child'
export default {
    components:{
        child
    },
    methods:{
        fatherMethods(data){
            console.log('这是父组件中的方法',data)
        }
    }
}
</script>
```
子组件：
```javascript
<template>
<div>
    <h3>这是子组件</h3>    
    <button @click="goFa">调用父组件中的方法</button>
</div>
</template>
<script>
export default {
    methods:{
        goFa(){
            this.$emit('fatherMethod','haha')
        }
    }
}
</script>
```
该结果是：当在子组件中点击‘调用父组件中的方法’这个按钮时，则会在控制台输出‘这是父组件中的方法 haha’

#### 3.父组件把方法名传给子组件，在子组件里调用这个方法
该方法原理类似于父传值给子，在父组件设置自定义属性，并在子组件通过props接收这个属性；
但注意：这里传递方法，必须要设置props

```javascript
<template>
<div>
    <h1>这是父组件</h1>
    <child :fatherMethod='fatherMethods'></child>
</div>
</template>
<script>
import child from '../components/child'

export default {
    components:{
        child
    },
    methods:{
        fatherMethods(data){
            console.log('这是父组件中的方法',data)
        }
    }
}
</script>
```
子组件：

```javascript
<template>
<div>
    <h3>这是子组件</h3>    
    <button @click="goFa">调用父组件中的方法</button>
</div>
</template>
<script>
export default {
    //这里必须设置props
    props:{
        fatherMethod: {
            type: Function,
            default: null
      }
    },
    methods:{
        goFa(){
            this.fatherMethod('haha')
        }
    }
}
</script>
```
该结果是：当在子组件中点击‘调用父组件中的方法’这个按钮时，则会在控制台输出‘这是父组件中的方法 haha’