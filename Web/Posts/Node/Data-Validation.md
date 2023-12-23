---
title: Data Validation in Express
date: '2022-9-2'
tags: ['Node', 'Tutorial', 'Express', 'MongoDB']
draft: false
summary: 'In this article we learn the Data Validation in Express.'
---

# Data Validation

## Validation

```js
const courseSchema = new mongoose.Schema({
  name: String,
  author: String,
  tags: [String],
  date: { type: Date, default: Date.now() },
  isPublished: Boolean,
})
```

这是一个 Schema，我们规定了一个课程应该具有什么属性，但是这些字段默认都是可选的，所以其实完全可以提交一个空的对象给 MongoDB。MongoDB 不会在意我们有什么属性，所以来看下如何使用 required 验证器来验证数据。

我们先让 name 字段必须

```js
name: { type: String, required: true },
```

现在如果提交一个没有 name 的课程，会提示失败。提示失败，所以我们要记得把提交处理的代码放到 `try catch` 代码块里。

```js
try {
  const result = await course.save()
  console.log(result)
  // await course.validate()
} catch (ex) {
  console.log(ex.message)
}
```

这里有一个另一个注释掉的代码，`course` 的 `validate` 方法，有了它我们不需要另外两行。如果出错他会直接跳转到 catch 方法。**注意：这个方法返回一个空地 Promise 对象，也就是说我们拿不到任何返回值。我们无法拿回一个布尔值做判断，只能使用 callback 的形式。**

```js
// const isValid = await course.validate()
course.validate((err) => {
if (err) { ....... }
})
```

**注意：此处对 name 的验证只在 mongoose 有意义，MongoDB 才不管有什么，所有如果直接在数据库内操作添加数据，没有 name 属性一样会通过。**

## Built in Validators

我们可以在 `required` 里做内建验证器，可以做一个函数，使其返回布尔值。例如，我们只需要价格在已发布的课程内必要。

```js
price: {
    type: Number,
    required: function () {
      // Using function
      return this.isPublished;
    }
  },
```

**注意：这里的 required 不可以使用箭头函数，箭头函数没有自己的 this，所以它会把 this 指向离它最近的对象。mongoose 在别的地方执行这个函数的化，就会无法指向到具体的 course 对象。**

同时，根据属性的不同类型，还有很多附加验证器，例如对于 String 可以限定其长度。

```js
name: {
    type: String,
    required: true,
    minlength: 5,
    maxlength: 255,
    // match: /pattern/,
  },
```

另一个有用的就是 `enum`，可以限定其值只能为特定的几个。新建一个 `category` 属性。

```js
category: {
  type: String,
  required: true,
  enum: ["web", "mobile", "network"],
},
```

现在 `category` 只能为 web、mobile、network 这几个。

对于数字也有他自己的附加验证器。

```js
price: {
    type: Number,
    required: function () {
      // Using function
      return this.isPublished;
    },
    min: 20,
    max: 200,
  },
```

## Custom Validators

```js
tags: [String],
```

对于这个 `tags` 属性，我们要验证他不可以使用 `required` 属性，我们如果就算只给一个空数组，也可以

通过认证。所以我们要自己做验证器，使用 `validate` 对象的 `validator` 属性。

```js
tags: {
  type: String,
  validate: {
    validator: function (v) {
      return v.length > 0;
    },
  },
},
```

我们也可以给它一个验证信息，使用 `message` 属性。

```js
tags: {
  type: String,
  validate: {
    validator: function (v) {
      return v && v.length > 0;
    },
    message: "A course should be have at least one tag",
  },
},
```

## Async Validators

有时候验证逻辑可能需要读取数据库或者远端 HTTP 服务，这时候就需要异步验证器了。看看如何将其改为一个异步验证器。设定 isAsync 为 true，同时我们需要一个 callback 来处理异步的返回结果。

```js
validate: {
    isAsync: true,
    validator: function (v, callback) {
      // Do some async work
      const result = v && v.length > 0;
      callback(result);
    },
    message: "A course should be have at least one tag",
  },
```

```js
tags: {
  type: String,
  validate: {
    isAsync: true,
    validator: function (v, callback) {
      // Do some async work
      const result = v && v.length > 0;
      callback(result);
    },
    message: "A course should be have at least one tag",
  },
},
```

## Validation Errors

我们来让 `try catch` 代码块里的错误处理信息更加相信。

在 mongoose 返回的 `error` 里有 `errors` 属性，在这个属性又可以单独获得特定属性错误。例如

