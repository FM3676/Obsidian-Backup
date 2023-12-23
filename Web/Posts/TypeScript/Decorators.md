---
title: Decorators
date: '2022-10-15'
tags: ['TypeScript', 'guide']
draft: false
summary: 'In this article we learn the Decorators of TypeScript.'
---

# Decorators

装饰器（Decorator）用来增强 JavaScript 类（class）的功能，许多面向对象的语言都有这种语法，目前有一个提案将其引入了 ECMAScript。
装饰器是一种特殊类型的声明，它能够被附加到类声明，方法，访问符，属性或参数上。 装饰器使用`@expression` 这种形式，`expression` 求值后必须为一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入。

## A First Class Decorator

类装饰器是我们最常使用到的，它的通常作用是，为该类扩展功能。

```ts
function Logger(constructor: Function) {
  console.log("Logging...");
  console.log(constructor);
}

@Logger
class Person {
  name = "Jack";
  constructor() {
    console.log("Creating person object....");
  }
}

const person = new Person();

console.log(person);
```

在这里，`Logger` 就是一个装饰器。使用的时候，在**类声明**前一行使用 `@` 后跟装饰器名字使用。这里用作类装饰器。
类装饰器应用于类构造函数，可以用来监视，修改或替换类定义。
类装饰器表达式会在运行时当做函数被调用，类的构造函数函数作为其唯一的参数。
现在运行这段代码，看看会有什么。

```terminal
Logging...
class Person {
    construcotr() {
        this.name = "Jack";
        console.log("Creating person object....")
    }
}
Creating person object....
Person { name: 'Jack' }
```

首先可以看的到，装饰器内的输出先于我们实例化 `Person` 的输出。装饰器在类被定义的时候执行，而不是实例化的时候，事实上不需要做实例化也会执行。

## Why Decorator?

设想有这样一个场景。
目前有一个 Tank 类，有一个 Plane 类，有一个 Animal 类。这三个类都需要一个公共的方法来获取他们所在的位置。我们第一可能想到使用继承来实现。

```ts
class BaseClass {
  getPosition() {
    return {
      x: 100,
      y: 200,
      z: 300,
    };
  }
}
class Tank extends BaseClass {}
class Plane extends BaseClass {}
class Animal extends BaseClass {}
```

这样三个类都可以调用 `getPosition` 方法来获取各自的位置了。到目前为止看起来没什么问题。

现在又有了一个新的诉求，`Tank` 类和`Plane`类需要一个新的方法`addPetrol`来给坦克和飞机加油。而动物不需要加油。此时这种写法好像不能继续进行下去了。而 js 目前没有直接语法提供多继承的功能，我们的继承方式好像行不通了。这时候装饰器可以很完美的实现这样的功能。此时就可以请我们的装饰器闪亮登场了

装饰器功能之——能力扩展
我们把`getPosition`和`addPertrol`都抽象成一个单独的功能，它们得作用是给宿主扩展对应的功能。

```ts
const getPositionDecorator: ClassDecorator = (constructor: Function) => {
  constructor.prototype.getPosition = () => {
    return [100, 200];
  };
};

const addPetrolDecorator: ClassDecorator = (constructor: Function) => {
  constructor.prototype.addPetrol = () => {
    // do something
    console.log(`${constructor.name}进行加油`);
  };
};

@addPetrolDecorator
@getPositionDecorator
class Tank {}
@addPetrolDecorator
@getPositionDecorator
class Plane {}

@getPositionDecorator
class Animal {}
```

这样的话，加入日后我们有其他的猫猫狗狗，都可以对他进行能力扩展，让其具有加油的能力。

> 多个装饰器叠加的时候，执行顺序为离被装饰对象越近的装饰器越先执行。下面有更详细的章节。

## Working with a Decorator Factories

可以通过装饰器工厂创建装饰器

```ts
function Logger(logString: string) {
  return function (constructor: Function) {
    console.log(logString);
    console.log(constructor);
  };
}

@Logger("LOGGING - PERSON")
class Person2 {
  name = "Jack";
  constructor() {
    console.log("Creating person object....");
  }
}
```

在这里面我们返回一个函数，同时，我们有能力接受参数了，这让我们可以对装饰器有更高的灵活性，更多的可能性。实用性会更强。

