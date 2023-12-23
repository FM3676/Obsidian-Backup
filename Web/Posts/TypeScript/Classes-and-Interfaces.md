---
title: Classes & Interfaces
date: '2022-10-15'
tags: ['TypeScript', 'guide']
draft: false
summary: 'In this article we learn the Classes & Interfaces of TypeScript.'
---

# Classes & Interfaces

## Create first Class

如何在 TypeScript 里创造类

```ts
class Department {
  name: string;

  constructor(n: string) {
    this.name = n;
  }
}

const accounting = new Department("Accounting");

console.log(accounting);
```

在这里我们指明了 `name` 的类型为 `string`，但是实际编译到 es6 的 JavaScript 里，是不会有这一行限制的。

```js
class Department {
  constructor(n) {
    this.name = n;
  }
}
```

## Constructor Functions & The this Keyword

来看一下 `this` 关键词

```ts
class Department {
  name: string;

  constructor(n: string) {
    this.name = n;
  }

  describe() {
    console.log("Department: " + this.name);
  }
}

const accounting = new Department("Accounting");

const accountingCopy = { describe: accounting.describe };

accountingCopy.describe();
```

如果运行这段代码，得到的结果会是什么？
`Department: undefined`
为什么是 undefined？ **因为我们只是把 `accounting` 的这个方法的引用 copy 了过去，在实际运行的 `accountingCopy` 的 Object 里，并没有 name 这个属性。this 指向的，是运行这个函数的对象。**

```ts
class Department {
  name: string;

  constructor(n: string) {
    this.name = n;
  }

  describe(this: Department) {
    console.log("Department: " + this.name);
  }
}

const accounting = new Department("Accounting");

const accountingCopy = {
  name: "DUMMY",
  describe: accounting.describe,
};

accountingCopy.describe();
```

我们在这里给 `describe` 加上了一个参数，`this`，还给它限定了类型，为 `Department`。TypeScript 会分析这段代码，知道这个方法，应该什么样的 object 下才能被调用。
如果这里吧 `accountingCopy` 的 `name` 去掉，会出现错误，因为 TypeScript 知道它和 `Department` 类的应该有的属性不一样。

## private and public Access Modifiers

TypeScript 提供了 public 和 private，这在 JavaScript 中没有。

```ts
class Department {
  public name: string;
  private employees: string[] = [];

  constructor(n: string) {
    this.name = n;
  }

  describe(this: Department) {
    console.log("Department: " + this.name);
  }
  addEmployee(employee: string) {
    this.employees.push(employee);
  }
}

const accounting = new Department("Accounting");
// accounting.employees=[];
```

默认情况下，声明的变量都为 public，设置了 private 的变量，无法从外部直接访问 。

## Shorthand Initialization

快速的初始化，从而摆脱重复的赋值

```ts
class Department {
  private name: string;
  private id: string;

  constructor(id: string, name: string) {
    this.name = name;
    this.id = id;
  }
}
```

如果我们初始化的值很多，这样不断地反复赋值会很幸苦，TypeScript 提供了更快捷的办法。

```ts
class Department {
  //   private name: string;
  //   private id: string;

  constructor(private id: string, public name: string) {
    // this.name = name;
    // this.id = id;
  }
}
```

直接在构造函数里一次性声明清楚 private 还是 public，会创建同名值并赋值。

## Readonly

只读属性，当你确保一个属性不会再更改且不希望更改，使用 readonly

```ts
class Department {
  constructor(private readonly id: string, public name: string) {}
}
```

## Inheritance

```ts
class Department {
  private employees: string[] = [];

  constructor(private readonly id: string, public name: string) {}

  describe(this: Department) {
    console.log(`Department (${this.id}): ${this.name}`);
  }

  addEmployee(employee: string) {
    this.employees.push(employee);
  }

  printEmployeeInformation() {
    console.log(this.employees.length);
    console.log(this.employees);
  }
}

class ITDepartment extends Department {
  admins: string[];
  constructor(id: string, admins: string[]) {
    super(id, "IT"); // Call the constructor function of the base Class at first
    this.admins = admins;
  }
}
```

TypeScript 的继承和 JavaScript 基本一样，使用 extends 关键字，同时调用 super 方法。
**super()会在构建这个类之前先运行父类的 constructor。**
同样也可以自己在新类中添加方法。

## Overriding Properties & The protected Modifier

在继承类中可以重写父类的方法。
父类的属性设置为 protected 而不是 private，即可在子类做修改。

```ts
class Department {
  protected employees: string[] = [];

  constructor(private readonly id: string, public name: string) {}
  addEmployee(employee: string) {
    // validation etc
    // this.id = 'd2';
    this.employees.push(employee);
  }
}

class ITDepartment extends Department {
  constructor(id: string) {
    super(id, "IT");
  }
  addEmployee(employee: string) {
    this.employees.push(employee);
  }
}
```

## Getter & Setter

Getter 用于获取类中的特定属性，设定一个 getter 可以在获取时候内部做处理后再返回。
Setter 可以设置类中的属性。使用的时候就和直接设置属性值一样。也可以在内部处理。

```ts
class Department {
  protected employees: string[] = [];

  get reverseEmployees() {
    return this.employees.reverse();
  }

  set setReverseEmployees(a: string[]) {
    this.employees = a.reverse();
  }

  constructor(private readonly id: string, public name: string) {}
  addEmployee(employee: string) {
    // validation etc
    // this.id = 'd2';
    this.employees.push(employee);
  }
}
const d = new Department("1", "ad");
console.log(d.reverseEmployees);
d.setReverseEmployees = ["1", "2"];
```

## Static

