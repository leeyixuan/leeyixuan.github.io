---
layout:     post
title:      "深入理解JS事件循环机制"
subtitle:   "浏览器和node.js中的事件循环机制的分析比较"
date:       2018-3-12
author:     "leeyixuan"
header-img: "img/background/post-bg-eventloop.jpg"
tags:
    - js
---



## 浏览器中的事件循环机制
### 浏览器的渲染进程
每打开一个浏览器tab页，就会开启一个**浏览器的渲染进程**，负责页面渲染，脚本执行，事件处理等。

该浏览器的渲染进程包括以下线程：
1. GUI渲染线程   
负责渲染浏览器界面，解析HTML，CSS，构建DOM树和RenderObject树，布局和绘制等；负责渲染浏览器界面，解析HTML，CSS，构建DOM树和RenderObject树，布局和绘制等。

2. JS引擎线程（JS内核，例如V8）   
负责处理Javascript脚本程序，运行代码。

3. 事件触发线程      
当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎的处理。当JS引擎执行代码块如setTimeOut时（也可来自浏览器内核的其他线程,如鼠标点击），会将对应任务添加到事件线程中。

4. 定时触发器线程    
setInterval与setTimeout所在线程，负责计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）。

5. 异步http请求线程     
XMLHttpRequest在连接后是通过浏览器新开一个线程请求，检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。

 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531703327669.png" />

**注意**：
- 由于JS的单线程关系，所以这些待处理队列中的事件都得排队等待JS引擎处理（当JS引擎空闲时才会去执行）。
- GUI渲染线程与JS引擎线程是互斥的，所以如果JS执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。


### JS单线程和异步
JS分为同步任务和异步任务。同步任务都在主线程上执行，形成一个执行栈。主线程之外，事件触发线程管理着一个任务队列，只要异步任务有了运行结果，就在任务队列之中放置一个事件。一旦执行栈中的所有同步任务执行完毕（此时JS引擎空闲），系统就会读取任务队列，将可运行的异步任务添加到可执行栈中，开始执行。


### 事件循环

工作线程（事件触发线程、定时触发器线程、异步http请求线程）将回调放到任务队列，主线程（JS引擎线程）通过事件循环过程依次轮训回调。JS实现异步的核心就是事件循环。

实质上，事件循环就是：检查调用栈是否为空，以及确定把哪个task加入调用栈。


<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531662515883.png" />

### ES6中的事件循环——宏任务和微任务
ES6推出的promise等概念，更加细化了事件循环。
现在除了广义的同步任务和异步任务，我们对任务有更精细的定义：

**macro-task(宏任务)**：包括整体代码script，setTimeout，setInterval，setImmediate（macro-task任务源非常宽泛，比如ajax的onload，click事件，基本上我们经常绑定的各种事件都是task任务源）
**micro-task(微任务)**：Promise，process.nextTick，MutationObserver

 
 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531665036247.png" />

**ES6中的事件循环包括**：一个宏任务，所有微任务（，更新渲染），一个宏任务，所有微任务（，更新渲染）......

>渲染更新：执行完microtask队列里的任务，有可能会渲染更新。在一帧以内的多次dom变动浏览器不会立即响应，而是会积攒变动以最高60HZ的频率更新视图。


## node.js中的事件循环机制
### 6个阶段
node.js中的事件循环由 libuv库实现，每次事件循环都包含了6个阶段。
<img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1531662515628.png" />

- timers 阶段：这个阶段执行timer（setTimeout、setInterval）的回调
- I/O callbacks 阶段：执行一些系统调用错误，比如网络通信的错误回调
- idle, prepare 阶段：仅node内部使用
- poll 阶段：获取新的I/O事件, 适当的条件下node将阻塞在这里
- check 阶段：执行 setImmediate() 的回调
- close callbacks 阶段：执行 socket 的 close 事件回调

日常开发中的绝大部分异步任务都是在timers、poll、check这3个阶段这处理的，我们只需要着重关注这三个阶段。


### 事件循环

step1.循环开始之前
- 同步任务
- 发出异步请求
- 规划定时器生效时间
- 执行process.nextTick()

