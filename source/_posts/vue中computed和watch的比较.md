---
title: vue中computed和watch的比较
date: 2019-11-20 21:58:21
tags: vue
---
### computed：
```html
<template>
  <div>
    <p>{{ reversedMessage }}</p>
  </div>
</template>
 
<script>
    export default {
        name: 'test1',
        data () {
            return {
            message: 'hello world',
            number: 1
            }
        },
        computed: {
            // 字符串反转
            reversedMessage () {
            return this.message.split('').reverse().join('') + this.number
            }
        }
    }
</script>
```
在computed中定义的每一个计算属性，都会被<strong style="color:red">缓存</strong>起来，只有当计算属性里面依赖的一个或多个属性变化了，才会重新计算当前计算属性的值。上面的代码片段中，在reversedMessage中，它依赖了message和number这两个属性，一旦其中一个变化了，reversedMessage会立刻重新计算输出新值。

 vue会缓存计算属性的计算结果，只要依赖的其它属性值不变，即使多次使用计算属性，就不会重复计算,效率更高。

 而vue不会缓存函数的执行结果，所以如果多次调用函数，会导致重复计算！

watch是属性监听器，一般用来监听属性的变化（也可以用来监听计算属性函数），并做一些逻辑。

### watch：
```html
<template>
  <div>
    <p>{{ this.number }}</p>
  </div>
</template>
 
<script>
export default {
  name: 'test1',
  data () {
    return {
      number: 1
    }
  },
  created () {
    setTimeout(() => {
      this.number = 100
    }, 2000)
  },
  watch: {
    number (newVal, oldVal) {
      console.log('number has changed: ', newVal)
    }
  }
}
</script>
```
watch是专门监听一个变量的变化，只要变量值发生变化，就自动触发的函数。

### 总结：
computed和watch的使用场景并不一样，

- computed的话是<span style="color:red">通过几个数据的变化，来影响一个数据</span>，computed是在HTML DOM加载后马上执行的，如赋值；

- 而watch的话，<span style="color:red">是可以一个数据的变化，去影响多个数据</span>。watch用于观察Vue实例上的数据变动。对应一个对象，键是观察表达式，值是对应回调。

- 另外，methods则必须要有<span style="color:red">一定的触发条件才能执行</span>，如点击事件

- 如果更关心执行过程时，<span style="color:red">不关心结果时，首选函数！</span>

- 如果更关心计算结果，<span style="color:red">不关心过程时，首选计算属性。</span>