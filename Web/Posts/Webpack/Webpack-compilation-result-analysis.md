---
title: Webpack compilation result analysis
date: '2023-9-27'
tags: ['webpack', 'tutorial', 'frontend engineering']
draft: false
summary: 'In this article we analyze the simple Webpack compilation result'
---

# Webpack compilation result analysis

**Webpack** 的作用是将 **Common JS** 或 **ES Module** 的模块进行打包，合并，转换为合适生产使用的文件产物，本文将对于 **Webpack** 如何处理 **Common JS** 的模块处理。

## Preparation

使用的 **Webpack** 版本如下

```json
{
  "devDependencies": {
    "webpack": "^4.41.5",
    "webpack-cli": "^3.3.10"
  }
}
```

打包命令采用

```bash
webpack --mode=development
```

文件准备

```js
// ./src/a.js

console.log("module a");
module.exports = "a";
```

```js
// ./src/index.js

console.log("index module");
var a = require("./a");
a.abc();
console.log(a);
```

## How Webpack handle with Common JS module

首先，Common JS 是如何处理模块化的呢？他会把每一个模块放在一个函数来运行一次。所以最简单的把这两个文件放到一个文件里面就是下面这样

```js
// dist/my-main.js

console.log("index module");

/*
    console.log("module a");
    module.exports = "a";
*/

var a = require("./a");
a.abc();
console.log(a);
```

这里运行到 `require` 就运行 `a.js` 的内容，运行到 `module.export = "a"` 导出 `"a"`，那么最后这段代码就变成了

```js
// dist/my-main.js

console.log("index module");

console.log("module a");

var a = "a";
console.log(a);
```

很明显这样是不行的，他依然污染了全局变量，那么 `Webpack` 是怎么做的呢

它会建立一个 `modules` 的 obj，以文件路径为 key，将文件内的执行代码放到一个函数里

```js
// dist/my-main.js

var modules = {
  // 此对象保存了所有的模块，以及模块对应的代码
  "./src/a.js": function (module, exports) {
    console.log("module a");
    module.exports = "a";
  },
  "./src/index.js": function (module, exports, require) {
    console.log("index module");
    var a = require("./src/a.js");
    console.log(a);
  },
};
```

在这里我们在 `function` 里传入第一个参数为 `module`，因为 在语句中使用到了 `module.exports`。第二个参数为 `exports，` 是因为文件内可能会有如 `exports.a = 1` 这样的代码。第三个参数为 `require` 函数，很明显是使用在 `var a = require("./src/a.js");` 的地方，同时，可以发现， 我们把 `require("./a");` 变成 `require("./src/a.js");` ，这样子的话当遇到了 `require` 函数就可以直接根据此字符串从 `modules` 对象通过 `key` 去到执行代码。

**那么我们应该把 modules 交给谁来处理呢？**

我们可以写一个函数，来进行处理。

```js
// dist/my-main.js

var modules = {
  // ...
}(function (modules) {})(modules);
```

我们使用一个立即执行函数来处理，这里又可以发现，我们声明了 `var modules` 也会污染全局变量，所以可以直接把这个对象塞到参数里面

```js
// dist/my-main.js
(function (modules) {})({
  "./src/a.js": function () {
    /**/
  },
  "./src/index.js": function () {
    /**/
  },
});
```

在这个函数里面，我们要执行入口文件，怎么执行入口文件呢?

```js
// dist/my-main.js
(function (modules) {
  // 执行入口文件
  require("./src/index.js"); // require函数相当于是运行一个模块，并得到模块导出结果
})({
  "./src/a.js": function () {
    /**/
  },
  "./src/index.js": function () {
    /**/
  },
});
```

现在来写这个 require 函数

```js
// dist/my-main.js
(function (modules) {
  // require函数相当于是运行一个模块，并得到模块导出结果
  function require(moduleId) {
    // 模块id
    var func = modules[moduleId];
    var module = { exports: {} };

    func(module, module.exports, require); //运行模块

    var result = module.exports; // 得到模块导出结果
    return result;
  }

  // 执行入口文件
  require("./src/index.js"); // require函数相当于是运行一个模块，并得到模块导出结果
})({
  /* ... */
});
```

这里 `require` 函数非常简单，首先根据 `moduleId` 获取到对应的执行函数，储存为 `func`，然后定义一个 `module` 对象，里面有一个 `exports` 属性，用于 `module.exports`，然后执行这个 `func`。在这里，声明变量 `result` 并将其储存为 `func` 执行后的导出返回结果，因为模块最后会写例如`modules.exports = "a";` ，那么也就是把 `modules` 对象内 `key` 为 `exports` 的属性的值写为 `"a"`，所以将其保存为 `result` 变量后 `return` 即可。

**到这里，最基础基本的打包其实就已经完成了**

## Set the Cache

Common JS 的模块是具有缓存功能的，即**一个模块被首次 `require` 执行后会缓存 exports 的结果，避免重复执行**。现在来实现这个功能

```js
var moduleExports = {}; // 缓存模块导出结果

function require(moduleId) {
  // 模块id
  if (moduleExports[moduleId]) {
    // 检查是否有缓存
    return moduleExports[moduleId];
  }

  var func = modules[moduleId];
  var module = { exports: {} };

  func(module, module.exports, require); //运行模块

  var result = module.exports; // 得到模块导出结果
  moduleExports[moduleId] = result; // 缓存导出结果
  return result;
}
```

同时，`webpack` 为了避免其实现的 `require` 函数 `node` 的重名，因此它会把其命名为 `__webpack_require`。

## Analyze Webpack compile result

我们执行 `webpack --mode=development` 后去除多余的注释和为了处理其他兼容性而出现的代码，可以得到以下结果

```js
(function (modules) {
  // The module cache
  var installedModules = {};
  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {},
    });
    // Execute the module function
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    );
    // Flag the module as loaded
    module.l = true;
    // Return the exports of the module
    return module.exports;
  }
  return __webpack_require__((__webpack_require__.s = "./src/index.js"));
})({
  "./src/a.js": function (module, exports) {
    eval(
      '\nconsole.log("module a")\nmodule.exports = "a";\n\n//# sourceURL=webpack:///./src/a.js?'
    );
  },
  "./src/index.js": function (module, exports, __webpack_require__) {
    eval(
      'console.log("index module")\nvar a = __webpack_require__(/*! ./a */ "./src/a.js")\na.abc();\nconsole.log(a)\n\n//# sourceURL=webpack:///./src/index.js?'
    );
  },
});
```

可以看到，和前面写的大差不差，有一些细微的差别。

### installedModules

用于处理缓存的对象名字叫做 `installedModules` ，同时他的结构也不太一样，示例如下

```js
var installedModules = {
  "./src/a.js": {
    i: "./src/a.js",
    l: true,
    exports: "a",
  },
};
```

这里 `i` 表示模块 Id，`l` 表示模块是否加载完成，`exports` 则是导出结果

### Why 'eval'

为什么选择用 `eval` 来执行模块代码呢？这是为了方便调试和定位错误，使用 `eval` 会有一个单独的运行空间，同时 `sourceURL` 选项就可以通过控制台直接定位到是哪一个文件出现了问题。
