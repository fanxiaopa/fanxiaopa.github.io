---
title: tapable实现
date: 2021-11-22 14:05:38
tags: webpack
---

tapable有以下9种钩子

- SyncHook

- SyncBailHook

- SyncLoopHook

- SyncWaterfallHook

- AsyncParallelHook

- AsyncParallelBailHook

- AsyncSeriesHook

- AsyncSeriesBailHook

- AsyncSeriesWaterfallHook

  

### SyncHook

#### 用法：

```javascript
const { SyncHook } = require('tapable');
const hook = new SyncHook(['arg1', 'arg2', 'arg3']);
// 注册两个监听函数，tap方法的第一个参数没有实际意义，只是用来做标识
hook.tap('yxfan', (say) => {
    console.log(say, ': yxfan');
});
hook.tap('cheney', (say) => {
    console.log(say, ' : cheney')
});
// 触发
hook.call('hello')
// 打印
// hello : yxfan
// hello : cheney
```

#### 实现：

```javascript
class SyncHook {
    constructor(args) { // 
        this.tasks = [];
    }
    // 订阅
    tap(name, task) {
        this.tasks.push(task);
    }
    // 发布
    call(...args) {
        this.tasks.forEach(task => {
            task(...args);
        });
    }
}
module.exports = SyncHook;
```

### SyncBailHook

SyncBailHook同步熔断保险钩子,即return一个非undefined的值，则不再继续执行后面的监听函数

#### 用法：

```javascript
const { SyncBailHook } = require('tapable');
class Test {
    hooks = {
        arch: new SyncBailHook(['arg1'])
    }
}
const test = new Test();
test.hooks.arch.tap('yxfan', (say) => {
    console.log(say, ': yxfan');
    return 'yxfan 今天调休啦'; // 不会继续执行下面的事件
    // return undefined 则继续往下执行
})
test.hooks.arch.tap('cheney', (say) => {
    console.log(say, ': cheney');
})
test.hooks.arch.call('hello')
// 打印：hello : yxfan
```

#### 实现：

```javascript
class SyncBailHook {
    constructor(args) {
        this.tasks = [];
    }
    // 订阅
    tap(name, task) {
        this.tasks.push(task);
    }
    // 发布
    call(...args) {
        for(let i = 0; i < this.tasks.length; i++) {
            const result = this.tasks[i](...args);
            if (result) return
        }
    }
}
module.exports = SyncBailHook;
```

### SyncWaterfallHook

SyncWaterfallHook 瀑布钩子,上一个监听函数的返回值，作为下一个监听函数的参数，像瀑布一样传递参数

#### 用法：

```javascript
const { SyncWaterfallHook } = require('tapable');
class Test {
    hooks = {
        arch: new SyncWaterfallHook(['arg1'])
    }
}
const test = new Test();
test.hooks.arch.tap('yxfan', (say) => {
    console.log(say, ': yxfan');
    return 'yxfan ok'
})
test.hooks.arch.tap('cheney', (data) => {
    console.log(data);
    return 'cheney ok'

})
test.hooks.arch.tap('xiaopa', (data) => {
    console.log(data);
})
test.hooks.arch.call('hello')
// 打印如下：
// hello : yxfan
// yxfan ok
// cheney ok
```

#### 实现：

```javascript
class SyncWaterfallHook {
    constructor(...args) {
        this.tasks = [];
    }
    tap(name, task) {
        this.tasks.push(task);
    }
    call(...args) {
        const [first, ...other] = this.tasks;
        let result = first(...args);
        other.reduce((pre, next) => {
            let ret = next(pre);
            if (ret) return ret;
            return result;
        }, result);
    }
}
module.exports = SyncWaterfallHook;
```

### SyncLoopHook

SyncLoopHook 只要监听函数不返回undefined，就循环执行该监听函数

```javascript
const { SyncLoopHook } = require('tapable');
class Test {
    hooks = {
        arch: new SyncLoopHook(['arg1', 'arg2'])
    }
}
let index = 0;
const test = new Test();
test.hooks.arch.tap('yxfan', (say) => {
    console.log(say, ': yxfan', index);    
    return ++index === 3 ? undefined : 'wait a moment'
})
test.hooks.arch.tap('cheney', (say) => {
    console.log(say, ': cheney');
})
test.hooks.arch.call('hello')
// 打印如下：
// hello : yxfan 0
// hello : yxfan 1
// hello : yxfan 2
// hello : cheney
```

#### 实现：

```javascript
class SyncLoopHook {
    constructor() {
        this.tasks = [];
    }
    tap(name, task) {
        this.tasks.push(task);
    }
    call(...args) {
        this.tasks.forEach(task => {
            let result;
            do {
                result = task(...args)
            } while(result != undefined)
        })
    }
}
module.exports = SyncLoopHook;
```

### AsyncParallelHook

AsyncParallelHook 异步并行钩子

tapable库中，有三种事件注册方式： 

