---
title: Webpack Pugins & Loaders
date: "2023-10-5"
tags: ["webpack", "tutorial", "frontend engineering"]
draft: false
summary: "In this article we learn what is  plugins and loaders in Webpack."
---

# Webpack Pugins & Loaders

Webpack 做的事情，仅仅是分析出各种模块的依赖关系，然后形成资源列表，最终打包生成到指定的文件中。
更多的功能需要借助 `webpack loaders` 和 `webpack plugins` 完成。

## Loader

loader 本质上是一个函数，它的作用是将某个源码字符串转换成另一个源码字符串返回。

![Webpack-plugin-and-loader-1](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-1.png)

loader 函数的将在模块解析的过程中被调用，以得到最终的源码。

**全流程：**

![Webpack-plugin-and-loader-2](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-2.png)

**chunk 中解析模块的流程：**

![Webpack-plugin-and-loader-3](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-3.png)

**chunk 中解析模块的更详细流程：**

![Webpack-plugin-and-loader-4](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-4.png)

**处理 loaders 流程：**

![Webpack-plugin-and-loader-5](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-5.png)

loader 会在读取文件内容后，获取到符合匹配规则的文件的源代码，然后经过处理后返回新的处理后的源代码。

### loader 配置

**完整配置**

```js
module.exports = {
  module: {
    //针对模块的配置，目前版本只有两个配置，rules、noParse
    rules: [
      //模块匹配规则，可以存在多个规则
      {
        //每个规则是一个对象
        test: /\.js$/, //匹配的模块正则
        use: [
          //匹配到后应用的规则模块
          {
            //其中一个规则
            loader: "模块路径", //loader模块的路径，该字符串会被放置到require中
            options: {
              //向对应loader传递的额外参数
            },
          },
        ],
      },
    ],
  },
};
```

**简化配置**

```js
module.exports = {
  module: {
    //针对模块的配置，目前版本只有两个配置，rules、noParse
    rules: [
      //模块匹配规则，可以存在多个规则
      {
        //每个规则是一个对象
        test: /\.js$/, //匹配的模块正则
        use: ["模块路径1", "模块路径2"], //loader模块的路径，该字符串会被放置到require中
      },
    ],
  },
};
```

### 练习：样式处理

我们假设有这么一段代码

```js
// index.js
const content = require("./assets/index.css");

console.log(content);
```

```css
/* index.css */
body {
  background: #333;
  color: #fff;
}
```

我们这里是用了 `require('./assets/index.css')`，正常来讲这是肯定报错的，因为根据 webpack 处理流程可知，文件读取后要抽象成 AST，而 css 内容是无法被抽象成 AST 的，那么这个时候就可以利用 loader 了。

```js
// style-loader.js
module.exports = function (sourceCode) {
  const code = `const style = document.createElement("style");
    style.innerHTML = \`${sourceCode}\`;
    document.head.appendChild(style);
    module.exports = \`${sourceCode}\``;
  return code;
};
```

这里，我们收到的参数 `sourceCode` 就会是 `css` 文件内本身的所有内容，我们拿到后，将其放入 `style` 标签内，此时在代码运行阶段，就会插入这个 `style` 标签。并且这段代码也可以被正确抽象成 AST。注意的是，我们以字符串的形式，将新的源代码 `return` 。

这个时候再去应用它

```js
// webpack.config.js

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.(png)|(jpg)|(gif)$/,
        use: [
          {
            loader: "./loaders/img-loader.js",
            options: {
              limit: 3000, //3000字节以上使用图片，3000字节以内使用base64
              filename: "img-[contenthash:5].[ext]",
            },
          },
        ],
      },
    ],
  },
};
```

这样就完成了对样式导入的简单处理。

### 练习：处理图片

对于导入的图片，应该做出一些什么样的处理呢？

```js
const src = require("./assets/webpack.png");
const img = document.createElement("img");
img.src = src;
document.body.appendChild(img);
```

我们导入后，希望拿到图片的 url，然后设置 img 标签的 src 属性值。所以我们需要正确返回一个可用的 url ,它可以是 base64 的编码，也可以是相对的路径。

#### base64

先看看如何转化为 base64

```js
function loader(buffer) {
  //给的是buffer
  console.log("文件数据大小：(字节)", buffer.byteLength);
  const content = "data:image/png;base64," + buffer.toString("base64");
  return `module.exports = \`${content}\``;
}

loader.raw = true; //该loader要处理的是原始数据

