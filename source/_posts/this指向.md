---
title: this指向
date: 2020-06-15 20:35:53
tags: js
---
this在开发中的重要性这里就不说了，咱们直接来讨论出现this的几种情形

- obj.fun() 
fun函数中的this指向调用者obj
-  fun()  或  (function(){ ... })()  或 setTimeout(function(){ ... }, ms) 
里面的this指向window
-  new Fun()  
this指向正在创建的对象
-  arr.forEach/map(() => {this})
这里面不管是普通函数还是箭头函数this都指向window
-  btn.onclick=function(){ this->btn }
      btn.addEventListener("click",function(){ this->btn })
      $btn.on("click",function(){ this->btn })
      $btn.click(function(){ this->btn })
- Array.prototype.sum=function(){this}
this->将来调用sum的.前的数组家子对象
- 只要new Vue中的this，一律指向当前的new Vue对象

这样直接看概念可能有点晦涩，这里上一个例子
```javascript
var name = 'window'

var person1 = {
  name: 'person1',
  show1: function () {
    console.log(this.name)
  },
  show2: () => console.log(this.name),
  show3: function () {
    return function () {
      console.log(this.name)
    }
  },
  show4: function () {
    // console.log(this)
    return () => console.log(this.name)
  }
}
var person2 = { name: 'person2' }

person1.show1() // person1
person1.show1.call(person2) //person2

person1.show2() // window
person1.show2.call(person2) // window  

person1.show3()() // window
person1.show3().call(person2) // person2
person1.show3.call(person2)() // window

person1.show4()() // person1
person1.show4().call(person2) // person1  
person1.show4.call(person2)() // person2
```
答案既注释
我们来分析一下：
person1.show1()和person1.show1.call(person2)这两个都不用细说，前者this指向函数调用者‘person1’；后者使用call改变了this指向，相当于是用person2调用了person1的show1方法，故this指向调用者person2；

person1.show2()这里是个箭头函数，注意箭头函数中的this是定义时就被确定的，指向执行上下文，即window；
而person1.show2.call(person2)即使调用了call来改变this指向，但是this在定义时就被确定了，因此call无效

person1.show3()()，先执行show3方法，会返回一个普通函数，全局再调用这个普通函数，因此this指向window；
person1.show3().call(person2)，先执行show3方法，返回一个普通函数，但是此时用了call，相当于是person2在调用这个普通函数，因此this指向person2
person1.show3.call(person2)()，此时相当于是person2在调用person1的show3方法，会返回一个普通函数，全局再调用它，因此this指向window

person1.show4()()，person1先调用show4方法，会返回一个箭头函数，注意箭头函数中的this是继承上下文而来的，而此时上下文的this指向person1，因此this指向person1；
person1.show4().call(person2) ，person1先调用show4方法，还是会返回一个箭头函数，即使用call也不能改变this指向，因为它在定义时就被确定了；还是指向上下文，即person1；
person1.show4.call(person2)()，用person2调用person1的show4方法，此时箭头函数的上下文是person2，因此this指向person2；

总而言之，言而总之记住一句话：
> <span style="color:red">箭头函数内部是没有this的，也就是说，箭头函数里面的this会继承自外部的this，判断箭头函数指向问题时，不要看被谁调用的，而是看在哪儿
定义的！！！</span>


