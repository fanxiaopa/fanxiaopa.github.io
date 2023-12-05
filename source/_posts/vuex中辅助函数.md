---
title: vuex中辅助函数
date: 2019-11-24 23:00:38
tags: vuex
---
<span>在正规的公司开发中很少用:</span>

this.$store.commit("xxx"),this.$store.dispatch("xxx"),this.$store.state.xxx,因为这样显得十分累赘，所以此时就推荐用辅助函数来替代。

以下三个例子都公用一个store.js
```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import axios from "axios"

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    uname:""
  },
  mutations: { //专门负责修改state中的变量
    setUname(state,uname){
      state.uname=uname;
    }
  },
  actions: { //专门负责发送异步ajax请求，从服务器端获取数据
    login(context,user){ //context代表整个vuex对象
      (async function(){
        var result=await axios.get("/users/signin",{
          params:user
        });
        context.commit("setUname",result.data.uname);
      })()
    }
  },
  modules: {
  }
})
```

### mapState

当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 mapState 辅助函数帮助我们生成<span style="color:red">计算属性</span>，让你少按几次键：

```html
<template>
    <h1 v-else>Welcome {{uname}}</h1>
</template>
<script>
import {mapState} from "vuex"
export default {
  computed:{
    // 使用对象展开运算符将此对象混入到外部对象中
    ...mapState(["uname"])
  }
}
</script>
```
### mapMutation
```html
<template>
    <button @click="logout">注销</button>
</template>
<script>
    import {mapMutations} from "vuex"

    export default {
    methods:{
        logout(){
            this.setUname("");
        },
        //和其他方法同级
        ...mapMutations(["setUname"])
        //setUname(uname){ this.$store.commit("setName",uanme) }
    }
</script>
```
### mapActions
```html
<template>
    <div class="home">
        用户名:<input v-model="uname"><br>
        密码:<input type="password" v-model="upwd"><br>
        <button @click="myLogin">登录</button>
  </div>
</template>
<script>
import {mapActions} from "vuex"

export default {
  name: 'home',
  data(){
    return {
      uname:"dingding",
      upwd:"123456"
    }
  },
  methods:{
    myLogin(){
      this.login({//给user
        uname:this.uname,
        upwd:this.upwd
      });
    },
    ...mapActions([//去vuex的actions中取出名为login的函数放到此地
      "login"//,"logout","registor"
    ])
    /**
    原方法:
     * login(user){ 
     *  this.$store.dispatch("login",user)
     * },
     */
  }
}
</script>
```