module.exports = loader;
```

> 在 webpack 中，一个 loader 可以接收文件的原始内容（buffer）或转换后的文本内容。通过设置`loader.raw`属性为`true`，您告诉 webpack 该 loader 将处理原始数据（buffer），而不是转换后的文本内容。
>
> 默认情况下，webpack 会将文件内容作为字符串传递给 loader 函数。但是，有些 loader 可能需要直接操作文件的原始二进制数据，例如图像文件的处理。在这种情况下，您可以将`loader.raw`属性设置为`true`，以便告诉 webpack 传递原始数据给该 loader。
>
> 在您的代码中，`loader.raw = true;`将`raw`属性设置为`true`，表示该 loader 将处理原始数据（buffer）。这使得您可以直接操作文件的二进制数据，而不需要进行额外的转换。
>
> 需要注意的是，当`loader.raw`设置为`true`时，loader 函数的参数将是原始的 buffer 数据，而不是文件内容的字符串形式。因此，在您的代码中，loader 函数的参数`buffer`表示文件的原始内容（buffer）而不是字符串。
>
> 这种设置使得您能够在 loader 中根据文件大小或其他条件，决定是对原始数据进行处理，还是执行其他操作（例如生成 base64 编码或输出为文件）。

我们拿到原始数据后，通过 `toString` 转换为 base64 格式后进行正确拼接并返回即可。此时 img 的 src 属性就会是一段 base64

#### File Path

```js
const loaderUtil = require("loader-utils");

function getFilePath(buffer, name) {
  const filename = loaderUtil.interpolateName(this, name, {
    content: buffer,
  });
  this.emitFile(filename, buffer);
  return filename;
}

function loader(buffer) {
  //给的是buffer
  console.log("文件数据大小：(字节)", buffer.byteLength);
  const filename = "[contenthash].[ext]";
  const content = getFilePath.call(this, buffer, filename);
  return `module.exports = \`${content}\``;
}

loader.raw = true; //该loader要处理的是原始数据

module.exports = loader;
```

> `emitFile` 是 Webpack LoaderContext 对象提供的一个方法，用于将文件输出到输出目录。它是 Webpack 提供的一种机制，允许 Loader 将处理后的文件发送到输出目录，以供后续的打包和使用。
>
> 使用 `this.emitFile` 方法，您可以指定要输出的文件路径和文件内容。这个方法通常在 Loader 中被用于处理资源文件（例如图像、字体等），并输出它们到最终的构建目录中。
>
> 在给定的代码中，`this.emitFile(filename, buffer)` 将 `buffer` 中的数据输出到输出目录，并使用 `filename` 作为文件的路径和名称。这样，通过调用 `emitFile` 方法，您可以确保处理后的文件被正确地输出到 Webpack 的构建目录中，以供后续使用。

> `loader-utils` 是一个常用的工具库，用于在 Webpack Loader 中处理各种任务。它提供了一系列实用函数，用于处理 Loader 配置、解析查询参数、生成唯一标识符等常见任务。

> `loaderUtil.interpolateName` 是 `loader-utils` 包提供的一个函数，它用于生成文件名或路径的字符串。它接受三个参数：
>
> 1. `this`：在此上下文中，它是指向当前 loader 的 Loader 上下文对象。它提供了一些有用的方法和属性，以便在 loader 中访问 loader 的配置、资源路径等信息。
> 2. `name`：是一个字符串，表示生成的文件名或路径的模板。它可以包含占位符，用于插入特定的值。占位符由一对尖括号 `< >` 包裹，例如 `` 表示使用文件内容的哈希值作为插入值。
> 3. `options`：是一个对象字面量，用于提供额外的选项或上下文信息。在给定代码中，`content` 是其中的一个属性，它的值是传递给 `getFilePath` 函数的 `buffer` 参数。
>
> `loaderUtil.interpolateName` 函数会将模板中的占位符替换为相应的值，并返回生成的字符串。在给定代码中，它会根据模板 `name` 和上下文信息生成一个文件名，用于后续的文件操作。

在这里我们使用`loaderUtil.interpolateName`生成文件名然后使用`emitFile`添加到最后的打包文件结果里。最后将结果拼接字符串返回就完成了文件的处理。

##### Options

我们还可以在 loader 里允许配置一些参数，例如，图片大于一定程度使用文件形式，小于一定程度使用 base64 形式。

```js
// webpack.config.js

module.exports = {
  mode: "development",
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.(png)|(jpg)|(gif)$/,
        use: [
          {
            loader: "./loaders/img-loader.js",
            options: {
              limit: 3000, //3000字节以上使用图片，3000字节以内使用base64
              filename: "img-[contenthash:5].[ext]",
            },
          },
        ],
      },
    ],
  },
};
```

那么我们就需要在 loader 里面添加 options 处理的代码。

```js
const loaderUtil = require("loader-utils");

function loader(buffer) {
  //给的是buffer
  console.log("文件数据大小：(字节)", buffer.byteLength);
  const { limit = 1000, filename = "[contenthash].[ext]" } =
    loaderUtil.getOptions(this);
  if (buffer.byteLength >= limit) {
    const content = getFilePath.call(this, buffer, filename);
  } else {
    const content = getBase64(buffer);
  }
  return `module.exports = \`${content}\``;
}

loader.raw = true; //该loader要处理的是原始数据

module.exports = loader;

function getBase64(buffer) {
  return "data:image/png;base64," + buffer.toString("base64");
}

