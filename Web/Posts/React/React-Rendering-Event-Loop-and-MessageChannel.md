---
title: "React Rendering: Event Loop and MessageChannel"
date: 2023-12-24T23:08:14Z
lastmod: "2023-12-24"
tags: ["React", "Behind The Scene", "Tutorial"]
draft: false
summary: "This article dive into React use of MessageChannel for efficient rendering."
layout: PostSimple
bibliography: references-data.bib
---

# React Rendering: Event Loop and MessageChannel

## Review Event Loop

以下这一张图，很好的展示了时间循环的基本流程。首先会执行同步代码，同步代码执行的时候，如果遇到异步代码，就会放到 `WebAPIs` 里面进行执行，当 `WebAPIs` 执行完毕之后，会将结果放入到 `Task Queue`（任务队列），同步代码执行完毕后，就会从任务队列中会获取一个一个的任务进行执行。

![React-Rendering-Event-Loop-and-MessageChannel-1](Images/React/React-Rendering-Event-Loop-and-MessageChannel/React-Rendering-Event-Loop-and-MessageChannel-1.gif)

> 以下是更细致的解释：
>
> 浏览器的渲染主线程遇到异步代码 `setTimeout` 后会通知计时线程开始计时，然后当前任务结束，即 `setTimeout` 这个函数运行结束了，开始运行下一个函数任务。时间到了以后，计时线程将事先传递的回调函数包装成任务，加入到消息队列的末尾排队，等待主线程调度执行。

如果将事件循环和浏览器渲染结合到一起，大致就是下面这张图：

![React-Rendering-Event-Loop-and-MessageChannel-2](Images/React/React-Rendering-Event-Loop-and-MessageChannel/React-Rendering-Event-Loop-and-MessageChannel-2.gif)

从上面的动图，我们可以看出，每一次事件循环，会从任务队列里面获取一个任务来执行。

之前有讲过，大多数设备的刷新率是 60hz，也就是 1 秒钟要绘制 60 次，这意味着浏览器每隔 16.66ms 就需要重新渲染一次。

总结一下：事件循环的机制就是每一次循环，会从任务队列中取一个任务来执行，如果还没有达到浏览器需要重新渲染的时间（16.66ms），那么就继续循环一次，从任务队列里面再取一个任务来执行，依此类推，直到浏览器需要重新渲染，这个时候就会执行重选渲染的任务，执行完毕后，回到之前的流程。

GIF 里面的 S、L、P 分别指代 `Recalculate Style`、`Layout`、`Paint`。而 rAF 则为 `requestAnimmationFrame`

`requestAnimationFrame Api` 是在每一次重新渲染之前执行，这个 API 的出现，就是专门拿来做动画的。以前我们做动画，用的更多的是 `setInterval` 或者 `setTimeout`，但是这些 API 本意不是拿来做动画的。使用 `requestAnimationFrame Api` 拿到做动画，最大的优点就是频率是和浏览器重新渲染的频率一致。

![React-Rendering-Event-Loop-and-MessageChannel-3](Images/React/React-Rendering-Event-Loop-and-MessageChannel/React-Rendering-Event-Loop-and-MessageChannel-3.png)

requestAnimationFrame 就不会存在这个问题，因为它是在渲染之前，保证了和浏览器渲染是同频

![React-Rendering-Event-Loop-and-MessageChannel-4](Images/React/React-Rendering-Event-Loop-and-MessageChannel/React-Rendering-Event-Loop-and-MessageChannel-4.png)

微任务：如果微任务队列里面存在任务，那么事件循环会在循环一次的时候，将整个微任务队列清空。

每一次事件循环时这几种任务的区别，如下图：

![React-Rendering-Event-Loop-and-MessageChannel-5](Images/React/React-Rendering-Event-Loop-and-MessageChannel/React-Rendering-Event-Loop-and-MessageChannel-5.gif)

## MessageChannel & Why choose it

MessageChannel 接口本身是用来做消息通信的，允许我们创建一个消息通道，通过它的两个 MessagePort 来进行信息的发送和接收。

