---
title: React中的数据绑定
date: 2019-11-27 22:33:48
tags: React
---
React中没有指令的概念，故v-model、ngModel都没有；使用“受控组件”来实现双向数据绑定的功能

### 条件渲染
使用内容绑定一个函数（该函数中根据条件返回不同的JSX片段）
```html
printWelcome(){
if(...) return (<div>...</div>)
else  return (<p>...</p>)
}
<div>{ this.printWelcome() }</div>
```
当然你在项目中也可以这样写：
```html
import React from 'react'
export default class App extends React.Component{
  constructor(){
    super()
    this.state={
      todoList:["eat","play","dance","xixi"],
    }
  }
  render(){
    return(
      <div>
        <h1>代办事项列表</h1>
        {
          (()=>{
            if(this.state.todoList.length===0){
            //可以采用匿名函数自调用的方式，返回一个JSX片段
              return <p>当前没有代办事项</p>
            }
          })()
        }
      </div>
    )
  }
}
```
### 循环渲染
前提：React中的内容绑定如果绑定了数组的话，会自动把每个元素提取出来作为当前父节点的子节点——利用此特性，只需要把数组转换为JSX数组直接做内容绑定即可
```html
import React from 'react'
export default class App extends React.Component{
  constructor(){
    super()
    this.state={
      todoList:["eat","play","dance","xixi"],
    }
  }
  render(){
    return(
      <div>
        <h1>代办事项列表</h1>
       <ul>
          {
            this.state.todoList.map((todo,i)=>{
            //遍历数组实现循环渲染
              return(
                <li key={i}>
                  {i+1}-{todo}
                </li>
              )
            })
          }
        </ul>
      </div>
    )
  }
}
```
### 双向数据绑定
提示：React中没有指令的概念，故v-model、ngModel都没有；使用“受控组件”来实现双向数据绑定的功能
```javascript
//方向1：Model=>View，状态数据绑定到表单输入元素的value
constructor(){
    super()
    this.state = { userInput: '' }
}
<input  value={ this.state.userInput }/>

//方向2：View=>Model，表单输入事件反过来要影响状态
doChange = ( e )=>{
    this.setState({ userInput: e.target.value })
}
<input  onChange={this.doChange}  value={ this.state.userInput }/>
```
### todoList
下面我们将条件渲染、循环渲染、和双向数据绑定联合起来实现代办事项列表
```javascript
import React from 'react';

export default class App extends React.Component{
  constructor(){
    super()
    this.state={
      todoList:["eat","play","dance","xixi"],
      userInput:''
    }
  }
  change=(e)=>{
    this.setState({
      userInput:e.target.value
    })
  }
  add=()=>{
   let todoList=this.state.todoList
   todoList.push(this.state.userInput)
   this.setState({
    todoList:todoList,
     userInput:''
   })
   
  }
  delete(i){
    let todoList=this.state.todoList
    todoList.splice(i,1)
    this.setState({todoList:todoList})
  }
  render(){
    return(
      <div>
        <h1>代办事项列表</h1>
        <hr/>
        <input onChange={this.change} value={this.state.userInput} />
        <button onClick={this.add}>添加事项</button>
        {  //条件渲染
          (()=>{
            if(this.state.todoList.length===0){
              return <p>当前没有代办事项</p>
            }
          })()
        }
        <ul>
          { //循环渲染
            this.state.todoList.map((todo,i)=>{
              return(
                <li key={i}>
                  {i+1}-{todo}
                <!-- 此时delete方法要传递参数，因此立即调用该函数-->
                  <button onClick={()=>{this.delete(i)}}>删除</button>
                </li>
              )
            })
          }
        </ul>
      </div>
    )
  }
}
```