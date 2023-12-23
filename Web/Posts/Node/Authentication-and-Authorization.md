---
title: Authentication and Authorization with Node & MongoDB
date: "2022-9-1"
tags: ["Node", "Tutorial", "Express", "MongoDB"]
draft: false
summary: "In this article we learn how to do Authentication and Authorization with Node & MongoDB."
---

# Authentication and Authorization

## Introduction

目前已创建四个 API： genres、movies、customers、rentals。

几乎所有的应用都需要进行认证和授权。其中

**认证(Authentication)：**就是验证一个用户是不是它声明的身份的过程。我们把用户名和密码发给服务器，服务器验证我是不是那个人。

**授权(Authorization)：**就是判断用户是否有做某件事情的权利，比如，在 vidly 程序中，我们只允许登陆成功的用户修改电影内容，或者是由管理员有删除权限。

为了达到这个效果，我们要新建两个终端。首先就是要让用户可以注册、登录

```js
// Register: POST /api/users
// Login: POST /api/logins
```

## Creating the User Model

首先定义 User Model 新建 `models/user.js`

```js
// models/user.js

const mongoose = require("mongoose");
const Joi = require("joi");
const config = require("config");

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    minlength: 5,
    maxlength: 50,
  },
  email: {
    type: String,
    unique: true,
    required: true,
    minlength: 5,
    maxlength: 255,
  },
  password: {
    type: String,
    required: true,
    unique: true,
    required: true,
    minlength: 5,
    maxlength: 1024,
  },
});

const User = mongoose.model("User", userSchema);

function validateUser(user) {
  const schema = Joi.object({
    name: Joi.string().min(5).max(50).required(),
    email: Joi.string().min(5).max(255).required().email(),
    password: Joi.string().min(5).max(255).required(),
  });

  return schema.validate(user);
}

exports.User = User;
exports.validate = validateUser;
```

**这里 Schema 中 email 设定了 unique 为 true，是为了保证数据库中不会有相同邮箱。**

## Registering Users

现在创建用于注册用户的新路由，新建 `routes/users.js`

```js
// routes/users.js
const { User, validate } = require("../models/user");
const mongoose = require("mongoose");
const express = require("express");
const router = express.Router();

router.post("/", async (req, res) => {});

modules.exports = router;
```

然后回 `index.js` 把这个路由添加

```js
// index.js
const users = require("./routes/users");
app.use("/api/users", users);
```

现在来具体实现路由规则。首先验证输入内容，不合法返回 400，通过就创建新的用户。

```js
router.post("/", async (req, res) => {
  const { error } = validate(req.body);
  if (error) return res.status(400).send(error.details[0].message);

  // Make sure the user is not already registered
  let user = await User.findOne({ email: req.body.email });
  if (user) return res.status(400).send("User already registered.");

  user = new User({
    name: req.body.name,
    email: req.body.email,
    password: req.body.password,
  });

  await user.save();

  res.send(user);
});
```

现在运行程序并尝试发送一个请求吧，可以尝试不同的错误格式内容，来验证我们的 Data Validation 是否可行。

但你会发现，他会把密码一并发回，这并不是我们想要的结果。

## Using Lodash

我们可以限制返回发送的内容，比如

```js
res.send({
  name: user.name,
  email: user.email,
});
```

但我们也可以使用一个库，**Lodash**，他给了我们非常多操作对象的有用工具。我们用 node 安装后使用它。

首先引入

```js
const _ = require("lodash");
```

这里为了方便使用下划线做变量名，当然也可以用别的名字，比如就叫 lodash。这其中有一个方法，`pick`。

我们给 `pick` 一个对象，然后给一个我们需要属性为元素的数组，这样会返回一个新的对象只包含这些键值对。

```js
res.send(_.pick(user, ["_id", "name", "email"]));
```

同样可以替换上面创建 user 的部分。

```js
user = new User(_.pick(req.body, ["name", "email", "password"]));
```

