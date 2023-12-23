---
title: Advanced Types
date: '2022-10-15'
tags: ['TypeScript', 'guide']
draft: false
summary: 'In this article we learn the Advanced Types of TypeScript.'
---

# Advanced Types

## Intersection Types

当 type 为对象时，联合 type 取他们的**并集**

```ts
type Admin = {
  name: string;
  privileges: string[];
};

type Employee = {
  name: string;
  startDate: Date;
};

type ElevatedEmployee = Admin & Employee;

const e1: ElevatedEmployee = {
  name: "Fan",
  privileges: ["Admin"],
  startDate: new Date(),
};
```

当 type 为 Union 的时候，联合 type 取他们的**交集**

```ts
type Combinable = string | number;
type Numeric = number | boolean;

type Unverisal = Combinable & Numeric; // type: number
```

## More on Type Guard

有几种方法做类型守卫
第一种，使用 `typeof`

```ts
type Combinable = string | number;
type Numeric = number | boolean;

type Unverisal = Combinable & Numeric; // type: number

const add = (a: Combinable, b: Combinable) =>
  typeof a === "string" || typeof b === "string"
    ? a.toString() + b.toString()
    : a + b;
```

第二种，在面对一个对象时，我们不知道这个属性在不在这个对象里，可以使用 `in` 关键字。

```ts
type Admin = {
  name: string;
  privileges: string[];
};

type Employee = {
  name: string;
  startDate: Date;
};

type UnknownEmployee = Admin | Employee;

function printEmployeeInformation(emp: UnknownEmployee) {
  console.log("Name" + emp.name);
  if ("privileges" in emp) {
    console.log("Privileges: " + emp.privileges);
  }
  if ("startDate" in emp) {
    console.log("Start Date: " + emp.startDate);
  }
}

printEmployeeInformation({ name: "Jack", startDate: new Date() });
```

第三种，如果是 Class 的联合，我们可以使用 `instanceof` 来判断是不是其中一个类的实例，来判断他有哪些属性和方法可用。

```ts
class Car {
  drive() {
    console.log("Driving...");
  }
}
class Truck {
  drive() {
    console.log("Driving a truck...0");
  }
  loadCargo(amount: number) {
    console.log("Loading cargo..." + amount);
  }
}

type Vehicle = Car | Truck;
const v1 = new Car();
const v2 = new Truck();

function useVehicle(vehicle: Vehicle) {
  vehicle.drive();
  if (vehicle instanceof Truck) {
    vehicle.loadCargo(1000);
  }
}
useVehicle(v1);
useVehicle(v2);
```

## Discriminated Union

我们可以设置一个我们已经知道的，一定存在的属性，去做类型守卫，去检查我们正在使用的类型。

```ts
interface Bird {
  type: "bird";
  flyingSpeed: number;
}

interface Horse {
  type: "horse";
  runningSpeed: number;
}

type Animal = Bird | Horse;

function moveAnimal(animal: Animal) {
  let speed;
  switch (animal.type) {
    case "bird":
      speed = animal.flyingSpeed;
      break;
    case "horse":
      speed = animal.runningSpeed;
      break;
  }
  console.log("Moving at speed: " + speed);
}

moveAnimal({ type: "horse", runningSpeed: 100 });
```

这里我们赋予了明面上的 type，在实例化对象的时候，我们就要写清楚我们的 type。同时，`switch` 的时候不容易写错单词，因为 TypeScript 会自动帮我们检查，有哪些是可选的选项，帮助我们 Coding。

## Type Casting

类型转换非常有用。有时候我们需要操作 `DOM` 节点，这个时候会有一些问题，**我们知道我们选择的 dom 节点时什么标签，TypeScript 不知道。**

```ts
const paragraph1 = document.querySelector("p");
```

在这个例子中，如果我们看 `paragraph1` 的类型，可以看到是`const paragraph1: HTMLParagraphElement`，因为我们是直接按类型选择的，可是如果按 ID 做选择呢？

```ts
const paragraph2 = document.getElementById("message-output");
```