## Building More Useful Decorator

来看看装饰器还能干嘛

```ts
function WithTemplate(template: string, hookId: string) {
  return function (constructor: any) {
    const hookEl = document.getElementById(hookId);
    const p = new constructor();
    if (hookEl) {
      hookEl.innerHTML = template;
      hookEl.querySelector("h1")!.textContent = p.name;
    }
  };
}

@WithTemplate("<h1></h1>", "app")
class P {
  namme = "Jack";
}
```

这个时候引入对应产生的 JavaScript 文件到 HTML，就可以看到屏幕显示 Jack 了。这就有一点像 Angular 的使用方法了。通过装饰器可以使用类里面的属性来生成我们需要的东西。

## Adding Multiple Decorators

```ts
function Deco1() {
  console.log("Deco1");
  return function (_: Function) {
    console.log("Deco 1 return");
  };
}
function Deco2() {
  console.log("Deco2");
  return function (_: Function) {
    console.log("Deco 2 return");
  };
}

@Deco1()
@Deco2()
class Multiple {}
```

现在添加了多个装饰器，来看执行顺序。

```terminal
Deco1
Deco2
Deco 2 return
Deco 1 return
```

从这里我们可以看到，**对于装饰器本身：**执行顺序是从上到下的。**对于装饰器工厂函数返回的函数：**执行顺序则是从下到上的。

## Divindinto Property Decorators

属性装饰器。
属性装饰器表达式会在运行时当作函数被调用，传入下列 2 个参数：

1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。 2.成员的名字。

```ts
function Log(target: any, propertyName: string | Symbol) {
  console.log("Property decorator!");
  console.log(target, propertyName);
}

class Product {
  @Log
  title: string; // Instance Member
  @Log
  static id: string; // Static Member
  private _price: number;
  constructor(t: string, p: number) {
    this.title = t;
    this._price = p;
  }

  set price(val: number) {
    if (val > 0) {
      this._price = val;
    } else {
      throw new Error("Invalid price.");
    }
  }

  getPriceWithTax(tax: number) {
    return this._price * (1 + tax);
  }
}
```

来看看两次输出分别是什么

```terminal
Property decorator!
{                                                  'title'
    getPriceWithTax: ƒ (tax)
    constructor: ƒ Product(t, p)
    set price: ƒ (val)
    [[Prototype]]: Object
}
```

```terminal
ƒ Product(t, p) {
        this.title = t;
        this._price = p;
    }
'id'
```

第一次是 title，是实例成员，输出类的原型对象。
第二次是 id，静态成员，输出类的构造函数。
同时，我们这里没有实例化任何一个对象，但是装饰器函数还是会正常运行。

## Accessor & Method & Parameter Decorators

### Accessor Decorator 访问器装饰器

访问器装饰器表达式会在运行时当作函数被调用，传入下列 3 个参数：

1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。 2.成员的名字。 3.成员的属性描述符。

```ts
function Log(target: any, name: string, descriptor: PropertyDescriptor) {
  console.log("Accessor decorator!");
  console.log(target);
  console.log(name);
  console.log(descriptor);
}

class Product {
  private _price: number;
  constructor(p: number) {
    this._price = p;
  }

  @Log
  set price(val: number) {
    if (val > 0) {
      this._price = val;
    } else {
      throw new Error("Invalid price.");
    }
  }

  // ...
}
```

对应的输出：

```terminal
{
    getPriceWithTax: ƒ (tax)
    constructor: ƒ Product(t, p)
    set price: ƒ (val)
    [[Prototype]]: Object
}
price
{
  get: [Function: get price],
  set: [Function: set price],
  enumerable: false,
  configurable: true
}
```

### Method Decorator 方法装饰器。

方法装饰器表达式会在运行时当作函数被调用，传入下列 3 个参数：

1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。 2.成员的名字。 3.成员的属性描述符。

```ts
function Log(
  target: any,
  name: string | Symbol,
  descriptor: PropertyDescriptor
) {
  console.log("Method decorator!");
  console.log(target);
  console.log(name);
  console.log(descriptor);
}

class Product {
  private _price: number;
  constructor(p: number) {
    this._price = p;
  }
  // ...
  @Log
  getPriceWithTax(tax: number) {
    return this._price * (1 + tax);
  }
}
```

