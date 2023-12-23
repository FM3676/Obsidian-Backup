---
title: Express Middleware
date: "2022-8-30"
tags: ["Node", "Tutorial", "Express"]
draft: false
summary: "In this article we learn what is Express Middleware and how to use it and some commonly used Middleware."
---

# Middleware

## Middleware

Express 中的一个核心概念就是中间件（中间函数/Middleware）。一个中间函数，得到一个请求对象后，要么反馈给客户端，要么传递给另一个中间函数。在 RESTful 中，已见过两个中间函数。

```js
app.get("/", (req, res) => {
  res.send("Hello World");
});
```

这是一个路由句柄函数（Route Handler Function），在 Express 中，所有路由句柄函数都是中间函数。因为它需要传入一个请求对象，并向客户端返回数据，他终结了请求函数循环。

```js
app.use(express.json()); // Enable parsing of JSON object
```

当我们调用 `express.json()` 时，它返回一个函数对象，一个中间函数。其作用就是读取请求，若请求体为一个 JSON，就会格式化这个 JSON 对象并以从设置 `req.body`属性。

> 想象一请求发送后进入一个管道，这个管道里有很多中间件，每个中间件处理完后将控制权交给下一个，全部完成后便发送回客户端

## Creating Custom Middleware

```js
// index.js
app.use(function (req, res, next) {
  console.log("Logging....");
  next(); // Call next() to pass control to the next midware function. Without doing this, we're not termiate the request response cycle. Our request will hang indefinitely
});
```

我们在 `app.use()` 里传入一个中间函数，其中有三个参数，`request`、`response` 和 `next`。其中，`next` 函数是用于当这个中间件处理完成后将控制权交给下一个，如果不调用 `next` 函数，请求将永远挂起。

但我们不会直接把中间件写在 `index.js`，我们要新建一个文件，分离出来

```js
// logger.js
function log(req, res, next) {
  console.log("Logging....");
  next();
}
export.default = log;
```

```js
// index.js
const logger = requiire("./logger");
app.use(logger);
```

现在我们可以理解，`express.json()` 就是会返回一个带有三个参数的中间函数，并对数据进行处理。

## Built in Middleware

Express 中也有一些内建的中间件。

```js
app.use(express.urlencoded({ extended: true })); // With this we can pass array and complicated object using the urlencoded
```

这个中间件可以让我们处理 `x-www-form-urlencoded` 请求。

