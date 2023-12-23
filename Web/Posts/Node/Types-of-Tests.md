---
title: Types of Tests
date: '2022-8-27'
tags: ['Node', 'Tutorial', 'Test']
draft: false
summary: 'In this article we learn the Types of Tests.'
---

# Types of Tests

## Unit Test

Test a unit of application **without** its **<u>external</u> dependencies**. Such as Files, Databases, Message queues, web services and so on.

Unit Test exercise your code without and external dependencies. They are **Cheap to write** and **Execute fast**. You can run hundreds of them just in few seconds. In this way, you can verify that each building block in your application it’s working as it’s expected.

**However**,since you’re not testing these classes or components with their external dependencies,

you can't get a lot of confidence in the reliability of your application. So, that's when integration tests come to the rescue

## Integration Test

Integration Test tests a class or a component **with** its **<u>external</u> dependencies**. It tests the integration of your application code with these concrete dependencies like Files, Databases and so on.

These tests **take longer to execute**, because they often involve reading or writing to a database, but **they give you more confidence in the health of your application.**

Traditionally an integration test is defined as a test that takes a few units or classes, and test their behavior as a whole. So based on this definition if you test 2 classes together some people believe you're writing an integration test and not a unit test **even if** none of these classes talk to an external resource like a database.

Chances are you've heard this definition before now. This definition is a great recipe for writing fragile tests that are coupled your implementation detail, so as you change the implementation of your classes, these tests are going to break and you will end up wasting a lot of time, fixing that. Not only one they give you any values, but they actually slow you down. if you have done unit testing before an failed. Chances are you've followed this definition.

#### **A unit test tests a class or multiple classes without their external dependencies, they test a unit of work.**

#### **An integration test tests a class or multiple classes with their external dependencies.**

## End to End Test

End to End test drives an application through its user interface.

There are specific tools built for creating end to end tests. One popular tool that you might have heard of is **selenium**, which allows us to record the interaction of a user with our application and then. Laid back and check if the application is returning the right result or not.

These tests give you the greatest amount of confidence about the health of your application, but they have 2 big problems.

he first problem is that they are very slow because they require launching the application and testing it through the UI. So every test is going to launch the application, potential log in, navigate to an internal page submit a form and inspect the results, is very slow.

The second problem is that there are very brittle, because a small enhancement to the application or a small change in the user interface can easily break these tests.

[Node.js: The Complete Guide to Build RESTful APIs (2018) | Udemy](https://www.udemy.com/course/nodejs-master-class/)