```js
// Using Joi to validate input
const schema = Joi.object({
  name: Joi.string().min(3).required(),
});
const result = schema.validate(req.body);
if (result.error) {
  res.status(400).send(result.error);
}
```

> **Joi 有一个用于验证密码安全的库，joi-password-complexity，限定大小写，符号等**

我们现在的密码都还是明文保存，很不安全，来看看如何哈希他们。

## Hashing Password

要使用哈希我们安装一个库 bcrypt（bcrypt 报错可以用 bvrypt.js）

先来看下他怎么用。新建一个 `hash.js` 做测试。首先引入。

```js
// hash.js
const bcrypt = require("bcrypt");
```

为了哈希密码，我们需要“加点盐(Salt)“，什么是 Salt？

> 假设有密码 1234，我们假设哈希的结果为 abcd，哈希算法是单向的，我们得到 abcd 不可以反得到 1234，如果有人骇进数据库看到 abcd，是得不到 1234 的。但是它可以用一些常见的密码来组合来算哈希值，这样他就破解了。所以我们需要 Salt，Salt 其实就是一个随机字符串，会前置或者后置与我们的密码，每次的 Salt 都是随机的，这样每次哈希的结果都会不一样。

我们使用 `genSalt` 方法，里面的参数是我们让它用多少次算法算出 Salt 的值，数字越大算的越久，也更难破解，默认为 10。

```js
async function run() {
  const salt = await bcrypt.genSalt(10);
  console.log(salt);
}

run();
```

运行就可以看到一个 Salt 值。

再看如何做哈希。使用 `hash` 方法

```js
async function run() {
  const salt = await bcrypt.genSalt(10);
  const hashed = await bcrypt.hash("1234", salt);
  console.log(salt);
  console.log(hashed);
}

run();
```

第一个参数是要加密的内容，第二个参数是 Salt 值

```cd
$2b$10$Kxj9jsuHKWKoVxEjhzK2Ve
$2b$10$Kxj9jsuHKWKoVxEjhzK2Vecf1S9KUkFst0WoTWtUsQslPl4pKKrGC
```

第一个为 Salt，第二个就是 Hash 后的结果，Salt 值是在 Hash 结果的头部。因为验证的时候，我们要把明文哈希一次，也要知道 Salt 值是多少。

现在来把关键的两句移到 `users` 路由句柄中。

```js
// routes/users.js
// .....
user = new User(_.pick(req.body, ["name", "email", "password"]));
const salt = await bcrypt.genSalt(10);
user.password = await bcrypt.hash(user.password, salt);

user = await user.save();
// .....
```

现在密码就加密了，可以尝试启动程序，创建新用户，看看 MongoDB Compass 里用户密码的部分

## Authenticating Users

现在来做验证用户，新建 `routes/auth.js`。先把 `users.js` 内容复制，然后在 `index.js` 启用。

```js
// routes./auth.js
const _ = require("lodash");
const { User, validate } = require("../models/user");
const mongoose = require("mongoose");
const bcrypt = require("bcrypt");
const express = require("express");
const Joi = require("joi");
const router = express.Router();

router.post("/", async (req, res) => {
  const { error } = validate(req.body);
  if (error) return res.status(400).send(error.details[0].message);

  // Make sure the user is not already registered
  let user = await User.findOne({ email: req.body.email });
  if (user) return res.status(400).send("User already registered.");

  user = new User(_.pick(req.body, ["name", "email", "password"]));
  const salt = await bcrypt.genSalt(10);
  user.password = await bcrypt.hash(user.password, salt);

  user = await user.save();
  res.send(_.pick(user, ["_id", "name", "email"]));
});
module.exports = router;
```

```js
// index.js
const auth = require("./routes/auth");
app.use("/api/auth", auth);
```

我们现在用的 validate 函数是验证 User 的，我们只需要验证用户名和密码就可以。所以写个新的 validator。

