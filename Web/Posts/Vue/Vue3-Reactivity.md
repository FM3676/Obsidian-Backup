---
title: Vue 3 Reactivity
date: "2022-11-30"
tags: ["vue3", "tutorial", "behine the scene"]
draft: false
summary: "In this article we learn how to implement vue3 reactivity"
---

# Vue 3 Reactivity

## A Simple Vue App

```js
var vm = new Vue({
  el: "#app",
  data: {
    price: 5.0,
    quantity: 2,
  },
  computed: {
    totalPriceWithTax() {
      return this.price * this.quantity * 1.03;
    },
  },
});
```

```js
<div id="#app">
    <div>Price: ${{ price }}</div>
    <div>Total: ${{ price * quantity }}</div>
    <div>Taxes: ${{ totalPriceWithTax }}</div>
</div>
```

这是一个 Vue2 的简易 App 模板，当我们的 price 或者 quantity 发生更新的时候，对应的 total 值也会更新，Vue 会对于值的变化做出响应式的处理，这是如何做到的，而且，响应式不是 JavaScript 的工作方式。

```js
let price = 5;
let quantity = 2;
let total = price * quantity;

console.log(total); // 10

price = 20;

console.log(total); // 10
```

total 的值并不会发生更新，所以要如何储存 total 的值和他的计算方式，才能在 price 或者 quantity 发生改变的时候，将 total 值重新计算一次呢？

### How to save the total calculation and run it again when price or quantity updates？

思路如下：

1. 假设一个变量为`x`，它在别的代码中被使用，例如`let a = x + y`，则我们将这条代码储存起来，这条被储存的代码称作**effect**。他会被用于以后当`x`值发生改变时重新计算出新的`a`值。
2. 将`effect`储存起来，并将这条`effect`与`x`关联后储存起来的函数（`x`可能有很多个相关的`effect`，这样所有和`x`值相关的变量都能得到更新）乘坐`track`。
3. 当`x`值变化后，运行所有与`x`相关的`effect`的函数称为`trigger`。

首先，先确认第一步，这次把代码储存在某种 storage 里。

```js
// Save this code to a sort of storage
let total = price * quantity;
```

然后运行这条代码，并且之后，会再次运行这个储存了的代码。并且我们可能不止保存了一种功能(代码)。

所以再次写一遍上面的代码，这次在一个 anonymous function 中计算我们的总数，并把它储存在 `effect` 里。并且这条就是我们要保存的代码。

```js
let price = 5;
let quantity = 2;
let total = 0;
let effect = function () {
  // This is the code we want to save。
  total = prcie * quantity;
};
```

当我们想要保存 `effect` 中的代码的时候（整个 function），我们需要调用 `track`。

```js
let effect = function () {
  // This is the code we want to save。
  total = prcie * quantity;
};
track(); // Save this code
```

然后运行 `effect` 进行首次计算，之后的某个时刻，我们调用 `trigger` 来运行所有储存了的代码。

```js
let effect = function() { // This is the code we want to save。
    total = prcie * quantity;
}
track() // Save this code
effect() // Run this effect
...
trigger() // Run all the code I've saved
```

现在需要储存 `effects`，使用 `dep` 变量，它表示依赖关系，是一个 `Set` 集合，然后为了跟踪(track)我们的依赖，我们将 effect 添加到 Set 中

```js
let dep = new Set(); // To store our effects
let effect = () => {
  total = price * quantity;
};
function track() {
  dep.add(effect);
}
```

> 这里使用 Set，因为 Set 不允许有重复值，如果尝试添加同样的 effect 是无效的

然后 trigger 函数会运行储存了的每一个 effect

```js
function trigger() {
  dep.forEach((effect) => effect());
} // Re-Run all the code in storage
```

```js
// All code
let price = 5;
let quantity = 2;
let total = 0;
let dep = new Set(); // To store our effects
let effect = () => {
  total = price * quantity;
};
function track() {
  dep.add(effect);
}
function trigger() {
  dep.forEach((effect) => effect());
} // Re-Run all the code in storage

track();
effect();
```

**现在，尝试运行这段代码**

设定 `total` 为 10，将 `quantity` 修改为 2 后，查看 `total` 依然会是 10，但如果运行 `trigger` 后就会变成 15

