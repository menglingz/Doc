## 一、vue详细目录：

```
├── scripts ------------------------------- 构建相关的文件，一般情况下我们不需要动
│   ├── git-hooks ------------------------- 存放git钩子的目录
│   ├── alias.js -------------------------- 别名配置
│   ├── config.js ------------------------- 生成rollup配置的文件
│   ├── build.js -------------------------- 对 config.js 中所有的rollup配置进行构建
│   ├── ci.sh ----------------------------- 持续集成运行的脚本
│   ├── release.sh ------------------------ 用于自动发布新版本的脚本
├── dist ---------------------------------- 构建后文件的输出目录
├── examples ------------------------------ 存放一些使用Vue开发的应用案例
├── flow ---------------------------------- 类型声明，使用开源项目 [Flow](https://flowtype.org/)
├── packages ------------------------------ 存放独立发布的包的目录
├── test ---------------------------------- 包含所有测试文件
├── src ----------------------------------- 这个是我们最应该关注的目录，包含了源码
│   ├── compiler -------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
│   ├── core ------------------------------ 存放通用的，与平台无关的代码
│   │   ├── observer ---------------------- 响应系统，包含数据观测的核心代码
│   │   ├── vdom -------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
│   │   ├── instance ---------------------- 包含Vue构造函数设计相关的代码
│   │   ├── global-api -------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── components -------------------- 包含抽象出来的通用组件
│   ├── server ---------------------------- 包含服务端渲染(server-side rendering)的相关代码
│   ├── platforms ------------------------- 包含平台特有的相关代码，不同平台的不同构建的入口文件也在这里
│   │   ├── web --------------------------- web平台
│   │   │   ├── entry-runtime.js ---------- 运行时构建的入口，不包含模板(template)到render函数的编译器，所以不支持 `template` 选项，我们使用vue默认导出的就是这个运行时的版本。大家使用的时候要注意
│   │   │   ├── entry-runtime-with-compiler.js -- 独立构建版本的入口，它在 entry-runtime 的基础上添加了模板(template)到render函数的编译器
│   │   │   ├── entry-compiler.js --------- vue-template-compiler 包的入口文件
│   │   │   ├── entry-server-renderer.js -- vue-server-renderer 包的入口文件
│   │   │   ├── entry-server-basic-renderer.js -- 输出 packages/vue-server-renderer/basic.js 文件
│   │   ├── weex -------------------------- 混合应用
│   ├── sfc ------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
│   ├── shared ---------------------------- 包含整个代码库通用的代码
├── package.json -------------------------- 不解释
├── yarn.lock ----------------------------- yarn 锁定文件
├── .editorconfig ------------------------- 针对编辑器的编码风格配置文件
├── .flowconfig --------------------------- flow 的配置文件
├── .babelrc ------------------------------ babel 配置文件
├── .eslintrc ----------------------------- eslint 配置文件
├── .eslintignore ------------------------- eslint 忽略配置
├── .gitignore ---------------------------- git 忽略配置
```



## 二、vue构建输出

### 1.vue构建输出的种类

- 按照输出的模块形式分类，vue有三种不同的构建输出，分别为：`umd`、`CommonJS` 以及`ESModule` 


- 三个构建配置的入口是相同的，但输出的格式（format）是不同的，分别为`cjs`、`es` 以及`umd`
- 每种模块输出又分别书发出了运行时版以及完整版，dev分别为`production` 和`development`
- 完整版比运行时版本多了`complier` ,它的作用是：**编译器代码的存放目录，将template编译为render函数**

### 2.不同构建输出的区别与作用

- 区分运行时版与完整版：`运行时版 + Complier = 完整版` ，Complier这个过程可以在构建的时候就可以完成，不需要在代码运行的时候完成，这样真正的代码就免去了这样一个步骤，提升了性能，同时将Complier抽离为单独的包，减小了库的体积；
- 完整版的应用场景：允许你在代码运行的时候去现场编译模板，在不配合构建工具的情况下可以直接使用；
- 区分输出的不同形式：`umd  ` 是使得你可以直接使用<script> 标签引用Vue的模块形式，但我们使用Vue的时候更多的是结合构建工具，比如`webpack` 之类；`cjs ` 是为`browserify` 和`webpack1` 提供的模块形式，但是他们在加载模块的时候不能直接加载`ES Module`;而`webpack2`以及`Rollup`是可以直接加载`ES Module`的，所以就有了`es`形式的模块输出； 

