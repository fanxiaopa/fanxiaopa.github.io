---
title: vue中的watch(深度监听)
date: 2019-11-25 23:04:05
tags: vue
---

让人容易忽视的是，watch中有两个属性，一个方法，分别是immediate，deep属性和handler方法

### handler方法和immediate属性
我们先来看一段代码
```html
<div>
    <p>FullName: {{fullName}}</p>
    <p>FirstName: <input type="text" v-model="firstName"></p>
</div>
new Vue({
  el: '#root',
  data: {
    firstName: 'Dawei',
    lastName: 'Lou',
    fullName: ''
  },
  watch: {
    firstName(newName, oldName) {
      this.fullName = newName + ' ' + this.lastName;
    }
  } 
})
```
上面的代码的效果是，当我们输入firstName后，wacth监听每次修改变化的新值，然后计算输出fullName。

这里 watch 的一个特点是，最初绑定的时候是不会执行的，要等到 firstName 改变时才执行监听计算。那我们想要一开始就让他最初绑定的时候就执行改怎么办呢？我们需要修改一下我们的 watch 写法，修改过后的 watch 代码如下：
```javascript
watch: {
  firstName: {
    handler(newName, oldName) {
      this.fullName = newName + ' ' + this.lastName;
    },
    // 代表在wacth里声明了firstName这个方法之后立即先去执行handler方法
    //而不是等待改变后
    immediate: true
  }
}
```
我们给 firstName 绑定了一个handler方法，之前我们写的 watch 方法其实默认写的就是这个handler，Vue.js会去处理这个逻辑，最终编译出来其实就是这个handler。

而immediate:true代表如果在 wacth 里声明了 firstName 之后，就会立即先去执行里面的handler方法，如果为 false就跟我们以前的效果一样，不会在绑定的时候就执行。

### deep属性
watch 里面还有一个属性 deep，默认值是 false，代表是否深度监听，比如我们 data 里有一个obj属性：
```html
<div>
      <p>obj.a: {{obj.a}}</p>
      <p>obj.a: <input type="text" v-model="obj.a"></p>
</div>
new Vue({
  el: '#root',
  data: {
    obj: {
      a: 123
    }
  },
  watch: {
    obj: {
      handler(newName, oldName) {
         console.log('obj.a changed');
      },
      immediate: true
    }
  } 
})
```
当我们改变input框中的属性时，控制台不会输出'obj.a changed'，因为此时监听的是obj这个对象（地址一直未改变），所以obj里面的属性的添加或删除并不会触发监听

当我们要满足这个需求时，deep属性就派上用场了！
```javascript
watch: {
  obj: {
    //当监听的是一个引用类型的值时，newValue和oldValue相同
    handler(newName, oldName) {
      console.log('obj.a changed');
    },
    immediate: true,
    deep: true
  }
} 
```
deep的意思就是深入观察，监听器会一层层的往下遍历，给对象的所有属性都加上这个监听器，但是这样性能开销就会非常大了，任何修改obj里面任何一个属性都会触发这个监听器里的 handler。

为了优化，我们可以使用字符串形式监听：
```javascript
watch: {
  'obj.a': {
    handler(newName, oldName) {
      console.log('obj.a changed');
    },
    immediate: true,
    //deep: true
  }
} 
```
这样Vue.js才会一层一层解析下去，直到遇到属性a，然后才给a设置监听函数。

注意：当监听的是一个引用类型的值时，newValue和oldValue相同

[戳此处👀别人家的博客](https://www.cnblogs.com/yesu/p/9546458.html)

