---
title: Vue 3 Reactivity - activeEffect & ref
date: '2022-11-30'
tags: ['vue3', 'tutorial', 'behine the scene']
draft: false
summary: 'In this article we learn the principle of activeEffect & ref in Vue3'
---

# activeEffect & ref

## Our Existing Code

```js
let product = reactive({ price: 5, quantity: 2 })
let total = 0
let effect = () => (total = product.price * product.quantity)

effect()
console.log(total)

product.quantity = 3

console.log(total)
```

这是我们现在的最后的代码

### Adding another GET

现在如果加多一个 get，比如 product.quantity

```js
console.log('Updated quantity to = ' + product.quantity)
```

它就会调用 track，然后去找 targetMap 等等，确保当前的 effect 会被记录下来

```js
track(product, 'quantity')
```

这不是我们想要的，我们只应该在 effect 里调用 track 函数（**track** gets called when we GET a property on our reactive object, even if we’re not in an effect, we should only be calling **track** when we’re inside of an effect.）

**也就是说，我们要能够对不同的 property，添加对应他们的 effect，例如这个 effect 和 quantity 有关，那访问 quantity 的时候，把与他有关的，现在正在运行的 effect 进行添加。**

## activeEffect

为此引入一个 activeEffect 变量，它是现在正在运行中的 effect。

```js
let activeEffect = null // The active effect running
```

接下来就是剩余的中间的其他的函数，然后现在来完成 effect 函数，他接受一个 anonymo function，先将其赋值到 activeEffect，然后运行它，最后再复位。

```js
function effect(eff) {
  activeEffect = eff // Set this as the activeEffect
  activeEffect() // Run it
  activeEffect = null // Unset it
}
```

然后我们用这个函数来计算 total，然后删除原先单独调用的 effect 函数。

```js
let product = reactive({ price: 5, quantity: 2 })
let total = 0
effect(() => (total = product.price * product.quantity))

// effect() ----> No longer need to call the effect, it's getting called when we send our function in.
// ....
```

现在需要更新 track 函数，让它去使用这个新的 activeEffect。现在回到 track 函数。

### Update track to use activeEffect

```js
const targetMap = new WeakMap()
let activeEffect = null

function track(target, key) {
  let depsMap = targetMap.get(target)
  if (!depsMap) targetMap.set(target, (depsMap = new Map()))

  let dep = depsMap.get(key)
  if (!dep) depsMap.set(key, (dep = new Set()))
  dep.add(effect)
}
```

一共要做两处修改

首先，我们只想在我们有 activeEffect 的时候运行这段代码，其次，当我们添加依赖时，我们要添加 activeEffect

```js
function track(target, key) {
  if (activeEffect) {
    // Only track if there is an activeEffect
    // ....
    dep.add(activeEffect)
  }
}
```

现在测试，简单修改测试代码

```js
let product = reactive({ price: 5, quantity: 2 })
let salePrice = 0
let total = 0

effect(() => (total = product.price * product.quantity))
effect(() => (salePrice = product.price * 0.9))

console.log(
  `Before updated total (should be 10 ) = ${total} salePrice (should be 4.5) = ${salePrice}`
)
product.quantity = 3

console.log(
  `After updated total (should be 15 ) = ${total} salePrice (should be 4.5) = ${salePrice}`
)
product.price = 10
console.log(`After updated total (should be 30 ) = ${total} salePrice (should be 9) = ${salePrice}`)
```

运行这段代码，当运行到 quantity = 3 时， 他应该运行第一个 effect，当运行到 price = 10 的时候，他应该运行两个 effect。

## Making salePrice Reactive

这段代码没有什么意义，我们为什么不用 salePrice 去计算 total 呢？

```js
effect(() => (total = salePrice * product.quantity))
effect(() => (salePrice = product.price * 0.9))
```

这不会生效，因为 salePrice 发生变化的时候，运算 total 的第一个 effect 并不会运行。salePrice 并不是响应式的。（When **salePrice** is set, we’d need to rerun the **total**. But **salePrice** isn’t reactive!）

## Using a refe on a single value

这里我们就可以开始使用 ref 了，它接受一个值并返回一个相应的，可变的 Ref 对象，其只有一个属性 value，指向内部的值本身。（Takes an inner value and returns a reactive and mutable **ref** object. The **ref** object has a single property **.value** that points to the inner value.）

```js
effect(() => (total = salePrice.value * product.quantity))
effect(() => (salePrice.value = product.price * 0.9))
```

## How to defin ref?

如何定义 ref 呢？

### Use Reactive

我们可以直接使用 reactive。

```js
// Use ractive
function ref(initialValue) {
  return reactive({ value: initialValue })
}
```

这可行，但 Vue3 不是这样做的。

### Use Object Accessors (aka. computed properties)

先了解什么时对象访问器（Object Accessors），也称之为计算属性（computed），**注意这不是 Vue 里的计算属性，而是 JavaScript 里的计算属性**
先看个例子

#### A simple example

```js
let user = {
  firstName: 'Gregg',
  lastName: 'Pollack',
  // Object Accessors are functions that get or set a value

  get fullName() {
    return `${this.firstName} ${this.lastName}`
  },

  set fullName(value) {
    ;[this.fullName, this.lastName] = value.split(' ')
  },
}

console.log(`Name is ${user.fullName}`)
user.fullName = 'Adam Jahr'
console.log(`Name is ${user.fullName}`)
```

对象访问器就是获取和设置值的函数，这就是我们如何使用 getter 和 setter 的方法。

现在利用这个来定义 ref 函数。

```js
// Use Object Accessors (aka. computed properties)
function ref(raw) {
  const r = {
    get value() {
      track(r, 'value')
      return raw
    },
    set value(newVal) {
      raw = newVal
      trigger(r, 'value')
    },
  }
  return r
}
```

这就是全部了。

## Testing

现在来测试一下

```js
let product = reactive({ price: 5, quantity: 2 })
let salePrice = 0
let total = 0

effect(() => (total = salePrice.value * product.quantity))
effect(() => (salePrice.value = product.price * 0.9))

console.log(
  `Before updated total (should be 10 ) = ${total} salePrice (should be 4.5) = ${salePrice}`
)
product.quantity = 3

console.log(
  `After updated total (should be 13.5 ) = ${total} salePrice (should be 4.5) = ${salePrice}`
)
product.price = 10
console.log(`After updated total (should be 27 ) = ${total} salePrice (should be 9) = ${salePrice}`)
```

计算 quantity = 3 的时候，运行第一个 effect 来更新 total。

当更新价格的时候，它会运行 effect 更新 salePrice。这个时候，salePrice 更新触发 effect 运算新的 total。

这就是全部了。

> [3 - activeEffect & ref\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1SZ4y1x7a9?p=3&vd_source=89f1af483c0e88361370c1c5c633d220)

 [Vue 3 Reactivity - Vue 3 Reactivity | Vue Mastery](https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity/) 