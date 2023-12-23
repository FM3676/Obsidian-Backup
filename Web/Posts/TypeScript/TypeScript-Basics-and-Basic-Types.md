---
title: TypeScript Basics & Basic Types
date: '2022-10-14'
tags: ['TypeScript', 'guide']
draft: false
summary: 'In this article we learn the basic of TypeScript and its basic types.'
---

# TypeScript Basics & Basic Types

## Using Types

一个简单的 JS 函数

```js
const add = (a, b) => a + b
```

正常情况下，他返回两个数字的和，但如果有一个数字以字符串形式输入，返回的结果则会是字符串的拼接

```js
console.log(add(5, 2.8)) // 7.8
console.log(add('5', 2.8)) // 52.8
```

使用 TS 在编写时就发现这个问题，而不是在编译的时候才发现问题。通过在参数后加`:`，再加上其限定类型即可。

```ts
const add = (a: number, b: number) => a + b
console.log(add('5', 2.8)) // error
```

## TypeScript Types vs JavaScript Types

JavaScript 中也有 typeof 操作符来检查参数的类型，就像这样

```js
const add = (a: number, b: number) =>
  typeof a !== 'number' || typeof b !== 'number' ? throw new Error('Incorrect input!') : a + b
```

这样同样可以做类型检查，但是这样很复杂，同时，依然无法在 Coding 的时候发现问题，只有当编译错误的时候，才会发现参数的类型错误，使用 TypeScript 规避这样的麻烦和错误，同时 TypeScript 拥有的类型也远远超过 JavaScript。

## Working with Numbers, Strings & Booleans

现在来进一步扩充我们的 `add` 函数，添加一个参数 `showResult`，类型为 `boolean`，来决定函数执行完后是答应结果还是返回结果。再增加一个 `phrase` 参数，类型为 `string`，如果要直接打印结果，前面想添加的输出内容作为 phrase 参数。

```ts
const add = (n1: number, n2: number, showResult: boolean, phrase: string) =>
  showResult ? console.log(phrase + n1 + n2) : n1 + n2

const number1 = 5 // 5.0
const number2 = 2.8
const printResult = true
const resultPhrase = 'Result is: '

add(number1, number2, printResult, resultPhrase)
```

## Type Assignment & Type Inference

TypeScript 代码会经过 Compiler 编译后变为绝大多数浏览器都可以理解的 JavaScript 代码。而且里面不带有类型推断，因为 JS 并没有这个能力。
TS 会对变量进行类型推断。
例如

```ts
const number = 5
```

如果鼠标放在 number 上，会看到显示 `const number: 5`，这是因为 number 这个时候是个常量，他会把它看作就是 5。如果使用 `let` 声明呢？

```ts
let number: number
number = 5
let n = 5
```

鼠标放在 number 和 n 上，就会显示 `let number: number`、`let n: number`，因为我们声明 `number` 是一个 number 类型，同时 TypeScript 会推断 `n` 也是 number 类型。

```ts
n = '5'
number = '5'
```

这种行为会报错，因为我们已经声明这两个变量是 `number` 类型，不可以被赋值为 string 类型。

## Object Types

对于 Object 如何处理？

```ts
const person = {
  name: 'jack',
  age: 30,
}
```

这是一个 object，可以使用点操作符访问内部键值对。鼠标放在 `person` 上，会出现它的类型。

```ts
const person: {
  name: string
  age: number
}
```

这看起来也像一个 object，但是不是，**每个键值对的分割使用的是';'而不是','，**而这就是 person 在 TS 里的类型。

如何确定类型呢

```ts
const person: object = {
  //...
}
```

如果这样子赋类型，我们是无法使用点操作符访问内部对象的，typesript 会认为这只是一个最简单的 object 类型，就是 JavaScript 里的与 number、boolean 齐名的 object 类型。

```ts
const person: {
  name: string
  age: number
} = {
  name: 'jack',
  age: 30,
}
```

采用这种方式，对 object 里面每一个键的值声明清楚其类型即可。

## Array Types

数组作为 JavaScript 里就存在的引用类型，也可以在 TypeScript 中做限制。

```ts
const person = {
  name: 'jack',
  age: 30,
  hobbies: ['Sports', 'Cooking'],
}
```

