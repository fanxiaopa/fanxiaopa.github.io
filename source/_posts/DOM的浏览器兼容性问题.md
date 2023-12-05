---
title: DOM与IE在事件处理上的区别
date: 2020-02-10 11:33:11
tags: DOM
---
### 事件模型
#### DOM：3个阶段
- 1.由外向内：捕获
- 2.触发
- 3.由内向外：冒泡

#### IE8：2两个阶段
- 1.触发
- 2.由内向外：冒泡

如图：DOM与IE8事件模型的比较
<img src='/images/DOM&IE8.png'>

### 事件绑定
在DOM中：elem.addEventListener('click',function(){},false)
注意：第三个参数 capture：是否在捕获阶段就提前触发，默认为false，一般省略；

在IE8中：elem.attachEvent('onclick',function(){})

### 事件移除
在DOM中：ele.removeEventListener('click',function(){},false)
在IE8中：ele.detachEvent('onclick',function(){})

### 获得事件对象的方法
在DOM中：ele.addEventListener('click',function(e){e->event})
在IE8中：不会自动传入时间对象e，事件对象event始终保持在一个全局变量window.event中
elem.attachEvent('onclick',function(){ var e = window.event })

### 获得目标元素
在DOM中：e.target
在IE8中：window.event.srcElement
兼容所有浏览器的写法：
```javascript
var target = e.target || window.event.srcElement;
```

### 阻止冒泡
在DOM中：e.stopPropagation()
在IE8中：window.event.cancelBubble = true
兼容所有浏览器的写法：
```javascript
if(window.event.cancelBubble != undefined) {
    window.event.cancelBubble = true //IE8
}else {
    e.stopProoagation() 
}
```

### 阻止默认行为
在DOM中：e.preventDefault()
在IE8中：事件处理函数中：window.event.returnValue = false
兼容所有浏览器的写法：
```javascript
if(typeof e.preventDefault !== 'function') {
   window.event.returnValue = false //IE8
}else {
    e.preventDefault()
}
```
### 案例
定义一个函数，可以支持所有浏览器中的处理函数绑定
```javascript
function bindEvent(elem, eventName, handler) {
    if(typeof elem.attachEvent!=='function'){
         //DOM
        elem.addEventListener(eventName, handler)
    }else{ //IE8
         elem.attachEvent('on'+eventName, function(){
             //this -> elem
             var e = window.event
             e.target = e.srcElement;
             handler.call(this,e)
         })
    }
}
bindEvent(btn, 'click', function(e){
    //this 当前事件冒泡到的父元素
    var target = e.target //目标元素
})

```