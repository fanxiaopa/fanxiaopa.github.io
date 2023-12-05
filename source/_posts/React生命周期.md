---
title: React不常用的生命周期
date: 2020-12-16 21:39:23
tags: React
index_img: /images/lifeCycle.png
---
React16.3之后的生命周期：
<img style="margin:0 auto;" src="/images/lifeCycle.png" />
几个不常用的生命周期：

#### getDerivedStateFromProps(props, state)
> 使用场景：在初始化组件数据时，我们想要将组件接收的参数props中的数据添加到组件内部中的state中去， 期望组件响应 props 的变化。

执行过程：该回调会在render之前执行，返回一个对象来更新state，或者返回null，什么也不进行更新。

栗子：将props中的数据添加到state中去
```javascript
state = {
    c: 1,
    t: 2
}

static getDerivedStateFromProps(props, state) {
    console.log('变化了111111111', props, state);
    return {
        c : props.count
    }
    // return null 不更新任何内容
}
```
#### shouldComponentUpdate(nextProps, nextState)
> 使用场景：直接决定 React 的组件是否进行更新

无论是与当前的 props 还是与当前的 state 进行比较，最终总是会得出一个 true 或者是 false，来决定组件是否更新(render)。

栗子：
```javascript
shouldComponentUpdate(nextProps, nextState) {
    if(....) {
        return false; // 不render
    } else {
        return true // 要render
    }
}
```

#### getSnapshotBeforeUpdate(prevProps, prevState)
> 使用场景: 这个方法能够使得组件可以在可能更改之前从 DOM 捕获一些信息，比如滚动的位置等等。

注意，这个方法的返回值会作为第三个参数，传给`componentDidUpdate(nextPros, nextState, snapshot)`;
该方法必须返回一个值或者是返回null

栗子：
```javascript
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 捕获滚动的位置，以便后面进行滚动 注意返回的值
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // 如果有 snapshot 会进行滚动的调整，这样子就不会立即将之前的内容直接弹上去
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```
