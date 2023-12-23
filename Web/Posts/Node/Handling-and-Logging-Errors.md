---
title: Handling and Logging Errors in Express
date: "2022-9-6"
tags: ["Node", "Tutorial", "Express", "MongoDB"]
draft: false
summary: "In this article we learn how to handle and log errors in Node and Express."
---

# Handling and Logging Errors

## Introduction

现实会出很多错误，例如数据库连接断开，要发送合适的消息给回 Client，并在 Server 记录日志。

## Handing Rejected Promises

如果手停止 MongoDB 数据库再发送请求就会出现错误 ![Handing-Rejected-Promises](//Images/Node/Handing-Rejected-Promises.png)

出现无法请求的 Promise 错误，且没有妥善处理。回到 `genres.js`

```js
// routes/genres.js
router.get("/", async (req, res) => {
  const genres = await Genre.find().sort("name");
  res.send(genres);
});
```

这里 `await` 了但是没有用 `try-catch` 去处理错误，类似于对 Promise 使用 `then` 以后没有使用 `catch` 一样。如果使用 Promise，要使用 `catch`、采用异步方法则使用 `try-catch`

```js
router.get("/", async (req, res) => {
  try {
    const genres = await Genre.find().sort("name");
    res.send(genres);
  } catch (ex) {
    res.status(500).send("Something failed.");
  }
});
```

这个时候就会再请求，Client 会收到提示。

## Express Error Middleware

如果是现实，我们要给每个句柄都套上 `try-catch`，而且要修改的话要到每一处都做好修改，所以要把处理错误的逻辑进行集中。回到 `index.js` 添加中间件。

需要一个 `err` 参数，为在应用中某处捕捉到的异常。

```js
// index.js
app.use(express.json());
// ...
app.use("/api/auth", auth);

app.use(function (err, req, res, next) {
  // Log the exception
  res.status(500).send("Something failed.");
});
```

现在看到 `genres.js` 刚才的 `try-catch` 块，在 `catch` 块中，要将控制权转移到错误处理函数，所以在路由句柄加上一个 `next`，来转移控制权。

```js
router.get("/", async (req, res, next) => {
  try {
    //...
  } catch (ex) {
    next();
  }
});
```

现实开发中，处理函数会很长，所以要做封装。新建 `middleware/error.js`

```js
// middleware/error.js
module.exports = function (err, req, res, next) {
  // Log the exception
  res.status(500).send("Something failed.");
};
```

```js
// index.js
const error = require("./middleware/error");
app.use(error);
```

**注意：此处不是没有调用函数，只是传来一了一个 error 的应用。**目前依然要以来 `try-catch` 来捕捉错误，所以要进一步优化。

## Removing Try Catch Blocks

要将那些高级代码放到单独的函数，在 `genres.js` 先新建一个 async Middleware，这个函数的基本结构是一个 `try-catch` 块。

```js
function asycMiddleware() {
  try {
    // ...
  } catch (ex) {
    next(ex);
  }
}
```

传入路由句柄函数，并在 try 当中调用。这里路由句柄函数是 async 的，所以要稍作修改。

```js
async function asycMiddleware(handler) {
  try {
    await handler();
  } catch (ex) {
    next(ex);
  }
}
```

```js
router.get("/", async asyncMiddleware((req, res) => {
  const genres = await Genre.find().sort("name");
  res.send(genres);
}));
```

把这给路由句柄作为参数传入。但又有新的问题，`request` 和 `response` 怎么传入。我们唯一的参数只有 `handler`。如何访问 `request`，`response` 和 `ex` 呢？

**我们在定义 Express 路由时，\*我们没有调用中中间函数，只是传入了一个引用。**例如

```js
routes.get("/another", (req, res, next) => {});
```

这里我们并没有调用这个 Arrow Function，只是传入了他的引用，是 Express 自己调用了它。**这三个参数也是由 Express 传给它的。**

在这里其实是调用了 async Middleware，我们没有传入一个参数有三个参数的函数，所以要对这个处理函数做修改，我们让它返回一个函数，这样 Express 会自己调用这个返回的函数，然后传入三个参数。这个函数就类似于一个 Factory Function。调用它，返回一个路由句柄函数。

```js
function asycMiddleware(handler) {
  return async (req, res, next) => {
    try {
      await handler(req, res);
    } catch (ex) {
      next(ex);
    }
  };
}
```

这里返回的是 async 函数，所以其本身就不需要为 async 函数。现在新建 `middleware/async.js`，将这个函数放入导出即可。

## Express Async Errors

刚才虽然封装好了错误处理函数，但依然需要在每个地方调用，现在可以利用一个模块，它会自动捕捉异常，当发送请求时，它会自动把路由句柄包裹。

```commonlisp
npm i express-async-errors
```

index.js 调用

```js
require("express-async-errors");
```

现在去掉 genres.js 刚才嵌套的错误处理函数即可。

```js
router.get("/", (req, res) => {
  const genres = await Genre.find().sort("name");
  res.send(genres);
});
```

## Logging Errors

使用 Winston 记录日志。`index.js` 调用，将 logger(记录器)对象储存。

```js
// index.js
const winston = require("winston");
```

这个 logger 对象，有输运特性(Transport)。**输运基本上就是日志记录的介质。**

Winston 有很多记录介质，Console：发送到控制台、File/Http：发送给指定终端地址，也可搭配其他附加的特定 Winston 模块使用，例如 MongoDB。刚才的加载默认导出为 logger。

```js
// index.js
winston.add(winston.transports.File, { filename: "logfile.log" });
```

回到 `error.js`。当捕捉到错误就可以记录了。

```js
// middleware/error.js
const winston = require("winston");

module.exports = function (err, req, res, next) {
  // Log the exception
  winston.log("error", err.message, err);
  // winston.error(err.message, err);

  // error
  // warn
  // info
  // verbose
  // debug
  // silly

  res.status(500).send("Something failed.");
};
```

使用 log，第一个参数为记录等级，从下往上等级越高。然后传入错误信息，也可以传入原始信息作为第三个参数。也可以直接使用 error 方法直接输出错误，这样少一个参数。

回到 `genres.js`，抛出一个错误并测试。

```js
// routes/genres.js
router.get("/", async (req, res, next) => {
  throw new Error("Could not get the genres.");
  //...
});
```

测试以后会发现多一个文件：`logfile.log`。里面的 JSON 记录了详细的错误信息。

## Logging to MongoDB

```com
npm i winston-mongodb
```

回到 index.js，添加运输介质。

```js
// index.js
require("winston-mongodb");
winston.add(winston.transports.MongoDB, { db: "mongodb://localhost/vidly" });
```

这里直接加载整个模块即可，现实开发中，日志记录要单独建一个数据库，此处直接使用同一个 vidly 库。

现在测试一下，打开 MongoDB 就能看到多一个文档 log 用于记录日志。

可以在设定传输介质的时候只记录特定等级的日志。假设记录 info

```js
winston.add(winston.transports.MongoDB, {
  db: "mongodb://localhost/vidly",
  level: "info",
});
```

这里 info 是第三级，意味着比它高级的 error 和 warn 等级的日志都会被记录。

## Uncaught Exceptions

现在添加到错误捕捉逻辑只能捕捉请求流程中的错误，是特别针对 Express 的，发生在 Express 之外的错误是不被捕捉的。例如

```js
// index.js
throw new Error("...");
```

程序会无法运行，但这个错误不会被记录在 `logfile.log`。使用 process 对象处理。

`process` 对象有一个事件发生器(Event Emitter)，它可以产生并发起事件，其有个方法 on 用于监听事件，Node 中有个特定的时间 uncaught Exception，它在 Node 处理过程中出现错误时发起，但我们没有专门捕捉它的 `try-catch`。

```js
process.on("uncaughtException", (ex) => {
  console.log("WE GOT AN UNCAUGHT EXCEPTION.");
  winston.error(ex.message, ex);
  process.exit(1);
});
```

第二个参数为错误处理函数，在里面使用 Winston 记录日志即可。

## Unhandled Promise Rejections

刚才的方法只能处理同步代码，不能处理异步 Promise

```js
// index.js
const p = Promise.reject(new Error("Somethind failed miserably!"));
p.then(() => consol.log("Done."));
```

这就是一个未被处理的被拒 Promise。

利用刚才类似的方法，事件换为 unhandled Rejection。

```js
process.on("unhandledRejection", (ex) => {
  console.log("WE GOT AN UNHANDLED REJECTION.");
  winston.error(ex.message, ex);
  process.exit(1);
});
```

此处 `process.exit(1)`是为了重启引用，0 为成功值，除此之外都为失败值。

还有一种方法，不需要 process 而是利用 Winston 的辅助函数。

```js
winston.handleExceptions(
  new winston.transports.File({ filename: "uncaughtExceptions.log" })
);
// With this, we can handle uncaughtExceptions as well, also unhandledRejection
```

还有另一种。我们直接抛出错误，让被拒 Promise 变为一个未处理的异常。

```js
process.on("unhandledRejection", (ex) => {
  throw ex;
});

// In this way, winston will catch the ex automaticly, so we can hadle both of them in less code.
```

## Extracting the Routes

现在的 `index.js` 太冗杂了，很多功能都集中在一块，现在把功能做单独封装。

新建文件夹 startup，新建文件 `route.js`。里面导出一个函数，并放置所有的路由和中间件。

```js
// startup/route.js
const genres = require("../routes/genres");
const customers = require("../routes/customers");
const movies = require("../routes/movies");
const rentals = require("../routes/rentals");
const users = require("../routes/users");
const auth = require("../routes/auth");
const error = require("../middleware/error");
const express = require("express");

module.exports = function (app) {
  app.use(express.json());
  app.use("/api/genres", genres);
  app.use("/api/customers", customers);
  app.use("/api/movies", movies);
  app.use("/api/rental", rentals);
  app.use("/api/users", users);
  app.use("/api/auth", auth);
  app.use(error);
};
```

这里我们接收一个 app 作为参数，整个应用程序应该只有一个单一的 app 实例，而不是在这另外创建一个

```js
// index.js
const express = require("express");
const app = express();

require("./startup/routes")(app);
```

**注意更改引用地址**

## Extracting the DB Logic

新建 startup/db.js

```js
// startup/db.js
const winston = require("winston");
const mongoose = require("mongoose");

module.exports = function () {
  mongoose
    .connect("mongodb://localhost/vidly")
    .then(() => winston.info("Connected to MongoDB..."));
};
```

这里改用 Winston 做记录，取代 Console，也不需要 catch 块了。因为已经做过处理了。

```js
// index.js
require("./startup/db")();
```

## Extracting the Logging Logic

新建 `startup/logging.js`

```js
// startup/logging.js
const winston = require("winston");
require("winston-mongodb");

module.exports = function () {
  winston.handleExceptions(
    new winston.transports.File({ filename: "uncaughtExceptions.log" })
  );

  process.on("unhandledRejection", (ex) => {
    throw ex;
  });
  winston.add(winston.transports.File, { filename: "logfile.log" });
  winston.add(winston.transports.MongoDB, {
    db: "mongodb://localhost/vidly",
    level: "info",
  }); // Only message level > info will be recored(error warn info),
};
```

```js
// index.js
require("./startup/logging")();
require("./startup/routes")(app);
require("./startup/db")();
```

这里 logging 放在最上面，以确保所有出错都会被捕捉。

## Extracting the Config Logic

新建 `startup/config.js`

```js
// startup/config.js
const config = require("config");

module.exports = function () {
  if (!config.get("jwtPrivateKey")) {
    throw new Error("FATAL ERROR: jwtPrivateKey is not defined");
  }
};
```

取代 console，抛出 Error 异常以由 Winston 记录错误并结束进程。

```js
// index.js
require("./startup/config")();
```

## Extracting the Validation Logic

新建 `startup/validation.js`

```js
// startup/validation.js
const Joi = require("joi");
// A MongoDB ObjectId validator for Joi.
module.exports = function () {
  Joi.objectId = require("joi-objectid")(Joi);
};
```

```js
// index.js
const winston = require("winston");
const express = require("express");
const app = express();

require("./startup/logging")();
require("./startup/routes")(app);
require("./startup/db")();
require("./startup/config")();
require("./startup/validation")();

const port = process.env.PORT || 3000;
app.listen(port, () => winston.info(`Listening on port ${port}...`));
```

现在 `index.js` 简短很多了。把 listen 的 console 改为 Winston 即可。

## Showing Unhandled Exceptions on the Console

我们需要将报错显示到控制台，不然有可能换台电脑，不会正常显示进程终止的原因。

```js
// startup/logging.js
module.exports = function () {
  winston.handleExceptions(
    new winston.transports.Console({ colorize: true, prettyPrint: true }),
    new winston.transports.File({ filename: "uncaughtExceptions.log" })
  );
  // ...
};
```

加上 Console 即可。里面设置了一些选项让可读性变强。

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
