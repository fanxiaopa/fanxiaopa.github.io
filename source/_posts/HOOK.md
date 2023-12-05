---
title: Hook
date: 2020-09-2 21:08:46
tags: React
---
### Hook简介

了解react的同鞋都知道，react组件有两种编写方式，即类组件和函数式组件。
函数式组件代码看起来更简洁，但是没有办法维护自身状态,
因此Hook的出现可以让我们在不编写class的情况下使用state以及其他的react特性。

### State Hook

useState方法必须要从react中引入，该方法接收一个参数（state的初始值），会返回一个数组，数组中第一个元素是当前状态（state），第二个元素是更新状态的函数

```javascript
import React, { useState } from 'react';
function Example() {
  // 声明一个叫 "count" 的 state 变量  
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
我们尝试这把这段代码转化成class写法：

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
当我们要声明多个state时：
```javascript
const [value, setValue] = useState({
    valA: 0,
    valB: 1
})
```

### Effect Hook

> useEffect Hook看做`componentDidMount`，`componentDidUpdate`和`componentWillUnmount`这三个函数的组合。

```javascript
import React, { useState, useEffect } from 'react';
function Example() {
  const [count, setCount] = useState(0);
  useEffect(() => {    document.title = `You clicked ${count} times`;  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

等价的class写法

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {    
    document.title = `You clicked ${this.state.count} times`;  
  }  
  componentDidUpdate() {    
    document.title = `You clicked ${this.state.count} times`;  
  }
  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
> <strong>useEffect 做了什么？</strong>
通过使用这个 Hook，你可以告诉 React 组件需要在渲染后执行某些操作。React 会保存你传递的函数（我们将它称之为 “effect”），并且在执行 DOM 更新之后调用它。在这个 effect 中，我们设置了 document 的 title 属性，不过我们也可以执行数据获取或调用其他命令式的 API。

> <strong>为什么在组件内部调用 useEffect？</strong>
将 useEffect 放在组件内部让我们可以在 effect 中直接访问 count state 变量（或其他 props）。我们不需要特殊的 API 来读取它 —— 它已经保存在函数作用域中。Hook 使用了 JavaScript 的闭包机制，而不用在 JavaScript 已经提供了解决方案的情况下，还引入特定的 React API。

> <strong>useEffect 会在每次渲染后都执行吗？</strong>
是的，默认情况下，它在第一次渲染之后和每次更新之后都会执行。（我们稍后会谈到如何控制它。）你可能会更容易接受 effect 发生在“渲染之后”这种概念，不用再去考虑“挂载”还是“更新”。React 保证了每次运行 effect 的同时，DOM 都已经更新完毕。

#### 需清楚的effect
useEffect可以替代生命周期的 `componentDidMount`，`componentDidUpdate`和`componentWillUnmount`
常常会遇到这种情形：在 componentDidMount 中监听resize事件，那么在组件卸载时，必须移除resize事件
```javascript
import React, {useState, useEffect} from 'react';
function UseEffect() {
    const [count, setCount] = useState(0);
    const resize = () => {
        console.log('窗口改变啦');
    }
    useEffect(() => {
        window.addEventListener('resize', resize)
        return () => {
            window.removeEventListener('resize', resize);
        }
    })
    return (
        <h2>
            useeffect测试
        </h2>
    )
}
export default UseEffect
```
等价的class组件
```javascript
resize = () => {
  console.log('窗口改变啦');
}
componentDidMount(){
     window.addEventListener('resize',this.resize)
}
componentWillUnmount(){
     window.removeEventListener('resize', resize);
}
```
这里useEffect方法中的参数（箭头函数）里返回了一个函数，这就是effect的清除机制
> <strong>为什么要在 effect 中返回一个函数？</strong> 
这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。

> <strong>React 何时清除 effect？</strong>
 React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 React 会在执行当前 effect 之前对上一个 effect 进行清除。我们稍后将讨论为什么这将助于避免 bug以及如何在遇到性能问题时跳过此行为。

 #### 跳过Effect进行性能优化
因为useEffect会在每次更新渲染后执行，着就会导致一个性能问题。在 class 组件中，我们可以通过在 componentDidUpdate 中添加对 prevProps 或 prevState 的比较逻辑解决：
```javascript
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```
只有在state中的count改变时，才修改document的title
在hook中实现：
```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```
上面这个示例中，我们传入 [count] 作为第二个参数。这个参数是什么作用呢？如果 count 的值是 5，而且我们的组件重渲染的时候 count 还是等于 5，React 将对前一次渲染的 [5] 和后一次渲染的 [5] 进行比较。因为数组中的所有元素都是相等的(5 === 5)，React 会跳过这个 effect，这就实现了性能的优化。

### useMemo & memo

`useMemo()`和`memo`方法总是让我有点分不清楚
- `memo()`指的是一个组件是否重复渲染，可以优化性能
- `useMemo()`指的是一段函数逻辑是否重复执行

#### memo
父组件：
```javascript
function Father() {
    const [count, setCount] = useState(0);
    function add(){
        setCount(count + 1)
    }
    return (
        <>
            父组件：计数：{count}
            <button onClick={add}>加一</button> <br/>
            <Son />
        </>
    )
}
export default Father
```
子组件：
```javascript
import React, { memo, useMemo } from 'react';
const Son =  memo((prop) => {
    console.log('子组件渲染了');
    return (
        <>  
            这是子组件
        </>
    )
})
export default Son
```
效果：点击加一按钮，控制台没有输出，因为子组件没有重复渲染
> 分析：正常情况下，父组件中的state改变了，会引起re-render，不管子组件有没有依赖父组件的状态，都会被re-render,
  但是这里用了memo()方法,子组件不依赖于父组件的状态，即使父组件状态更新了，子组件也能保持不变，不用重复渲染，因此节约性能

#### useMemo
父组件：
```javascript
function Father() {
    const [count, setCount] = useState(0);
    function add(){
        setCount(count + 1)
    }
    return (
        <>
            父组件：计数：{count}
            <button onClick={add}>加一</button> <br/>
            <Son count={count}/>
        </>
    )
}
export default Father
```
子组件：
```javascript
import React, { memo, useMemo } from 'react';
const Son =  memo((prop) => {
	const [anotherCount, setAnotherCount] = useState(0);
    	console.log('子组件渲染了');

	const useMemoTest = useMemo(() => {
		console.log('useMemo方法执行了')
	}, [anotherCount])
	
    return (
        <>  
            这是子组件{useMemoTest()}
        </>
    )
})
export default Son
```
效果：点击加一按钮，控制台输出：'子组件渲染了'
> 分析：子组件依赖了父组件的count，所以父组件的count改变时，引起子组件的re-render;
	正常情况re-render会调用useMemoTest()方法，但是却没有打印信息；
	因为这里用了useMemo()方法，它只有在第二个参数[anotherCount]改变时，才执行useMemoTest()