这个时候去看 hobbies 的类型。显示的是 `(property) hobbies: string[]`。这表明他是一个有 string 组成的 Array。
若要声明一个有 string 组成的 Array，可以这样

```ts
let favouriteActivites: string[]
favouriteActivites = ['Sports']
```

我们可以遍历这个数组看看

```ts
for (const hobby of person.hobbies) {
  console.log(hobby.toUpperCase())
  console.log(hobby.map()) //---> error: TS knows that the string type doesn't have the method 'map'
}
```

但是这样会报错，在遍历的时候，TypeScript 知道 hobbies 数组里面的值类型为 hobby，所以当我们在 hobby 下调用点操作符时，会为我们显示属于 string 的方法。但是 `map` 方法不会，他知道 string 类型是不会有 `map` 方法的

## Working with Tuple

Tuple 是 TypeScript 提供的一种类型，他像是一个数组，但是它的**长度，每一个位置的值的类型都是固定的。**

```ts
const person: {
  // ....
  role: [number, string]
} = {
  //....
  role: [2, 'author'],
}
```

现在已经声明好了 `role` 作为一个 tuple，他长度只能是 2，第一个必须是 number 且第二个一定是 string。如下的代码会报错，因为类型不符合。同样的，超出长度的赋值也是会报错的。

```ts
person.role[1] = 10
person.role = [0, 'admin', 'user']
```

但是很可惜，对于 `push` 这样的操作，TypeScript 无法检测推入的长度是否会超出，以及推入的这一位，是否是这一位应该为的类型。(即通过 `push` 在第二个位置推入一个 number)。

```ts
person.role.push('admin')
```

## Working with Enums

TypeScript 提供了枚举 Enum 类型。
在 JavaScript 中会遇到一些问题，给一些东西赋固定值的时候，会输入错具体的值，使得程序无法正常运行，如下：

```js
const person = {
  role: 'READ ONLY',
}

if (person.role === 'READ-ONLY') console.log('Read only.')
```

一个字符之差，导致这条信息永远不被输出，所以我们有其他的解决方案

```js
const ADMIN = 'ADMIN'
const READ_ONLY = 'READ_ONLY'
const person = {
  role: READ_ONLY,
}

if (person.role === READ_ONLY) console.log('Read only.')
```

这可以规避上述的错误，但依然略显繁琐，可以使用 TypeScript 的 enmu 类型。

```ts
enum Role {
  ADMIN,
  READ_ONLY,
  AUTHOR,
}

const person = {
  // ...
  role: Role.ADMIN,
}

if (person.role === Role.ADMIN) console.log('I am Admin')
```

这样又方便而且不会出错，类型分明。同时，如果把鼠标移到 Role 里每个项的上，可以看到他们的具体值。被分别按顺序赋予了 1、2、3。
如果想要改变他们的初始值也可以，例如

```ts
enum Role {
  ADMIN = 200,
  READ_ONLY,
  AUTHOR,
}
```

这样 ADMIN 的值会是 200，剩余两个也是以此增加，分别为 201 和 202。
当然也可以赋予其他的类型。

```ts
enum Role {
  ADMIN = 'ADMIN',
  READ_ONLY = 100,
  AUTHOR = 'AUTHOR',
}
```

这同样可行。

## The any Type

any 告诉 TypeScript 这个值的类型可以为任何，例如

```ts
const a = any[];
```

这个数组里面的值可以是任何类型，没有限制。
**尽量避免使用 any，TypeScript 没有办法对他做任何检查，any 用的多和写 JavaScript 没有区别**

## Union Types

有时候我们希望我们的函数有更多的功能，接收的参数类型不同，返回的结果也不同，这个时候可以使用 Union Type

```ts
const combine = (a: number | string, b: number | string) =>
  typeof a === 'number' && typeof b === 'number' ? a + b : a.toString() + b.toString

const combineAges = combine(30, 26)
console.log(combineAges)

const combineNames = combine('Du', 'Lang')
console.log(combineNames)
```

在这个例子中，combine 函数的参数接收两种类型的参数，string 和 number，如果两个参数都为 number 类型，那就对他们做简单的相加，否则就做及简单的字符串拼接。
使用 Union type 我们限制了参数的类型的同时，扩充函数的功能