类中有静态方法，他可以不用创造实例，直接调用，就像 `Math` 类的 `PI`，可以直接获取，或者 `pow` 方法，直接调用。

```ts
class Department {
  static fiscialYear = 2020;
  protected employees: string[] = [];

  constructor(private readonly id: string, public name: string) {
    // console.log(this.fiscialYear); Not available in a instance
    console.log(Department.fiscialYear);
  }
  addEmployee(employee: string) {
    this.employees.push(employee);
  }

  static createEmployee(name: string) {
    return { name };
  }
}

const pi = Math.PI;
const p = Math.pow(1, 2);

const employee = Department.createEmployee("Max");
console.log(Department.fiscialYear);
```

**注意，在构造函数可以看到，如果要输出 `fiscialYear` 变量，不可以使用 `this`，因为 static 属性不属于任何一个实例。所以直接从 Department 调用就好。**

## Abstract Classes

当你想确保某种方法可用，而且对于每一个具体的，基于这个类继承的所有子类的这个方法的具体实现都不一样，也就是**我们不能提供一个通用的方法的时候，但你想强制这个方法存在且要求子类实现，使用 abstract**。

当你想强制所有基于其他类的类，分享一些常用的方法或属性，同时你不想提供具体的值或者实现，使用 abstract。

```ts
abstract class Department {
  constructor(protected readonly id: string, public name: string) {}
  abstract describe(this: Department): void;
}
```

在 class 关键字前添加 abstract，同时再要 abstract 的方法或者属性前也添加 abstract，不需要提供具体实现，只需要写清楚返回类型和参数类型即可。

**注意：被 abstract 的类不可以再实例化，也就是无法创建一个新的基于 Department 的 Object。**

```ts
class ITDepartment extends Department {
  constructor(id: string, admins: string[]) {
    super(id, "IT");
  }

  describe() {
    console.log("IT Department - ID: " + this.id);
  }
}
```

本例中，其扩展类必须补充完整 describe 方法，不然会报错。

## Singletons & Private Constructor

当需要一个类只有一个具体的实例化对象时候，使用单例模式 Singletons

```ts
class Department {
  static instance: Department;

  private constructor(private readonly id: string, public name: string) {}

  static getInstance() {
    if (Department.instance) return this.instance;
    this.instance = new Department("1", "n");
    return this.instance;
  }
}

const theOnlyInstance = Department.getInstance();
```

首先设置 constructor 为 private，使其无法从外直接调用。
随后建立一个 static 变量，instance。用于储存唯一的，实例化的对象。
最后添加静态方法，实例存在就返回它，不存在就新建一个再返回。

## A First Interface

使用 Interface 去定义一个 Object 的结构，他应该拥有什么属性，每个属性又该有什么特点，同时也可以使用它来做类型检查。

```ts
interface Person {
  name: string;
  age: number;
  greet(phrase: string): void;
}

let user1: Person;
user1 = {
  name: "Jack",
  age: 28,
  greet(phrase: string) {
    console.log(phrase);
  },
};
```

## Using Interface with Classes

我们已经有 type 了，为什么还需要 Interface？

```ts
type Person = {
  name: string;
  age: number;
  greet(phrase: string): void;
};
```

有时候可以互换，但是 Interface 之可以用于描述对象的结构。当定义某些东西为接口的时候，Interface 会更加清晰的定义对象的结构。

```ts
interface Greetable {
  name: string;
  greet(phrase: string): void;
}

class Person implements Greetable {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  greet(phrase: string) {
    console.log(phrase);
  }
}

let user: Greetable;
user = new Person("Jack");
```

我们定义了 Interface 后在 Class 使用，因为我们希望这个 Class 有这个结构，使用 `implements` 关键字。可以同时应用多个 Interface，逗号相连即可。

可以看到，`user` 的类型设置为 Greetable，但可以新机按一个 Person 实例，因为 Person 就是按照 Greetable 建的。

**Interface 和 Abstract Class 有区别。Interface 内不能提供任何具体实现，Abstract Class 除了抽象部分方法外，内部可以写完整的函数功能。**

## Readonly Interface Properties

在 Interface 里设置一个属性为 readonly，在建 Class 时如果应用，那么这个属性会自动只读

```ts
interface Greetable {
  readonly name: string;
}

class Person implements Greetable {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

let user: Greetable;
user = new Person("Jack");
user.name = "Fan"; // !!ERROR!!
```

## Extending Interface

```ts
interface Named {
  readonly name: string;
}

interface Greetable extends Named {
  greet(phrase: string): void;
}
```

如果我们确定一个新的 Interface 一定会拥有一部分和另一个 Interface 一样的内容，那可以做扩展。可以同时扩展多个，使用逗号隔开。

## Interface as Function Type

也可以使用 Interface 作为函数类型。

```ts
// type AddFn = (a: number, b: number) => number;
interface AddFn {
  (a: number, b: number): number;
}

let add: AddFn;
add = (a: number, b: number) => a + b;
```

**注意和 type 的区别，最后用的是`:`而不是`=>`。**

## Optional Parameters & Properties

通过`?`操作符来让一个属性变得可选，不一定要具备。其可以是函数，也可以是参数，也可以是属性变量。

```ts
interface Named {
  readonly name?: string;
  outputName?: string;
}

interface Greetable extends Named {
  greet?(phrase: string): void;
}

class Person implements Greetable {
  name?: string;
  constructor(n?: string) {
    if (n) this.name = n;
  }
}
```

## Compiling Interfaces to JavaScript

如果编译 TypeScript 文件，找到对应的 JavaScript 文件其实会发现 Interface 部分什么内容都没有被编译。JavaScript 并没有这个概念，这只是 TypeScript 给我们提供的工具而已。