- tap -同步注册  
- tapAsync-异步注册，接受第二个参数cb ，cb方法调用表示改事件执行完了 
- tapPromise-接受一个参数Promise

有三种事件发布方式：

- call
- callAsync
- promise

```javascript
/**
 * tapable库中，有三种注册方式 tap -同步注册  tapAsync-异步注册，接受第二个参数cb  tapPromise-一个参数Promise
 */
const { AsyncParallelHook } = require('tapable');
class Hook {
    constructor() {
        this.hooks = {
            arch: new AsyncParallelHook(['arg1'])
        }
    }
    // 注册监听函数
    subscribe() {
        this.hooks.arch.tapAsync('primary', (name, cb) => {
            setTimeout(() => {
                console.log('初级前端：', name);
                cb()
            }, 1000)
        })
        this.hooks.arch.tapAsync('intermediate', (name, cb) => {
            setTimeout(() => {
                console.log('中级前端：', name);
                cb()
            }, 1000)
        })
        this.hooks.arch.tapAsync('senior', (name, cb) => {
            console.log('高级前端：', name);
            cb()
        })
    }
    // 执行监听函数
    publish() {
        this.hooks.arch.callAsync('yxfan', () => {
            console.log('封神啦！');
        })
    }
}

const hook = new Hook;
hook.subscribe();
hook.publish();
```

因为是异步并行钩子，所以一秒后一次性打印：“初级前端： yxfan”，“中级前端： yxfan”，“高级前端： yxfan”，异步都执行完了，最后才打印“封神啦！”

用tapPromise比较简单，注意用`tapPromise`注册监听函数，触发时要用`promise`

```javascript
class HookPromise {
    constructor() {
        this.hooks = {
            arch: new AsyncParallelHook(['arg1'])
        }
    }
    // 注册监听函数
    subscribe() {
        this.hooks.arch.tapPromise('primary', (name) => {
            return new Promise(resolve => {
                setTimeout(() => {
                    console.log('初级前端：', name);
                    resolve()
                }, 1000)
            })
            
        })
        this.hooks.arch.tapPromise('intermediate', (name) => {
            return new Promise(resolve => {
                setTimeout(() => {
                    console.log('中级前端：', name);
                    resolve()
                }, 1000)
            })
        })
        this.hooks.arch.tapPromise('senior', (name) => {
            return new Promise(resolve => {
                setTimeout(() => {
                    console.log('高级前端：', name);
                    resolve()
                }, 1000)
            })
        })
    }
    // 执行监听函数
    publish() {
        this.hooks.arch.promise('yxfan').then(() => {
            console.log('封神啦！');
        })
    }
}
const hook = new HookPromise();
hook.subscribe();
hook.publish();
```

#### 实现：

```javascript
class AsyncParallelHook {
    constructor() {
        this.tasks = [];
    }
    tapAsync(name, task) {
        this.tasks.push(task);
    }
    callAsync(...args) {
        const endCb = args.pop();
        let index = 0;
        const done = () => {
            index++;
            if (index === this.tasks.length) {
                endCb();
            }
        }
        this.tasks.forEach(task => {
            task(...args, done)
        });
    }
    tapPromise(name, task) {
        this.tasks.push(task);
    }
    promise(...args) {
        let promiseTasks = this.tasks.map(task => task(...args));
        return Promise.all(promiseTasks);
    }
}
module.exports = AsyncParallelHook;
```

### AsyncSeriesHook

AsyncSeriesHook 异步串行钩子

```javascript
const { AsyncSeriesHook } = require('tapable');
class Hook {
    constructor() {
        this.hooks = {
            arch: new AsyncSeriesHook(['arg1'])
        }
    }
    // 注册监听函数
    subscribe() {
        this.hooks.arch.tapAsync('primary', (name, cb) => {
            setTimeout(() => {
                console.log('初级前端：', name);
                cb()
            }, 1000)
        })
        this.hooks.arch.tapAsync('intermediate', (name, cb) => {
            setTimeout(() => {
                console.log('中级前端：', name);
                cb()
            }, 1000)
        })
        this.hooks.arch.tapAsync('senior', (name, cb) => {
            setTimeout(() => {
                console.log('高级前端：', name);
                cb()
            }, 1000)
        })
    }
    // 执行监听函数
    publish() {
        this.hooks.arch.callAsync('yxfan', () => {
            console.log('封神啦！');
        })
    }
}
const hook = new Hook();
hook.subscribe();
hook.publish();
```

因为是串行钩子，所以依次间隔1秒打印：“初级前端： yxfan”，“中级前端： yxfan”，“高级前端： yxfan”，异步都执行完了，最后才打印“封神啦！”

同理用tapPromise的情况