对应的输出：

```terminal
{
    getPriceWithTax: ƒ (tax)
    constructor: ƒ Product(t, p)
    set price: ƒ (val)
    [[Prototype]]: Object
}
getPriceWithTax
{
  value: [Function: getPriceWithTax],
  writable: true,
  enumerable: false,
  configurable: true
}
```

### Parameter Decorator 参数装饰器

参数装饰器表达式会在运行时当作函数被调用，传入下列 3 个参数：

1.对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。 2.成员的名字。 3.参数在函数参数列表中的索引。

```ts
function Log(target: any, name: string | Symbol, position: number) {
  console.log("Parameter decorator!");
  console.log(target);
  console.log(name);
  console.log(position);
}

class Product {
  // ...
  getPriceWithTax(@Log tax: number) {
    return this._price * (1 + tax);
  }
}
```

对应的输出：

```terminal
{
    getPriceWithTax: ƒ (tax)
    constructor: ƒ Product(t, p)
    set price: ƒ (val)
    [[Prototype]]: Object
}
getPriceWithTax
0
```

## When Do Decorators Execute

装饰器会在我们定义变量，类等东西的时候执行。装饰器很擅长在不破坏原有代码结构的情况下，为其扩展功能。装饰器配合 metadata 可以实现很多强大的功能。

## Returning (and changing) a Class in a Class Decorator

在类装饰器中，我们可以返回一个新的构造函数，它会替换我们原来使用了这个装饰器的类。所以我们可以选择返回一个新的构造函数或者返回一个 class，这里选择返回一个 class

```ts
function WithTemplate(template: string, hookId: string) {
  console.log("TEMPLATE FACTORY");

  return function (originalConstructor: any) {
    return class extends originalConstructor {
      constructor() {
        super();
        const hookEl = document.getElementById(hookId);
        const p = new originalConstructor();
        if (hookEl) {
          hookEl.innerHTML = template;
          hookEl.querySelector("h1")!.textContent = p.name;
        }
      }
    };
  };
}

@WithTemplate("<h1></h1>", "app")
class PP {
  name = "Max";
}
```

现在我们要在保有原来类的属性和方法的基础上，去修改和增加类的方法和属性。现在我们希望这个类，只在实例化的时候，而不是在装饰器执行(类定义)的时候，在 HTML 渲染 `<h1></h1>` 标签。

为此我们不生成`p`变量，使用`this`来获取 name。

```ts
function WithTemplate(template: string, hookId: string) {
    // ...
      constructor() {
        super();
        const hookEl = document.getElementById(hookId);
        if (hookEl) {
          hookEl.innerHTML = template;
          hookEl.querySelector("h1")!.textContent = this.name;
        }
      }
    };
  };
}

@WithTemplate("<h1></h1>", "app")
class PP {
  name = "Max";
}

```

但现在其实`@WithTemplate("<h1></h1>", "app")`是报错的，说类型不正确。

```ts
function WithTemplate(template: string, hookId: string) {
  return function <T extends { new (...args: any[]): { name: string } }>(
    originalConstructor: T
  ) {
    return class extends originalConstructor {
      constructor(..._: any[]) {
        // ...
      }
    };
  };
}
```

在这里，我们首先使用泛型 T，并将其设置为`originalConstructor`的类型。接下来就要声明清楚：
T 是一个构造函数。我们声明其是一个特殊的类型（一个 Object）来说明其实构造函数。使用 new 关键字来告诉 TypeScript：这最终会是一个对象，一个可以被 new 构建的对象，所以这是一个构造函数。我们可以通过 new 来生成新的 Object。
在这个函数里，我们又可能会有若干个参数，所以这里使用 Rest parameters，即`new (...args: any[])`，类型为 any。
最后，我们设置这个函数会返回一个对象，同时下面又用到了`this.name`，所以我们最终写成`{ name: string } `。

> **注意：**这里 `constructor(..._: any[])`使用下划线而不是`...args: any[]`的原因是，`_`意味着这个参数不被使用，不然 TypeScript 会提示你没有使用该参数

```ts
@WithTemplate("<h1></h1>", "app")
class PP {
  name = "Jack";
}

const PPinstance = new PP();
```

