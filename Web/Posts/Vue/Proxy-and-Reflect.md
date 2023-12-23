---
title: Vue 3 Reactivity - Proxy and Reflect
date: "2022-11-30"
tags: ["vue3", "tutorial", "behine the scene"]
draft: false
summary: "In this article we learn what are Proxy and Reflect and their feature in building Vue3 Reactivity"
---

# Proxy and Reflect

## Our current code

```js
let product = { price: 5, quantity: 2 };
let total = 0;
let effect = () => (total = product.price * product.quantity);

track(product, "quantity");
effect();
console.log(total);

product.quantity = 3;
trigger(product, "quantity");
console.log(total);
```

目前仍然在调用 track 和 trigger 来记录保存我们的 effect，并且调用它。我们希望它会自动调用 track 和 trigger。

我们在运行 effect 的时候，如果访问了 product 的 properties，或者说是用了**GET**，那正是我们想调用**track**去保存 effect 的时候。

当 product 的 properties 改变，或者说是用了**SET**，则调用**trigger**。

```js
// When running an effect if product properties ara accessed (GET)
// then call
track(product, property); // ---> to save this effect
// If product properties are changed (SET)
// then call
trigger(product, property); // ---> to run saved effects
```

问题来到了，如何拦截 SET 和 GET，Vue2 采用 ES5 Object.defineProperty(), Vue3 中使用 ES6 的 Reflect 和 Proxy。

## How to intercept GET&SET?

### Understanding ES6 Reflect

```js
let product = { price: 5, quantity: 2 };
```

这是一个对象，有三个方法访问它。

```js
// Three ways to print out a property on an object
console.log("quantity is " + product.quantity); // Dot notation
console.log("quantity is " + product["quantity"]); // Bracket notation
console.log("quantity is " + Reflect.get(product, "quantity"));
// All three are valid, but Reflect has a super power you'll see in a minute
```

三种都可以，但是 Reflect 拥有一种特殊能力，在这之前先理解**Proxy**。

### Understanding ES6 Proxy

**Proxy 是另一个对象的占位符，默认情况下对该对象进行委托。(a placeholder for another object which by default delegates to that object)**

```js
let product = { price: 5, quantity: 2 };

let proxiedProduct = new Proxy(product, {});

console.log(proxiedProduct.quantity); // proxiedProduct -> product -> proxiedProduct -> console.log
// 2
```

现在声明一个 proxy，当我们调用 proxiedProduct.quantity 时，他会先调用 Proxy(proxiedProduct)，再调用 product，再返回到 Proxy(proxiedProduct)，再返回这个 product 到 console 然后输出。

#### Using a Proxy get trap

声明 Proxy 的时候，第二个参数称为**handler**，再 handler 里可以传递一个**trap**，它可以让我们拦截一些基本操作，例如属性查找、枚举、函数调用（Property lookup、Enumeration、Function invocation）。

```js
proxiedProduct = new Proxy(product, {
  get() {
    console.log("Get was called");
    return "Not the value";
  },
});

console.log(proxiedProduct.quantity);
```

现在调用 proxiedProduct.quantity，他会先打印，Get was called，然后返回 Not the value，再打印。现在修改一下。

#### Intercepting GET using ES Proxy

```js
proxiedProduct = new Proxy(product, {
  get(target, key) {
    console.log("Get was called with key = " + key);
    return target[key];
  },
});

console.log(proxiedProduct.quantity);
// In this case, the target is our product, which we send as the first parameters.
// And the key is quantity
```

在这里，当我们调用 proxiedProduct.quantity 时，传递到 proxiedProduct 的 get，第一个参数 target 即为 product，我们把 product 在声明 Proxy 的时候作为第一个参数传入，第二个参数则为 quantity。

输出 Get was called with key = quantity 和 2。这里 Reflect 就来了。

#### Using Proxy with Reflect

```js
proxiedProduct = new Proxy(product, {
  get(target, key, receiver) {
    console.log("Get was called with key = " + key);
    return Reflect.get(target, key, receiver);
  },
});
```

这里我们给多一个参数 receiver 给 get，同时把它传递给 Reflect。**它保证了当我们的对象有继承自其他对象的值或函数时，this 可以正确的执行使用(的对象)。**(Ensures the proper value of **this** is used when our object has inherited values or functions from another object)打印的结果和以前相同。

#### Now with a Setter Method

现在拦截 set。

```js
proxiedProduct = new Proxy(product, {
  get(target, key, receiver) {
    console.log("Get was called with key = " + key);
    return Reflect.get(target, key, receiver);
  },
  set(target, key, value, receiver) {
    console.log("Set was called with key = " + key + " and value = " + value);
    return Reflect.set(target, key, value, receiver);
  },
});
proxiedProduct.quantity = 4;
console.log(proxiedProduct.quantity);
```