## Literal Types

Literal type 为写死的内容。例如之前的

```js
const number = 5
```

查看它的类型会发现它就只写了一个 5，我们可以利用这个特性。

```ts
const combine = (
  a: number | string,
  b: number | string,
  resultConversion: 'as-number' | 'as-text'
) =>
  (typeof a === 'number' && typeof b === 'number') || resultConversion === 'as-number'
    ? +a + +b
    : a.toString() + b.toString

const combineAges = combine(30, 26, 'as-number')
console.log(combineAges)

const combineNames = combine('Du', 'Lang', 'as-text')
console.log(combineNames)
```

这个时候，第三个参数 resultConvension 的值就只能为二者其一，被严格限定，避免出现错误。
如果传的参数或者在函数内判断其的值不为这二者其一，TypeScript 都会报错，从而规避错误。

## Type Aliases Custom Types

类型别名，使用 type 操作符定义。

```ts
type Combinable = number | string
type ConversionDescriptor = 'as-number' | 'as-text'

const combine = (a: Combinable, b: Combinable, resultConversion: ConversionDescriptor) =>
  (typeof a === 'number' && typeof b === 'number') || resultConversion === 'as-number'
    ? +a + +b
    : a.toString() + b.toString
```

如果我们的参数类型复杂，这样会更清晰，同时也更能明确为什么是这些类型。也可以编码更加复杂的类型定义。并且有着良好的复用性。

## Function Return Types && void

我们可以声明 function 的返回类型。先来看看

```ts
const add = (a: number, b: number) => a + b
```

鼠标放大 add 上 会显示 `const add: (a: number, b: number) => number`，箭头后面的 number 表面它会返回一个 number 类型的值，这是由 TypeScript 自己推断出来的。我们也可以显式的声明

```ts
const add = (a: number, b: number): number => a + b
```

也是通过 `:` 来做声明，**如果你不知道返回类型究竟是什么，让 TypeScript 自己推断就好。**

对于不返回东西的 function，也提供了 void 类型，就和其它语言一样。

```ts
const printResult = (n: number): void => console.log('Result: ' + n)
```

这里即使不写清楚 void，TypeScript 也会自己推断出这是 void 类型。
还有另一种返回类型，undefined，他会写 return 语句但是不不返回任何东西

```ts
function returnUndefined(): undefined {
  return
}
```

## Functions as Types

我们有时候也会希望函数本身可以作为一种类型，或者特定的一种函数，作为一种类型。

```ts
const add = (a: number, b: number): number => a + b

let combineFunction
combineFunction = add
```

`combineFunction` 的类型是 type 是 any，所以可以赋值为 add。
我们也可以限定其类型为 Function。

```ts
let combineFunction: Funtcion
```

但这样太宽泛了,它可以被赋值为任意一个函数都可以。可以进一步限制。

```ts
const add = (a: number, b: number): number => a + b

const printResult = (n: number): void => console.log('Result: ' + n)

let combineFunction: (a: number, b: number) => number

combineFunction = add
// combineFunction = printResult;
```

这个时候，`combineFunction` 就只可以被赋值为 add 了，因为他接受两个参数，类型为 number，且返回一个 number 值。

## Functions Types & Callback

有时候需要传入回调函数，我们也可以对回调函数，作出一定的限制

```ts
const add = (a: number, b: number, callback: (n: number) => number): number => callback(a + b)

add(1, 2, (num) => num * 10)
```

这样我们就限定了，回到函数的类型，参数类型，和返回类型。

## Unknown Type

如果不知道你要储存的变量的类型，但你知道你最终要用他做什么，使用 `unknown` 类型

```ts
let userInput: unknown
let userName: string

userInput = 5
userInput = 'Max'

if (typeof userInput === 'string') userName = userInput
```

可以看到，unknown 类型可以被赋值为任何值，类似 unknown。但不能随意给其他变量赋值，可以在判断类型后赋值，否则会报错。
**同样，尽量不要用 unknown 类型，如果知道有哪些类型，union 是更好的选择。**

## Never Type

还有一种返回类型，never。他表示从不返回任何值，例如抛出异常或者其他的等等。

```ts
function generateError(msg: string, code: number): never {
  throw { msg, code }
}

generateError('An error', 500)
```
