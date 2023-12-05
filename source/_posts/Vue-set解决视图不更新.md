---
title: Vue.$set解决视图不更新
date: 2020-01-08 20:10:35
tags: vue
---
### 问题
实际开发时，遇到过这种情况，我们给data中的一个对象添加新属性时，视图层却不更新
```javascript
<div id="app">
    <div v-for="(item, i) of user" :key="i">
        姓名：{{item.name}}
        年龄：{{item.age}}
        <button @click="change(i)">爱好</button>
        <div v-if="item.love">
            爱好：吃饭
        </div>
    </div>
</div>
<script>
    var vue = new Vue({
        el: '#app',
        data () {
            return {
                user:[
                    { name:'fan', age:20 },
                    { name:'lijing', age:30 },
                    { name:'yucheng', age:40 },
                    { name:'rongxin', age:50 }
                ]
            }
        },
        methods: {
            change(i){
                this.user.forEach((value, index)=>{
                    if(index === i){
                        //相当于添加一个新属性
                        this.user[i].love = !value.love
                    }
                })
            }
        }
    })
</script>
```
最初item.love是undefined，点击事件触发后，给item.love赋值为true，相当于给对象新添了一个love属性，但是视图层却没更新

由于受JavaScript的限制，vue.js不能监听对象属性的添加和删除，因为在vue组件初始化的过程中，会调用getter和setter方法，所以该属性必须是存在在data中，视图层才会响应该数据的变化

### 解决
使用vue.set(obj, key, value)方法
```javascript
 methods: {
            change(i){
                this.user.forEach((value,index)=>{
                    if(index === i){
                        this.$set(this.user[i], 'love', !value.love)
                    }else{                        
                        this.$set(this.user[index], 'love', value.love)
                    }
                })
            }
        }
```