```terminal
Set was called with key = quantity and value = 4
Get was called with key = quantity
4
```

#### Encapsulated like Vue3

现在封装这段 get 和 set 使它变得更新 Vue 3 的源代码

```js
function reactive(target) {
  const handler = {
    get(target, key, receiver) {
      console.log("Get was called with key = " + key);
      return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
      console.log("Set was called with key = " + key + " and value = " + value);
      return Reflect.set(target, key, value, receiver);
    },
  };
  return new Proxy(target, handler);
}

product = reactive({ price: 5, quantity: 2 });
proxiedProduct.quantity = 4;
console.log(proxiedProduct.quantity);
```

取名 reactive，就像 Composition API 一样。

#### Back to our current code

记得开头，为了调用 track，我们需要一些方法监听 GET 请求，调用 trigger，需要监听 SET 请求。现在来看我们应该分把他们放到 reactive 函数的哪个位置。

```js
function reactive(target) {
  const handler = {
    get(target, key, receiver) {
      let result = Reflect.get(target, key, receiver);
      track(target, key);
      return result;
    },
    set(target, key, value, receiver) {
      let oldValue = target[key];
      let result = Reflect.set(target, key, value, receiver);
      if (!oldValue != result) trigger(target, key);
      return result;
    },
  };
  return new Proxy(target, handler);
}

let product = reactive({ price: 5, quantity: 2 });
```

#### Removing track and trigger

```js
let product = reactive({ price: 5, quantity: 2 });
let total = 0;
let effect = () => (total = product.price * product.quantity);

track(product, "quantity");
effect();
console.log(total);

product.quantity = 3;
trigger(product, "quantity");
console.log(total);
```

我们不再需要调用 track 和 trigger 了。删掉

```js
let product = reactive({ price: 5, quantity: 2 });
let total = 0;
let effect = () => (total = product.price * product.quantity);

effect();
console.log(total);

product.quantity = 3;
console.log(total);
```

#### Now with Reactivity

现在从头到尾看一次这段代码执行过程。

首先声明变量。

```js
let product = reactive({ price: 5, quantity: 2 });
let total = 0;
let effect = () => (total = product.price * product.quantity);
```

effect 里，当我们试图得到价格(product.price)时候，运行

```js
track(product, "price");
```

而此时，在 targetMap 里，它会为 product 创建一个新的映射。其值是一个新的 depsMap，这将映射价格属性到一个新的 dep，而这个 dep 就是 effects 集(Set)，把我们所有 effects 添加到集(Set)中。

试图得到 product.quantity 时候，运行

```js
track(product, "quantity");
```

这时他会访问 targetMap 里的 product，找到其 depsMap 建一个新的值 quantity，并映射到一个新的 dep 中。

此时继续往下运行。

```js
effect();
console.log(total);
```

输出 total（10）到控制台。

此时

```js
product.quantity = 3;
```

触发

```js
trigger(product, "quantity");
```

然后运行储存了的 effect。更新了 total

此时 total 成了 15

```js
// All code
const targetMap = new WeakMap(); // For storing the dependencies for each reactive object

function track(target, key) {
  // Add the code
  let depsMap = targetMap.get(target); // Get the current depsMap for this target(reactive object)
  if (!depsMap) targetMap.set(target, (depsMap = new Map())); // If it doesn't exist, create it(product)

  let dep = depsMap.get(key); // Get the dependency object for this property (quantity)
  if (!dep) depsMap.set(key, (dep = new Set())); // If it doesn't exist, create it
  dep.add(effect); // Add the effect to the dependency
}

function trigger(target, key) {
  // Run again all the code.
  const depsMap = targetMap.get(target); // Does the object have any properties that have dependencies?
  if (!depsMap) return; // If no, return from the function immediately
  let dep = depsMap.get(key); // Otherwise, check if this property has a dependency.
  if (dep) dep.forEach((effect) => effect()); // Run those
}
//------------------------------------------------------------------------------------------------------------------------
//------------------------------------------------------------------------------------------------------------------------
//------------------------------------------------------------------------------------------------------------------------
function reactive(target) {
  const handler = {
    get(target, key, receiver) {
      let result = Reflect.get(target, key, receiver);
      track(target, key);
      return result;
    },
    set(target, key, value, receiver) {
      let oldValue = target[key];
      let result = Reflect.set(target, key, value, receiver);
      if (!oldValue != result) trigger(target, key);
      return result;
    },
  };
  return new Proxy(target, handler);
}

let product = reactive({ price: 5, quantity: 2 });
let total = 0;
let effect = () => (total = product.price * product.quantity);

effect();
console.log(total);

product.quantity = 3;

console.log(total);
```

![1](//Images/Vue/Proxy-and-Reflect/How-to-track-deps.png)

[Vue 3 Reactivity - Vue 3 Reactivity | Vue Mastery](https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity/)
