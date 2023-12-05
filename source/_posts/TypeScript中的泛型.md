---
title: TypeScript中的泛型
date: 2019-12-24 21:52:33
tags: TypeScript
---
### 什么是泛型
软件工程中，我们不仅要创建一致的定义良好的API，同时也要考虑可重用性。 组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。

在像C#和Java这样的语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。

通俗理解：泛型就是解决 类 接口 方法的复用性、以及对不特定数据类型的支持(类型校验)

我们用 T 来表示泛型，当然也可以用其他字母表示

### 泛型类
我们举一个例子，找数组中最小的元素，它可以是number，也可以是string
```javascript
class MinClas<T>{
    public list:T[]=[];
    add(value:T):void{
        this.list.push(value);
    }
    min():T{        
        var minNum=this.list[0];
        for(var i=0;i<this.list.length;i++){
            if(minNum>this.list[i]){
                minNum=this.list[i];
            }
        }
        return minNum;
    }
}
/*实例化类 并且制定了类的T代表的类型是number*/
var m1=new MinClas<number>();   
m1.add(11);
m1.add(3);
m1.add(2);
alert(m1.min()) // 2

/*实例化类 并且制定了类的T代表的类型是string*/
var m2=new MinClas<string>();   
m2.add('c');
m2.add('a');
m2.add('v');
alert(m2.min()) // 'a'
```
注意：T 表示泛型，它具体指什么类型是调用这个方法的时候决定的

### 泛型接口
泛型解决、类、接口、方法的复用性
而接口是一种规范的定义，它定义了行为和动作的规范（约束）
```javascript
//定义一个方法接口
interface ConfigFn{
   <T>(value:T):T;
}
var getData:ConfigFn=function<T>(value:T):T{
  return value;
}
//此时给出了泛型的具体类型
getData<string>('张三');
getData<string>(1243);  //错误
```
当然也可以这样写，两者等效
```javascript
interface ConfigFn<T>{
    (value:T):T;
}
function getData<T>(value:T):T{
     return value;
}
//此时给出了泛型的具体类型
var myGetData:ConfigFn<string>=getData;     
myGetData('20');  /*正确*/
myGetData(20)  //错误
```
### 泛型接口和泛型类混合
注意：要实现泛型接口 这个类也应该是一个泛型类
定义一个操作数据库的库  支持 Mysql Mssql  MongoDb：
```javascript
interface DBI<T>{
    add(info:T):boolean;
    update(info:T,id:number):void;
}
//定义一个操作mysql数据库的类       
//注意：要实现泛型接口 这个类也应该是一个泛型类
class MysqlDb<T> implements DBI<T>{
    constructor(){
        console.log('数据库建立连接');
    }
    //必须实现接口的方法
    add(info: T): boolean {
        console.log(info);
        return true;
    }
    //必须实现接口的方法    
    update(info: T, id: number): void {
        throw new Error("Method not implemented.");
    }
}
//操作用户表   定义一个User类和数据表做映射
class User{
    username:string | undefined;
    password:string | undefined;
}
var u=new User();
u.username='张三111';
u.password='123456';

var oMysql=new MysqlDb<User>(); //类作为参数来约束数据传入的类型 
oMysql.add(u);//User {username: "张三111", password: "123456"}
```