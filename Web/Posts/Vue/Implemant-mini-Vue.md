---
title: Implement Mini Vue3
date: '2022-11-30'
tags: ['vue3', 'tutorial', 'behine the scene']
draft: false
summary: 'The code snippets of implementing mini Vue3 step by step'
---

# Implement Mini Vue3

## Monut Function

```js
function h(tag, props, children) {
  return {
    tag,
    props,
    children,
  }
}

function mount(vnode, container) {
  const el = (vnode.el = document.createElement(vnode.tag))
  // Props & Attributes
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key]
      el.setAttribute(
        /* For simplified, assume that all the props are attributes. */
        key,
        value
      )
    }
  }

  // Childaren
  if (vnode.children) {
    if (typeof vnode.children === 'string') {
      el.textContent = vnode.children
    } else {
      vnode.children.forEach((child) => {
        mount(child, el)
      })
    }
  }
  container.appendChild(el)
}
const vdom = h('div', { class: 'red' }, [h('span', null, 'hello')])
mount(vdom, document.getElementById('app'))

/* To figure out what is the minimal amount of DOM manipulations that it needs to perform to update our screen to the desired state.*/
function patch(n1 /* Previous Snapshot */, n2 /* New Snapshot */) {}

const vdom2 = h('div', { class: 'green' }, [h('span', null, 'hello')])
patch(vodm, vdom2)
```

## Path Function

```js
function h(tag, props, children) {
  return {
    tag,
    props,
    children,
  }
}

function mount(vnode, container) {
  const el = (vnode.el = document.createElement(vnode.tag))
  // Props & Attributes
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key]
      el.setAttribute(
        /* For simplified, assume that all the props are attributes. */
        key,
        value
      )
    }
  }

  // Childaren
  if (vnode.children) {
    if (typeof vnode.children === 'string') {
      el.textContent = vnode.children
    } else {
      vnode.children.forEach((child) => {
        mount(child, el)
      })
    }
  }
  container.appendChild(el)
}
const vdom = h('div', { class: 'red' }, [h('span', null, 'hello')])
mount(vdom, document.getElementById('app'))

/* -------------------------------------------------------------------------------------------------------------------------------------------- */
/* To figure out what is the minimal amount of DOM manipulations that it needs to perform to update our screen to the desired state.*/
function patch(n1 /* Previous Snapshot */, n2 /* New Snapshot */) {
  // If they are the same node
  if (n1.tag === n2.tag) {
    const el = (n2.el = n1.el)
    // Check the props
    const oldProps = n1.props || {}
    const newProps = n2.props || {}

    // Props change
    for (const key in newProps) {
      const oldValue = oldProps[key]
      const newValue = newProps[key]
      if (newValue !== oldValue) {
        el.setAttribute(key, newValue)
      }
    }

    // Props removed
    for (const key in oldProps) {
      if (!key in newProps) {
        el.removeAttribute(key)
      }
    }

    // Children
    const oldChildren = n1.children
    const newChildren = n2.children
    // Children both are string
    if (typeof newChildren === 'string') {
      if (typeof oldChildren === 'string') {
        if (newChildren !== oldChildren) {
          el.textContent = newChildren
        }
      } else {
        el.textContent = newChildren
      }
    } else {
      // New is Array
      if (typeof oldChildren === 'string') {
        el.innerHTML = ''
        newChildren.forEach((child) => mount(child, el))
      } else {
        // Both are Array
        /* Vue acutally has two modes to handle it, a list loop render with key and the normal one. Here is the normal one */
        // Assume that nothing is keyed
        const commonLength = Math.min(oldChildren.length, newChildren.length)
        for (let i = 0; i < commonLength; i++) {
          patch(oldChildren[i], newChildren[i])
        }
        if (newChildren.length > oldChildren.length) {
          newChildren.slice(oldChildren.length).forEach((child) => mount(child, el))
        } else if (newChildren.length < oldChildren.length) {
          oldChildren.slice(newChildren.length).forEach((child) => el.removeChild(child.el))
        }
      }
    }
  } else {
    // replace
  }
}

const vdom2 = h('div', { class: 'green' }, [h('span', null, 'changed!')])
patch(vdom, vdom2)
```

## Deps

```js
let activeEffect = null
class Dep {
  constructor(value) {
    this.subscribers = new Set()
    this._value = value
  }
  get value() {
    this.depend()
    return this._value
  }
  set value(newValue) {
    this._value = newValue
    this.notify()
  }
  depend() {
    if (activeEffect) this.subscribers.add(activeEffect)
  }
  notify() {
    this.subscribers.forEach((effect) => effect())
  }
}

function watchEffect(effect) {
  activeEffect = effect
  effect()
  activeEffect = null
}

// const dep = new Dep("hello");
const dep = new Dep(1)

watchEffect(() => {
  // dep.depend();
  console.log(dep.value)
}) // effect run

// dep.value = "changed";
dep.value++
dep.value++
dep.value++
dep.value++
dep.value++
//   dep.notify(); // effect run
```

## Reactive