step2.开始循环
- 清空当前循环内的 Timers Queue，清空NextTick Queue，清空Microtask Queue
- 清空当前循环内的 Poll Queue，清空NextTick Queue，清空Microtask Queue
- 清空当前循环内的 Close Queue，清空NextTick Queue，清空Microtask Queue

step3.进入下一轮循环

**总结**：
事件循环的每个阶段都有一个任务队列。当事件循环到达某个阶段时，将执行该阶段的任务队列，直到队列清空或执行的回调达到系统上限后，才会转入下一个阶段。当所有阶段被顺序执行一次后，称事件循环完成了一次。

### setTimeout |setImmediate|process.nextTick
>1.不在同一个I/O cycle中的时候，回调的调度顺序是不被保证的；在同一个I/O cycle中，immediate 总比 timeout 更早被调度。

setTimeout 设定一个最短的调度该脚本的时间阈值。实际上是在该时间阈值到达之后，加入timer阶段，因此往往会被添加到下一次事件循环的timer阶段执行。
setImmediate 一旦当前poll阶段结束就执行一次脚本。由于setImmediate是check阶段的执行，所以往往可以赶在本次事件循环poll之后立即执行。


>2.setImmediate()实际上在 process.nextTick() 之后执行。


`process.nextTick() `会在各个事件阶段之间执行，一旦执行，要直到nextTick队列被清空，才会进入到下一个事件阶段，所以如果递归调用 process.nextTick()，则会发生I/O阻塞。



## Node.js 与浏览器的 Event Loop 差异
> 1.微任务队列的执行时机不同：

- Node.js中，微任务在事件循环的各个阶段之间执行，清空完当前阶段的任务队列之后，再清空微任务队列。
- 浏览器中，微任务在事件循环的宏任务执行完之后执行，执行一个宏任务，清空所有微任务。


> 2.事件添加到任务队列中的顺序不同：

- Node.js中，有6个阶段，每个阶段对应一个当前阶段的任务队列和一个微任务队列。根据事件源的不同将事件添加到不同阶段的任务队列中。一次事件循环依次清空每个阶段对应的任务队列和微任务队列。
- 浏览器和node.js相比，始终维护一个主任务队列。

多个任务队列，不同的优先级
## 附加题
```
setTimeout(() => console.log('setTimeout1'), 0);    // 1#
setTimeout(() => {                  // 2#
    console.log('setTimeout2');     // 2-1#
    Promise.resolve().then(() => {  // 2-2#
        console.log('promise2');        // 2-2-1#
        Promise.resolve().then(() => {  // 2-2-2#
            console.log('promise3');
        })
        console.log(5)                  // 2-2-3#
    })
    setTimeout(() => console.log('setTimeout4'), 0);    // 2-3#
}, 0);
setTimeout(() => console.log('setTimeout3'), 0);    // 3#
Promise.resolve().then(() => {  // 4#
    console.log('promise1');
})
```
在浏览器环境中的结果：
```
promise1
setTimeout1
setTimeout2
promise2
5
promise3
setTimeout3
setTimeout4
```
在node中的结果：
```
promise1
setTimeout1
setTimeout2
setTimeout3
promise2
5
promise3
setTimeout4
```


## Reference
1. [并发模型与事件循环 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
2. [JavaScript 运行机制详解：再谈Event Loop - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
3. [详解JavaScript中的Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/33058983)
4. [从一道题浅说 JavaScript 的事件循环 · Issue #61 · dwqs/blog](https://github.com/dwqs/blog/issues/61)
5. [有别于浏览器环境的Node.js Event loop](http://sweetalkto.me/2017/12/08/%E6%9C%89%E5%88%AB%E4%BA%8E%E6%B5%8F%E8%A7%88%E5%99%A8%E7%8E%AF%E5%A2%83%E7%9A%84Node.js%20Event%20loop/)
6. [这一次，彻底弄懂 JavaScript 执行机制 - 掘金](https://juejin.im/post/59e85eebf265da430d571f89)
7. [从event loop规范探究javaScript异步及浏览器更新渲染时机 · Issue #5 · aooy/blog](https://github.com/aooy/blog/issues/5)
8. [浏览器和NodeJS中不同的Event Loop · Issue #234 · kaola-fed/blog](https://github.com/kaola-fed/blog/issues/234#code-analysis-b-a)