### 3.package.json中的运行时版本

```javascript
"main": "dist/vue.runtime.common.js",
"module": "dist/vue.runtime.esm.js",
```

- `main`和`module`指向的都是运行时版的Vue，不同的是，前者是`cjs`模块，后者是`es`模块；

- 其中`main`字段和`module`字段分别用于`browserify`、`webpack1`和`webpack2+`、`Rollup`，后者可以直接加载`ES Module`且会根据`module`字段的配置进行加载；

- package.json中的scripts字段

  ```javascript
  "scripts": {
  	  // 构建完整版 umd 模块的 Vue
      "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
      // 构建运行时 cjs 模块的 Vue
      "dev:cjs": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-cjs",
      // 构建运行时 es 模块的 Vue
      "dev:esm": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-esm",
      // 构建 web-server-renderer 包
      "dev:ssr": "rollup -w -c scripts/config.js --environment TARGET:web-server-renderer",
      // 构建 Compiler 包
      "dev:compiler": "rollup -w -c scripts/config.js --environment TARGET:web-compiler ",
      "build": "node scripts/build.js",
      "build:ssr": "npm run build -- vue.runtime.common.js,vue-server-renderer",
      "lint": "eslint src build test",
      "flow": "flow check",
      "release": "bash scripts/release.sh",
      "release:note": "node scripts/gen-release-note.js",
      "commit": "git-cz"
    },
  ```
  ## 


## 三、Vue构造函数

在使用`Vue`的时候，要使用`new`操作符进行调用，这说明`Vue`应该是一个构造函数，因此第一件事就是清楚构造函数的原理；

### 1.Vue构造函数的原型

- 当执行`npm run dev`时，执行的是`scripts/config.js`中的`full-dev`，根据`scripts/config.js`文件中的配置：

```javascript
'full-dev': {
	entry: resolve('web/entry-runtime-with-compiler.js'),
	dest: resolve('dist/vue.js'),
	format: 'umd',
	env: 'development',
	alias: { he: './entity-decoder' },
	banner
}
```

- 由代码可知，入口文件为`web/entry-runtime-with-compiler.js`，最终输出为`dist/vue.js` ，它是一个`umd`模块，我们可以以入口文件为起点，找到`Vue`构造函数；

- 入口文件中的`web/entry-runtime-with-compiler.js`中的`web`是一个别名配置，在`scripts/alias.js`文件中可以查看；

  ```javascript
  const path = require('path')

  const resolve = p => path.resolve(__dirname, '../', p)

  module.exports = {
    vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
    compiler: resolve('src/compiler'),
    core: resolve('src/core'),
    shared: resolve('src/shared'),
    web: resolve('src/platforms/web'),
    server: resolve('packages/server-renderer/src'),
    sfc: resolve('packages/compiler-sfc/src')
  }
  ```


- 从中可以得知，`web`指向的应该是`src/platforms/web` ，因此当运行`npm run dev`命令时，会首先执行`src/platforms/web/entry-runtime-with-compiler`这个文件，如下所示：

  ![屏幕截图 2022-06-30 201033](E:\笔记\源代码\img\屏幕截图 2022-06-30 201033.png)


- 从上图可以看出，从`src/platforms/web/runtime-with-compiler`里引入了`Vue`，而这个`Vue`就是我们所说的Vue构造函数，接下来打开`src/platforms/web/runtime-with-compiler`文件，由下图所示：

  ![屏幕截图 2022-06-30 201544](E:\笔记\源代码\img\屏幕截图 2022-06-30 201544.png)


- 从上图可以看出，`Vue`构造函数同样是从`./runtime/index` 导入进来的，因此打开`src/platforms/web/runtime/index`文件，由下图所示：

  ![屏幕截图 2022-06-30 201937](E:\笔记\源代码\img\屏幕截图 2022-06-30 201937.png)