现在，如果看 HTML 文件，可以看到 Jack 的字样。但是：如果去掉`PPinstance`，你就看不到了，因为现在只有实例化了才可以看得到。
同时，在使用的时候我们不一定要`extends originalConstructor`，这样就是一个全新的构造函数，写了则是增加功能。

## Other Decorators Return Type

除了类装饰器会返回东西，剩下的几种也会。其中：**Property Decorators**和**Parameter Decorators**虽然可以返回东西，但是 typescipt 会忽略他们，所以可以返回的只有 Accessor Decorators 和 Method Decorators。

在

```ts
function AccessorDecorator(
  target: any,
  name: string,
  descriptor: PropertyDescriptor
) {
  //...
}
function MethodDecorator(
  target: any,
  name: string | Symbol,
  descriptor: PropertyDescriptor
) {
  // ...
}
```

在这两种 Decorator，都有一个参数 descriptor，类型都为 PropertyDescriptor。通过它我们可以获得其对应的 Property 描述的一个 Object。

```terminal
**------------Accessor------------**
{
  get: [Function: get price],
  set: [Function: set price],
  enumerable: false,
  configurable: true
}
**------------Method------------**
{
  value: [Function: getPriceWithTax],
  writable: true,
  enumerable: false,
  configurable: true
}
```

可以看到他们两种对象的内容不同吗，我们可以修改里面的内容，或者增加内容，来进行修改。

我们可以设置其返回类型为`PropertyDescriptor `，然后返回一个 object，作为新的 PropertyObject，下面以 Method Decorators 举例。

## Example Creating an Autobind Decorators

现在，我们在 HTML 文件里面，弄一个 button 标签。随后在对应 TypeScript 写下这些。

```ts
class Printer {
  message = "This works!";

  showMessage() {
    console.log(this.message);
  }
}

const p = new Printer();

const button = document.querySelector("button")!;
button.addEventListener("click", p.showMessage);
```

我们希望当 button 被点击的时候，控制台输出正确信息，但实际上，输出的是`undefined`。为什么？因为方法绑定后执行这个方法的`this`指向并不是实例 p。想要代码正常运行，可以这样写

```ts
button.addEventListener("click", p.showMessage.bind(p));
```

将 this 指向 p，这个时候就可以正常输出了。

现在尝试用装饰器来实现自动绑定 this。

```ts
function Autobind(
  _: any,
  __: string | Symbol,
  descriptor: PropertyDescriptor
): PropertyDescriptor {
  const originalMethod = descriptor.value;
  const adjDescriptor: PropertyDescriptor = {
    configurable: true,
    enumerable: false,
    get() {
      const boundFn = originalMethod.bind(this); // this keyword assign to the variable who call get(), and that's the variable who visit the property.
      return boundFn;
    },
    // extra properties ......
  };
  return adjDescriptor;
}
```

在这里， `const originalMethod = descriptor.value;`是获取 Method 本身，之前在控制台可以看到 Function 本身储存在 value 属性上。

随后新建一个变量`adjDescriptor`，类型为 PropertyDescriptor。我们设置它的 get()，来绑定 this 指向。

`const boundFn = originalMethod.bind(this);`意味着重新存储这个函数，这个函数的 this，**永远指向调用 get()方法的 Object**，也就是访问这个变量的 Object 本身。这样就做好了绑定，随后返回即可。

现在，不需要再对每一次调用绑定 this，也会自动将 this 指向 p 了。这个例子很好的说明了，装饰器可以帮助我们节省很多工作，而且更简洁。

## Validation with Decorators

现在用 Decorator 做一个 Validator。
我们可能会从别处获取数据，或者让用户输入表单，这都需要对数据做 Validation。
现在假设一个场景，首先再 HTML 里有这样的表单：

```html
<form>
  <input type="text" placeholder="Course title" id="title" />
  <input type="text" placeholder="Course price" id="price" />
  <button type="submit">Save</button>
</form>
```

然后创建一个`Course`类，再对 DOM 进行获取然后操作，生成一个 Course 的实例。

```ts
class Course {
  title: string;
  price: number;

  constructor(t: string, p: number) {
    this.title = t;
    this.price = p;
  }
}

const courseForm = document.querySelector("form");
courseForm?.addEventListener("submit", (event) => {
  event.preventDefault();
  const titleEl = document.getElementById("title") as HTMLInputElement;
  const priceEl = document.getElementById("price") as HTMLInputElement;

  const title = titleEl.value;
  const price = +priceEl.value;

  const createdCourse = new Course(title, price);
  console.log(createdCourse);
});
```

