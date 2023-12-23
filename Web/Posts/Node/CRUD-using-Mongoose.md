---
title: CRUD using Mongoose
date: "2022-9-1"
tags: ["Node", "Tutorial", "Express", "MongoDB"]
draft: false
summary: "In this article we learn how to Implement CRUD on MongoDB with Mongoose."
---

# CRUD using Mongoose

## Introducing MongoDB

再 Express 和 Node 中常用 MongoDB 作为数据库，这是一个文件型（非关系型）数据库，不同于 MySQL 等传统数据库，它没 u 有表、是视图、记录、等概念。所以不同于关系型数据库，我们需要设计数据库，MongoDB

中没有设计或者结构，我们只是简单的再 MongoDB 中保存 JSON 对象。

## Installing MongoDB

进入 MongoDB 后选择对应系统后下载安装包即可，本次安装为 4.4.15 版本，同时下载 MongoDB Compass，这是一个使 i 数据库可视化的工具。安装完成后在 Windows 服务启用即可（或在 CMD 输入 mongod， 如果没有则需要添加系统变量路径），同时在创建 C:\data\db 文件夹，MongoDB 将把数据存放于此。

打开 MongoDB Compass 进行连接即可。默认端口 27017。

## Connecting to MongoDB

我们通过 mongoose 库来连接 MongoDB，**注意：不同版本的 MongoDB 需要使用不同版本的 mongoose**

| MongoDB Version       | mongoose Version              |
| :-------------------- | ----------------------------- |
| MongoDB Server 2.4.x: | mongoose ^3.8 or 4.x          |
| MongoDB Server 2.6.x: | mongoose ^3.8.8 or 4.x        |
| MongoDB Server 3.0.x: | mongoose ^3.8.22, 4.x, or 5.x |
| MongoDB Server 3.2.x: | mongoose ^4.3.0 or 5.x        |
| MongoDB Server 3.4.x: | mongoose ^4.7.3 or 5.x        |
| MongoDB Server 3.6.x: | mongoose 5.x                  |
| MongoDB Server 4.0.x: | mongoose ^5.2.0               |

新建 index.js

```js
// index.js
const mongoose = require("mongoose");

mongoose
  .connect("mongodb://localhost/playground")
  .then(() => console.log("Connected to MongoDB"))
  .catch((err) => console.log("Could not connect to MongoDB...", err));
```

这里我们连接了一个 playground 的数据库，尽管我们没有创建它，但当我们进行操作的时候，MongoDB 会自动建立它。现在运行运行 `index.js`，即可成功连接数据库

## Schemas

现在我们来创建一个 Schema，我们使用 Schema 来创建一种 MongoDB 数据集合的结构。这是一个 mongoose 中的概念，**不是 MongoDB 的概念**，我们使用 Schema 来设计符合 MongoDB 集合的文档结构，定义一个文档中该有什么属性。

```js
const courseSchema = new mongoose.Schema({
  name: String,
  author: String,
  tags: [String],
  date: { type: Date, default: Date.now },
  isPublished: Boolean,
});
```

这里我们就建好了一个 Schema，我们希望一个合规的文档有着 `name`、`author` 等属性。

## Models

现在来看看如何根据 Schema 创建 `courses` 的文档。我们使用 courseSchema 来定义数据库中 course 的文档结构，我们需要把它弄成一个模型（Model）。

> 对于类与对象，我们可以创建一个 Class，拥有不同 Classes，然后实例化出不同的 Object。同样的，我们可以类比为有一个 Course 类，然后有不同的具体课程，例如 Node Course。为了创建一个 Course 类，我们需要将 Schema 弄成模型。

```js
// To create a Class like course, we need to compiled the schema to model
const Course = mongoose.model("Course", courseSchema);
```

我们传入两个参数，第一个参数是目标集合的单数模型名称。在数据库中，我们有个集合叫 Courses，所以我们传入这个集合的单数名称（去掉 s），第二个参数就是需要的 Schema。这样就创建好了，现在我们来创建一个实际对象。

```js
const course = new Course({
  name: "Node.js Course",
  author: "Mosh",
  tags: ["node", "backend"],
  isPublished: true,
});
```

**这里我们之前设置了 date 的默认值，所以不需要传入一个值**