```javascript
class HookPromise {
    constructor() {
        this.hooks = {
            arch: new AsyncSeriesHook(['arg1'])
        }
    }
    // 注册监听函数
    subscribe() {
        this.hooks.arch.tapPromise('primary', (name) => {
            return new Promise((resolve) => {
                setTimeout(() => {
                    console.log('初级前端：', name);
                    resolve()
                }, 1000)
            })
        })
        this.hooks.arch.tapPromise('intermediate', (name) => {
            return new Promise((resolve) => {
                setTimeout(() => {
                    console.log('中级前端：', name);
                    resolve()
                }, 1000)
            })
        })
        this.hooks.arch.tapPromise('senior', (name) => {
            return new Promise((resolve) => {
                setTimeout(() => {
                    console.log('高级前端：', name);
                    resolve()
                }, 1000)
            })
        })
    }
    // 执行监听函数
    publish() {
        this.hooks.arch.promise('yxfan')
            .then(() => {
                console.log('封神啦！');
            })
    }
}
const hookPromise = new HookPromise();
hookPromise.subscribe();
hookPromise.publish();
```

#### 实现：

```javascript
class AsyncSeriesHook {
    constructor() {
        this.tasks = [];
    }
    tapAsync(name, task) {
        this.tasks.push(task);
    }
    callAsync(...args) {
        const endCb = args.pop();
        let index = 0;
        // 递归
        let next = () => {
            if (index === this.tasks.length) {
                return endCb();
            }
            let task = this.tasks[index];
            index++;
            task(...args, next);
        }
        next();
    }

    tapPromise(name, task) {
        this.tasks.push(task);
    }
    promise(...args) {
        const [first, ...others] = this.tasks;
        return others.reduce((pre, next) => {
            return pre.then(() => next(...args));
        }, first(...args));
    }
}
module.exports = AsyncSeriesHook;
```

### AsyncSeriesWaterfallHook

AsyncSeriesWaterfallHook 异步串行瀑布钩子，像瀑布一样的传递参数

注意：cb方法接受两个参数，第一个参数是错误信息，为null时，数据流向下，不为null则表示出错了，后续事件都不执行，直接执行callAsync的回调。第二个参数就要传递的数据

```javascript
const { AsyncSeriesWaterfallHook } = require('../real-tapable');

class Hook {
    constructor() {
        this.hooks = {
            arch: new AsyncSeriesWaterfallHook(['arg1'])
        }
    }
    // 注册监听函数
    subscribe() {
        this.hooks.arch.tapAsync('primary', (name, cb) => {
            setTimeout(() => {
                console.log('初级前端：', name);
                cb(null, 'cheney')
            }, 1000)
        })
        this.hooks.arch.tapAsync('intermediate', (data, cb) => {
            setTimeout(() => {
                console.log('中级前端：', data);
                cb('error', 'xiaopa') // 出错啦，senior事件不会执行
            }, 1000)
        })
        this.hooks.arch.tapAsync('senior', (data, cb) => {
            setTimeout(() => {
                console.log('高级前端：', data);
                cb(null)
            }, 1000)
        })
    }
    // 执行监听函数
    publish() {
        this.hooks.arch.callAsync('yxfan', () => {
            console.log('封神啦!');
        })
    }
}
const hook = new Hook();
hook.subscribe();
hook.publish();
```

依次间隔1秒打印："初级前端： yxfan"，“中级前端： yxfan”，“封神啦!”

用`tapPromise`注册事件：
```javascript
class HookPromise {
    constructor() {
        this.hooks = {
            arch: new AsyncSeriesWaterfallHook(['arg1'])
        }
    }
    // 注册监听函数
    subscribe() {
        this.hooks.arch.tapPromise('primary', (name) => {
            return new Promise((resolve) => {
                setTimeout(() => {
                    console.log('初级前端：', name);
                    resolve('cheney')
                }, 1000)
            })
        })
        this.hooks.arch.tapPromise('intermediate', (data) => {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    console.log('中级前端：', data);
                    // resolve('xiaopa')
                  	reject()
                }, 1000)
            })
        })
        this.hooks.arch.tapPromise('senior', (data) => {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    console.log('高级前端：', data);
                    resolve()
                }, 1000)
            })
        })
    }
    // 执行监听函数
    publish() {
        this.hooks.arch.promise('yxfan')
            .then(() => {
                console.log('封神啦！');
            })
    }
}
const hookPromise = new HookPromise();
hookPromise.subscribe();
hookPromise.publish();
```

#### 实现：

```javascript
class AsyncSeriesWaterfallHook {
    constructor() {
        this.tasks = [];
    }
    tapAsync(name, task) {
        this.tasks.push(task);
    }
    callAsync(...args) {
        const endCb = args.pop();
        let index = 0;
        // 递归
        let next = (err, data) => {
            let task = this.tasks[index];
            if (!task || err) return endCb();
            if (index == 0) {
                task(...args, next);
            } else {
                task(data, next);
            }
            index++;
        }
        next();
    }
    tapPromise(name, task) {
        this.tasks.push(task);
    }
    promise(...args) {
        const [first, ...others] = this.tasks;
        return others.reduce((pre, next) => {
            return pre.then((data) => next(data));
        }, first(...args));
    }
}
module.exports = AsyncSeriesWaterfallHook;
```

[代码地址](https://gitee.com/fanxiaopa/tabaple)