```js
ex.errors.name
```

所以我们可以采用 `for` 来输出错误消息，或者更详细的错误报告，堆栈信息。

```js
try {
  const result = await course.save()
  console.log(result)
  // await course.validate()
} catch (ex) {
  //----------------------------------------------------------------
  for (field in ex.errors) {
    console.log(ex.errors[field].message)
  }
}
```

## Schema Type Options

之前我们已经知道，可以把 Schema 里的属性设置成一个对象，对于特定的类型，还可以有不同的 Schema 属性对数据做处理。例如 lowercase，可以把传入的数据自动传为小写，对应就有 uppercase。还有 `trim` 属性，把字符串的前后空格都删掉。

```js
category: {
  type: String,
  required: true,
  enum: ["web", "mobile", "network"],
  // lowercase: true,
  // uppercase: true,
  // trim:true ------------> Delete blank before and after the string.
},
```

再或者对于 price，我们永远对 price 的数值四舍五入小数部分，就可以使用 `get` 和 `set`

```js
price: {
  type: Number,
  required: function () {
    // Using function
    return this.isPublished;
  },
  min: 20,
  max: 200,
  get: (v) => Math.round(v),
  set: (v) => Math.round(v),
},
```

这样我们在输入数据的时候，`set` 就会自动去掉小数。如果数据库有了有小数的课程，那我们在访问的时候，`get` 就会把请求回来的 price 小数去掉。

## Project - Add Persistence to Genres API

我们在我们之前做到 vidly 中已经有一个 genres 的路由句柄。现在来尝试把数据库移到 MongoDB 后修改路由句柄来对数据库做操作。

```js
// routes/genres.js
const express = require('express')
const router = express.Router()

const genres = [
  { id: 1, name: 'Action' },
  { id: 2, name: 'Horror' },
  { id: 3, name: 'Romance' },
]

router.get('/', (req, res) => {
  res.send(genres)
})

router.post('/', (req, res) => {
  const { error } = validateGenre(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  const genre = {
    id: genres.length + 1,
    name: req.body.name,
  }
  genres.push(genre)
  res.send(genre)
})

router.put('/:id', (req, res) => {
  const genre = genres.find((c) => c.id === parseInt(req.params.id))
  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  const { error } = validateGenre(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  genre.name = req.body.name
  res.send(genre)
})

router.delete('/:id', (req, res) => {
  const genre = genres.find((c) => c.id === parseInt(req.params.id))
  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  const index = genres.indexOf(genre)
  genres.splice(index, 1)

  res.send(genre)
})

router.get('/:id', (req, res) => {
  const genre = genres.find((c) => c.id === parseInt(req.params.id))
  if (!genre) return res.status(404).send('The genre with the given ID was not found.')
  res.send(genre)
})

function validateGenre(genre) {
  const schema = Joi.object({
    name: Joi.string().min(3).required(),
  })
  return schema.validate(genre)
}

module.exports = router
```

首先要使用 mongoose 连接到 MongoDB，但我们不在 `genres.js` 里直接连接，因为这只是一个负责路由的文件，要在 `index.js` 中连接

```js
// index.js
const mongoose = require('mongoose')

mongoose
  .connect('mongodb://localhost/vidly', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => console.log('Connected to MongoDB...'))
  .catch((err) => console.log('Could not connect to MongoDB...', err.message))
```

这个时候回到 `genres.js`，加载 `mongoose`。先来定义一个 Schema，这里不需要赋予 ID 属性，数据库会自动赋予。

然后生成出一个 `Genre` 的 Model，这里我们可以直接写在一起，不需要单独的 genre Schema 变量。

```js
const Genre = mongoose.model(
  'Genre',
  new mongoose.Schema({
    name: { type: String, required: true, minlength: 5, maxlength: 50 },
  })
)
```

现在来处理第一个 `get` 句柄。

```js
router.get('/', async (req, res) => {
  const genres = await Genre.find().sort('name')
  res.send(genres)
})
```

再来处理 `post` 句柄。

```js
router.post('/', async (req, res) => {
  const { error } = validateGenre(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  let genre = new Genre({ name: req.body.name })
  genre = await genre.save()

  res.send(genre)
})
```

再到 `put`。

```js
router.put('/:id', async (req, res) => {
  const { error } = validateGenre(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  const genre = await Genre.findByIdAndUpdate(req.params.id, { name: req.body.name }, { new: true })

  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  res.send(genre)
})
```

