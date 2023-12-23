---
title: How JavaScript Works Behind the Scenes
date: "2023-1-28"
tags: ["JavaScript", "tutorial", "behine the scene"]
draft: false
summary: "In this article we learn How JavaScript Works Behind the Scenes, and what is JavScript Engine and How it works."
---

# How JavaScript Works Behind the Scenes

## An High-Level Overview Of JavaScript

> JavaScript is a High-Level, Prototype-Based Object-Oriented, Multi-Paradigm, Interpreted Or Just-In-Time Complied, Dynamic, Single-Threaded, Garabage-Collected Programming Language With First-Class Functions And a Non-Blocking Event Loop Concurrency Model.

> JavaScript 是一种高级的、基于原型的、面向对象的、多范式、解释或即时编译、动态的、单线程的、有垃圾收集的编程语言。具有允许将函数赋值给变量，作为参数传递给其他函数的特性和非阻塞事件循环并发模型

### High Level

低级语言例如 C，你需要手动管理资源，如向计算机内存以创建新变量。但高级语言如 JavaScript 或 Python，开发者不需要操心管理内存资源，这些语言更容易学习，但它们的运行速度通常没有低级语言那么快，最大可优化成都也不如低级语言。

### Garabage Collected

垃圾收集是电脑根据算法，自动的从内存删除旧的、未使用的对象。

### Interpreted Or Just-In-Time Complied

机器只认识 0 和 1，但写代码不回去直接写 0 和 1。都是将人类可读的代码转换编译为机器码后由机器运行，所有语言都是。

### Multi-Paradigm

> Paradigm：An Apporach and mindset of structuring code, which will direct your coding style and technique.

> 范式： 是一种构建代码的风格和思维，它会影响指导你的代码风格和技术水平

三种流行的范式有：

1. Procedural programming 程序化的
2. Object-oriented programming(OOP)面向对象
3. Functional programming(FP) 函数式

同时还可以将范式分为命令式和声明式的。

许多语言是只能有三者其一的特性的，如只能程序式编写，或 OOP 编写。但 JavaScript 都可以做得到。

### Prototype-Based Object-Oriented

JavaScript 的面向对象特性，是一种基于原型、面向对象的方法。什么意思呢？

JavaScript 中的几乎所有东西都是一个对象 Object，除了原始值，如字符 String，数字 Number 等。

当新建一个数组以后，可以直接在这个叔祖上面使用`push`方法，是因为原型继承。Array 原型上有这个方法，新建的数组会自动继承这上面的所有方法。

### First-Class Functions

支持 First-Class Functions 的语言允许将函数赋值给变量,作为参数传递给其他函数,并从其他函数返回

### Dynamic

变量类型轻松改变，像 C 需要在声明变量时声明其类型，JavaScript 不需要，其变量可以被重新赋值为任意类型。

## The JavaScript Engine and Runtime

### What is a JavaScript Engine?

JavaScript 引擎用于执行 JavaScript 代码，每个浏览器都有自己的引擎，最出名的是 Google 的 V8，其次还有 Node.js 可以用于构建服务端。但任何的 JavaScript 都会有两个东西：调用栈（Call Stack）和堆（Heap）。Call Stack 是代码实际执行的地方，Heap 则是储存了应用程序所需要的所有对象。那么，JavaScript 代码是如何进入到 Call Stack 并被执行的呢？

### Compilation vs. Interpretation

机器只懂 0 和 1，所以所有的程序都需要经过编译，或解释为机器能读懂的机器码。

编译：指代码会先被翻译为机器码，生成 PE 文件（Portable File）例如 exe，然后再被计算机执行

解释：代码被逐行运行，不会提前编译，运行到哪里，编译到哪里。

JavaScript 曾经是纯粹的解释型语言，但它的效率会非常低，所以现在的 JavaScript 不再是解释型语言了。它两种混用，称为即时编译（Just-In-Time Compilation）。它也会提前编译，但编译过后不会生成 PE 文件，而是马上执行，这大大提高了 JavaScript 的效率

### Morden Just-In-Time Compilation Of JavaScript