## Saving a Document

现在我们来把这个 course 对象保存到数据库。这个 `course` 对象有一个 `save` 方法，是一个异步操作，因为连接到数据库需要时间，当储存到 MongoDB 后，MongoDB 会给对象或文档一个唯一的标识。**注意：要使用 await，必须在一个 async 异步函数里执行。**

```js
async function createCourse() {
  const course = new Course({
    name: "Angular Course",
    author: "Mosh",
    tags: ["node", "frontend"],
    isPublished: true,
  });

  const result = await course.save();
  console.log(result);
}

createCourse();
```

运行它，就会发现已经储存好了

![Saving-a-Document-Compass](//Images/Node/Saving-a-Document-Compass.png)

![Saving-a-Document-Terminal](//Images/Node/Saving-a-Document-Terminal.png)

现在我们再创建一个 Node Courses 并储存，然后来看看如何做查询。

## Querying Documents

现在来看下如何查询。我们之前创建的 Course 类有很多查询的方法：有 `find`，用于获得一个文档的列表、`findById`，通过 ID 查找、`findOne`，返回一个单一的文档等等等等。先来看看 `find` 方法。

它返回一个文档查询对象（Document Query），有点像 Promise，他也有 `then` 方法。

```js
async function getCourses() {
  const courses = await Course.find();
}

getCourses();
```

现在它会返回所有的课程，我们可添加筛选条件以返回指定课程。

```js
const courses = await Course.find({ author: "Mosh", isPublished: true }); // A filter, only return the courses that author is Mosh and isPbulished
```

但返回的课程会有很多属性，我们只需要少数课程以及特定的属性，所以我们可以进一步添加限制

```js
const courses = await Course.find({ author: "Mosh", isPublished: true }) // A filter, only return the courses that author is Mosh and isPbulished
  .limit(10) // and only return 10 documents
  .sort({ name: 1 }) // Sort by name in ascending order, -1 means descending order.
  .select({ name: 1, tags: 1 }); // Select the propeties that we want to return.
```

现在他只会最多返回 10 个课程，并且以名字正序排序，包含 `tags` 和 `name` 两个属性

## Comparison Query Operators

```js
async function getCourses() {
  // eq (equal)
  // ne (not equal)
  // gt (greater than)
  // gte (greater than or euqal to)
  // lt (less than)
  // lte (less than or equal to)
  // in
  // nin (not in)
  const courses = await Course
    // .find({ author: "Mosh", isPublished: true })
    // .find({ price: { $gte: 10, $lte: 20 } })  -------> 10$ < Prices < 20$
    .find({ prices: { $in: [10, 15, 20] } }) // prices = 10 || 15 || 20
    .limit(10)
    .sort({ name: 1 })
    .select({ name: 1, tags: 1 });
  console.log(courses);
}

getCourses();
```

## Logical Query Operators

```js
async function getCourses() {
  // or
  // and
  const courses = await Course
    //.find({ author: "Mosh", isPublished: true })
    .find()
    .or([{ author: "Mosh" }, { isPublished: true }])
    .and()
    .limit(10)
    .sort({ name: 1 })
    .select({ name: 1, tags: 1 });
  console.log(courses);
}

getCourses();
```

这样我们可以找到发布了的 或 作者是 Mosh 的课程， 对于 and 操作符做法是类似的

## Regular Expressions

我们在最开始的 `find` 中传入的 author 是特定的字符串，要对查询字符串有更多控制，就需要正则表达式

```js
async function getCourses() {
  const courses = await Course
    // .find({ author: "Mosh", isPublished: true })

    // Starts with Mosh
    .find({ author: /^Mosh/ })

    // End with Hamedani
    .find({ author: /Hamedani$/i })

    //Contains Mosh
    .find({ author: /.*Mosh.*/ })

    .limit(10)
    .sort({ name: 1 })
    .select({ name: 1, tags: 1 });
  console.log(courses);
}

getCourses();
```

## Counting

有时候我们并不关心文档的具体内容，只需要知道有几个符合要求的文档就可以，可以使用 `count` 方法

```js
async function getCourses() {
  const courses = await Course.find({ author: "Mosh", isPublished: true })
    .limit(10)
    .sort({ name: 1 })
    //-----------
    .count(); // Return documents num.
  //--------------
  console.log(courses);
}

getCourses();
```

## Pagination

我们已经知道了 `limit` 方法，与 `limit` 方法如影随形的是 `skip` 方法，来看看如何用 `skip` 实现分页功能。先定义两个常量

```js
const pageNumber = 2;
const pageSize = 10;
// In real world, we use RESTful API to get the value, like /api/courses?pageNumber=2&pageSize=10
```

**此处采用硬编码，显示开发中通过 RESTful API 的查询字符串来传入这两个值。**为了实现分页功能，我们需要跳过前一页的所有文档，公式是这样的：**页数-1 乘 页码**。此处我声明页码是从 1 开始的，页码不代表页的序列，然后再把 `pageSize` 传给 `limit` 即可。

```js
const courses = await Course.find({ author: "Mosh", isPublished: true })
  // In order to implement pagination, we need to skip all the documents previous page.
  .skip((pageNumber - 1) * pageSize)
  .limit(pageSize);
// And we get the documents in the given page
```

## Exercise 1

```json
// exercise-data.json
[
  {
    "_id": "5a68fdc3615eda645bc6bdec",
    "tags": ["express", "backend"],
    "date": "2018-01-24T21:42:27.388Z",
    "name": "Express.js Course",
    "author": "Mosh",
    "isPublished": true,
    "price": 10,
    "__v": 0
  },
  {
    "_id": "5a68fdd7bee8ea64649c2777",
    "tags": ["node", "backend"],
    "date": "2018-01-24T21:42:47.912Z",
    "name": "Node.js Course",
    "author": "Mosh",
    "isPublished": true,
    "price": 20,
    "__v": 0
  },
  {
    "_id": "5a68fde3f09ad7646ddec17e",
    "tags": ["aspnet", "backend"],
    "date": "2018-01-24T21:42:59.605Z",
    "name": "ASP.NET MVC Course",
    "author": "Mosh",
    "isPublished": true,
    "price": 15,
    "__v": 0
  },
  {
    "_id": "5a68fdf95db93f6477053ddd",
    "tags": ["react", "frontend"],
    "date": "2018-01-24T21:43:21.589Z",
    "name": "React Course",
    "author": "Mosh",
    "isPublished": false,
    "__v": 0
  },
  {
    "_id": "5a68fe2142ae6a6482c4c9cb",
    "tags": ["node", "backend"],
    "date": "2018-01-24T21:44:01.075Z",
    "name": "Node.js Course by Jack",
    "author": "Jack",
    "isPublished": true,
    "price": 12,
    "__v": 0
  },
  {
    "_id": "5a68ff090c553064a218a547",
    "tags": ["node", "backend"],
    "date": "2018-01-24T21:47:53.128Z",
    "name": "Node.js Course by Mary",
    "author": "Mary",
    "isPublished": false,
    "price": 12,
    "__v": 0
  },
  {
    "_id": "5a6900fff467be65019a9001",
    "tags": ["angular", "frontend"],
    "date": "2018-01-24T21:56:15.353Z",
    "name": "Angular Course",
    "author": "Mosh",
    "isPublished": true,
    "price": 15,
    "__v": 0
  }
]
```

这是要导入数据库的数据

```commonlisp
mongoimport --db mongo-exercises --collection courses --drop --file exercise-data.json --jsonArray
```

这是要在控制台输入的指令，来导入数据库。

现在，写一个程序，获取所有已发布的后端课程，以名称排序，只获取课程的作者和名称。

```js
const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/mongo-exercises");

const courseSchema = new mongoose.Schema({
  name: String,
  author: String,
  tags: [String],
  date: Date,
  isPublished: Boolean,
  price: Number,
});

const Course = mongoose.model("Course", courseSchema);

// Get all the published backend courses, sort them by their name, pick onliy their name and author and display them
async function getCourses() {
  return await Course.find({ isPublished: true, tags: "backend" })
    .sort({ name: 1 })
    .select({ name: 1, author: 1 });
}

async function run() {
  const courses = await getCourses();
  console.log(courses);
}

run();
```

## Exercise 2

写一个程序，获取所有已发布的前后端课程，价格降序，只获取名字和课程名字。

```js
const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/mongo-exercises");

const courseSchema = new mongoose.Schema({
  name: String,
  author: String,
  tags: [String],
  date: Date,
  isPublished: Boolean,
  price: Number,
});

const Course = mongoose.model("Course", courseSchema);

// Get all the published frontend and backend courses, sort them by their price in a descending order.

async function getCourses() {
  return await Course.find({
    isPublished: true,
    tags: { $in: ["forntend", "backend"] },
  })

    .sort("-price")
    .select("name author price");
}

// Soultion 2
/* 
    .find({ isPublished: true, })
    .or([{ tags: "frontend" }], [{ tags: "bakcend" }]); 
*/

async function run() {
  const courses = await getCourses();
  console.log(courses);
}

run();
```

## Exercise 3

写一个程序， 包含所有课程单价大于等于 15 刀的且名字含有 by 关键字的。

```js
const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/mongo-exercises");

const courseSchema = new mongoose.Schema({
  name: String,
  author: String,
  tags: [String],
  date: Date,
  isPublished: Boolean,
  price: Number,
});

const Course = mongoose.model("Course", courseSchema);

// Get all the published courses that are $15 more, or have the word 'by' in their title

async function getCourses() {
  return await Course.find({ isPublished: true })
    .or([{ price: { $gte: 15 } }, { name: /.*by.*/i }])
    .sort("-price")
    .select("name author price");
}
async function run() {
  const courses = await getCourses();
  console.log(courses);
}

run();
```

## Updating a Document - Query First

来看下如何更新文档。我们有两种方式，一种是先查询（Query First）,先通过 ID 找到文档，然后编辑，然后再保存。在别的框架中也是这么实现的。另一种则是先更新，直接接入数据库修改文档，同时可选的获得已更新的文档。先看看 Query First。

```js
// 	Query First
async function updateCourse(id) {
  const course = await Course.findById(id);
  if (!course) return;
  // course.isPublished = true;
  // course.author = 'Another Auther';
  course.set({
    isPublished: true,
    author: "Another Author",
  });

  const result = await course.save();
  console.log(result);
}
```

## Updating a Document - Update First

先查询的方法对于我们收到客户端的更新数据又想验证的时候很有用，例如，我们不希望已经发布的课程被修改作者，我们就可以写一些 `if` 规则来限制这种情况。但有时候我们并不需要，直接修改就可以，则采用另一种方式。

使用 `update` 来代替 `findById`，第一个参数是一个过滤器，第二个参数则是更新的数据。这里我们就需要使用一个或多个更新运算符了。我们可以搜索 MongoDB update operators，就可以找到所有的更新运算符。

```js
async function updateCourse(id) {
  const result = await Course.update(
    { _id: id },
    {
      // First argument is a filter
      // Second argument is the update Object
      // Here we need to use mongodb update operators
      $set: {
        author: "Mosh",
        isPublished: false,
      },
    }
  );
  console.log(result);
}
```

有时候我们想得到已经被修改好的文档，则使用 `findByIdAndUpdate` 方法，第一个传入的参数要改为 `id`。

```js
async function updateCourseAnotherWay(id) {
  const course = await Course.findByIdAndUpdate(
    id,
    {
      $set: {
        author: "Jason",
        isPublished: false,
      },
    },
    { new: true } // With this, it returns the updated document
  );
  console.log(course);
}
```

**还有第三个参数，设置第三个参数以后，可以获得更新后的文档，不然获得的实际为更新前的文档**

## Removing Documents

```js
async function removeCourseOne(id) {
  const result = await Course.deleteOne({ _id: id }); // This method will find the first one and delete
  console.log(result);
}
```

此处过滤器的参数我们可以传一个更通用的，比如未发布的，那他会删除第一个符合条件的。

```js
async function removeCourseMultiply(id) {
  const result = await Course.deleteMany({ isPublished: false });
  console.log(result);
}
```

这个方法可以一次删除多个文档。得到的返回值为删除的文档数。

```js
async function removeCoursWithReturn(id) {
  const course = await Course.findByIdAndRemove(id);
  console.log(course);
}
```

这个方法可以得到被 删除的文档。

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