```js
function validate(req) {
  const schema = Joi.object({
    email: Joi.string().min(5).max(255).required().email(),
    password: Joi.string().min(5).max(255).required(),
  });

  return schema.validate(req);
}
```

然后验证邮箱是否存在

```js
let user = await User.findOne({ email: req.body.email });
if (user) return res.status(400).send("Invalid email or password.");
```

此处不写邮箱错误是为了防止被人恶意尝试邮箱暴力破解。

然后验证密码，使用 bcrypt 的 `compare`

```js
const validPassword = await bcrypt.compare(req.body.password, user.password);
if (!validPassword) return res.status(400).send("Invalid email or password");
res.send(true);
```

如果不通过，返回错误信息，否则发送 `true` 回 client。

## JSON Web Tokens

现在在用户验证通过后只会发送一个 true，现在改为发送一个 JSON Web Token，JWT 本质是一个长字符串，可以用来验证用户的身份。**当一个用户登陆成功，服务器就会产生一个 JWT，就像一个护照一样，然后给回用户，下次用户再请求无论什么终端地址，客户端都会把这个 JWT 本地储存然后在下次请求的时候发还服务器。**

我们可以使用 JWT，访问 jwt.io，输入一个已经写好的 JWT 进行解码。

![JWT](//Images/Node/JWT.png)

可以看到 JWT 包含三个部分，第一部分红色，包含 JWT 的基本信息，例如利用了什么算法。第二部分则是主要的，**用户可以公开的明文信息**，例如用户名等等，这里可以包含简单的信息。这样每次发回服务器以后，服务器就可以轻松知道一些基本信息例如 ID，名字，不需要再回到数据库去查。我们也可以设定用户是否为 admin。第三部分是数字签名，与装载项的信息和私钥一起生成，这个私钥只在服务器上有，所以如果有人得到这个 JWT 并且尝试修改，例如修改 admin，数字签名会无法得到验证。就可以避免被修改。

## Generating Authentication Tokens

NPM 安装 `jsonwebtoken`。在 `auth.js` 中使用。

```js
const jwt = require("jsonwebtoken");
```

使用 `sign` 方法，在里面放装载项，还有一个私钥。**此处先采用硬编码，应该要放到环境变量。**

```js
const token = jwt.sign({ _id: user.id }, "jwtPrivateKey");
res.send(token);
```

测试就会发现它返回一个 JWT，可以到 jwt.io 取解码验证。

## Storing Secrets

使用 `config` 将私钥保存至环境变量。

新建 `config/default.json`

```json
{
  "jwtPrivateKey": ""
}
```

此处是空值，只是为应用设置了一个模板。再创建 `custom-environment-variables.json`。这个文件将映射应用设置和环境变量的关系。

```json
{
  "jwtPrivateKey": "vidly_jwtPrivateKey"
}
```

回到 `auth.js`，加载 config。修改刚才的方法。

```js
const token = jwt.sign({ _id: user.id }, config.get("jwtPrivateKey"));
res.send(token);
```

然后回到 `index.js`，我们要确保我们的环境变量设置好了，程序再正常工作。

```js
// index.js
const config = require("config");

if (!config.get("jwtPrivateKey")) {
  console.error("FATAL ERROR: jwtPrivateKey is not defined");
  process.exit(0);
}
```

现在运行程序会发现无法运行，设置环境变量。Terminal 输入

```cmd
set vidly_jwtPrivateKey=mySourceKey
```

## Setting Response Headers

如何让用户注册成功就直接登录呢，注册成功后，不用再次进行登录。现在去到 users.js，我们注册的路由。

```js
res.send(_.pick(user, ["_id", "name", "email"]));
```

我们可以选择将 JWT 作为另一个属性，一起发回，但是 JWT 并不是 user 的一个属性，最好的是把 JWT 放在 header 返回。

先将 `auth.js` 里 token 生成的代码复制，然后对于使用 `res` 的 `header` 方法，对自定义的头部属性，都应该在属性前加 `x-` ，第二参数就是内容。

```js
const config = require("config");
const jwt = require("jsonwebtoken");

const token = jwt.sign({ _id: user.id }, config.get("jwtPrivateKey"));
res.header("x-auth-token", token).send(_.pick(user, ["_id", "name", "email"]));
```

使用 postman 做测试，打开 Header 部分，就可以看到返回的 JWT 了。

## Encapsulating Logic in Models

现在我们在 `auth.js` 和 `users.js` 里有俩段一样的获取 Token 的代码，现在的 token 只需要蕴含 id 信息，如果以后还有更多信息，那修改的地方就会越来越多，所以要封装他们。

可以新建一个模块，然后弄个新函数，`generateAuthToken`，这样可以，但是往后可能这样的函数会越来越多。

在 OOP 中有个原则，信息专精原则(Information Expert Principle)(源自软件工程学中的 GRASP)，**一个对象只要有足够的信息，他就是某个给定域的专精，这个对象要负责做出决定或完成任务。**就像一个大厨，他有所有做菜的知识，这就是为什么他是专门做菜的，而不是一个服务员去做。

所以我们这些功能，应该封装到 User Object 里面，类似这种

```js
const token = user.generateAuthToken();
```

如何实现？回到 `user.js`，将 Schema 单独提取为 user Schema

```js
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    minlength: 5,
    maxlength: 50,
  },
  email: {
    type: String,
    unique: true,
    required: true,
    minlength: 5,
    maxlength: 255,
  },
  password: {
    type: String,
    required: true,
    unique: true,
    required: true,
    minlength: 5,
    maxlength: 1024,
  },
});
```

```js
// modles/user.js
const jwt = require("jsonwebtoken");
const config = require("config");

userSchema.methods.generateAuthToken = function () {
  return jwt.sign({ _id: this.id }, config.get("jwtPrivateKey"));
};
```

使用 Schema 中的 methods 属性，来添加方法，这里获取 id 直接使用 this，也就是这个 Object 自己。所以不可以用 Arrow Function，因为 Arrow Function 没有自己的 this。

运行程序并验证。

## Authorization Middleware

现在要设计保护数据不被轻易的修改，只有在通过认证才可以做更改，来到 `genres.js`，要让 POST 只有通过认证后才可以操作。

```js
router.post("/", async (req, res) => {
  const token = req.header("x-auth-token");
  //...
});
```

这是一个大的方向，但我们不想再所有数据修改请求前面都这样重复，所以把这个逻辑放入中间件。

新建 `middleware/auth.js` ,把基本这一段码复制过去

```js
// middleware/auth.js
const jwt = require("jsonwebtoken");
const config = require("config");

module.exports = function (req, res, next) {
  const token = req.header("x-auth-token");
  if (!token) return res.status(401).send("Access denied. No token provided.");
};
```

使用 JWT 的 `verify` 方法，第一参数给它 Token，第二参数这是解码令牌的私钥。如果合法，则解码并返回加载项，不合法就会抛出异常，所以把它放到 `try-catch` 里

```js
try {
  const decoded = jwt.verify(token, config.get("jwtPrivateKey"));
  // verify method will return an Error if fail
} catch (ex) {
  res.status(400).send("Invalid token.");
}
```

我们解码过后，就可以把他放到 `request` 中去了。**（这是一个中间件，把它放到 request 里交给下一个中间件取进一步处理。）**

```js
req.user = decoded;
```

之前只在装载项里得到了用户的 ID

```js
// modles/user.js
userSchema.methods.generateAuthToken = function () {
  return jwt.sign({ _id: this.id }, config.get("jwtPrivateKey"));
};
```

我们把它放到 `request` 中作为 `user` 对象，这样就可以以 `request.user.\_id` 来做访问，然后需要将控制权给到下一个，使用 `next()`。

```js
req.user = decoded;
next();
```

```js
// middleware/auth.js
const jwt = require("jsonwebtoken");
const config = require("config");

module.exports = function (req, res, next) {
  const token = req.header("x-auth-token");
  if (!token) return res.status(401).send("Access denied. No token provided.");

  try {
    const decoded = jwt.verify(token, config.get("jwtPrivateKey"));
    // verify method will return an Error if fail
    req.user = decoded;
    next();
  } catch (ex) {
    res.status(400).send("Invalid token.");
  }
};
```

## Protecting

现在写好了中间件，但不在 `index.js` 使用，因为不是所有终端都需要得到这个保护。回到 `genres.js`。这里 post 语句的第二个参数可以写入中间件。

```js
// routes/genres
const auth = require("../middleware/auth");
// Second argument is optionally mddleware.
router.post("/", auth, async (req, res) => {
  // ....
});
```

## Getting the Current User

很多时候，我们都需要获取当前已经登录的用户，所以现在新建一个终端。来到 `users.js`

```js
// routes/users.js
router.get("/:id");
```

我们可以在路由带上 ID，但这意味着客户端需要发送用户的 ID，出于安全角度考虑，不能这样做，如果我发送另一个用户的 ID，就能看到不该看的东西了。常见做法是，有一个类似 me 的终端地址，Client 就不用发送 ID，可以从 JWT 获得，而且 JWT 不可以伪造。

```js
// routes/users.js
const auth = require("../middleware/auth");
// Getting current user

router.get("/me", auth, async (req, res) => {
  const user = await User.findById(req.user._id).select("-password");
  res.send(user);
});
```

使用之前写好的验证中间件，解析 Header 的 Token，返回 ID，然后通过 ID 查找用户，就可以知道现在是谁在登录。返回时排除掉密码选项。

## Logging Out Users

用户退出，不需要写一个新的终端。Token，没有存于服务器。客户要退出只需要删除储存的 Token 即可，不应该在服务器储存 Token，若有人骇入等于直接拿着其身份，不需要密码。若一定要储存，要哈希加密。

当 Client 将 Token 发送给 Server 时候，使用 HTTP 保护加密。

## Role based Authorization

目前完成了验证和授权，现在优化程序，加深现在的操作，例如删除，是只有管理员才可以做的操作，即实现角色授权。

先到 `user.js`，修改 User Schema

```js
// models/user.js
const userSchema = new mongoose.Schema({
  name: {
    //...
  },
  email: {
    //...
  },
  password: {
    //...
  },
  isAdmin: Boolean,
});
```

现在在 MongoDB Compass 随便选一个用户添加 is Admin 属性并设定为 true。

登陆的时候，要将这个属性放到 JWT 中，这样 Server 收到后可以直接知道此用户是否为 Admin。

```js
// models/user.js
userSchema.methods.generateAuthToken = function () {
  return jwt.sign(
    { _id: this.id, isAdmin: this.isAdmin },
    config.get("jwtPrivateKey")
  );
};
```

现在需要一个新的 Middleware 来检测是否为管理员，新建 `middleware/admin.js`。

```js
// middleware/admin.js
module.exports = function (req, res, next) {
  // 401 : Unauthorized
  // 403 : Forbidden
  if (!req.user.isAdmin) return res.status(403).send("Access denied.");

  next();
};
```

这个函数将在登录验证函数后运行，在 request 中已经设置了 user，所以检测 is Admin 就可以。不是则返回 403，意为禁止。

现在修改 `genres.js` 修改 delete 句柄。第二个参数传入数组，添加两个 Middleware 即可。

```js
// routes.genres.js
const admin = require("../middleware/admin");
router.delete("/:id", [auth, admin], async (req, res) => {
  // ...
});
```

完成功能后验证即可。

最后，目前只需要查看是否为管理员，大型应用可能有很多其他的角色，例如版主等等，此时可以选择在 Schema 添加一个 roles 属性，为一个数组。储存用户是什么身份。或者直接设置一个 operations 数组，写明用户可以进行什么操作。

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
