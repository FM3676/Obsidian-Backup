---
title: Generics
date: '2022-10-15'
tags: ['TypeScript', 'guide']
draft: false
summary: 'In this article we learn the Generics of TypeScript.'
---

# Generics

## Built-in Genrics & What are Generics

当我们定义一个变量不确定类型的时候有两种解决方式：any 或 泛型。
先来看看内建的默认的泛型，Array。

```ts
const names = ["Jack"]; // type string[]
// ---------------------------------------------------------
const names = []; // type any[]
```

我们上下两种都是建立数组的方法，他们现在的类型会变得不同。如果我们给他指明类型呢？指明它就是 Array

```ts
const names: Array = [];
```

这会报错，提示`Generic type 'Array<T>' requires 1 type arguments(s)`，即泛型类型`Array<T>`需要 1 个类型参数。
当出现`<>`的时候，说明我们正在处理一个泛型。

**泛型指的是在定义函数/接口/类型时，不预先指定具体的类型，而是在使用的时候在指定类型限制的一种特性。**

在这里，数组本身就是一种类型。但是，数组本身其实并不关心什么东西存在里面。但我们任然要说明，会存什么类型的东西进去。

```ts
const names: Array<string> = []; // string[]
names[0].split(" ");
```

这个时候，其实和直接指定类型为 `string[]`是一样的。这个时候，我们可以在它的 `item` 上调用 `split` 了，因为我们知道他肯定是 string。

另一个内建泛型就是 Promise。

```ts
const promise = new Promise((res, rej) => {
  setTimeout(() => {
    res("Resolved!");
  }, 2000);
});
```

这个时候去看 promise 的类型，会发现为`const promise: Promise<unknown>`，那么这有什么用呢？
现在我们限定它返回的为 string。

```ts
const promise: Promise<string> = new Promise((res, rej) => {
  setTimeout(() => {
    res("Resolved!");
  }, 2000);
});

promise.then((res) => res.split(" "));
```

我们在接下来的 `then` 调用的时候，就可以获得更好的对数据处理的支持。TypeScript 和编辑器知道我们返回的是 string 类型，所以我们可以使用 split 方法。

## Creating a Generic Function

先来看看正常情况会有什么问题。

```ts
function merge(objA: object, objB: object) {
  return Object.assign(objA, objB);
}

const mergeObj = merge({ name: "Jack" }, { age: 30 });

console.log(mergeObj.name);
```

在这里，`mergeObj.name` 会报错，因为我们虽然合并了两个对象，但是 TypeScript 还是不知道，`mergeObj` 会有 name 属性，除非加上一句：

```ts
const mergeObj = merge({ name: "Jack" }, { age: 30 }) as {
  name: string;
  age: number;
};
```

所以我们需要泛型。

```ts
function merge<T, U>(objA: T, objB: U) {
  return Object.assign(objA, objB);
}
```

在这里，我们在函数名字后面加上一个`<>`，第一个写个 T（不一定是 T，但通常是），第二个写个 U，因为我们有这两个参数。随后在两个参数后面分别指定类型为 T 和 U。

**此处如果报错，可以写 `<T extends object, U extends object> `是因为 assign 方法需要确定你是一个 object，否则可以不写。**

此时我们来看看函数的类型定义。

`function merge<T extends object, U extends object>(objA: T, objB: U): T & U`
可以看到，TypeScript 推断出这个函数返回的将是两个 object 的结合。我们就可以正常访问属性了。

mergeObj 的 type 此时为
`const mergeObj: {name: string;} & { age: number; }`
当然我们也可以明确指明参数类型，但这是多此一举。

```ts
const mergeObj = merge<{ name: string }, { age: number }>(
  { name: "Jack" },
  { age: 30 }
);
```

## Working with Constrains

对于泛型中的类型，我们也可以做约束。
例如，对于刚才，如果我们这样定义 `mergeObj`，age 就没有办法储存。

```ts
const mergeObj = merge({ name: "Jack" }, 30); // {name: 'Jack'}
```

所以我们可以做类型约束，我们希望我们的两个参数，一定为 object。

```ts
function merge<T extends object, U extends object>(objA: T, objB: U) {
  return Object.assign(objA, objB);
}
```

此时我们就不能传一个数字作为参数了，而必须是一个对象才可以。

## Another Generics Function

再做一个泛型函数，这次我们只希望，参数他一定要包含某种属性即可。

```ts
interface Lengthy {
  length: number;
}

function countAndDescribe<T extends Lengthy>(element: T): [T, string] {
  let descriptionText =
    element.length > 0 ? `Got ${element.length} elements.` : "Got no value.";
  return [element, descriptionText];
}

console.log(countAndDescribe(["Sports"]));
console.log(countAndDescribe("I am a string!"));
```

这里我们定义了一个 interface，然后使用 extends 希望参数一定拥有这个 interface 里声明的属性（此处为 length）。
可以看到，在使用的时候，可以传入 string，也可以穿一个 Array，因为他们都具有 length 属性。

## The keyof Constraint

