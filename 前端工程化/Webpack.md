## Webpack

### 一、入口起点

**概念：**指示webpack应该使用哪个模块，来作为构建其内部依赖图的开始，进入入口起点后，webpack会找出有哪些模块和库是入口起点依赖的

**基本结构：**

```js
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```

在webpack中有多种方式定义entry属性；

#### 1.单个入口（简写）语法

```js
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```

entry属性的单个入口语法，是以下形式的简写：

```js
module.exports = {
  entry: {
    main: './path/to/my/entry/file.js',
  },
};
```

也可以给entry属性





### 二、输出

**概念：**output属性告诉webpack在哪里输出他所创建的bundle，以及如何命名这些文件这些文件。主要输出文件的默认值是./dist/main.js，其他生成文件默认放置在./dist文件夹中。可以通过在配置中指定一个output字段，来配置这些处理过程。

**基本结构：**

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js',
  },
};
```

通过output.filename和output.path属性，来告诉webpack bundle的名称，以及我们想要 bundle 生成(emit)到哪里。

### 三、loader

**概念：**webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。**loader** 让webpack 能够去处理其他类型的文件，并将它们转换为有效模块，以供应用程序使用，以及被添加到依赖图中

在 webpack 的配置中，**loader** 有两个属性：

- `test` 属性，识别出哪些文件会被转换。
- `use` 属性，定义出在进行转换时，应该使用哪个 loader

```js
const path = require('path');

module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js',
  },
  module: {
    // 当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先 use(使用) raw-loader 转换一下
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
};
```

### 四、插件

**概念：**loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量

**使用：**使用一个插件，你只需要 `require()` 它，然后把它添加到 `plugins` 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建一个插件实例

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
};
```

### 五、模式

**概念：**通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`

```js
module.exports = {
  mode: 'production',
};
```