基本使用如下：

```html
<div>
  <input type="text" id="content" placeholder="请输入消息" />
</div>
<div>
  <button id="btn1">给 port1 发消息</button>
  <button id="btn2">给 port2 发消息</button>
</div>
```

```js
const channel = new MessageChannel();
// 两个信息端口，这两个信息端口可以进行信息的通信
const port1 = channel.port1;
const port2 = channel.port2;
btn1.onclick = function () {
  // 给 port1 发消息
  // 那么这个信息就应该由 port2 来进行发送
  port2.postMessage(content.value);
};
// port1 需要监听发送给自己的消息
port1.onmessage = function (event) {
  console.log(`port1 收到了来自 port2 的消息：${event.data}`);
};

btn2.onclick = function () {
  // 给 port2 发消息
  // 那么这个信息就应该由 port1 来进行发送
  port1.postMessage(content.value);
};
// port2 需要监听发送给自己的消息
port2.onmessage = function (event) {
  console.log(`port2 收到了来自 port1 的消息：${event.data}`);
};
```

那么这个和 `Scheduler` 有什么关系呢？

之前，我们有说过 `Scheduler` 是用来调度任务，调度任务需要满足两个条件：

- JavaScript 暂停，将主线程还给浏览器，让浏览器能够有序的重新渲染页面
- 暂停了的 JavaScript（说明还没有执行完），需要再下一次接着来执行

那么这里自然而然就会想到**事件循环**，我们可以将没有执行完的 JavaScript 放入到任务队列，下一次事件循环的时候再取出来执行。

那么，如何将没有执行完的任务放入到任务队列中呢？

那么这里就需要产生一个任务（宏任务），这里就可以使用 `MessageChannel`，因为 `MessageChannel` 能够产生宏任务。

### Why not <u>setTimeout</u>

以前要创建一个宏任务，可以采用 `setTimeout(fn, 0)` 这种方式，但是 _React_ 团队没有采用这种方式。

这是因为 `setTimeout` 在嵌套层级超过 5 层，`timeout`（延时）如果小于 **4ms**，那么则会设置为 **4ms**。

> 这个可以参阅 _HTML_ 规范：*https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#dom-settimeout*

> If the nesting level is greater than 5, and the `timeout` is less than 4, then set the `timeout` to 4.

一个例子来进行验证：

```js
let count = 0; // 计数器
let startTime = new Date(); // 获取当前的时间戳
console.log("start time:", 0, 0);
function fn() {
  setTimeout(function () {
    console.log("exec time:", ++count, new Date() - startTime);
    if (count === 50) return;
    fn();
  }, 0);
}
fn();
```

执行结果部分截图如下：

![React-Rendering-Event-Loop-and-MessageChannel-6](Images/React/React-Rendering-Event-Loop-and-MessageChannel/React-Rendering-Event-Loop-and-MessageChannel-6.png)

正因为这个原因，所以 _React_ 团队没有选择使用 `setTimeout` 来产生任务，因为 **4ms** 的时间的浪费还是不可忽视的。

### Why not <u>requestAnimationFrame</u>

这个也不合适，因为这个只能在重新渲染之前，才能够执行一次，而如果我们包装成一个任务，放入到任务队列中，那么只要没到重新渲染的时间，就可以一直从任务队列里面获取任务来执行。

而且 `requestAnimationFrame` 还会有一定的兼容性问题，**safari** 和 **Edge** 浏览器是将 `requestAnimationFrame` 放到渲染之后执行的，**Chrome** 和 **Firefox** 是将 `requestAnimationFrame` 放到渲染之前执行的，所以这里存在不同的浏览器有不同的执行顺序的问题。

> 根据标准，应该是放在渲染之前。

### Why not wrap it into a microtask?

这是因为和微任务的执行机制有关系，微任务队列会在清空整个队列之后才会结束。那么微任务会在页面更新前一直执行，直到队列被清空，达不到将主线程还给浏览器的目的。
