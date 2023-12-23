---
title: Test methods for Unit Test(with Jest)
date: '2022-8-27'
tags: ['Node', 'Tutorial', 'Test']
draft: false
summary: 'In this article we learn how to do unit test wuth Jest.'
---

# Test methods for Unit Test(with Jest)

## Writing Your First Unit Test

First run this command in Terminal

```commonlisp
npm i jest --save-dev
```

Back to `package.json` , write down the test command

```json
"scripts": {
    "test": "jest"
  }
```

If we want to test the file, run `npm test` in Terminal, all the files end with `test|spec.js` will be treated as test file.

We already have a file named `lib.js`, now we create a new folder `Test` and `lib.test.js`

In lib.test.js, we call `test()`, which is a function from jest. We give it two arguments, the first is the name of the test, and that’s what we will see in the console. The second argument is a function where we implement our test. When we run are test, jest will call this function.

```javascript
// lib.test.js
test('Our first test', () => {
  //throw new Error("Something failed.")
})
```

Now we can run `npm test` in Terminal, and see what happen.

## Testing Numbers

In `lib.js` we have a function for testing numbers, and now we write a unit test for this function.

```js
// lib.js
// Testing numbers
module.exports.absolute = function (number) {
  if (number > 0) return number
  if (number < 0) return -number
  return 0
}
```

So how many tests do we need here?

> A basic guideline is that the number of unit tests, you have for a given function should be greater than or equal to the number of execution paths

Here we have three execution paths

- If the number is positive
- If the number is negative
- If the number is zero

In unit test, We want to make sure that all the execution paths in a given function are covered. You are testing all the logic thoroughly.

Now back to lib.test.js and write the test.

```js
// lib.test.js
const lib = require('../../lib')

test('absolute - should return a postive number if input is positive', () => {
  const result = lib.absolute(1)
  expect(result).toBe(1)
})

test('absolute - should return a postive number if input is negative', () => {
  const result = lib.absolute(-1)
  expect(result).toBe(1)
})

test('absolute - should return a zerro if input is zeero', () => {
  const result = lib.absolute(0)
  expect(result).toBe(0)
})
```

We can visit the Jest documents and find more matchers.

If we are testing the floating number

```js
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2
  // expect(value).toBe(0.3) --> This won't work because of rounding error
  expect(value).toBeCloseTo(0.3)
})
```

## Grouping Testing

Currently we have 3 tests for absolute function as we write more tests. It becomes critical to organize this test so they aren’t clean and maintainable.

> **Remember**, test are the first class citizen’s in your source code, they are as important as the production code. It's better not to write tests, which has ugly fat and unmaintainable code. So let's see how we can clean up this code and make it more maintainable ingest.

We use `describe()` for grouping a bunch of related tests, and use `it()` to replace test()

```js
const lib = require('../../lib')

describe('absolute', () => {
  it('should return a postive number if input is positive', () => {
    const result = lib.absolute(1)
    expect(result).toBe(1)
  })

  it('should return a postive number if input is negative', () => {
    const result = lib.absolute(-1)
    expect(result).toBe(1)
  })

  it('should return a zerro if input is zeero', () => {
    const result = lib.absolute(0)
    expect(result).toBe(0)
  })
})
```

## Refactoring with Confidence

One of the benefits of these tests is that they allow us to refactor our code with confidence.

> **Refactoring** means changing the structure of the code without changing its external behavior

```js
// lib.js
// Testing numbers
module.exports.absolute = function (number) {
  if (number > 0) return number
  if (number < 0) return -number
  return 0
}
```

Here’s our absolute function. We have covered all the execution paths in this function. So i can change it.

```js
// lib.js
// Testing numbers
module.exports.absolute = function (number) {
  if (number >= 0) return number
  return -number
}
```

Now we run the test again, if all the tests passed, I will be confident that my new implementation will work.

```js
// lib.js
// Testing numbers
module.exports.absolute = function (number) {
  return number >= 0 ? number : number
}
```

All the test are passed!

## Testing String

```js
// lib.js
// Testing strings
module.exports.greet = function (name) {
  return 'Welcome ' + name
}
```

Now we write a test for this function.