一段 JavaScript 进入引擎，会有以下几个步骤

第一步，解析代码。本质上是阅读代码，在阅读代码解析的过程中，这些代码会被解析成一种数据结构，抽象语法树（AST）。它首先将每一行代码拆分合并成有意义的语言片段，例如`const`或`function`，然后将这些转化成一种数据结构后存入树中。这一步还会检查是否有语法错误，生成的 AST 会在稍后用于生成机器码。

第二步，AST 被编译为机器码。最后，编译好的机器码进入 Call Stack 被执行。

但是现在 JavaScript 的引擎还会有一些聪明的优化策略，

它会先编译一段未经优化的、开头的 JavaScript 代码并执行，为了能够快速开始程序。在程序执行的时候，剩余那些代码会被优化然后重新编译然后再执行。这个过程周而复始。这个过程发生在一些特殊的线程，我们没有办法通过代码去访问这些线程。不同引擎会略微有区别，但是基本都是这个模式。

### JavaScript Runtime

运行环境除了需要引擎，还有 WEB APIs，包括 DOM 操作相关的 API、Timers、Fetch API 等等。它是**浏览器**提供给引擎一些功能，JavaScript 通过 global window 对象访问这些 API。

其次就是回调队列（Callback Queue），它是一个数据结构，包含所有准备要执行的回调函数。例如`setTimeOut`的回调、`addEventListener`的回调。例如，当点击 DOM 元素时，对应的回调会进入到回调队列，当 Call Stack 为空时，队列里的 Callback Function 就会进入 Call Stack 执行，这个过程周而复始，即事件循环（Event Loop）

## Execution Contexts and The Call Stack

### What is an Execution Context？

现在假设代码已经被编译完成，可以执行了。那么接下来，它会创建一个所谓的**全局执行上下文（Global Execution Context -- for top-level code）** 只有顶级代码才会在这段时间被执行，例如变量声明，函数会被编译。但只有在调用时才执行。

执行上下文（Execution Context）是 JavaScript 运行的地方，这是一个抽象的概念，它提供 JavaScript 代码运行所需要的一切如变量等等。就好像一个外卖盒子，里面除了吃的本身，还会有刀叉等让你吃饭的工具。

整个执行环境只有一个全局上下文（Exactly one global execution context -- EC）他为函数外的代码创建。每一个函数也会有自己的执行上下文，这些上下文会组成 Call Stack。

在执行完 top-level code 以后就会执行函数，随后就会等待回调函数进入并执行

### Execution Context in Detail

执行上下文里都有什么？

首先：变量环境（Variable Environment）。包括`let`、`const`、`var`等声明，还有函数。以及一个特殊的 arguments 对象，这个对象里包含了所有要传给当前正在上下文执行的函数的参数。每一个函数在被执行的时候都会创建自己的上下文，所以，当这个函数被调用的时候，其函数内声明的变量等最终都会进入变量环境。

其次就是：作用域链（Scope Chain），它由在当前执行函数意外对于变量的引用组成，其存在于每一个上下文。

最后：就是 this 关键字（this keyword）

就上三个都是在创建阶段生成的（Generated during ‘creation phase’, right before execution），此外，箭头函数没有自己的 arguments object 和 this keyword，其所使用的都是其最近的父级函数或对象。

下面来看一个实际的例子。

