---
title: typescript数据类型
date: 2019-12-21 14:13:35
tags: TypeScript
---
typescript中为了使编写的代码更规范，更有利于维护，增加了类型校验，在typescript中主要给我们提供了以下数据类型
- 布尔类型（boolean）
- 数字类型（number）
- 字符串类型(string)
- 数组类型（array）
- 元组类型（tuple）
- 枚举类型（enum）
- 任意类型（any）
- null 和 undefined
- void类型
- never类型

### 布尔类型（boolean）
```javascript
var flag:boolean=true;
// flag=123;  //错误
flag=false;  //正确
console.log(flag);
```
### 数字类型（number）
```javascript
var num:number=123;
num=456;
console.log(num);  //正确
num='str';    //错误
```
### 字符串类型(string)
```javascript
var str:string='this is ts';
str='haha';  //正确
str=true;  //错误
```
### 数组类型(array)
ts中定义数组有两种方式

```javascript
// 1.第一种定义数组的方式
var arr:number[]=[11,22,33];
console.log(arr);

// 2.第二种定义数组的方式
var arr:Array<number>=[11,22,33];
console.log(arr)
```
### 元组类型(tuple)  属于数组的一种
元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为string和number类型的元组。

```javascript
var arr:Array<number>=[11,22,33];
 console.log(arr)
//元祖类型
let arr:[number,string];
arr = [123,'this is ts']; //正确
arr = ['this is ts',123]; //错误
```
当访问一个越界的元素，会使用联合类型替代：
```javascript
//x[3] 不存在，不知道它的类型
x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
x[6] = true; // Error, 布尔不是(string | number)类型
```
### 枚举类型(enum)
在程序中用自然语言中有相应含义的单词来代表某一状态，则程序就很容易阅读和理解。

```javascript
enum Flag {success=1,error=2};
let s:Flag=Flag.success;
console.log(s);//1
enum Flag {success=1,error=2};
let f:Flag=Flag.error;
console.log(f);//2
```
如果标识符没有赋值 它的值就是下标：
```javascript
enum Color {blue,red,'orange'};
var c:Color=Color.red;
 console.log(c);   //1  
 /当有些有值时，有些没值时/
enum Color {blue,red=3,'orange'};
var c:Color=Color.red;
console.log(c);   //3
var c:Color=Color.orange;
console.log(c);   //4
var d:string=Color[4]
console.log(d)   // orange

enum Err {'undefined'=-1,'null'=-2,'success'=1};
var e:Err=Err.success;
console.log(e);//1
```
### 任意类型(any)
```javascript
var num:any=123;
num='str';
num=true;
console.log(num)//true
```
### null 和 undefined
```javascript
var num:number;  //必须要赋一个值
console.log(num)  //输出：undefined   报错

var num:undefined;
console.log(num)  //输出：undefined  //正确
```
```javascript
var num:number | undefined;
num=123;
console.log(num);//123
 
 //定义没有赋值就是undefined
var num:number | undefined;
console.log(num); //undefined
```
一个元素可能是 number类型 可能是null 可能是undefined
```javascript
var num:number | null | undefined;
num=1234;
console.log(num) //1234
```
### void
typescript中的void表示没有任何类型，一般用于定义方法的时候方法没有返回值。
```javascript
//typescript中 
function run():void{
    console.log('run')
}
run();
//错误写法
function run():undefined{
    console.log('run')
}
run();
```
### never类型
是其他类型 （包括 null 和 undefined）的子类型，代表从不会出现的值。
这意味着声明never的变量只能被never类型所赋值。
```javascript
var a:never;
//    a=123; //错误的写法
a=(()=>{
    throw new Error('错误');
})()
```