可以看到类型为`const paragraph2: HTMLElement`，这个时候 TypeScript 就只知道他是一个 HTML 元素，不知道他是 `p` 标签，只可以获取一些通用的属性，例如 text 等。
如果这是一个 input 标签呢？

```ts
const userInputElement = document.getElementById("user-input");

userInputElement.value = "Hi!";
```

这个时候如果我们修改它的 value 是会报错的。因为 TypeScript 并不知道你具体是什么标签，至少 `HTMLElement` 上没有 value，所以我们需要高速 TypeScript，这是一个 Input 标签。

```ts
const userInputElement = <HTMLInputElement>(
  document.getElementById("user-input")
);
//------------------------------------------------------------------------------
const userInputElement = document.getElementById(
  "user-input"
)! as HTMLInputElement;
```

可以有两种方法，一种是在前面加`<>`，另一种这是使用 `as` 关键字，**其中的'!'是告诉 TypeScript 这里不会为 null。**

如果我们不确定这个元素会不会 `null` 可以做检查。

```ts
if (userInputElement) (userInputElement as HTMLInputElement).value = "Hi";
```

## Index Properties

有时候我们不希望把 Interface 写的太死。我们知道它的键值对分别是什么 type，同时数量又是可选的，那就用`[]`括起来。

```ts
interface ErrorContainer {
  // eg. {email: 'Not a valid email', username: 'Must start with a character'}
  [props: string]: string;
}

const errorBag: ErrorContainer = {
  email: "Not a valid email",
  username: "Must start with a character",
};
```

## Function Overloads

函数重载。没有函数重载我们会遇到一些问题。

```ts
type Combinable = string | number;

const add = (a: Combinable, b: Combinable) =>
  typeof a === "string" || typeof b === "string"
    ? a.toString() + b.toString()
    : a + b;

const addResult = add("Jack", "Du"); as string;

result.split(' ')
```

在这个例子中，如果我们没有写明`as string`，是没有办法使用 `split` 的，会报错。因为 `result` 可能为 `number`，使用重载来说清楚每种情况的返回类型，可以是函数更易维护和好用。

```ts
function overloadAddFunc(a: number, b: number): number;
function overloadAddFunc(a: string, b: string): string;
function overloadAddFunc(a: number, b: string): string;
function overloadAddFunc(a: string, b: number): string;
function overloadAddFunc(a: Combinable, b: Combinable) {
  return typeof a === "string" || typeof b === "string"
    ? a.toString() + b.toString()
    : a + b;
}

const result = overloadAddFunc("Jack", "Du");

result.split(" ");
```

不同的参数类型对于应不同的返回类型。这个时候我们使用 `split` 就会有提示而且不报错了。

## Optional Chaining

如果我们有一个，来自后端或数据库或任何其他不确定来源的数据，如果在对象中我们需要某个属性，但是这里面还没有做填充，或者不知道什么原因没获取到。我们需要做判断以防报错。
在 JavaScript 中这样操作。

```ts
const fetchUserData = {
  id: "1",
  name: "Jack",
  //   job: { title: "CEO", description: "My own company" },
};

console.log(fetchUserData.job && fetchUserData.job.title);
```

TypeScript 有更好的选择：Optional Chaining

```ts
console.log(fetchUserData?.job?.title);
```

'?'表面：如果有这个选项，就继续往下访问。类似于做了 if 检查。

## Nullish Coalescing

有时候我们会获取用户的输入，例如 input，如果他没有输入我们就采取一个默认值。
在 JavaScript 中可以这样做：

```ts
// const userInput = undefined;
// const userInput = null;
const userInput = "";

console.log(userInput || "DEFAULT");
```

需要注意的是：**如果 userInput 为空字符串，仍然会输出 DEFAULT，因为会视为 false。**如果我们**只想在非 null 和 undefined 的情况下**，输入'DEFAULT',那么就使用`??`

```ts
// const userInput = undefined;
// const userInput = null;
const userInput = "";

console.log(userInput ?? "DEFAULT");
```

这个时候，哪怕 userInput 为空字符串，也会输出，输出内容为空。