再到 `delete`。

```js
router.delete('/:id', async (req, res) => {
  const genre = await Genre.findByIdAndRemove(req.params.id)
  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  res.send(genre)
})
```

最后还有一个得到单一分类的 `get` 句柄。

```js
router.get('/:id', async (req, res) => {
  const genre = await Genre.findById(req.params.id)

  if (!genre) return res.status(404).send('The genre with the given ID was not found.')
  res.send(genre)
})
```

现在尝试着启动，然后在 Postman 里测试吧。

```js
// routes/genres.js
const Joi = require('joi')
const mongoose = require('mongoose')
const express = require('express')
const router = express.Router()

const Genre = mongoose.model(
  'Genre',
  new mongoose.Schema({
    name: { type: String, required: true, minlength: 5, maxlength: 50 },
  })
)

router.get('/', async (req, res) => {
  const genres = await Genre.find().sort('name')
  res.send(genres)
})

router.post('/', async (req, res) => {
  const { error } = validateGenre(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  let genre = new Genre({ name: req.body.name })
  genre = await genre.save()

  res.send(genre)
})

router.put('/:id', async (req, res) => {
  const { error } = validateGenre(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  const genre = await Genre.findByIdAndUpdate(req.params.id, { name: req.body.name }, { new: true })

  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  res.send(genre)
})

router.delete('/:id', async (req, res) => {
  const genre = await Genre.findByIdAndRemove(req.params.id)
  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  res.send(genre)
})

router.get('/:id', async (req, res) => {
  const genre = await Genre.findById(req.params.id)

  if (!genre) return res.status(404).send('The genre with the given ID was not found.')
  res.send(genre)
})

function validateGenre(genre) {
  const schema = Joi.object({
    name: Joi.string().min(3).required(),
  })
  return schema.validate(genre)
}

module.exports = router
```

## Project - Build the Customers API

创建一个管理用户的终端，有几个属性。

```
isGold
name
phone
```

首先新建 `routes/customers.js`，然后在 `index.js` 引用。

```js
// index.js
// ...
const customers = require('./routes/customers')
app.use('/api/customers', customers)
// ...
```

创建 `customers` 和 `genres` 类似，所以可以复制后在做更改，一样先建 Schem 和 Model。

```js
const Customer = mongoose.model(
  'Customer',
  new mongoose.Schema({
    name: { type: String, required: true, minlength: 5, maxlength: 50 },
    isGold: {
      type: Boolean,
      default: false,
    },
    phone: {
      type: String,
      required: true,
      minlength: 5,
    },
  })
)
```

同时建一个新的 `validate` 函数。

```js
function validateCustomers(customer) {
  const schema = Joi.object({
    name: Joi.string().min(5).max(50).required(),
    phone: Joi.string().min(5).max(50).required(),
    isGold: Joi.boolean(),
  })
  return schema.validate(customer)
}
```

剩下的就是对具体的句柄做简单的更改了。

```js
// routes/customers.js
const Joi = require('joi')
const mongoose = require('mongoose')
const express = require('express')
const router = express.Router()

const Customer = mongoose.model(
  'Customer',
  new mongoose.Schema({
    name: { type: String, required: true, minlength: 5, maxlength: 50 },
    isGold: {
      type: Boolean,
      default: false,
    },
    phone: {
      type: String,
      required: true,
      minlength: 5,
    },
  })
)

router.get('/', async (req, res) => {
  const customers = await Customer.find().sort('name')
  res.send(customers)
})

router.post('/', async (req, res) => {
  const { error } = validateCustomers(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  let customer = new Customer({
    name: req.body.name,
    phone: req.body.phone,
    isGold: req.body.isGold,
  })
  customer = await customer.save()

  res.send(customer)
})

router.put('/:id', async (req, res) => {
  const { error } = validateCustomers(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  const customer = await Customer.findByIdAndUpdate(
    req.params.id,
    { name: req.body.name, phone: req.body.phone, isGold: req.body.isGold },
    { new: true }
  )

  if (!customer) return res.status(404).send('The customer with the given ID was not found.')

  res.send(customer)
})

router.delete('/:id', async (req, res) => {
  const customer = await Customer.findByIdAndRemove(req.params.id)
  if (!customer) return res.status(404).send('The customer with the given ID was not found.')

  res.send(customer)
})

router.get('/:id', async (req, res) => {
  const customer = await Customer.findById(req.params.id)

  if (!customer) return res.status(404).send('The customer with the given ID was not found.')
  res.send(customer)
})

function validateCustomers(customer) {
  const schema = Joi.object({
    name: Joi.string().min(5).max(50).required(),
    phone: Joi.string().min(5).max(50).required(),
    isGold: Joi.boolean(),
  })
  return schema.validate(customer)
}

module.exports = router
```