```ts
function extractAndConvert(obj: object, key: string) {
  return obj[key];
}
extractAndConvert({}, "name");
```

这个函数会有潜在的问题，如果我们传入的 key 写错了或者其他原因，并不是 obj 内部拥有的 key 怎么办？

```ts
function extractAndConvert<T extends object, U extends keyof T>(
  obj: T,
  key: U
) {
  return obj[key];
}

extractAndConvert({ name: "Jack" }, "name");
```

这里使用了 keyof 关键字，它告诉 TypeScript，这个参数类型一定会是 T 这个 obj 的 key。所以，我们在使用的时候，第一个参数的 object 如果没有 name 这个 key，会报错。

## Generics Classes

Class 也同样具有泛型

```ts
class DataStorage {
  private data = [];

  addItem(item) {
    this.data.push(item);
  }

  removeItem(item) {
    this.data.splice(this.data.indexOf(item), 1);
  }
  getItems() {
    return [...this.data];
  }
}
```

我们希望这个类里的 data 存的对象，只为 string 或只为 number 或其他等等，总之为单一类型，这个时候就可以使用泛型。

```ts
class DataStorage<T> {
  private data: T[] = [];

  addItem(item: T) {
    this.data.push(item);
  }

  removeItem(item: T) {
    this.data.splice(this.data.indexOf(item), 1);
  }
  getItems() {
    return [...this.data];
  }
}

const textStorage = new DataStorage<string>();
const numberStorage = new DataStorage<number>();
```

这里我们就做好了限制，如果想在 `textStorage` 加入 number 是不可能的，因为限定了其类型为 string，一切操作都和 string 相关，对于 `numberStorage` 也一样。
但对于 object 我们遇到了一些问题。

```ts
const objStorage = new DataStorage<object>();

objStorage.addItem({ name: "Jack" });
objStorage.addItem({ name: "Du" });
// ...
objStorage.removeItem({ name: "Jack" });
console.log(objStorage.getItems()); // { name: "Du" }
// Fail to remove, because the object is a totally new one, even if they have same content.
```

我们无法正确删除指定的 item，因为这是引用类型，对于 JavaScript 而言虽然内容相同，但确实一个全新的对象，`indexOf` 的返回值会是-1，也就是会把数组的最后一个元素给删掉。所以可以给 `removeItem` 做优化。

```ts
  removeItem(item: T) {
    if (this.data.indexOf(item) === -1) return;
    this.data.splice(this.data.indexOf(item), 1);
  }
```

但这并不是长久之计，也还有一种解决办法

```ts
const Lang = { name: "Lang" };
objStorage.addItem(Lang);
objStorage.removeItem(Lang);
```

将要添加的储存为一个固定变量，但这样会很复杂。所以最好的还是不要让这个类适用于 object，对于 object 应该有专门的 Class 最好。

```ts
class DataStorage<T extends string | number | boolean> {
  //...
}
```

## Generic Utility Types

TypeScript 自带了很多高级的类型，可以在编写代码的时候帮助我们。这些不会在编译的时候被编译，但却可以对我们的代码做额外的严格检查。

### Partial (可选属性，但仍然不允许添加接口中没有的属性)

```ts
interface CourseGoal {
  title: string;
  description: string;
  completeUntil: Date;
}

function createCourseGoal(
  title: string,
  description: string,
  date: Date
): CourseGoal {
  let courseGoal: Partial<CourseGoal> = {};
  courseGoal.title = title;
  courseGoal.description = description;
  courseGoal.completeUntil = date;
  return courseGoal as CourseGoal;
}
```

在这个例子当中，我们对 `courseGoal` 的类型写的是`Partial<CourseGoal>`，意思是，结构和 CourseGoal 的结构一样，**但是！里面的每一个属性，都会是可选的，可以有也可以没有。**如果我们不这样写，下面的赋值语句会出错，因为我们最开始赋予 course Goal 的是一个空的 object。最后返回的时候，以 CourseGoal 返回即可。

### Readonly （只读属性）

让一个变量只读

```ts
const names: Readonly<string[]> = ["Max", "Anna"];
names.pop();
```

pop 方法会报错，因为 `Readonly` 让这个变量只读。

> http://t.csdn.cn/i8R9K

## Generic Types vs Union Type

为什么我们有时候需要泛型呢？看下面这个例子

```ts
class DataStorage<T extends string | number | boolean> {
  //...
}
```

这是之前写的，如果我们不用泛型，使用 Union Type 呢？

```ts
class DataStorage {
  private data: (string | number | boolean)[] = [];

  addItem(item: string | number | boolean) {
    //...
  }

  removeItem(item: string | number | boolean) {
    //...
  }
  getItems() {
    return [...this.data];
  }
}
```

这样看好像可以，但实际上他的意思是，data 是可以存储三种类型的数组，下面的 method 参数，也都可以传三种。如果再换个方法呢？

```ts
class DataStorage {
  private data: string[] | number[] | boolean[] = [];
  // ...
}
```

这样好像可以，但是我们在传参数的时候。要判断他传的参数类型才可以做得到，因为 data 只能是一种的类型的 Array。