function getFilePath(buffer, name) {
  const filename = loaderUtil.interpolateName(this, name, {
    content: buffer,
  });
  this.emitFile(filename, buffer);
  return filename;
}
```

这里使用 `loaderUtil.getOptions(this)` 来获得 options 的具体内容，然后以此就可以进行 `if` 判断，来决定执行哪个方法了。

## Plugins

loader 的功能定位是转换代码，而一些其他的操作难以使用 loader 完成，比如：

- 当 webpack 生成文件时，顺便多生成一个说明描述文件
- 当 webpack 编译启动时，控制台输出一句话表示 webpack 启动了
- 当 xxxx 时，xxxx

这种类似的功能需要把功能嵌入到 webpack 的编译流程中，而这种事情的实现是依托于 plugin 的

![Webpack-plugin-and-loader-6](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-6.png)

plugin 的**本质**是一个带有 apply 方法的对象

```js
const plugin = {
  apply: function (compiler) {},
};
```

通常，习惯上，我们会将该对象写成构造函数的模式

```js
class MyPlugin {
  apply(compiler) {}
}

const plugin = new MyPlugin();
```

要将插件应用到 webpack，需要把插件对象配置到 webpack 的 plugins 数组中，如下：

```js
module.exports = {
  plugins: [new MyPlugin()],
};
```

apply 函数会在初始化阶段，创建好 Compiler 对象后运行。

compiler 对象是在初始化阶段构建的，整个 webpack 打包期间只有一个 compiler 对象，后续完成打包工作的是 compiler 对象内部创建的 compilation

apply 方法会在**创建好 compiler 对象后调用**，并向方法传入一个 compiler 对象

![Webpack-plugin-and-loader-7](//Images/Webpack/Webpack-plugin-&-loader/Webpack-plugin-and-loader-7.png)

compiler 对象提供了大量的钩子函数（hooks，可以理解为事件），plugin 的开发者可以注册这些钩子函数，参与 webpack 编译和生成。

你可以在 apply 方法中使用下面的代码注册钩子函数:

```js
class MyPlugin {
  apply(compiler) {
    compiler.hooks.事件名称.事件类型(name, function (compilation) {
      //事件处理函数
    });
  }
}
```

具体可以如下

```js
module.exports = class MyPlugin {
  apply(compiler) {
    //在这里注册事件，类似于window.onload  $(function(){})
    compiler.hooks.done.tap("MyPlugin-done", function (compilation) {
      //事件处理函数
      console.log("编译完成");
    });
  }
};
```

`compilation` 这个阶段只会执行一次，也就是最开始初始化的时候，通常而言，**我们会在这个时候为接下来的几个阶段注册好监听事件**。

### 事件名称

即要监听的事件名，即钩子名，所有的钩子：https://www.webpackjs.com/api/compiler-hooks

### 事件类型

这一部分使用的是 Tapable API，这个小型的库是一个专门用于钩子函数监听的库。

它提供了一些事件类型：

- tap：注册一个同步的钩子函数，函数运行完毕则表示事件处理结束
- tapAsync：注册一个基于回调的异步的钩子函数，函数通过调用一个回调表示事件处理结束
- tapPromise：注册一个基于 Promise 的异步的钩子函数，函数通过返回的 Promise 进入已决状态表示事件处理结束

### 处理函数

处理函数有一个事件参数`compilation`，可以以此来获取打包过程中的一些信息，来做处理。

#### 添加文件目录

下面来看一个小的应用场景，我们希望打包后可以生成一个 txt，里面包含了打包生成的文件目录和大小，这个时候就需要用到 plugins 了。

```js
module.exports = class FileListPlugin {
  constructor(filename = "filelist.txt") {
    this.filename = filename;
  }

  apply(compiler) {
    compiler.hooks.emit.tap("FileListPlugin", (complation) => {
      const fileList = [];
      for (const key in complation.assets) {
        const content = `【${key}】
大小：${complation.assets[key].size() / 1000}KB`;
        fileList.push(content);
      }

      const str = fileList.join("\n\n");
      complation.assets[this.filename] = {
        source() {
          return str;
        },
        size() {
          return str.length;
        },
      };
    });
  }
};
```

```js
// webpack.config.js

const FileListPlugin = require("./plugins/FileListPlugin");

module.exports = {
  mode: "development",
  devtool: "source-map",
  plugins: [new FileListPlugin("文件列表.md")],
};
```

首先，我们可以给 `plugin` 类添加 `constructor` ，以此允许进行配置。随后我们再进行监听事件的注册，利用 `complation.asset` 获取到打包信息，然后以此生成好 `txt` 内的文件内容，最后通过 `complation.assets[this.filename]` 进行打包内容的添加。值得注意的是：里面需要包含有 `source` 和 `size` 两个函数，这是固定的，分别用来表示内容和大小。这样，就完成了 `plugin` 的编写。

```md
【main.js】
大小：4.076KB

【main.js.map】
大小：3.7KB
```
