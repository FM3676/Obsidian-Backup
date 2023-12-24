---
title: 'Understanding React Fiber Architecture and Double Buffering Mechanism'
date: 2023-12-23T23:08:14Z
lastmod: '2023-12-23'
tags: ['React', 'Behind The Scene', 'Tutorial']
draft: false
summary: 'This article help us to understand React Fiber Architecture and Double Buffering Mechanism'
layout: PostSimple
bibliography: references-data.bib
---

# Understanding React Fiber Architecture and Double Buffering Mechanism

## Understanding React Fiber

可以从三个维度来理解 Fiber：

- 是一种架构，称之为 Fiber 架构
- 是一种数据类型
- 动态的工作单元

### 是一种架构，称之为 Fiber 架构

在 React v16 之前，使用的是 `Stack Reconciler`，因此那个时候的 React 架构被称之为 Stack 架构。从 React v16 开始，重构了整个架构，引入了 Fiber，因此新的架构也被称之为 Fiber 架构，`Stack Reconciler` 也变成了 `Fiber Reconciler`。各个 `FiberNode` 之间通过链表的形式串联起来：

```js
function FiberNode(tag, pendingProps, key, mode) {
  // ...

  // 周围的 Fiber Node 通过链表的形式进行关联
  this.return = null
  this.child = null
  this.sibling = null
  this.index = 0

  // ...
}
```

### 是一种数据类型

Fiber 本质上也是一个对象，是在之前 React 元素基础上的一种升级版本。每个 `FiberNode` 对象里面会包含 React 元素的类型、周围链接的 `FiberNode` 以及 DOM 相关信息：

```js
function FiberNode(tag, pendingProps, key, mode) {
  // 类型
  this.tag = tag
  this.key = key
  this.elementType = null
  this.type = null
  this.stateNode = null // 映射真实 DOM

  // ...
}
```

### 动态的工作单元

在每个 `FiberNode` 中，保存了本次更新中该 React 元素变化的数据，还有就是要执行的工作（增、删、更新）以及副作用的信息：

```js
function FiberNode(tag, pendingProps, key, mode) {
  // ...

  // 副作用相关
  this.flags = NoFlags
  this.subtreeFlags = NoFlags
  this.deletions = null
  // 与调度优先级有关
  this.lanes = NoLanes
  this.childLanes = NoLanes

  // ...
}
```

> 为什么指向父 `FiberNode` 的字段叫做 `return` 而非 `parent`？
>
> 因为作为一个动态的工作单元，`return` 指代的是 `FiberNode` 执行完 `completeWork` 后返回的下一个 `FiberNode`，这里会有一个返回的动作，因此通过 `return` 来指代父 `FiberNode`

## Fiber 双缓冲

Fiber 架构中的双缓冲工作原理类似于显卡的工作原理。

显卡分为**前缓冲区**和**后缓冲区**。首先，**前缓冲区**会显示图像，之后，合成的新的图像会被写入到**后缓冲区**，一旦**后缓冲区**写入图像完毕，就会前后缓冲区进行一个互换，这种将数据保存在缓冲区再进行互换的技术，就被称之为**双缓冲技术**。

Fiber 架构同样用到了这个技术，在 Fiber 架构中，同时存在两颗 `Fiber Tree`，一颗是真实 UI 对应的 `Fiber Tree`，可以类比为显卡的**前缓冲区**，另外一颗是在内存中构建的 `FiberTree`，可以类比为显卡的**后缓冲区**。

在 React 源码中，很多方法都需要接收两颗 `FiberTree`：

```js
function cloneChildFibers(current, workInProgress) {
  // ...
}
```

`current` 指的就是前缓冲区的 `FiberNode`，`workInProgress` 指的就是后缓冲区的 `FiberNode`。

两个 `FiberNode` 会通过 `alternate` 属性相互指向：

```js
current.alternate = workInProgress
workInProgress.alternate = current
```

接下来从首次渲染（mount）和更新（update）这两个阶段来看一下 `FiberTree` 的形成以及双缓存机制：