![Built-in-Middleware-urlencoded](//Images/Node/Built-in-Middleware-urlencoded.png)

```js
app.use(express.static("public")); // We put all the static data in the public folder, like imgae text...
// Now visit /--staticSource  (Eg: /readme.txt)
// Here we don't have public in url, it start contact form the root top side.
```

static 这个中间件可以让我们向外提供静态内容，很目录下创建一个 public 文件夹，我们将所有静态内容，如 CSS、图片等都放到这个文件夹中。

先创建一个简单的 txt 文本。

```js
// /public/readme.txt
This is readme file.
```

访问 `localhost:3000/readme.txt`，出现对应内容。**注意：我们的路径不包含 public，static 的方法是从根目录开始有效的**

## Third-party Middleware

访问 expressjs.com 可以看到找到很多我们可以使用的第三方中间件。现在看下 Helmet 这个中间件，它可以设置 HTTP 头部来增加安全性。先使用 npm 进行安装。

```commonlisp
npm i helmet
```

```js
const helmet = require("helmet");
app.use(helmet());
```

这样就已经完成调用了。我们再看 Morgan 这个中间件，它可以进行 HTTP 请求的日志记录。

```js
const morgan = require("morgan");
app.use(morgan("tiny"));
```

**此处 tiny 是一个字符串参数 format，更多选择请看文档**。

现在我们每次 HTTP 请求都会被记录了。![Third-party-Middleware-morgan](//Images/Node/Third-party-Middleware-morgan.png)

## Environment

有时候我们需要区分不同环境，生产环境和测试环境，来启用不同功能，例如刚才的 Morgan 我们只在测试环境起用，因为他会影响性能。

在 `process.env` 中有一个标准环境变量 `NODE_ENV`，他返回当前 node 所处环境值，未设置的时候为 undefined。

我们也有另一种获取当前环境变量的方法，`app.get()` ，它可以获取当前系统的多个设定值，其中一个就是 `env`

```js
app.get("env");
```

它内部调用了 `NODE_ENV` 的值，在未设置 `NODE_ENV` 的时候，获取值默认为 development。

```js
if (app.get("env") === "development") {
  // We can run set NODE_ENV=production, so the morgan will stop working.
  app.use(morgan("tiny"));
  console.log("Morgan enabled...");
}
```

这样 Morgan 就只在开发环境调用了。

## Configuration

我们需要如何保存应用的配置数据，并在不同环境复写对应配置呢？

我们有很多管理的包，最火的是**RC**， 但我们用里那个一个包**config**。

安装完后在根目录创建 config 文件夹，然后创建一个默认配置文件，default.json。

```json
// config/default.json
{
  "name": "My Express App"
}
```

再建一个 `development.json` 和 `production.json`，这里面的配置只会在对应环境下使用，同时会覆盖掉 `default.json` 里的配置。

```json
// config/development.json
{
  "name": "My Express App - Development",
  "mail": {
    "host": "dev-mail-server"
  }
}
```

```json
// config/production.json
{
  "name": "My Express App - Production",
  "mail": {
    "host": "pro-mail-server"
  }
}
```

回到 `index.js` 看看如何使用

```js
// index.js
const config = require("config");

// Configuration
console.log("Application Name: " + config.get("name"));
console.log("Mail Name: " + config.get("mail.host"));
// Run the code we can see
// Application Name: My Express App - Development
// Mail Name: dev-mail-server
// And these are come from development.json

// Now we run set NODE_ENV = production
// Run again
// Application Name: My Express App - Production
// Mail Name: pro-mail-server
```

设置 `NODE_ENV` 值为 development 和 production 看看有什么区别。

**但我们不可以直接把应用的机密信息保存在这里**，所以当我们处理这些机密的时候，我们应该把他们放在环境变量中。

我们来设置一个保存邮件服务器密码的环境变量。

```commonlisp
run set app_password=1234
```

开发环境中，我们手动设置这些变量，生产环境则可能会有一个面板来操作。

现在在 config 文件夹内创建一个新的文件，`custom-environment-variables.json`。**注意：拼写不可以错**。

在这个文件中映射环境变量和应用配置关系。

```json
{
  "mail": {
    "password": "app_password"
  }
}
```

```js
console.log("Mail Password: " + config.get("mail.password")); // Mail Password: 1234
```

现在我们就获得了邮箱密码了。

## Debugging

我们常用 `console.log` 调试，在每次调试完又要注释掉或者删除掉这些代码。我们可以使用 node 中的 debug 模块，这样就只在特定环境下输出信息，并且可以限制层级，例如我调试数据库，只想看到数据库相关的调试信息。

```js
npm i debug
```

```js
const startupDebugger = require("debug")("app:startup");
```

这里 `require` 返回一个函数，我们调用它，并给一个参数，一个用于调试的专用命名空间。

我们现在又需要一个调试数据库的函数。

```js
const startupDebugger = require("debug")("app:startup");
const dbDebugger = require("debug")("app:db");
```

现在我们来修改 console.log

```js
if (app.get("env") === "development") {
  // We can run set NODE_ENV=production, so the morgan will stop working.
  app.use(morgan("tiny"));
  startupDebugger("Morgan enabled..."); // !!!!!!!!!!!!!!!!!!!!!! Replace console.log
}

// Db work
dbDebugger("Connected to the database");
```

来到控制台

```cmake
set DEBUG=app:startup
```

现在我们只会看到 startup 环境的调试信息。

```
set DEBUG=app:startup,app:db
```

现在我们可以看到两个空间的调试信息。

```
set DEBUG=
```

这样我将看不到任何调试信息

```
set DEBUG=app:*
```

看到所有空间的调试信息

```js
/* 
    Command set DEBUG=app:startup, run the code
    and Command DEBUG=app:db, run the code
    and Command DEBUG= , run the code
    and Command DEBUG=app:startup,app:db, run the code
    and Command DEBUG=app:*(to see all debug information), run the code
*/
```

我们可以在启动应用时设置 DEBUG 环境

```
DEBUG=app:db node index.js
```

如果我们只有一个调试，比如现在不需要对 DB 做调试了，可以把函数名称简化为 debug。

```js
const debuge = require("debug")("app:startup");
if (app.get("env") === "development") {
  app.use(morgan("tiny"));
  debug("Morgan enabled..."); // !!!!!!!!!!!!!!!!!!!!!! Replace console.log
}
```

## Templating-Engines

有时候我们需要返回 HTML 给客户端，我们可以用 Pug、Mustache、EJS 来反馈 HTML。本次采用 Pug。

回到 `index.js`，设置应用的图形引擎。

```js
app.set("view engine", "pug");
```

设置过后，Express 会在内部自己导入 Pug 而不用我们手工导入。还有另外一个属性，用于更改模板路径。

```js
app.set("views", "./views"); // default
```

新建一个 views 文件夹，再建一个 `index.pug` 文件，使用 pug 语法创建 HTML 模板

```pug
html
    head
        title=title
    body
        h1=message
```

**此处 title 和 message 为动态变量**。回到 index.js

```js
app.get("/", (req, res) => {
  res.render("index", { title: "My Express App", message: "Hello" });
});
```

**注意此处用的是 render 而不是 send**。此时我们浏览器访问 `localhost:3000`，就能看到 hello 字样

## Structuring Express Applications

现在我们的 `index.js` 文件非常庞大而且杂乱，我们来整理一下。新建一个 routes 文件夹，创建 `courses.js`，将 course 的 API 移植

```js
// routes/courses.js
const express = require("express");
const router = express.Router();

const courses = [
  { id: 1, name: "courses1" },
  { id: 2, name: "courses2" },
  { id: 3, name: "courses3" },
];

router.get("/api/courses", (req, res) => {
  res.send(courses);
});

router.post("/api/courses", (req, res) => {
  const { error } = validateCourse(req.body); // get the req.body.error
  if (error) return res.status(400).send(error);

  const course = {
    id: courses.length + 1,
    name: req.body.name,
  };
  courses.push(course);
  res.send(course);
});

router.put("/api/courses/:id", (req, res) => {
  // Look up the course
  // If not existing, return 404
  const course = courses.find((c) => c.id === parseInt(req.params.id));
  if (!course)
    return res.status(404).send("The course with the given ID was not found.");

  // Validate
  // If invalid, return 400 - Bad Request

  const { error } = validateCourse(req.body); // get the req.body.error
  if (error) return res.status(400).send(error);

  // Upadate course
  course.name = req.body.name;
  // Return the updated course
  res.send(course);
});

module.exports = router;
```

此处没有采用

```js
const app = express();
```

因为我们将路由放到了单独文件，这样会失效。因此使用 `express.Router()`c 方法。

```js
const courses = require("./routes/courses");
app.use("/api/courses", courses);
```

我们加载模块后，使用 `app.use()`，第一个参数表面，凡是路径由`/api/courses` 开始的，都由 courses 处理。因此我们其实可以更精简 `courses.js`

```js
router.get("/", (req, res) => {
  res.send(courses);
});

router.post("/", (req, res) => {
  //...
});

router.put("/:id", (req, res) => {
  // ...
});
```

```js
// index.js
const Joi = require("joi");
const express = require("express");
const app = express();
const helmet = require("helmet");
const courses = require("./routes/courses");

app.use(express.json()); // Enable parsing of JSON object
app.use(express.urlencoded({ extended: true })); // With this we can pass array and complicated object using the urlencoded
app.use(express.static("public")); // We put all the static data in the public folder, like imgae text...
app.use(helmet());
app.use("/api/courses", courses);

const port = process.env.PORT || 3000;

app.listen(3000, () => console.log(`Listening on the port ${port}...`));
```

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