```js
// lib.test.js
const lib = require('../../lib')

describe('greet', () => {
  it('should return the grretting message', () => {
    const result = lib.greet('Mosh')
    expect(result).toBe('Welcome Mosh')
  })
})
```

Now run npm test, we will past the test. However, there is a problem with this test. This test is too specific and can easily break if we change our great function. Just by simply adding an exclamation mark at the end, our test is going to break.

```js
module.exports.greet = function (name) {
  return 'Welcome ' + name + '!'
}
```

So here’s the lesson.

> Your tests should neither be too specific nor to general, there should be at the right balance. If there are too specific like in this case, they can easily break if there are too general. They may not give you the confidence that your code is actually working

So for testing string, we can look for certain patterns. We can use a regular expression

```js
expect(result).toMatch(/Mosh/)
```

Or use another matcher

```js
expect(result).toContain('Mosh') // Use it if you don't want to use Regex
```

## Testing Arrays

```js
// lib.js
// Testing arrays
module.exports.getCurrencies = function () {
  return ['USD', 'AUD', 'EUR']
}
```

Now let’s write the test.

```js
// lib.test.js
const lib = require("../../lib");

describe("getCurreencies", () => {
  it("should return ssupporteed curreencis", () => {
    const result = lib.getCurrencies();
	// expect()......
});
```

**Remember, tests should neither be too general nor too specific**

```js
// Too general
expect(result).toBeDefined()
expect(result).not.toBeNull()
```

This is too general. It doesn’t give us any value, if i create a bug by simply returning, the test will still pass.

```js
// return ["USD", "AUD", "EUR"];
return 1
```

```js
// Too specific
expect(result[0]).toBe('USD')
expect(result[1]).toBe('AUD')
expect(result[2]).toBe('EUR')
expect(result.length).toBe(3)
```

We are testing the exact location of these elements in the array. If tomorrow we change our sorting algorithm and sort this currency in a different way, this test is going to break.

So here is the proper way to test this function.

```js
// Proper way
expect(result).toContain('USD')
expect(result).toContain('AUD')
expect(result).toContain('EUR')
```

But this is not the ideal way, instead of these 3 assertions, we can have one assertion and compare the result of this function with an array that includes.

```js
// Ideal way
expect(result).toEqual(expect.arrayContaining(['EUR', 'USD', 'AUD']))
```

```js
// lib.test.js
const lib = require('../../lib')

describe('getCurreencies', () => {
  it('should return ssupporteed curreencis', () => {
    const result = lib.getCurrencies()

    // Too general
    expect(result).toBeDefined()
    expect(result).not.toBeNull()

    // Too specific
    expect(result[0]).toBe('USD')
    expect(result[1]).toBe('AUD')
    expect(result[2]).toBe('EUR')
    expect(result.length).toBe(3)

    // Proper way
    expect(result).toContain('USD')
    expect(result).toContain('AUD')
    expect(result).toContain('EUR')

    // Ideal way
    expect(result).toEqual(expect.arrayContaining(['EUR', 'USD', 'AUD']))
  })
})
```

## Testing Objects

```js
// lib.js
// Testing objects
module.exports.getProduct = function (productId) {
  return {
    id: productId,
    price: 10,
    // category: "A",
  }
}
```

Now let’s write the test

```js
// Tests are first-class citizens in your source code

const lib = require("../../lib");

describe('getProduct', () => {
  it('should return the product with the given id', () => {
    const result = lib.getProduct(1);
    expect(result).toBe({id : 1 , price : 10})
})

```

Now let’s run npm test. The test won’t pass, because these two objects are in different location in memory. When we use the `toBe()` matcher, this function compares the references of these objects in memory. That’s why are test failed.

> If it should pass with deep equality, replace "toBe" with "toStrictEqual"

```js
expect(result).toStrictEqual({ id: 1, price: 10 })
```

In this case, we want ensure that this this object has these properties, we don’t care about there location in memory. So we use `toEqual()`. With this we can check for object equality.

```js
expect(result).toEqual({ id: 1, price: 10 })
// With this matcher, we can act only the properties that we're interesting in.
```

There is also another matcher: `toMatchObject()`. So how are these approaches different? With the first approach, both the source and target objects should have exactly 2 properties and they should be ID and Price. But sometimes you may get an object from somewhere else and then object might have 50 properties. When testing the equality. You don't want to list all those properties here. In that case. You can use `toMatchObject()`.