```js
let activeEffect
class Dep {
  subscribers = new Set()
  depend() {
    if (activeEffect) this.subscribers.add(activeEffect)
  }
  notify() {
    this.subscribers.forEach((effect) => {
      effect()
    })
  }
}

/* ----------------------------------------------------------------------------------------------------------------------- */

// Vue2 Version
function reactiveV2(raw) {
  Object.keys(RAW).forEach((key) => {
    const dep = new Dep()
    let value = raw[key]

    Object.defineProperty(raw, key, {
      get() {
        dep.depend()
        return value
      },
      set(newValue) {
        value = newValue
        dep.notify()
      },
    })
  })
  return raw
  /* The properties that new added in is not reactive, because we didn;t set the getter and setter on those properties */
}

// Vue3 Version

const targetMap = new WeakMap()

function getDep(target, key) {
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    depsMap = new Map()
    targetMap.set(target, depsMap)
  }
  let dep = depsMap.get(key)
  if (!dep) {
    dep = new Dep()
    depsMap.set(key, dep)
  }
  return dep
}

const reactiveHandlers = {
  get(target, key, receiver) {
    const dep = getDep(target, key)
    dep.depend()
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    const dep = getDep(target, key)
    // In Proxy, the set needs to return a Boolean to indeicate whether this operation is has succeed or not.
    const result = Reflect.set(target, key, value, receiver)
    dep.notify()
    return result
  },
}

function reactiveV3(raw) {
  return new Proxy(raw, reactiveHandlers)
}

/* ----------------------------------------------------------------------------------------------------------------------- */

function watchEffect(effect) {
  activeEffect = effect
  effect()
  activeEffect = null
}

const state = reactiveV3({
  count: 0,
})

watchEffect(() => {
  console.log(state.count)
}) // 0

state.count++
```

## Mini Vue

```js
// v-dom
function h(tag, props, children) {
  return {
    tag,
    props,
    children,
  }
}

function mount(vnode, container) {
  const el = (vnode.el = document.createElement(vnode.tag))
  // Props & Attributes
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key]
      if (key.startsWith('on')) {
        el.addEventListener(key.slice(2).toLowerCase(), value)
      } else {
        el.setAttribute(
          /* For simplified, assume that all the props are attributes. */
          key,
          value
        )
      }
    }
  }

  // Childaren
  if (vnode.children) {
    if (typeof vnode.children === 'string') {
      el.textContent = vnode.children
    } else {
      vnode.children.forEach((child) => {
        mount(child, el)
      })
    }
  }
  container.appendChild(el)
}

/* -------------------------------------------------------------------------------------------------------------------------------------------- */
/* To figure out what is the minimal amount of DOM manipulations that it needs to perform to update our screen to the desired state.*/
function patch(n1 /* Previous Snapshot */, n2 /* New Snapshot */) {
  // If they are the same node
  if (n1.tag === n2.tag) {
    const el = (n2.el = n1.el)
    // Check the props
    const oldProps = n1.props || {}
    const newProps = n2.props || {}

    // Props change
    for (const key in newProps) {
      const oldValue = oldProps[key]
      const newValue = newProps[key]
      if (newValue !== oldValue) {
        el.setAttribute(key, newValue)
      }
    }

    // Props removed
    for (const key in oldProps) {
      if (!key in newProps) {
        el.removeAttribute(key)
      }
    }

    // Children
    const oldChildren = n1.children
    const newChildren = n2.children
    // Children both are string
    if (typeof newChildren === 'string') {
      if (typeof oldChildren === 'string') {
        if (newChildren !== oldChildren) {
          el.textContent = newChildren
        }
      } else {
        el.textContent = newChildren
      }
    } else {
      // New is Array
      if (typeof oldChildren === 'string') {
        el.innerHTML = ''
        newChildren.forEach((child) => mount(child, el))
      } else {
        // Both are Array
        /* Vue acutally has two modes to handle it, a list loop render with key and the normal one. 
          Here is the normal one */
        // Assume that nothing is keyed
        const commonLength = Math.min(oldChildren.length, newChildren.length)
        for (let i = 0; i < commonLength; i++) {
          patch(oldChildren[i], newChildren[i])
        }
        if (newChildren.length > oldChildren.length) {
          newChildren.slice(oldChildren.length).forEach((child) => mount(child, el))
        } else if (newChildren.length < oldChildren.length) {
          oldChildren.slice(newChildren.length).forEach((child) => el.removeChild(child.el))
        }
      }
    }
  } else {
    // replace
  }
}

// Reactivity
let activeEffect = null
class Dep {
  subscribers = new Set()
  depend() {
    if (activeEffect) this.subscribers.add(activeEffect)
    console.log(this.subscribers)
  }
  notify() {
    this.subscribers.forEach((effect) => effect())
  }
}

const targetMap = new WeakMap()

function getDep(target, key) {
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    depsMap = new Map()
    targetMap.set(target, depsMap)
  }
  let dep = depsMap.get(key)
  if (!dep) {
    dep = new Dep()
    depsMap.set(key, dep)
  }
  return dep
}

const reactiveHandlers = {
  get(target, key, receiver) {
    const dep = getDep(target, key)
    dep.depend()
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    const dep = getDep(target, key)
    // In Proxy, the set needs to return a Boolean to indeicate whether this operation is has succeed or not.
    const result = Reflect.set(target, key, value, receiver)
    dep.notify()
    return result
  },
}

function reactive(raw) {
  return new Proxy(raw, reactiveHandlers)
}

function watchEffect(effect) {
  activeEffect = effect
  effect()
  activeEffect = null
}

/* ----------------------------------------------------------------------------------------------------------------------- */

const App = {
  data: reactive({
    count: 0,
  }),
  render() {
    return h(
      'div',
      {
        onClick: () => this.data.count++,
      },
      String(this.data.count)
    )
  },
}

function mountApp(component, container) {
  let isMounted = false
  let prevVdom
  watchEffect(() => {
    console.log(isMounted)
    if (!isMounted) {
      prevVdom = component.render()
      mount(prevVdom, container)
      isMounted = true
    } else {
      const newVdom = component.render()
      patch(prevVdom, newVdom)
      prevVdom = newVdom
    }
  })
}

mountApp(App, document.getElementById('app'))
```
