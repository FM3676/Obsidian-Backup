---
title: Vue 3 Reactivity - Computed & Vue3 Source
date: '2022-11-30'
tags: ['vue3', 'tutorial', 'behine the scene']
draft: false
summary: 'In this article we learn the principle of Computed of and the Vue3 Source Code'
---

# Computed & Vue3 Source

## Our Existing Code

```js
let product = reactive({ price: 5, quantity: 2 })
let salePrice = 0
let total = 0

effect(() => (total = salePrice.value * product.quantity))
effect(() => (salePrice.value = product.price * 0.9))
```

这是我们现在的代码，可以看到我们用了两个 effect，如果这是 Vue3，为什么不使用 computed 呢？

## Now with Computed Properties

```js
let product = reactive({ price: 5, quantity: 2 })
let salePrice = computed(() => (total = salePrice.value * product.quantity))
let total = computed(() => (salePrice.value = product.price * 0.9))
```

如何定义 computed 方法？他让对象也同样是响应式的，和 ref 一样

## Defining Computed Property

首先可以明确，computed 要接受一个参数 getter，作为初始值。

其次他要创建一个响应式的值，叫 result。

在 effect 中运行 getter，因为我们需要监听响应值。

最后返回 result。

```js
function computed(getter) {
  // Create a reactive reference called result
  let result = ref()
  // Run the getter in an effect() whichsets the result.value
  effect(() => (result.value = getter()))
  // Return result
  return result
}
```

```js
let product = reactive({ price: 5, quantity: 2 })
let salePrice = computed(() => (total = salePrice.value * product.quantity))
let total = computed(() => (salePrice.value = product.price * 0.9))

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

现在我们重新测试，这一段测试代码的输出内容应该和之前相同，我们的 computed 属性应该要正常运行。

[Vue 3 Reactivity - Vue 3 Reactivity | Vue Mastery](https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity/)
