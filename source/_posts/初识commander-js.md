---
title: 初识commander.js
date: 2019-12-17 11:45:16
tags: commander
---
commander.js 是 express 框架的开发者号称无所不能的 TJ 大神写的工具包，用于快捷开发命令行工具，提高项目的开发效率。[commander.js 官网地址](https://github.com/tj/commander.js/blob/master/Readme_zh-CN.md)

### 初识命令行工具
下面是npm安装全局包的实例
```javascript
npm install commander -g
```
上述命令行代码由三部分组成：
- npm 命令行工具名称
- install 指令，后面的 commander 为该指令携带的参数
- -g 选项

### 定义命令行工具能够提供哪些选项
```javascript
var program = require('commander');
program.option(flag, desc, fn, defaultValue)
```
通过 option 方法我们可以定义命令行工具提供哪些选项，这些选项可以带参数，我们可以对参数进行自定义的再处理，我们还可以给参数赋值默认值。
- flag 必填。定义命令行参数能够提供哪些参数；
- desc 非必填。对参数进行描述；
- fn 非必填。是一个函数；
- defaultValue 非必填。默认值;
- <> 符号表示参数必填，不填会报错;
- [] 符号表示参数非必填。

```javascript
// 表示命令行提供 -i 参数，--install 参数是 -i 的别名，二者作用相同
// 如果指令用到了这个参数，那么 program.install 的值就变成 true，否则为 false
program.option('-i, --install')    
```
```javascript
// 第二个参数是对选项 -i 的描述信息
program.option('-i, --install <packagename>', 'install some package')  
```
```javascript
// 更改 program.install 的值，变成 handle 函数的返回值
function handle (arg) {     // 这里的 arg 等于选项后面跟着的参数 packagename 的值
    // 对 arg 处理然后返回处理后的值 afterarg，如将参数转换为数组并返回
    return arg.split(',');
}
// 第三个参数为函数，名称可以任意取值
program.option('-i, --install <packagename>', 'install some package', handle)
```
```javascript
// 如果第三个参数不是函数而是字符串，表示是 packagename 的默认值，packagename 如果不输入值，就赋值默认值
program.option('-i, --install <packagename>', 'install some package', 'commander')
```
```javascript
function handle (arg) {
    // 处理后返回值
}
// 第三个参数是函数，第四个参数依然是 packagename 的默认值，如果 packagename 没有值，
// 就会将第一个参数的值赋值给 packagename，再将 packagename 作为参数传给 handle 函数
program.option('-i, --install <packagename>', 'install some package', handle, 'commander')
```
### 案例
```javascript
var program = require('commander');

function delfun (arg) {
  console.log('接收到的参数 packagename 为' + arg);
  console.log('可以对参数 arg 进行处理并返回');
  console.log('返回值同时会赋值给 program.delete 属性');
}

program
  .version('0.0.1')
  .option('-i, --install [packagename]', 'install some package')
  .option('-u, --update [packagename]', 'update some package')
  .option('-d, --del [packagename]', 'delete some package', delfun, 'defval')
  .parse(process.argv);
	
// 命令行中使用到的参数会变成 true 进行标识，没有使用到的参数标识值为 false
if (program.install) console.log('调用了 -i 参数，我们可以对 -i 参数做具体处理')
if (program.update) console.log('调用了 -u 参数，我们可以对 -u 参数做具体处理')
if (program.del) console.log('调用了 -d 参数，我们可以对 -d 参数做具体处理')

console.log('验证 program.delete 属性值：', program.del);
```
[完整的博客](https://my.oschina.net/dkvirus/blog/1204210)