## Restructuring the Project

现在我们的 `customers.js` 比较短，只有 100 行左右，但是在真实开发中，我们可能光是 Customer 的 Model 创建就超过 100 多行了，所以我们要重构，要确保 routes 下的文件只负责处理路由。所以我们新建 models 文件夹，存放各种类型的模型。在这之下新建 `customer.js`。移植 Model 和 Validators。

```js
// models/customer.js
const Joi = require('joi')
const mongoose = require('mongoose')

const Customer = mongoose.model(
  'Customer',
  new mongoose.Schema({
    name: { type: String, required: true, minlength: 5, maxlength: 50 },
    isGold: {
      type: Boolean,
      default: false,
    },
    phone: {
      type: String,
      required: true,
      minlength: 5,
    },
  })
)

function validateCustomers(customer) {
  const schema = Joi.object({
    name: Joi.string().min(5).max(50).required(),
    phone: Joi.string().min(5).max(50).required(),
    isGold: Joi.boolean(),
  })
  return schema.validate(customer)
}

exports.Customer = Customer
exports.validate = validateCustomers
```

再回到 `customers.js` 引用。

```js
// routes/customers.js
const { Customer, validate } = require('../models/customer')
const mongoose = require('mongoose')
const express = require('express')
const router = express.Router()

router.get('/', async (req, res) => {
  const customers = await Customer.find().sort('name')
  res.send(customers)
})

router.post('/', async (req, res) => {
  const { error } = validate(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  let customer = new Customer({
    name: req.body.name,
    phone: req.body.phone,
    isGold: req.body.isGold,
  })
  customer = await customer.save()

  res.send(customer)
})

router.put('/:id', async (req, res) => {
  const { error } = validate(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  const customer = await Customer.findByIdAndUpdate(
    req.params.id,
    { name: req.body.name, phone: req.body.phone, isGold: req.body.isGold },
    { new: true }
  )

  if (!customer) return res.status(404).send('The customer with the given ID was not found.')

  res.send(customer)
})

router.delete('/:id', async (req, res) => {
  const customer = await Customer.findByIdAndRemove(req.params.id)
  if (!customer) return res.status(404).send('The customer with the given ID was not found.')

  res.send(customer)
})

router.get('/:id', async (req, res) => {
  const customer = await Customer.findById(req.params.id)

  if (!customer) return res.status(404).send('The customer with the given ID was not found.')
  res.send(customer)
})

module.exports = router
```

现在代码更简短，而且更明确，对 genres 做同样处理。

```js
// models/genre.js
const Joi = require('joi')
const mongoose = require('mongoose')

const Genre = mongoose.model(
  'Genre',
  new mongoose.Schema({
    name: { type: String, required: true, minlength: 5, maxlength: 50 },
  })
)

function validateGenre(genre) {
  const schema = Joi.object({
    name: Joi.string().min(3).required(),
  })
  return schema.validate(genre)
}

exports.Genre = Genre
exports.validate = validateGenre
```

```js
// routes/genres.js
const { Genre, validate } = require('../models/genre')
const mongoose = require('mongoose')
const express = require('express')
const router = express.Router()

router.get('/', async (req, res) => {
  const genres = await Genre.find().sort('name')
  res.send(genres)
})

router.post('/', async (req, res) => {
  const { error } = validate(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  let genre = new Genre({ name: req.body.name })
  genre = await genre.save()

  res.send(genre)
})

router.put('/:id', async (req, res) => {
  const { error } = validate(req.body)
  if (error) return res.status(400).send(error.details[0].message)

  const genre = await Genre.findByIdAndUpdate(req.params.id, { name: req.body.name }, { new: true })

  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  res.send(genre)
})

router.delete('/:id', async (req, res) => {
  const genre = await Genre.findByIdAndRemove(req.params.id)
  if (!genre) return res.status(404).send('The genre with the given ID was not found.')

  res.send(genre)
})

router.get('/:id', async (req, res) => {
  const genre = await Genre.findById(req.params.id)

  if (!genre) return res.status(404).send('The genre with the given ID was not found.')
  res.send(genre)
})

module.exports = router
```

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