但现在有些问题，如果表单内容都为空，一样可以新建实例。当然可以通过`if (title.trim() > 0 ) //...`来做检查，但如果东西很多这就很复杂，能否用装饰器来做到呢？

看看答题的思路：我们需要一个`Required`装饰器，一个`Positive`装饰器，还有一个`validate`函数，让给我们可以进行验证。

```ts
function Required() {
  // ...
}

function PositiveNumber() {
  // ...
}

function validate(obj: object) {
  // ...
}

class Course {
  @Required
  title: string;
  @PositiveNumber
  price: number;

  constructor(t: string, p: number) {
    this.title = t;
    this.price = p;
  }
}
// ...
const createdCourse = new Course(title, price);
validate(createdCourse);
// ...
```

首先建立一个 Interface 去储存配置，这里面有若干个 properties，然后在这里面我们又有具体的验证的属性，我们的要求。类似于`['required', 'positive']`的感觉。

```ts
interface ValidatorConfig {
  [property: string]: {
    [validatableProp: string]: string[]; // ['required', 'positive']
  };
}
```

然后建一个`registeredValidators`Object 来储存我们想要的验证配置，在`Required`函数，我们获取这个类的 Constructor 的名字，用于储存这个类下的配置，然后再到具体的每个属性，来分配所需要验证的类型，这里需要验证他是否必须，即 required。

```ts
const registeredValidators: ValidatorConfig = {};

function Required(target: any, propName: string) {
  registeredValidators[target.constructor.name] = {
    ...registeredValidators[target.constructor.name],
    [propName]: ["required"],
  };
}
```

对应`Positive`函数则是一样的思路：

```ts
function PositiveNumber(target: any, propName: string) {
  registeredValidators[target.constructor.name] = {
    ...registeredValidators[target.constructor.name],
    [propName]: ["positive"],
  };
}
```

现在来看`validate`函数，我们首先获取我们要验证的 Object 的配置，然后遍历去查看每一种情况是否符合，符合则返回 true。

```ts
function validate(obj: any) {
  const objValidatorConfig = registeredValidators[obj.constructor.name];
  if (!objValidatorConfig) return true;

  for (const prop in objValidatorConfig) {
    for (const validator of objValidatorConfig[prop]) {
      switch (validator) {
        case "required":
          return !!obj[prop];
        case "positive":
          return obj[prop] > 0;
      }
    }
  }

  return true;
}
```

现在大部分情况都可以正常工作，但是其实，如果 title 为 0，price>0 也可以通过，但实际上应该不行，为什么会这样？

```ts
for (const prop in objValidatorConfig) {
  console.log(prop);
  // ...
}
```

我们加一句看看哪个属性会率先被验证，会发现是 price，这意味着什么？如果 price 被验证通过，返回了 true，那么剩下的就不重要了，无论是 true 还是 false 函数返回过 true 以后都会被通过，所以要修改逻辑。

```ts
function validate(obj: any) {
  const objValidatorConfig = registeredValidators[obj.constructor.name];
  if (!objValidatorConfig) return true;

  let isValid = true;
  for (const prop in objValidatorConfig) {
    //   console.log(prop);
    for (const validator of objValidatorConfig[prop]) {
      switch (validator) {
        case "required":
          isValid = isValid && !!obj[prop];
          break;
        case "positive":
          isValid = isValid && obj[prop] > 0;
          break;
      }
    }
  }

  return isValid;
}
```

增加`isValid`变量。这样只有所有都通过，函数才会返回 true。

> GitHub 上有许多的，基于 TypeScript 制作的验证器，我们都可以直接使用，同时 Angular 和 Nest 都使用到了装饰器来使用。

> Reference：
>
> [ TS 装饰器\_曲径通幽~的博客-CSDN 博客\_ts 装饰器](https://blog.csdn.net/qq_36205941/article/details/124675122)
>
> [Decorators - TypeScript 中文手册 (bootcss.com)](https://TypeScript.bootcss.com/decorators.html#class-decorators)