### mount Progress

首先最顶层有一个 `FiberNode`，称之为 `FiberRootNode`，该 `FiberNode` 会有一些自己的任务：

- Current Fiber Tree 与 Wip Fiber Tree 之间的切换
- 应用中的过期时间
- 应用的任务调度信息

现在假设有这么一个结构：

```html
<body>
  <div id="root"></div>
</body>
```

```jsx
function App() {
  const [num, add] = useState(0)
  return <p onClick={() => add(num + 1)}>{num}</p>
}
const rootElement = document.getElementById('root')
ReactDOM.createRoot(rootElement).render(<App />)
```

当执行 `ReactDOM.createRoot` 的时候，会创建如下的结构：

![Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-1](Images/React/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-1.png)

> 在这里，`HostRootFiber` 便是 `<div id="root"></div>`

此时会有一个 `HostRootFiber`，`FiberRootNode` 通过 `current` 来指向 `HostRootFiber`。

接下来进入到 `mount` 流程，该流程会基于每个 React 元素以深度优先的原则依次生成 `wipFiberNode`，并且每一个 `wipFiberNode` 会连接起来，如下图所示：

![Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-2](Images/React/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-2.png)

生成的 `wipFiberTree` 里面的每一个 `FiberNode` 会和 `currentFiberTree` 里面的 `FiberNode` 进行关联，关联的方式就是通过 `alternate`。但是目前 `currentFiberTree` 里面只有一个 `HostRootFiber`，因此就只有这个 `HostRootFiber` 进行了 `alternate` 的关联。

当 `wipFiberTree` 生成完毕后，也就意味着 `render` 阶段完毕了，此时 `FiberRootNode` 就会被传递给 `Renderer`（渲染器），接下来就是进行渲染工作。渲染工作完毕后，浏览器中就显示了对应的 UI，此时 `FiberRootNode.current` 就会指向这颗 `wipFiberTree`，曾经的 `wipFiberTree` 它就会变成 `currentFiberTree`，完成了双缓存的工作：

> 这里说的是 render 阶段，来回忆一下相关知识：
>
> 对应到 React 里面就两大阶段：
>
> - render 阶段：调合虚拟 DOM，计算出最终要渲染出来的虚拟 DOM
> - commit 阶段：根据上一步计算出来的虚拟 DOM，渲染具体的 UI
>
> 每个阶段对应不同的组件：
>
> - 调度器(Scheduer)：调度任务，为任务排序优先级，让优先级高的任务先进入到 Reconciler
> - 协调器(Reconciler)：生成 Fiber 对象，收集副作用，找出哪些节点发生了变化，打上不同的 flags，著名的 diff 算法也是在这个组件中执行的。
> - 渲染器(Renderer)：根据协调器计算出来的虚拟 DOM 同步的渲染节点到视图上。
>
> 以下是更为细致的解释：
>
> - **Reconciliation（协调/对比）阶段**：这是 React 内部的一个过程，旨在对比新旧虚拟 DOM（Fiber 树）的差异，并计算出需要进行的 DOM 更新。这个阶段可以看作是 React 的“diff”算法的一部分。它决定了哪些组件需要更新，以及具体的更新方式。
> - **Render 阶段**：在 Reconciliation 阶段之后，React 会进入 Render 阶段，这个阶段会根据 Reconciliation 阶段的结果，生成一个新的 Fiber 树（Work In Progress Tree），然后这个树会转换为 DOM 更新。
> - **Commit 阶段**：一旦 Render 阶段完成，React 进入 Commit 阶段，这个阶段将所有的变更实际应用到 DOM 上，这也是 React 更新 DOM 的唯一时间。也就是说，用户现在可以看到更新后的界面。
>
> “当 wip FiberTree 生成完毕后，也就意味着 render 阶段完毕了”，实际上是指的 Render 阶段，而不是整个 Reconciliation 过程。在 React 18 之前，这个过程通常被称为“Reconciliation”，但在 React 18 引入并发特性（Concurrent Features）之后，整个流程被划分为更细致的阶段。
>
> 在 React 18+ 的并发模式下，这些阶段被进一步细分，允许 React "中断" Reconciliation 和 Render 阶段的工作，以便于调度更重要的任务（如用户输入），这提高了应用程序的响应性和性能表现。
>
> 所以，通常我们说的“render 阶段结束”指的是生成了新的 Work In Progress Fiber 树的过程完成，而 Commit 阶段是将这些更改实际应用到 DOM 上。
>
> 如果要描述整个过程的话，可以这样说：当 wip FiberTree 生成并完成协调/对比后，render 阶段结束，接下来 React 会在 Commit 阶段将变更应用到 DOM，从而完成用户界面的更新。此时，`FiberRootNode.current`会指向新的 Fiber 树，确保 React 可以为后续的更新提供正确的基准。

![Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-3](Images/React/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-3.png)

### update Progress

点击 p 元素，会触发更新，这一操作就会开启 update 流程，此时就会生成一颗新的 wip Fiber Tree，流程和之前是一样的

![Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-4](Images/React/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-4.png)

新的 `wipFiberTree` 里面的每一个 `FiberNode` 和 `currentFiberTree` 的每一个 `FiberNode` 通过 alternate 属性进行关联。

当 `wipFiberTree` 生成完毕后，就会经历和之前一样的流程，`FiberRootNode` 会被传递给 `Renderer` 进行渲染，此时宿主环境所渲染出来的真实 UI 对应的就是左边 `wipFiberTree` 所对应的 DOM 结构，`FiberRootNode.current` 就会指向左边这棵树，右边的树就再次成为了新的 `wipFiberTree`

![Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-5](Images/React/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism/Understanding-React-Fiber-Architecture-and-Double-Buffering-Mechanism-5.png)

这个就是 Fiber 双缓存的工作原理。

另外值得一提的是，开发者是可以在一个页面创建多个应用的，比如：

```js
ReactDOM.createRoot(rootElement1).render(<App1 />)
ReactDOM.createRoot(rootElement2).render(<App2 />)
ReactDOM.createRoot(rootElement3).render(<App3 />)
```

在上面的代码中，我们创建了 3 个应用，此时就会存在 3 个 FiberRootNode，以及对应最多 6 棵 Fiber Tree 树。

## 真题解析

> 题目：谈一谈你对 React 中 Fiber 的理解以及什么是 Fiber 双缓冲？
>
> 参考答案：
>
> Fiber 可以从三个方面去理解：
>
> - **FiberNode 作为一种架构**：在 React v15 以及之前的版本中，Reconceiler 采用的是递归的方式，因此被称之为 `Stack Reconciler`，到了 React v16 版本之后，引入了 Fiber，Reconceiler 也从 `Stack Reconciler` 变为了 `Fiber Reconceiler`，各个 `FiberNode` 之间通过链表的形式串联了起来。
> - **FiberNode 作为一种数据类型**：Fiber 本质上也是一个对象，是之前虚拟 DOM 对象（React 元素，createElement 的返回值）的一种升级版本，每个 Fiber 对象里面会包含 React 元素的类型，周围链接的 `FiberNode`，DOM 相关信息。
> - **FiberNode 作为动态的工作单元**：在每个 `FiberNode` 中，保存了“本次更新中该 React 元素变化的数据、要执行的工作（增、删、改、更新 Ref、副作用等）”等信息。
>
> 所谓 Fiber 双缓冲树，指的是在内存中构建两颗树，并直接在内存中进行替换的技术。在 React 中使用 `WipFiberTree` 和 `CurrentFiberTree` 这两颗树来实现更新的逻辑。`WipFiberTree` 在内存中完成更新，而 `CurrentFiberTree` 是最终要渲染的树，两颗树通过 alternate 指针相互指向，这样在下一次渲染的时候，直接复用 `WipFiberTree` 作为下一次的渲染树，而上一次的渲染树又作为新的 `WipFiberTree`，这样可以加快 DOM 节点的替换与更新。
