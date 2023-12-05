---
title: json-server
date: 2019-11-23 22:17:12
tags: json-server
---
[别人家的博客](https://blog.csdn.net/lhjuejiang/article/details/81475993)
### 什么是json-server
一个在前端本地运行，专门模拟后端接口地址的可以存储json数据的简易版server。

- 今后，都是前后端分离方式开发
- 前后端是完全独立的两个项目
- 前后端两个项目是并行开发，也就是说前端项目开发时，后端接口还没开发好呢

### json-server的安装
全局环境下安装
```javascript
npm install -g json-server
```

### json-server的使用
#### 1.定义db.json文件
基本格式如下
```javascript
{

    "接口名1": [
        {
            第一个数据对象的属性和值
		},
        ... ...
    ],
    "接口名2":[
        {
            数据对象的属性和值
         },
         ... ...
    ]
}

```
在db.json文件所在目录，运行json-server
```javascript
 json-server --watch --port 5050 db.json
```

#### 2.get请求
get请求专门用于查询数据直接输入：http://localhost:5050/index

当get请求携带参数时

- http://localhost:5050/details/1，直接只返回id为1的一个商品对象
- http://localhost:5050/details?id=1，却返回一个数组，包含查询到的结果对象

解决：定义一个路由翻译文件：<span style="color:red">routes.json</span>，他可以将服务器端的路由翻译成json-server的路由
```javascript
//routes.json文件中
 {
  "/details\\?id=:id": "/details/:id"
 }
```
但是服务器端不是用id检索的，都是用lid检索的，为了和服务器端的接口统一应该写成：
```javascript
 {
  "/details\\?lid=:id": "/details/:id"
 }
```

<span style="color:red">注意：启动json-server时加载自定义路由</span>

```javascript
json-server --watch --port 5050 --routes routes.json db.json
```

<span style="font-size:25px">模糊检索</span>

如果希望与服务器端kws参数保持一致，也可用自定义路由将title_like转化为kws:
```javascript
 {
    "/products\\?kws=:a":"/products?title_like=:a"
 }
```
#### 3.post请求
该请求专门用于插入数据
```javascript
$("#postBtn").click(function(){
    $.ajax({
        type: 'post',
        url: 'http://localhost:3003/fruits',
        data: {
            name:  $("#fruitName").val(),
            price: $("#fruitPrice").val()
        },
        success: function(data){
            console.log("post success")
        },
        error:function(){
            alert("post error")
        }
    })
})
 
```

#### 4.put请求
该请求专门用于修改数据
```javascript
$("#putBtn").click(function(){
    $.ajax({
        type: 'put',
        url: 'http://localhost:3003/fruits/'+ $("#putId").val(),
        data: {
            price: $("#putPrice").val()
        },
        success: function(data){
            console.log("put success")
        },
        error:function(){
            alert("put error")
        }
    })
})
 
```

#### 5.delete请求
专门用于删除数据
```javascript
$("#delOne").click(function(){
    $.ajax({
        type: 'delete',
        url: 'http://localhost:3003/fruits/'+ $("#delId").val(),
        success: function(data){
            console.log("del success")
        },
        error:function(){
            alert("del error")
        }
    })
})
 
```

### 强调
- 如果只是修改json文件的内容，不用重启服务器
- post,put,delete操作都是直接修改硬盘上的db.json文件，所以，修改前，一定做好备份！