- 从上图可以看出，Vue构造函数同样是从`core/index` 文件中引入的，在 `scripts/alias.js` 的配置中，`core` 指向的是 `src/core`，打开 `src/core/index 文件，由下图所示：

  ![屏幕截图 2022-06-30 202126](E:\笔记\源代码\img\屏幕截图 2022-06-30 202126.png)


- 按照之前的套路，继续打开 `src/core/instance/index` 文件，由下图所示：

  ![屏幕截图 2022-06-30 202439](E:\笔记\源代码\img\屏幕截图 2022-06-30 202439.png)


- 可以看到，Vue构造函数的出生地最终在`src/core/instance/index`文件中，分析此文件的代码，首先分别从 `./init.js`、`./state.js`、`./render.js`、`./events.js`、`./lifecycle.js` 这五个文件中导入五个方法，分别是：`initMixin`、`stateMixin`、`renderMixin`、`eventsMixin` 以及 `lifecycleMixin`，然后定义了 `Vue` 构造函数，其中使用了安全模式来提醒你要使用 `new` 操作符来调用 `Vue`，接着将 `Vue` 构造函数作为参数，分别传递给了导入进来的这五个方法，最后导出 `Vue`。

- 我们分别看这5个方法有什么作用；

  #### （1）initMixin（）

  - 打开 `src/core/instance/init` 文件，找到 `initMixin` 方法，如下：

  ```typescript
  export function initMixin (Vue: Class<Component>) {
    Vue.prototype._init = function (options?: Object) {
      // ... _init 方法的函数体，此处省略
    }
  }
  ```

  - 可以看出，`initMixin`这个方法在`Vue`的原型上挂载了`_init`这个方法，这个方法看上去应该是内部初始化的方法，在`src/core/instance/index`文件中也调用过，为`this._init(options)`,这句代码在Vue构造函数内部，说明当我们执行`new Vue（）`的时候，`_init`方法会被触发；
  - 其大致逻辑是`initMixn()`方法接受`options`作为参数，`options`如果是组件，则通过`initInternalComponent(vm,options)`方法将`options`参数合并到`vm.options`上。否则,将从vm的构造函数中解析出`options`进行合并。接下来，调用`initProxy(vm)`对属性进行拦截。最后，调用`vm.$mount(vm.$options.el)`进行挂载。

  #### （2）stateMixin（）

  - 打开 `src/core/instance/state.js` 文件，找到 `stateMixin` 方法，如下：

    ![屏幕截图 2022-07-01 133541](E:\笔记\源代码\img\屏幕截图 2022-07-01 133541.png)


  - 最后两句，使用`Object.defineProperty` 在 `Vue.prototype` 上定义了两个属性，就是大家熟悉的：`$data` 和 `$props`，这两个属性的定义分别写在了 `dataDef` 以及 `propsDef` 这两个对象里；
  - 这两个对象的`get`实际上代理的是`_data` 和`_props`实例属性，然后有一个生产环境的判断，如果不是生产环境的话，就为`_data` 和`_props`这两个实例属性设置一下`set`，也就是说，这两个属性是只读的属性；
  - 也定义了几个方法，`$set`、`$delete` 以及 `$watch`;

  #### （3）eventsMixin（）

  - 这个方法在 `src/core/instance/events` 文件中，打开这个文件找到 `eventsMixin` 方法，这个方法在 `Vue.prototype` 上添加了四个方法，分别是：`$on`、`$once`、`$off`、`$emit`;

    ![屏幕截图 2022-07-01 201827](E:\笔记\源代码\img\屏幕截图 2022-07-01 201827.png)

  #### （4）lifecycleMixin（）

  - 打开 `src/core/instance/lifecycle` 文件找到相应方法，这个方法在 `Vue.prototype` 上添加了三个方法：

    ![屏幕截图 2022-07-01 202033](E:\笔记\源代码\img\屏幕截图 2022-07-01 202033.png)


  - 分别为`_update`、`$forceUpdate`、`$destroy`;

  #### （5）renderMixin（）

  - 它在 `src/core/instance/render` 文件中，这个方法的一开始以 `Vue.prototype` 为参数调用了 `installRenderHelpers` 函数，这个函数来自于与 `src/core/instance/render-helpers/index` 文件，打开这个文件找到 `installRenderHelpers` 函数，如下所示：

    ![屏幕截图 2022-07-01 202611](E:\笔记\源代码\img\屏幕截图 2022-07-01 202611.png)


  - 这个函数的作用就是在 `Vue.prototype` 上添加一系列方法；
  - `renderMixin` 方法在执行完 `installRenderHelpers` 函数之后，又在 `Vue.prototype` 上添加了两个方法，分别是：`$nextTick` 和 `_render`；

- 至此，`src/core/instance/index` 文件中的代码就运行完毕了，我们大概了解了每个 `*Mixin` 方法的作用其实就是包装 `Vue.prototype`，在其上挂载一些属性和方法。

### 2.Vue构造函数的静态属性和方法(全局API)

- 到目前为止，`src/core/instance/index` 文件，也就是 `Vue` 的出生文件的代码我们就看完了，按照之前我们寻找 `Vue` 构造函数时的文件路径回溯，下一个我们要看的文件应该就是 `src/core/index` 文件，这个文件将 `Vue` 从 `src/core/instance/index` 文件中导入了进来;

  ![屏幕截图 2022-07-01 203954](E:\笔记\源代码\img\屏幕截图 2022-07-01 203954.png)


- 上面的代码中，分别从三个文件导入了三个变量，将Vue构造函数作为参数调用了initGlobalAPI这个方法，然后在 `Vue.prototype` 上分别添加了两个只读的属性，分别是：`$isServer` 和 `$ssrContext`。接着又在 `Vue` 构造函数上定义了 `FunctionalRenderContext` 静态属性，之所以在 `Vue` 构造函数上暴露该属性，是为了在 `ssr` 中使用它。

- 最后，在 `Vue` 构造函数上添加了一个静态属性 `version`，代表`Vue`的版本号；

- 我们分别看看导入的几个变量有什么作用？

  #### （1）initGlobalAPI（）

  - 打开 `src/core/global-api/index` 文件找到 `initGlobalAPI` 方法，我们来看看 `initGlobalAPI` 方法都做了什么：

  - 首先是这样一段代码：

    ![屏幕截图 2022-07-02 084221](E:\笔记\源代码\img\屏幕截图 2022-07-02 084221.png)

  - 这段代码的作用就是在`Vue`构造函数上添加`config`属性，这个属性同 `$data` 以及 `$props` 也是一个只读的属性，当你设置其值得时候，在非生产环境下会给你一个友好的提示；

  - `config `这个属性是从`src/core/config` 文件导出的对象；

  - 接下来是这样一段代码：

    ![屏幕截图 2022-07-02 084614](E:\笔记\源代码\img\屏幕截图 2022-07-02 084614.png)

  - 这段代码的作用是在Vue上添加了util属性，这是一个对象，者和对象拥有四个属性分别是：`warn`、`extend`、`mergeOptions` 以及 `defineReactive`。这四个属性来自于 `src/core/util/index` 文件。这里的注释的大概意思就是`Vue.util` 以及 `util` 下的四个方法都不被认为是公共API的一部分，要避免依赖他们，但是你依然可以使用，只不过风险你要自己控制。并且，在官方文档上也并没有介绍这个全局API，所以能不用尽量不要用。


  - 然后是这样一段代码：

  ![屏幕截图 2022-07-02 084838](E:\笔记\源代码\img\屏幕截图 2022-07-02 084838.png)

  - 这段代码在 `Vue` 上添加了四个属性分别是 `set`、`delete`、`nextTick` 以及 `options` ，一个方法`observable` 。`Vue.options`通过 `Object.create(null)` 创建是一个空对象。但接下来遍历`ASSET_TYPES` 数组改变了Vue.options的值，`ASSET_TYPES` 来自于 `src/core/shared/constants` 文件，打开这个文件，发现 `ASSET_TYPES` 是一个数组：

  ![屏幕截图 2022-07-02 085514](E:\笔记\源代码\img\屏幕截图 2022-07-02 085514.png)

  - 所以当遍历完成后，`Vue.options` 将变成这样：



  - ```typescript
    Vue.options = {
    	components: Object.create(null),
    	directives: Object.create(null),
    	filters: Object.create(null),
    	_base: Vue
    }
    ```


  - 紧接着使用`Vue`全局方法`extend`将`builtInComponents` 的属性混合到 `Vue.options.components` 中，其中 `builtInComponents` 来自于 `src/core/components/index` 文件，该文件如下：

  ![屏幕截图 2022-07-02 085846](E:\笔记\源代码\img\屏幕截图 2022-07-02 085846.png)

  - 所以最终 `Vue.options.components` 的值如下：


  ```typescript
  Vue.options.components = {
  	KeepAlive
  }
  ```
  - `Vue.options` 的值如下：

  ```typescript
  Vue.options = {
  	components: {
  		KeepAlive
  	},
  	directives: Object.create(null),
  	filters: Object.create(null),
  	_base: Vue
  }
  ```

  - 在 `initGlobalAPI` 方法的最后部分，以 `Vue` 为参数调用了四个 `init*` 方法，这四个方法从上至下分别来自于 `src/core/global-api/use.js`、`src/core/global-api/mixin.js`、`src/core/global-api/extend.js` 以及 `src/core/global-api/assets.js` 这四个文件：

  ```
  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
  ```

  - 打开 `src/core/global-api/use` 文件，我们发现这个文件只有一个 `initUse` 方法，如下：

    ![屏幕截图 2022-07-02 200051](E:\笔记\源代码\img\屏幕截图 2022-07-02 200051.png)


  - 该方法的作用是在`Vue`构造函数上添加`use`方法，也就是`Vue.use` 这个全局API，这个方法用来安装 `Vue` 插件；再打开 `src/core/global-api/mixin` 文件，这个文件更简单，全部代码如下：

    ![屏幕截图 2022-07-02 200229](E:\笔记\源代码\img\屏幕截图 2022-07-02 200229.png)


  - `initMixin` 方法的作用是，在 `Vue` 上添加 `mixin` 这个全局API。

    再打开 `src/core/global-api/extend` 文件，找到 `initExtend` 方法，如下：

    ![屏幕截图 2022-07-02 200459](E:\笔记\源代码\img\屏幕截图 2022-07-02 200459.png)


  - `initExtend` 方法在 `Vue` 上添加了 `Vue.cid` 静态属性，和 `Vue.extend` 静态方法。最后一个是 `initAssetRegisters`，我们打开 `src/core/global-api/assets` 文件，找到 `initAssetRegisters` 方法如下：

    ![屏幕截图 2022-07-02 200607](E:\笔记\源代码\img\屏幕截图 2022-07-02 200607.png)


  -  `initAssetRegisters` 方法的作用是遍历`ASSET_TYPES`这个对象，将`ASSET_TYPES`对象里的每一个成员都添加到`Vue`构造函数上，所以，最终经过 `initAssetRegisters` 方法，`Vue` 又多了三个静态方法：

    ```
    Vue.component
    Vue.directive
    Vue.filter
    ```


  - 因此，`initGlobalAPI` 方法的全部功能就是在 `Vue` 构造函数上添加全局的API；

- 至此，对于 `core/index.js` 文件的作用我们也大概清楚了，在这个文件里，它首先将核心的 `Vue`，也就是在 `core/instance/index.js` 文件中的 `Vue`，也可以说是原型被包装(添加属性和方法)后的 `Vue` 导入，然后使用 `initGlobalAPI` 方法给 `Vue` 添加静态方法和属性，除此之外，在这个文件里，也对原型进行了修改，为其添加了两个属性：`$isServer` 和 `$ssrContext`，最后添加了 `Vue.version` 属性并导出了 `Vue`。

### 3.Vue平台化的包装

- 现在，在我们弄清 `Vue` 构造函数的过程中已经看了两个主要的文件，分别是：`core/instance/index.js` 文件以及 `core/index.js` 文件，前者是 `Vue` 构造函数的定义文件，我们一直都叫其 `Vue` 的出生文件，主要作用是定义 `Vue` 构造函数，并对其原型添加属性和方法，即实例属性和实例方法。后者的主要作用是，为 `Vue` 添加全局的API，也就是静态的方法和属性。这两个文件有个共同点，就是它们都在 `core` 目录下，我们在介绍 `Vue` 项目目录结构的时候说过：`core` 目录存放的是与平台无关的代码，所以无论是 `core/instance/index.js` 文件还是 `core/index.js` 文件，它们都在包装核心的 `Vue`，且这些包装是与平台无关的。但是，`Vue` 是一个 `Multi-platform` 的项目（web和weex），不同平台可能会内置不同的组件、指令，或者一些平台特有的功能等等，那么这就需要对 `Vue` 根据不同的平台进行平台化地包装，这就是接下来我们要看的文件，这个文件也出现在我们寻找 `Vue` 构造函数的路线上，它就是：`platforms/web/runtime/index.js` 文件。

- 在看这个文件之前，大家可以先打开 `platforms` 目录，可以发现有两个子目录 `web` 和 `weex`。这两个子目录的作用就是分别为相应的平台对核心的 `Vue` 进行包装的。而我们所要研究的 `web` 平台，就在 `web` 这个目录里。

- 接下来，我们就打开 `platforms/web/runtime/index.js` 文件，看一看里面的代码，这个文件的一开始，是一大堆 `import` 语句，其中就包括从 `core/index.js` 文件导入 `Vue` 的那句。在 `import` 语句下面是这样一段代码：

  ```
  // install platform specific utils
  Vue.config.mustUseProp = mustUseProp
  Vue.config.isReservedTag = isReservedTag
  Vue.config.isReservedAttr = isReservedAttr
  Vue.config.getTagNamespace = getTagNamespace
  Vue.config.isUnknownElement = isUnknownElement
  ```


- `Vue.config`其代理的值是从 `core/config.js` 文件导出的对象，这几句代码的作用就是覆盖默认导出的 `config` 对象的属性，注释已经写得很清楚了，安装平台特定的工具方法，接着是这两句代码：

  ```
  // install platform runtime directives & components
  extend(Vue.options.directives, platformDirectives)
  extend(Vue.options.components, platformComponents)
  ```


- 安装特定平台运行时的指令和组件，将`platformDirectives` 和 `platformComponents` 这两个方法里的值与 `Vue.options` 合并，合并后的 `Vue.options` 的值为：

  ```
  Vue.options = {
  	components: {
  		KeepAlive,
  		Transition,
  		TransitionGroup
  	},
  	directives: {
  		model,
  		show
  	},
  	filters: Object.create(null),
  	_base: Vue
  }
  ```


- 我们继续往下看代码，接下来是这段：

  ```
  // install platform patch function
  Vue.prototype.__patch__ = inBrowser ? patch : noop

  // public mount method
  Vue.prototype.$mount = function (
    el?: string | Element,
    hydrating?: boolean
  ): Component {
    el = el && inBrowser ? query(el) : undefined
    return mountComponent(this, el, hydrating)
  }
  ```


- 首先在 `Vue.prototype` 上添加 `__patch__` 方法，如果在浏览器环境运行的话，这个方法的值为 `patch` 函数，否则是一个空函数 `noop`。然后又在 `Vue.prototype` 上添加了 `$mount` 方法，我们暂且不关心 `$mount` 方法的内容和作用。
- 再往下的一段代码是 `vue-devtools` 的全局钩子，它被包裹在 `setTimeout` 中，最后导出了 `Vue`。
- 现在我们就看完了 `src/platforms/web/runtime/index.js` 文件，该文件的作用是对 `Vue` 进行平台化地包装：
  - 设置平台化的 `Vue.config`。
  - 在 `Vue.options` 上混合了两个指令(`directives`)，分别是 `model` 和 `show`。
  - 在 `Vue.options` 上混合了两个组件(`components`)，分别是 `Transition` 和 `TransitionGroup`。
  - 在 `Vue.prototype` 上添加了两个方法：`__patch__` 和 `$mount`。