```js
// With this matcher, we can act only the properties that we're interesting in.
// For example, is this object has more than 50 properties, we can't write down all the properties in arguments. So this is why we use toMatchObject()
expect(result).toMatchObject({ id: 1, price: 10 })
// There is also another matcher
expect(result).toHaveProperty('id', 1)
```

```js
// lib.test.js
const lib = require('../../lib')

describe('getProduct', () => {
  it('should return the product with the given id', () => {
    const result = lib.getProduct(1)
    // expect(result).toBe({id : 1 , price : 10}) ----> This will be failed, because toBe() will compare this two object location in memory, so we should use toEqual()
    // If it should pass with deep equality, replace "toBe" with "toStrictEqual"
    expect(result).toEqual({ id: 1, price: 10 })
    // With this matcher, we can act only the properties that we're interesting in.
    // For example, is this object has more than 50 properties, we can't write down all the properties in arguments. So this is why we use toMatchObject()
    expect(result).toMatchObject({ id: 1, price: 10 })
    // There is also another matcher
    expect(result).toHaveProperty('id', 1)
  })
})
```

## Testing Exceptions

```js
// Testing exceptions
module.exports.registerUser = function (username) {
  if (!username) throw new Error('Username is required.')

  return { id: new Date().getTime(), username: username }
}
```

Now write the test.

What are false value in JavaScript? `Null`、`Undefined`、`NaN`、`""`、`0`、`false`. To test the function, we need to call this function every time with a different input and ensure that this function throw an exception.

```js
// lib.test.js
const lib = require("../../lib");

describe("registerUser", () => {
  it("should throw if username is falsy", () => {
    const result = lib.registerUser(null);
    expect().toThrow();
});
```

This work won't make sense. Because we won't get the result from this function, so we can't expect the result to throw an exception. So when testing an exceptions, we're using another approaches.

We pass a callback function, instead of passing result.

```js
expect(() => lib.registerUser(null)).toThrow()
```

Now, in this case, our test code is only a single line, so we can easily repeat this expectation with different values. However, in more real world examples, sometimes your test function might be a few lines long. In that case we should use **Parameterized Tests**

> If the Jest version is under 23, it doesn’t support parameterized test. So we have alternative approach.

```js
const args = [null, undefined, NaN, '', 0, false]
args.forEach((a) => {
  expect(() => lib.registerUser(a)).toThrow()
})
```

> If your Jest version is higher than 23, Jest support Parameterized Test. Btw, if you have the lower version, install the jest-each package will work as well.

```js
it.each([null, undefined, NaN, '', 0, false])('', (v) =>
  expect(() => lib.registerUser(a)).toThrow()
)
```

