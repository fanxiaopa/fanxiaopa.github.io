---
title: useEffect & useLayoutEffect的区别
date: 2021-09-17 16:19:02
tags:
---

#### 先说结论：

两者执行时机不同，`useEffect`在渲染之后异步执行；`useLayoutEffect`在渲染之前同步执行，如果执行复杂的操作，会阻塞页面渲染

-  1、useLayoutEffect和componentDidMount和componentDidUpdate触发时机一致（都在在DOM修改后且浏览器渲染之前），会阻塞浏览器渲染！
- 2、useEffect执行时机是commit阶段后异步执行（可理解为浏览器渲染之后），不会阻塞浏览器渲染