![Execution-Context-Example](//Images/How-JavaScript-Works-Behind-the-Scenes/Execution-Context-Example.png)

上图详细描述了具体的过程，首先是 Global Context，进行变量初始化，每一个函数，也会有自己的上下文，初始化其内部的变量。其中有的变量，需要先执行其他函数才可以知道具体为什么。这是一个很简单的例子，但实际的程序有上百个函数，上百个上下文。引擎如何知道调用顺序，以及目前正在调用哪个函数呢？

之前提到的 Call Stack，其实就由一个执行上下文的地方。上下文依次入栈，并每次执行栈顶的函数并使其出栈，以此实现追踪和按顺序调用。

下面来看看具体在 Call Stack 会发生什么。

第一步生成 Global 环境，并且入栈，并从顶层依次往下执行，生成对应的变量。当执行到`const x = first()`时，调用了`first()`，所以，生成`first()`的上下文并入栈，执行其里面的上下文。

在`first()`内执行到了`const b = second(7, 9)`。调用了`second()`函数，所以生成`second()`的上下文并入栈，执行其内部代码。当执行完`return c`以后，函数执行完毕，`second()`的上下文出栈，继续执行`first()`的上下文的内容。

执行到了`return a`，`first()`的上下文也出栈，只剩下了 Global，然后继续执行，将变量赋给`x`以后，代码执行完毕。此时 Global 对象可能不会出栈，也可能会。如果关闭浏览器那就一定会。

## Scope And The Scope Chain

### Scope Concepts

先看一些基本概念

> **Scoping**：How our program's variables are **organized** and **accessed**. "Where do variables live?" or "Where can we access a certain variable".
>
> **Lexical Scoping**: Scoping is controlled by **placement** of functions and blocks in the code.
>
> **Scope**: Space or environment in which a certain variable is **declared** (variable environment in case of funtions). There is **global** scope, **function** scope, and **block** scope.
>
> **Scope of a variable**: Region of our code where a certain variable can be **accessed**

> **作用域**：使用 JavaScript Engine 来控制，变量如何储存和访问。变量放在哪里，以及哪些变量可以访问，哪些不能？
>
> **词法作用域**：由函数和代码块决定的作用域范围
>
> **作用域**：一个在已声明变量内的环境或空间。有全局作用域，函数作用域，块作用域。
>
> **变量域 **： 指变量能够被引用的区域,也就是可以使用该变量的程序范围

### The 3 Types of Scope

![Three-Types-of-Scope](//Images/How-JavaScript-Works-Behind-the-Scenes/Three-Types-of-Scope.png)

### The Scope Chain

![The-Scope-Chain](//Images/How-JavaScript-Works-Behind-the-Scenes/The-Scope-Chain.png)

这张图片展示了什么是 Scope Chain，以及它如何工作的。

1. 下一级的作用域可以访问上一级的作用域内的变量。例如：`second()`可以访问`first()`内的变量，因为`second()`被包含在`first()`，但是`first()`不可以访问`second()`内的变量。而全局变量则是都可以访问
2. 在`if`语句内，`let`、`const`是属于块级作用于的，其内部声明的`var`变量，是属于函数作用域的，因此`second()`也可以访问得到
3. 同级别的作用域不可以互相访问。如：`if`、`second()`不可以访问其内部作用域的变量。

### Scope Chain vs. Call Stack

看看 Scope Chain 和 Call Stack 的关系

![Scope-Chain-vs-Call-Stack--Call-Stack-Part](//Images/How-JavaScript-Works-Behind-the-Scenes/Scope-Chain-vs-Call-Stack--Call-Stack-Part.png)

左边是一个小片段，右边是调用时 Call Stack 内的顺序，Call Stack 内的都是一段段执行上下文，里面是变量的环境等等。目前，这一切都和 Scope Chain 无关，这里所做的都只是为对应的函数创建上下文并根据函数的变量进行填充。

当这一切准备充分，就可以开始构建 Scope Chain

![Scope-Chain-vs-Call-Stack](//Images/How-JavaScript-Works-Behind-the-Scenes/Scope-Chain-vs-Call-Stack.png)

首先最顶层，是 Global Scope。往下走，第一个先是`first()`作用域，其内部可以访问到 Global Scope 的变量，`second()`声明在`first()`以内，因此`second()`可以访问到 Global Scope 和`first()`作用域的变量。

`second()`内调用了`thrid()`，但是，`thrid()`是在 Global Scope 下声明的，因此它并不能访问到`b`和`c`，其声明层级和`first()`是相同的。

此外，作用域链与调用顺序无关，与之相关的是声明的嵌套关系。

### Summary

1. Scoping asks the question "Where do variables live?" or "Where can we access a certain variable, and where not?"
2. There are 3 types of scope in JavaScript: the global scope, scopes defined by functions, and scopes defined by blocks;
3. Only `let` and `const` variables are block-scoped. Variables declared witeh `var` end up in the closet function scope;
4. In JavaScript, we have lexical scoping, so the rules of where we can access variables are based on exactly where in the code functions and blocks are written;
5. Every scope always has access to all the variables from all its outer scopes. This is the scope chain!
6. Rhen a variable is not in the current scope, the engine looks up in the scope chain until it finds the variables it's looking for. This is called variable lookup.
7. The scope chain is a one-way street: a scope will never, ever have access to the variables of an inner scope;
8. The scope chain in a certain scope is euqal to adding together all the variables environments of the all parent scopes;
9. The scope chain has nothing to do with the order in which functions were called. It does not affect the scope chain at all!

## Variable Enviorment: Hoisting and the TDZ

这个 Scetion 来关注 Execution Context 的 Variable Enviorment，以及变量，是如何在 JavaScript 中被创建的。

### Hoisting in JavaScript

> **Hoisting**: Makes some types of variables accessible/usable in the code before they are declared. "Variables lifted to the top of their scope".
>
> -----BEHIND THE SCENES-----
>
> **Before execution,** code is scanned for variable declarations, and for each variable, a new property is created in the **variable enviroment object.**

> **变量提升**：让一些变量在它们被实际声明前可以被使用。
>
> -----实质-----
>
> 在代码执行之前，代码会被扫描以进行变量声明。对于每个变量，都会在 variable enviroment object 内创建一个新属性

![Hoisting-in-JavaScript](//Images/How-JavaScript-Works-Behind-the-Scenes/Hoisting-in-JavaScript.png)

1. `var`声明会发生变量提升，提升初值为`undefined`。例如

   ```js
   console.log(b);
   var b = 0;
   ```

   结果会是`undefined`

2. 对于`let`、`const`，表面上是不会发生变量提升的。但实际理论上是会的但他们的值被设置为`<uninitialized>`，相当于没有值，就好像根本没有被变量提升一样。这些变量被称为，放置在所谓暂时性死区（TDZ，Temporal Dead Zone）

3. 对于赋值函数（`const a = function(){}`）和箭头函数，他们是否被变量提升，取决于其被赋的变量使用的是`var`、`const`还是`let`。

### Temporal Dead Zone，let And const

![TDZ](//Images/How-JavaScript-Works-Behind-the-Scenes/TDZ.png)

可以看到，这段代码在`job`声明之前就使用了它，结果报错提示的是：不可以在`job`没有初始化以前访问它

同时下面也访问了`x`变量，这个`x`变量没有在之前声明，发生变量提升，所以它提升的是，`x`没有被定义。

为什么会有 TDZ？

首先，这会减少代码出错的可能，在变量声明之前使用它不是什么好习惯。其次，它真正的使`const`发挥出了它应有的作用。`const`变量不可以被重新赋值，所以也不存在先赋值为`undefined`然后再赋予其真实的变量值。

为什么会有 Hoisting 呢？

可以在函数声明前使用这个函数，这是有作用的，例如在相互递归中。同时，`var`是 function hoisting 的副产品。

## The this Keyword

> **this keyword/variable**: Special variable that is created for every execution context (every function). Takes the value of (points to) the "owner" of the function in which **this** keyword is used.

> **this 关键词**：每一个执行上下文都有的特殊变量。其指向使用了 this 关键词的函数的所有者，也可以说，指向函数的父级（所有者）。（谁调用 this 就指向谁）

**this** is **NOT** static. It depends on **how** the function is called, and its value is only assigned when the function **is actually called.**

this 关键词不是静态的，他取决于函数怎么被调用，其值在函数真正被调用的时候赋值。

![this-keyword-four-methods](//Images/How-JavaScript-Works-Behind-the-Scenes/this-keyword-four-methods.png)

以上是四个例子，分别有四种情况，this 的指向不同：

1. 作为一个方法被调用，那么 this 会指向这个方法隶属的 Object，就如右边的例子一样
2. 普通函数被调用，如果为严格模式下，应该为`undefined`，如果不是，会指向`Global`或`Window`对象
3. 箭头函数，其没有自己的 this， **它的 this 是继承而来; 默认指向在定义它时所处的对象(宿主对象)，此处指父级作用域，**而不是执行时的对象
4. 事件监听，会指向所监听的 DOM 元素。

### Regular Functions vs. Arrow Functions

```js
const jonas = {
  firstName: "Jonas",
  greet: () => {
    console.log(this);
    console.log(this.firstName);
  },
};
jonas.greet();
```

以上代码运行的结果为

```js
Window {0: global, 1: Window, 2: Window, window: Window, self: Window, document: document, name: '', location: Location, …}
undefined
```

为什么这里的`this`没有指向`jonas`呢？上面提到，Arrow Functions 的`this`指向在定义它时所处的父级作用域的`this`，而不是执行对象。这里虽然定义在`jonas`里，但是此处的`{}`并不是一个代码块，只是一个定义 Object 的语法。其所属的为 Global Scope，所以指向的是 Window 对象，如果是 Regular Functions，则不会有这样的问题。

同时，上文提到的，使用`var`定义变量会添加 Window 对象的属性。例子如下，添加下一行代码

```js
var firstName = "ABC";
```

如果现在再去执行，会发现输出为`ABC`，因为`Window`对象上的属性多了一个 `firstName`，自然就可以调用上面的值。

![Var-variable-add-property-on-Window-Object](//Images/How-JavaScript-Works-Behind-the-Scenes/Var-variable-add-property-on-Window-Object.png)

所以，定义对象方法时候，**不要使用 Arrow Functions**

```js
const jonas = {
  firstName: "Jonas",
  greet: function () {
    const g = function () {
      console.log(this);
      console.log(this.firstName);
    };
    g();
  },
};
jonas.greet();
```

这段代码里，`g`是无法读取到`jonas`为`this`的，因为定义它的环境是一个 function，是有自己的作用域的，而这个作用域内的`this`并没有`firstName`。所以时会报错的

```js
const jonas = {
  firstName: "Jonas",
  greet: function () {
    const self = this;
    const g = function () {
      console.log(self);
      console.log(self.firstName);
    };
    g();
  },
};
jonas.greet();
```

这样就可以正确获取到`firstName`，因为它暂存了`this`为`self`，还可以利用 Arrow Functions 来解决。

```js
const jonas = {
  firstName: "Jonas",
  greet: function () {
    const g = () => {
      console.log(this);
      console.log(this.firstName);
    };
    g();
  },
};
jonas.greet();
```

Arrow Functions 的`this`会指向其父级作用域的`this`，也就是`greet`这个函数的`this`，就是`jonas`这个 Object。

## Primitives vs. Objects (Primitive vs. Reference Types)

JavaScript 里变量有原始（Primitive）类型（`Number`、`Boolean`等）和引用（ Reference）类型（` Array``Object `等）两种。他们在内存中的储存方式不相同。原始类型储存于 Call Stack，引用类型储存于 Heap 中。

![Primitive-vs-Reference-values](//Images/How-JavaScript-Works-Behind-the-Scenes/Primitive-vs-Reference-values.png)

首先，对于原始类型变量`age`，他储存的实际为 Call Stack 里的一个地址，此处假设为`0001`，这个地址上储存的值为 30。现在让`oldAge`等于`age`，实际是将`oldAge`也指向地址`0001`。

现在修改`age`的值为 31，Call Stack 内会在`0002`处新储存一个 31，然后将`age`的指向改到`0002`。原地址上的值不变，仍为 30

对于引用类型变量`me`，是一个对象。首先会在 Heap 上储存这个对象，地址为`D30F`，然后，`me`指向 Call Stack 地址`0003`，该地址上储存的值，为 Heap 上储存了那个对象的地址，即`D30F`。新变量`friend`等于`me`，其指向的 Heap 地址，仍然为`D30F`。因此，在`me`上修改对象的`age`属性，会在`friend`变量上也看到修改后的结果。

[Learn Modern Javascript (Build and Test Apps) - Full Course | Udemy](https://www.udemy.com/course/the-complete-javascript-course/)