> Read more:
>
> [Jest Parameterised Testing](https://www.jianshu.com/p/428919f2a954)
>
> [jest-each - npm (npmjs.com)](https://www.npmjs.com/package/jest-each)

Now write second test case.

```js
it('should return a user pbject if valid username is passed', () => {
  const result = lib.registerUser('Fan')
  expect(result).toMatchObject({ username: 'Fan' })
  expect(result.id).toBeGreaterThan(0)
})
```

```js
// lib.test.js
describe('registerUser', () => {
  it('should throw if username is falsy', () => {
    // Null
    // undefined
    // Nan
    // ""
    // false
    /* 
      This work won't make sense.
        const result = lib.registerUser(null);
        expect().toThrow();
      Because we won't get the result from this function, so we can't expect the result to throw an exception.
      So when testing an exceptions, we're using another approaches.
    */
    expect(() => lib.registerUser(null)).toThrow()
    // In this test, are test is only an single line. So we can easily repeat this expection with different value.
    // However, in real more example, our test function may be few lines long.
    // In that case, we should use Parameterised Testing.
    // This is the approaches for Parameterised Testing with Jest version under 23.
    const args = [null, undefined, NaN, '', 0, false]
    args.forEach((a) => {
      expect(() => lib.registerUser(a)).toThrow()
    })

    // If your Jest version is 23 or above, or lower version with package jest-each
    // Visit the Unit-Test-Methods.md, or https://www.jianshu.com/p/428919f2a954
  })

  // Parameterised Testing for Jest with virsion hgiher than 23.
  it.each([null, undefined, NaN, '', 0, false])('', (v) =>
    expect(() => lib.registerUser(a)).toThrow()
  )

  it('should return a user pbject if valid username is passed', () => {
    const result = lib.registerUser('Fan')
    expect(result).toMatchObject({ username: 'Fan' })
    expect(result.id).toBeGreaterThan(0)
  })
})
```

## Continuously Running Tests

Open the package.json, change the test command.

```json
"scripts": {
    "test": "jest --watchAll"
  }
```

## Creating Simple Mock Functions

So far, we were working with functions that had no dependencies on external resources.

```js
// db.js
module.exports.getCustomerSync = function (id) {
  console.log('Reading a customer from MongoDB...')
  return { id: id, points: 11 }
}
```

```js
// lib.js
// Mock functions
module.exports.applyDiscount = function (order) {
  const customer = db.getCustomerSync(order.customerId)

  if (customer.points > 10) order.totalPrice *= 0.9
}
```

This `applyDiscount()` function using the `getCustomerSync()` that touches on external resource in this case, a database. **In Unit Test, we should not talk to external resources. Otherwise, you’re writing an integration test.** Now see how to unit test a function that directly or indirectly talks to an external resource.

To unit test this function, we need to replace the real implementation of `getCustomerSync()` method with a fake or mock implementation. This mock implementation looks exactly like this function from the outside. It gets a customer ID and returns a customer object. But the implementation is different. It doesn't talk to a database or a remote HTTP service

```js
// lib.test.js
const lib = require('../../lib')
const db = require('../../db')

describe('applyDiscount', () => {
  it('should apply 10% discount if customer has more than 10 points', () => {
    // Replace real implementation of getCustomerSync method with fake implementation.
    db.getCustomerSync = function (customerId) {
      console.log('Fake reading customer...')
      return { id: customerId, points: 20 }
    }
    const order = { customerId: 1, totalPrice: 10 }
    lib.applyDiscount(order)
    expect(order.totalPrice).toBe(9)
  })
})
```

# Interaction Testing

```js
// lib.js
// Mock functions
module.exports.notifyCustomer = function (order) {
  const customer = db.getCustomerSync(order.customerId)

  mail.send(customer.email, 'Your order was placed successfully.')
}
```

```js
// lib.test.js
const lib = require('../../lib')
const db = require('../../db')
const mail = require('../../mail')

describe('notifyCustomer', () => {
  it('should send an email to the customer', () => {
    db.getCustomerSync = function (customerId) {
      return { email: 'a' }
    }

    let mailSent = false
    mail.send = function (email, message) {
      mailSent = true
    }

    lib.notifyCustomer({ customerId: 1 })

    expect(mailSent).toBe(true)
  })
})
```

When we require a module in node that module is loaded and then cashed in memory. So if we load this Mail module in 5 different places in our application, there will be a single instance of that module loaded in the memory. That’s why this Mail object that we get from this Mail module is exactly the same Mail object that is available.

# Jest Mock Function

In Jest we have a method for creating mock functions, `jest.fn()`.So what we get here is a mock function mock function. This is like a function that we can call like this, but this function has no implementation in other words, it's a function with no code.

```js
const mockFunction = jest.fn()
mockFunction()
```

Now we can program this mark to return a specific value for example. We can call mark function that mark return value.

```js
mockFunction.mockReturnValue(1)
const result = mockFunction() // result = 1
```

We can also program this mark function to return a Promise.

```js
mockFunction.mockRejectedValue(new Error('....'))
// mockFunction.mockResolvedValue(1);
const result = await mockFunction()
```

With this we can simplify the implementation of this test. Now let’s rewrite the test in last chapter and make it simpler.

```js
/* 
  db.getCustomerSync = function (customerId) {
    return { email: "a" };
  }; 
*/
db.getCustomerSync = jest.fn().mockResolvedValue({ email: 'a' })

/* 
  let mailSent = false;
  mail.send = function (email, message) {
    mailSent = true;
  }; 
*/
mail.send = jest.fn()

lib.notifyCustomer({ customerId: 1 })

// expect(mailSent).toBe(true);
expect(mail.send).toHaveBeenCalled()
```

Sometimes you also want to check the arguments that are passed to the method.

```js
expect(mail.send).toHaveBeenCalledWith('a', '...')
```

We shouldn't check for the exact equality. So this approach works well when the arguments you pass through a method are not strings. Like Numbers Boolean's and so on.

If you still want to check the arguments that are passed through that method. You need to use a different approach.

```js
expect(mail.send.mock.calls[0][0]).toBe('a')
expect(mail.send.mock.calls[0][1]).toMatch(/order/)
```

The first [0] means the first call to this function , second [0] means the first argument.

# What to Unit Test

Open up the **Vidly Project**, let’s see what part of this application is a great candidate for unit testing.

```js
// middleware/admin.js

module.exports = function (req, res, next) {
  // 401 Unauthorized
  // 403 Forbidden

  if (!req.user.isAdmin) return res.status(403).send('Access denied.')

  next()
}
```

This function are working with external dependencies, `request` and `response` objects.

When Unit testing this function. We don't want to send a real HTTP request here because that would be an Integration test not a Unit Test.

If you really want to unit test this function we have to mock the request and response arguments here so we need to create mock objects that have the same interface as the request and response objects in express. We have to do a lot of marking and this makes our unit tests very fat and Unmaintainable. So it's better to test this function by Using an Integration test

> If you're doing, too much mocking it's better to write an Integration Test

So what can we unit test in this application?

```js
// models/user.js
userSchema.methods.generateAuthToken = function () {
  const token = jwt.sign({ _id: this._id, isAdmin: this.isAdmin }, config.get('jwtPrivateKey'))
  return token
}
```

We have zero dependency to action or resources in this method, we simply generate a jsonWebToken and return it.

So we can call this method using a unit test get the result verify it's a valid Jason Web token and in the payload. We have those properties.

**Create Folder and File: `tests/unit/models/user.test.js`**

```js
// tests/unit/models/user.test.js
const { User } = require('../../../models/user')
const jwt = require('jsonwebtoken')
const config = require('config')

describe('user.generateAuthToken', () => {
  it('should return a valid JWT', () => {
    const user = new User({ _id: 1, isAdmin: true })
    const token = user.generateAuthToken()
    const decoded = jwt.verify(token, config.get('jwtPrivateKey'))
    expect(decoded).toMatchObject({ _id: 1, isAdmin: true })
  })
})
```

If you run npm test right now, you will find our tests are failed, let’s see what happen.

```nginx
● user.generateAuthToken › should return a valid JWT
```

The reason we get this error is because when we run our tests using Jest, Jest configures NODE_ENV or node environment to test.

Now here in our configuration folder. We don't have a configuration file for our testing environment.

So we add a new file here test.json, copy everything we have in default.json into this file now.

We don't have to set an environment variable on a machine where we want to run our tests. We can simply read the private key from this file.

```json
// config/test.json
{
  "jwtPrivateKey": "1234"
}
```

Now we run npm test, you may will find your test fail again. If your test fail again, like

```commonlisp
Expected:
{"_id": 1, "isAdmin": true}
Received:
{"iat: 1518545670, "isAdmin": true}
```

So what happened to the \_id property? Because_id is type of object ID, we cannot set it to a simple number like one. So when creating this user object, we need to set this property to a valid object ID for that. We need to load Mongoose. And then we can set ID to a Mongoose.Types.ObjectId()

```js
const user = new User({ _id: new mongoose.Types.ObjectId(), isAdmin: true })
```

Now we run npm test, you may will find your test fail again. If your test may still fail again, add the toHexString().

```js
const user = new User({ _id: new mongoose.Types.ObjectId().toHexString(), isAdmin: true })
```

```js
// tests/unit/models/user.test.js
const { User } = require('../../../models/user')
const jwt = require('jsonwebtoken')
const config = require('config')
const mongoose = require('mongoose')

describe('user.generateAuthToken', () => {
  it('should return a valid JWT', () => {
    const payload = { _id: new mongoose.Types.ObjectId().toHexString(), isAdmin: true }
    const user = new User(payload)
    const token = user.generateAuthToken()
    const decoded = jwt.verify(token, config.get('jwtPrivateKey'))
    expect(decoded).toMatchObject(payload)
  })
})
```

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