![1](//Images/Vue/Vue3-Reactivity/1.png)

### Often our objects will have multiple properties and each property will need their own dep. How can we store these?

通常，我们的每个对象会有多个属性，每个属性都需要自己的 dep(依赖属性)，或者说 effect 的 Set 集。那我们如何储存，或者说让**每个属性拥有(自己的)依赖？**

```js
let product = { price: 5, quantity: 2 };
```

就和这个 product 一样，每一个属性(price、quantity)都需要自己的依赖 dep，dep 其实就是一个 effect 集(Set)，这个 effect 集应该在值发生改变时重新运行。（A dependency which is a set of effects that should get re-run when values change）。在 dep 这个 Set 集中，每一个值都只是一个我们需要执行的 effect，就像刚才的计算 total 的 anonymous function 一样。我们要将 dep 储存起来蛮方便日后找到他们。

### Use depsMap

创建一个 deps 图(depsMap)，一张储存了每个属性其 dep 对象的图(A map where we store the dependency object for each property)。

在 Map 中有 key 和 value，我们的 key 就是属性(price、quantity)，此时我们的 value 值就是一个 dep。

**现在先创建一个 depsMap**

```js
const depsMap = new Map();
```

我们的 track 函数要拿到这个特定属性的 dep。

```js
function track() {
  let dep = depsMap.get(key); // Get the dep for this property
  if (!dep) depsMap.set(key, (dep = new Set())); // No dep yet, so let's create one
  dep.add(effect); // Add this effect
}
```

在这个例子中，这里的 key 不是 price 就是 quantity。如它还没有 dep，那就建一个 dep，并把它放到 Map 中对应的键值上。

```js
function trigger(key) {
  let dep = depsMap.get(key); // Get the dep for this key
  if (dep) dep.forEach((effect) => effect()); // If it exists, run each effect
}
```

我们的 trigger 函数将先获取这个键的 dep，**如果 dep 存在，就遍历它，并且运行其每个 effect**

现在看回我们的原始代码。

```js
// Original
let product = { price: 5, quantity: 2 };
let total = 0;

let effect = () => {
  total = price * quantity;
};
```

现在，调用我们的 track，它将储存 effect，然后我们运行 effect。

```js
track("quantity");
effect();
```

**现在尝试运行测试这段逻辑代码。**

![2](//Images/Vue/Vue3-Reactivity/2.png)

**很好，现在有方法对不同的属性跟踪依赖。**

### What if we have multiple reactive objects that each need to track effects?

如果我们有多个响应式对象呢？例如再加一个 user object。

```js
let product = { price: 5, quantity: 2 };
let user = { fistName: "Jack", lastName: "James" };
```

现在我们有一个 depsMap，用于储存每个属性自己的依赖对象（属性到自己依赖对象的映射）(A map where we store the dependency object for each property)。其 key 为属性名字(Reactive object’s property name: quantiy、 price)，然后每一个属性都拥有他们自己的，可以重新运行 effect 的 dep(Effect to re-run： total)。

现在我们需要有一个新的储存对象，可能还是一个 Map，它的键以某种方式引用了我们的响应式对象(Where we store the dependencies associated with each reactive objects’s properties)(product、user)。**在 Vue3 中称之为 targetMap，它的 type 是 WeakMap。**

> WeakMap 是一种 Map，但它的键是一个对象，举个例子

```js
const targetMap = new WeakMap();
targetMap.set(product, "example code to test");
console.log(targetMap.get(product));
```

**现在来写完整代码。**

首先创建一个 targetMap，它储存着每个响应式对象的依赖。

```js
const targetMap = new WeakMap(); // For storing the dependencies for each reactive object
```

然后在我们的跟踪方法中，我们要首先拿到这个目标的 deps 图，在这个例子中就是 product，如果不存在就为其创建一个 depsMap。

然后我们将获得这个属性的依赖对象 dep，可能是 quantity。同样的，如果不存在就为它创建一个新的 Set，然后把 effect 添加进去。

```js
function track(target, key) {
  // Add the code
  let depsMap = targetMap.get(target); // Get the current depsMap for this target(reactive object)
  if (!depsMap) targetMap.set(target, (depsMap = new Map())); // If it doesn't exist, create it(product)

  let dep = depsMap.get(key); // Get the dependency object for this property (quantity)
  if (!dep) depsMap.set(key, (dep = new Set())); // If it doesn't exist, create it
  dep.add(effect); // Add the effect to the dependency
}
```

然后 trigger 函数会检查这个 object 是否有“拥有依赖”的属性(Has any properties that have dependencies)，如果没有就可以直接返回。否则检查此属性是否具有依赖。如果有的话，遍历 dep 并运行每个 effect。

```js
function trigger(target, key) {
  // Run again all the code.
  const depsMap = targetMap.get(target); // Does the object have any properties that have dependencies?
  if (!depsMap) return; // If no, return from the function immediately
  let dep = depsMap.get(key); // Otherwise, check if this property has a dependency.
  if (dep) dep.forEach((effect) => effect()); // Run those
}
```

看回原始代码

```js
// Original
let product = { price: 5, quantity: 2 };
let total = 0;

let effect = () => {
  total = price * quantity;
};
```

这次调用 track 的时候传入 product 作为目标 target，和 product 的属性 quantity，并调用 effect()

```js
track(product, "quantity");
effect();
```

![3](//Images/Vue/Vue3-Reactivity/3.png)

这就素全部了。现在我们已经有方法储存 effect，但还是没有办法让它自动运行的。

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

let product = { price: 5, quantity: 2 };
let total = 0;
let effect = () => (total = product.price * product.quantity);

track(product, "quantity");
effect();
```

![4](//Images/Vue/Vue3-Reactivity/Data-Structure-of-Deps-Storage.png)

[Vue 3 Reactivity - Vue 3 Reactivity | Vue Mastery](https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity/)
