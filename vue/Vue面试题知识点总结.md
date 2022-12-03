## Vue面试题知识点总结

### 一、Vue基本使用

#### 1.vue父子组件如何通讯

##### （1）props属性绑定（父组件向子组件传递数据）

- 在子组件prop中可以注册一些自定义组件属性，父组件调用子组件时可以向prop中的自定义属性传值

  ```vue
  <!-- 子组件 -->
  <template>
    <div class="ChildrenDemo">
      <h1>{{title}}</h1>
    </div>
  </template>

  <script>
  export default {
    name: 'ChildrenDemo',
    props:['title'],
  }
  </script>

  <!-- 父组件 -->
  <template>
    <div class="parent">
      <ChildrenDemo title="向子组件传递的title值"></ChildrenDemo>
    </div>
  </template>
  ```


- prop也可以通过`v-bind`动态赋值

  ```vue
  <ChildrenDemo :title="xxx"></ChildrenDemo>
  ```


- 指定props的验证要求

  ```js
   props: {
      // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
      propA: Number,
      // 多个可能的类型
      propB: [String, Number],
      // 必填的字符串
      propC: {
        type: String,
        required: true
      },
      // 带有默认值的数字
      propD: {
        type: Number,
        default: 100
      },
      // 带有默认值的对象
      propE: {
        type: Object,
        // 对象或数组默认值必须从一个工厂函数获取
        default: function () {
          return { message: 'hello' }
        }
      },
      // 自定义验证函数
      propF: {
        validator: function (value) {
          // 这个值必须匹配下列字符串中的一个
          return ['success', 'warning', 'danger'].indexOf(value) !== -1
        }
      }
    }
  ```


- prop双向绑定：一般情况下，子组件**不能直接修改从父组件接收的属性值**，否则会报错，如果子组件需要接收值后处理在使用，可以将接受的值赋给子组件本身的属性，如`data`或者计算属性；

- 如果希望子组件prop的值改变时将变化同步到父组件中，可以使用**事件监听**或者**.sync修饰符**

  ```vue
  <!-- 父组件 -->
  <h1>父组件title值：{{ title }}</h1>
  <ChildrenDemo :title.sync="title"></ChildrenDemo>

  <!-- 子组件 -->
  <template>
    <div class="ChildrenDemo">
      <h1>子组件</h1>
      <input type="text" v-model="childTitle" />
    </div>
  </template>

  <script>
  export default {
    props: ["title"],
    computed: {
      childTitle: {
        get() {
          return this.title;
        },
        set(val) {
          this.$emit("update:title", val);  //更新父组件中的title
        },
      },
    }
  };
  </script>
  ```

##### （2）监听自定义事件（子组件向父组件传递数据，子组件触发父组件方法）

- 通过Vue实例方法`vm.$emit`子组件可以自定义一个事件提交给父组件，触发父组件的方法，父组件通过监听子组件自定义事件可以接收子组件传递的数据

  ```vue
  <!-- 子组件 -->
  <template>
    <div class="ChildrenDemo">
      <h1>子组件</h1>
      <button @click="changeParentTitle">点击更改父组件title</button>
    </div>
  </template>

  <script>
  export default {
    methods: {
      changeParentTitle() {
        this.$emit("childEvent", "子组件传给父组件的title")//第一个参数是提交的事件名，后面的参数可以是多个需要传递给父组件的参数
      }
    }
  };
  </script>

  <!-- 父组件 -->
  <template>
    <div class="home">
      <h1>父组件title值：{{ title }}</h1>
      <ChildrenDemo :title="title" @childEvent="changeTitle"></ChildrenDemo>
    </div>
  </template>

  <script>
  export default {
    components: {
      ChildrenDemo,
    },
    data() {
      return {
        title: "My Journey with Vue"
      };
    },
    methods: {
      changeTitle: function (str) {
        this.title = str
      },
    },
  };
  </script>
  ```


- 在监听自定义事件时可以使用`$event`传递值

  ```vue
  <!-- 子组件 -->
  <button @click="$emit('childEvent', '子组件传给父组件的title')">点击更改父组件title</button>

  <!-- 父组件 -->
  <ChildrenDemo :title="title" @childEvent="title = $event"></ChildrenDemo>
  ```

##### （3）使用$refs（父组件访问子组件的数据和方法）

- 父组件可以使用`$refs`访问子组件的数据和方法，使用时需要在调用子组件时给子组件定义一个ref名字

  ```vue
  <ChildrenDemo ref="childrenDemo"></ChildrenDemo>
  <button @click="getChildData">点击获取子组件数据</button>

  getChildData: function () {
    let child = this.$refs.childrenDemo//获取子组件实例
    console.log(child.value);//访问子组件属性
    child.childFn() //调用子组件的childFn()方法
  },
  ```


- `$refs` **只会在组件渲染完成之后生效，并且它们不是响应式的**。这仅作为一个用于直接操作子组件的“逃生舱”——你应该避免在模板或计算属性中访问 `$refs`
- 由于**ref 需要在dom渲染完成后才会有**，在使用的时候确保dom已经渲染完成。比如在生命周期 `mounted(){}` 钩子中调用，或者在 `this.$nextTick(()=>{})` 中调用

##### （4）使用$parent（子组件访问父组件的数据和方法）

- `$parent`可以用来从一个子组件访问父组件的实例

  ```js
  getParentData(){
    let parent = this.$parent //获取父组件实例
    console.log(parent.parentValue) //访问父组件属性
    parent.parentFn() //调用父组件的方法parentFn()
  }
  ```



#### 2. vue父子组件生命周期调用顺序

- 加载渲染过程

  **父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted**

- 子组件更新过程

  **父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated**

- 父组件更新过程

  **父 beforeUpdate -> 父 updated**

- 销毁过程

  **父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed**

  ​

#### 3.vue如何自己实现v-model(自定义)

- `v-bind`绑定一个`value`属性

- `v-on`监听当前元素的`input`事件，当数据变化时，将值传递给`value`实时更新数据

  ​

#### 4.vue组件更新之后如何获取最新DOM

- 组件数据变化时将重新渲染虚拟DOM，而网页渲染通常是一个异步任务，因此在数据e 属性刚刚更改时（一个函数中是同步过程），DOM 渲染还没有进行，因此此刻获取不到此时还不存在的 i DOM元素。

- 为了解决这个问题，Vue 提供了全局 api `Vue.nextTick()`，它的作用是提供下次 DOM 更新之后的回调。也就是说，在更新数据后调用 api，就能够获取到重新渲染后的 DOM 并进行相关操作

  ​

#### 5.插槽（slot）

- 概念：**插槽**（Slot）是`vue`为组件的封装者提供的能力，允许开发者在封装组件时，把不确定的、希望由用户指定的部分定义为插槽
- 要想在组根件中可以在子组件中渲染页面，需要在子组件中创建一个插槽即`slot`标签
- 规定：每个插槽都要有一个名称，即`name`，若没写则是`default`
- 如果要把内容填充到指定名称的插槽中，需要使用`v-slot:`这个指令 
- `v-slot`:后面要跟上插槽的名字
- `v-slot:`指令不能用在元素身上,必须用在`template`标签上
- `template`这个标签,它是一个虚拟的标签,只起到包裹性的作用,但是,不会被渲染为任何实质性的HTML元素

```vue
<!-- 父组件 -->
<div id="">
  <h1>App根组件</h1>
  <div class="comsbox">
    <Left>
      <template v-slot:default>
        <p>这是在Left组件得到内容区域，声明的p标签</p>
	  </template>
    </Left>
  </div>
</div>

<!-- 子组件 -->
<div id="">
  <h3>Left根组件</h3>
  <hr>
  <!-- 声明一个插槽区域 -->
  <!-- Vue官方规定：每个slot插槽，都要有一个name名称 -->
  <!-- 如果省略了slot的name属性，则有一个默认名叫做default -->
  <slot name="default"></slot>
</div>
```

##### （1）后备内容

- 有时为一个插槽设置具体的后备（也就是默认的）内容是很有用的，他只会在没有提供内容的时候被渲染，如果提供内容，则这个提供的内容会被渲染从而取代后背内容

  ```vue
  <!-- 没有提供内容，slot标签内的内容会被渲染 -->
  <submit-button></submit-button>  // 父组件
  <button type="submit">Submit</button>  // 子组件

  <!-- 提供内容，slot标签内的内容不会被渲染，而会渲染提供的内容 -->
  <submit-button>Save</submit-button>  // 父组件
  <button type="submit">Submit</button>  // 子组件
  ```

##### （2）具名插槽

- 有时我们需要多个插槽，对于这样的情况，`<slot>` 元素有一个特殊的attribute：`name`,这个attribute可以用来定义额外的插槽
- 一个不带 `name` 的 `<slot>` 出口会带有隐含的名字default
- 在向具名插槽提供内容的时候，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称

```vue
<!-- 父组件 -->
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>

<!-- 子组件 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

- 现在，`<template>` 元素中的所有内容都将会被传入相应的插槽。任何没有被包裹在带有 `v-slot` 的 `<template>` 中的内容都会被视为默认插槽的内容 `（v-slot:default ）`

##### （3）作用域插槽

- 有时让插槽内容能够访问子组件中才有的数据是很有用的，但有一个问题：数据只有在组件中才可以访问到，而我们需要在父组件中将其渲染

- 为了让数据在父级的插槽内容中可用，我们可以将要渲染的数据作为元素` <slot>`的一个`attribue`绑定上去

  ```vue
  <!-- 子组件 -->
  <span>
    <slot v-bind:user="user">
      {{ user.lastName }}
    </slot>
  </span>

  <!-- 父组件 -->
  <current-user>
    <template v-slot:default="slotProps">
      {{ slotProps.user.firstName }}
    </template>
  </current-user>
  <!--在这个例子中，我们选择将包含所有插槽 prop 的对象命名为 slotProps，但你也可以使用任意你喜欢的名字 -->
  ```

- 绑定在`<slot>` 元素上的 attribute 被称为**插槽 prop**。在父级作用域中，我们可以使用带值的 `v-slot` 来定义我们提供的插槽 prop 的名字

- 写法还可以更简单。就像假定未指明的内容对应默认插槽一样，不带参数的 `v-slot` 被假定对应默认插槽

  ```vue
  <current-user v-slot="slotProps">
    {{ slotProps.user.firstName }}
  </current-user>
  ```


- 注意默认插槽的缩写语法**不能**和具名插槽混用，因为它会导致作用域不明确

- 只要出现多个插槽，请始终为所有的插槽使用完整的基于 `<template>` 的语法

  ```vue
  <current-user>
    <template v-slot:default="slotProps">
      {{ slotProps.user.firstName }}
    </template>

    <template v-slot:other="otherSlotProps">
      ...
    </template>
  </current-user>
  ```

##### （4）解构插槽prop

- 作用域插槽的内部工作原理是将你的插槽内容包裹在一个拥有单个参数的函数里

  ```js
  function (slotProps) {
    // 插槽内容
  }
  ```


- 这意味着 `v-slot` 的值实际上可以是任何能够作为函数定义中的参数的 JavaScript 表达式。所以在支持的环境下，你也可以使用ES2015结构来传入具体的插槽 prop

  ```vue
  <current-user v-slot="{ user }">
    {{ user.firstName }}
  </current-user>
  ```


- prop重命名

  ```vue
  <current-user v-slot="{ user: person }">
    {{ person.firstName }}
  </current-user>
  ```


- 定义后备内容，用于插槽prop是undefined的情形

  ```vue
  <current-user v-slot="{ user = { firstName: 'Guest' } }">
    {{ user.firstName }}
  </current-user>
  ```

##### （5）动态插槽名

- 动态指令参数可以用在`v-slot`上，来定义动态的插槽名

  ```vue
  <base-layout>
    <template v-slot:[dynamicSlotName]>
      ...
    </template>
  </base-layout>
  ```



#### 6.动态组件

- 动态组件是在运行时在组件之间动态切换的方法，在一个挂载点使用多个组件，并进行动态切换

- 使用：

  ```vue
  <component :is='component' />
  ```


- `v-bind:is` 属性可以传递两种类型的参数：
  - 组件的名称
  - 组件的选项对象



#### 7.缓存组件

- 在一个多标签的界面中使用`is`attribute来切换不同的组件时这些组建的状态不会保存，这是因为在每次切换新组件的时候，`Vue`都创建了一个新的`currentTabComponent`

- 为了在切换组件的时候可以保存组件的状态，可以使用`<keep-alive>` 元素将其动态组件包裹起来

  ```vue
  <!-- 失活的组件将会被缓存！-->
  <keep-alive>
    <component v-bind:is="currentTabComponent"></component>
  </keep-alive>
  ```


- 注意这个 `<keep-alive>` 要求被切换到的组件都有自己的名字，不论是通过组件的 `name` 选项还是局部/全局注册

  ​


### 二、Vue 原理

#### 1.如何理解MVVM

- `MVVM` 表示的是 `Model-View-ViewModel`。
  - **Model**: 数据模型层；负责处理业务逻辑以及和服务器端进行交互。
  - **View**: 视图层；负责把数据模型转化为UI展示，可以简单的理解为HTML。
  - **View Model**: 视图模型层；用于连接Model和View,是Model和View的之间的通信桥梁。


- 在MVVM架构中，是不允许数据和视图直接通信的，只能通过ViewModel来通信，而ViewModel就是定义了一个Observer观察者。ViewModel是连接View和Model的中间件。

- ViewModel能够观察到数据的变化，并对视图对应的内容进行更新。

- ViewModel能够监听到视图的变化，并能够通知数据发生变化。

  ​

#### 2.监听data变化的核心API

- **vue2.0核心api是Object.defineProperty，vue3.0是启用proxy实现响应式**

- 当你把一个普通的 JavaScript 对象传入 Vue 实例作为 `data` 选项，Vue 将遍历此对象所有的 property，并使用 `Object.defineProperty` 把这些 property 全部转为 `getter/setter`

- 这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue 能够追踪依赖，在 property 被访问和修改时通知变更

- 每个组件实例都对应一个 **watcher** 实例，它会在组件渲染的过程中把“接触”过的数据 property 记录为依赖。之后当依赖项的 setter 触发时，会通知 watcher，从而使它关联的组件重新渲染

- 缺点

  - 无法监听到新增属性（Vue.set）/删除属性(Vue.delete）
  - 深度监听，需要递归到底，一次性计算量大；
  - 无法原生监听数组，需要特殊处理

  ```js
  // 触发更新视图
  function updateView() {
      console.log('视图更新')
  }
   
  // 重新定义数组原型
  const oldArrayProperty = Array.prototype
  // 创建新对象，原型指向 oldArrayProperty ，再扩展新的方法不会影响原型
  const arrProto = Object.create(oldArrayProperty);
  ['push', 'pop', 'shift', 'unshift', 'splice'].forEach(methodName => {
      arrProto[methodName] = function () {
          updateView() // 触发视图更新
          oldArrayProperty[methodName].call(this, ...arguments)
          // Array.prototype.push.call(this, ...arguments)
      }
  })
   
  // 重新定义属性，监听起来
  function defineReactive(target, key, value) {
      // 深度监听
      observer(value)
   
      // 核心 API
      Object.defineProperty(target, key, {
          get() {
              return value
          },
          set(newValue) {
              if (newValue !== value) {
                  // 深度监听
                  observer(newValue)
   
                  // 设置新值：value 一直在闭包中，此处设置完之后，再 get 时也是会获取最新的值
                  value = newValue
   
                  // 触发更新视图
                  updateView()
              }
          }
      })
  }
   
  // 监听对象属性
  function observer(target) {
      if (typeof target !== 'object' || target === null) {
          // 不是对象或数组
          return target
      }
   
      // 污染全局的 Array 原型
      // Array.prototype.push = function () {
      //     updateView()
      //     ...
      // }
   
      if (Array.isArray(target)) {
          target.__proto__ = arrProto
      }
   
      // 重新定义各个属性（for in 也可以遍历数组）
      for (let key in target) {
          defineReactive(target, key, target[key])
      }
  }
   
  // 准备数据
  const data = {
      name: 'zhangsan',
      age: 20,
      info: {
          address: '北京' // 需要深度监听
      },
      nums: [10, 20, 30]
  }
   
  // 监听数据
  observer(data)
   
  // 测试
  // data.name = 'lisi'
  // data.age = 21
  // // console.log('age', data.age)
  // data.x = '100' // 新增属性，监听不到 —— 所以有 Vue.set
  // delete data.name // 删除属性，监听不到 —— 所有已 Vue.delete
  // data.info.address = '上海' // 深度监听
  data.nums.push(4) // 监听数组
  ```



#### 3.vue如何监听数组变化

- 更改数组的原型

  ​

#### 4.虚拟DOM

- 从本质上来说，`Vistual DOM` 是一个**javascript对象**，通过对象的方式来表示DOM结构。将页面的形状抽象为JS对象的形式，配合不同的渲染工具，使跨平台渲染成为可能。

- 通过事务处理机制，将多次DOM修改的结果一次性更新到页面上，从而**有效的减少页面渲染的次数，减少修改DOM重绘重排次数，提高渲染性能**

- 虚拟DOM是对DOM的抽象，这个对象是更加轻量级的对DOM的描述，他设计的最初目的就是更好的跨平台，比如 Node.js 就没有 DOM，如果想实现 SSR，那么就只能借助虚拟 DOM，因为虚拟 DOM 本身就是 JS 对象，在代码渲染到页面之前，Vue 和 React 会把代码转化成一个对象（虚拟 DOM），以对象的形式来描述真实 DOM 结构，最终渲染到页面。**在每次数据发生变化前，虚拟DOM都会缓存一份，变化之时，现在的虚拟DOM会与缓存的虚拟DOM进行比较。**

- **Vue中的虚拟DOM**

  - vue中状态变化时，只能通知到组件，组件内部的变化需要通过虚拟DOM去进行比对与渲染
  - 在vue中，我们使用模板来描述状态与DOM之间的映射关系。vue通过编译将模板转换成渲染函数，执行渲染函数就可以得到一个虚拟节点树，使用虚拟节点数就可以渲染页面
  - 虚拟DOM在vue中主要提供与真实节点对应的虚拟节点vnode,然后需要将vnode和oldVnode进行比对，然后更新视图，对比两个虚拟节点的算法是patch算法

  ​


#### 5.diff算法

- **概念：**diff算法就是进行虚拟节点对比，并返回一个patch对象，用来存储两个节点不同的地方，最后用patch记录的消息去局部更新DOM（diff的过程就是调用名为patch的函数，比较新旧节点，一边比较一边给真实的DOM打补丁）
- **特点：** 比较只会在同层级进行，不会跨层级比较；在diff算法比较的过程中，循环从两边向中间比较
- **步骤：**
  - 用javascript对象结构表示DOM树结构，然后用这个树构建一个真正的DOM树，插到文档当中
  - 当状态变更的时候，重新构造一棵新的对象树，然后用新的对象树和旧的对象树进行比较（diff），记录两棵树的差异。
  - 把第二颗树所记录的差异应用到第一棵树所构建的真正的DOM树上（patch），视图进行更新


- **比较方式：**深度优先，同层比较

- **原理：**

  **(1)根据节点是否相同初步判断：patch函数**
  **此函数可以说的diff算法的入口函数了，当数据发生改变时，defineProperty => get会调用Dep的notify方法调用Watcher进行更新，当每次走get调用_update的时候，都会走patch函数，更新真实DOM在调用patch函数时，会将新老vnode都传入oldVnode、Vnode（不考虑服务器渲染）**

  ```js
  function patch(oldVnode, vnode, hydrating, removeOnly) {
      if (isUndef(vnode)) { // 没有新节点，直接执行destory钩子函数
          if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
          return
      }

      let isInitialPatch = false
      const insertedVnodeQueue = []

      if (isUndef(oldVnode)) {
          isInitialPatch = true
          createElm(vnode, insertedVnodeQueue) // 没有旧节点，直接用新节点生成dom元素
      } else {
          const isRealElement = isDef(oldVnode.nodeType)
          if (!isRealElement && sameVnode(oldVnode, vnode)) {
              // 判断旧节点和新节点自身一样，一致执行patchVnode
              patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
          } else {
              // 否则直接销毁及旧节点，根据新节点生成dom元素
              if (isRealElement) {

                  if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
                      oldVnode.removeAttribute(SSR_ATTR)
                      hydrating = true
                  }
                  if (isTrue(hydrating)) {
                      if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
                          invokeInsertHook(vnode, insertedVnodeQueue, true)
                          return oldVnode
                      }
                  }
                  oldVnode = emptyNodeAt(oldVnode)
              }
              return vnode.elm
          }
      }
  }

  ```

  - patch函数前两个参数位为oldVnode 和 Vnode ，分别代表新的节点和之前的旧节点，主要做了四个判断：

    - 没有新节点，直接触发旧节点的destory钩子（代表组件被删除）
    - 没有旧节点，说明是页面刚开始初始化的时候，此时，根本不需要比较了，直接全是新建，所以只调用 createElm（代表创建新组件）
    - 旧节点和新节点自身一样，通过 sameVnode 判断节点是否一样，一样时，直接调用 patchVnode去处理这两个节点
    - 旧节点和新节点自身不一样，当两个节点不一样的时候，直接创建新节点，删除旧节点

    ```
    无新节点时：代表组件被删除，调用invokeDestroyHook卸载组件
    无老节点：代表要创建的是一个新组件，无需算法比较直接createElm创建新组件
    有老节点，有新节点：通过sameVnode比对组件
    	老节点、新节点比对为true：调用patchVnode对比与更新
    	老节点、新节点比对为false：抛弃旧节点，调用createElm生成新的
    ```

  **（2）更新+对比子节点patchNode**

  **该函数是递归调用`updateChildren`的入口，除了比对子节点以外，还会将老节点上的东西更新到新节点中，在`updateChildren`函数中，更新节点调用的就是该方法**

  - patchVnode主要做了几个判断：

    - 新节点是否是文本节点，如果是，则直接更新dom的文本内容为新节点的文本内容
    - 新节点和旧节点如果都有子节点，则处理比较更新子节点
    - 只有新节点有子节点，旧节点没有，那么不用比较了，所有节点都是全新的，所以直接全部新建就好了，新建是指创建出所有新DOM，并且添加进父节点
    - 只有旧节点有子节点而新节点没有，说明更新后的页面，旧节点全部都不见了，那么要做的，就是把所有的旧节点删除，也就是直接把DOM 删除
    - 子节点不完全一致，则调用updateChildren

    ```
    新节点是文本节点：走setTextContent，更新文本内容
    新节点有子节点
    	新节点和老节点都有子节点：如果子集完全一致不更新，不一致走updateChildren比较
    	只有新节点有子节点：不需要比较，直接将新节点插入到elm(父dom)下addVnodes
    	只有老节点有子节点：不需要比较，删除所有子节点removeVnodes
    	老节点是文本节点：走setTextContent清空文本
    ```


  - updateChildren主要进行了以下判断
    - 当新老 VNode 节点的 start 相同时，直接 patchVnode ，同时新老 VNode 节点的开始索引都加 1
    - 当新老 VNode 节点的 end相同时，同样直接 patchVnode ，同时新老 VNode 节点的结束索引都减 1
    - 当老 VNode 节点的 start 和新 VNode 节点的 end 相同时，这时候在 patchVnode 后，还需要将当前真实 dom 节点移动到 oldEndVnode 的后面，同时老 VNode 节点开始索引加 1，新 VNode 节点的结束索引减 1
    - 当老 VNode 节点的 end 和新 VNode 节点的 start 相同时，这时候在 patchVnode 后，还需要将当前真实 dom 节点移动到 oldStartVnode 的前面，同时老 VNode 节点结束索引减 1，新 VNode 节点的开始索引加 1
    - 如果都不满足以上四种情形，那说明没有相同的节点可以复用，则会分为以下两种情况：
      从旧的 VNode 为 key 值，对应 index 序列为 value 值的哈希表中找到与 newStartVnode 一致 key 的旧的 VNode 节点，再进行patchVnode，同时 将这个真实 dom移动到 oldStartVnode 对应的真实 dom 的前面
    - 调用 createElm 创建一个新的 dom 节点放到当前 newStartIdx 的位置

  ```js
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
      let oldStartIdx = 0 // 旧头索引
      let newStartIdx = 0 // 新头索引
      let oldEndIdx = oldCh.length - 1 // 旧尾索引
      let newEndIdx = newCh.length - 1 // 新尾索引
      let oldStartVnode = oldCh[0] // oldVnode的第一个child
      let oldEndVnode = oldCh[oldEndIdx] // oldVnode的最后一个child
      let newStartVnode = newCh[0] // newVnode的第一个child
      let newEndVnode = newCh[newEndIdx] // newVnode的最后一个child
      let oldKeyToIdx, idxInOld, vnodeToMove, refElm

      // removeOnly is a special flag used only by <transition-group>
      // to ensure removed elements stay in correct relative positions
      // during leaving transitions
      const canMove = !removeOnly

      // 如果oldStartVnode和oldEndVnode重合，并且新的也都重合了，证明diff完了，循环结束
      while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        // 如果oldVnode的第一个child不存在
        if (isUndef(oldStartVnode)) {
          // oldStart索引右移
          oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left

        // 如果oldVnode的最后一个child不存在
        } else if (isUndef(oldEndVnode)) {
          // oldEnd索引左移
          oldEndVnode = oldCh[--oldEndIdx]

        // oldStartVnode和newStartVnode是同一个节点
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
          // patch oldStartVnode和newStartVnode， 索引左移，继续循环
          patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
          oldStartVnode = oldCh[++oldStartIdx]
          newStartVnode = newCh[++newStartIdx]

        // oldEndVnode和newEndVnode是同一个节点
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
          // patch oldEndVnode和newEndVnode，索引右移，继续循环
          patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
          oldEndVnode = oldCh[--oldEndIdx]
          newEndVnode = newCh[--newEndIdx]

        // oldStartVnode和newEndVnode是同一个节点
        } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
          // patch oldStartVnode和newEndVnode
          patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
          // 如果removeOnly是false，则将oldStartVnode.eml移动到oldEndVnode.elm之后
          canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
          // oldStart索引右移，newEnd索引左移
          oldStartVnode = oldCh[++oldStartIdx]
          newEndVnode = newCh[--newEndIdx]

        // 如果oldEndVnode和newStartVnode是同一个节点
        } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
          // patch oldEndVnode和newStartVnode
          patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
          // 如果removeOnly是false，则将oldEndVnode.elm移动到oldStartVnode.elm之前
          canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
          // oldEnd索引左移，newStart索引右移
          oldEndVnode = oldCh[--oldEndIdx]
          newStartVnode = newCh[++newStartIdx]

        // 如果都不匹配
        } else {
          if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)

          // 尝试在oldChildren中寻找和newStartVnode的具有相同的key的Vnode
          idxInOld = isDef(newStartVnode.key)
            ? oldKeyToIdx[newStartVnode.key]
            : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)

          // 如果未找到，说明newStartVnode是一个新的节点
          if (isUndef(idxInOld)) { // New element
            // 创建一个新Vnode
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)

          // 如果找到了和newStartVnodej具有相同的key的Vnode，叫vnodeToMove
          } else {
            vnodeToMove = oldCh[idxInOld]
            /* istanbul ignore if */
            if (process.env.NODE_ENV !== 'production' && !vnodeToMove) {
              warn(
                'It seems there are duplicate keys that is causing an update error. ' +
                'Make sure each v-for item has a unique key.'
              )
            }

            // 比较两个具有相同的key的新节点是否是同一个节点
            //不设key，newCh和oldCh只会进行头尾两端的相互比较，设key后，除了头尾两端的比较外，还会从用key生成的对象oldKeyToIdx中查找匹配的节点，所以为节点设置key可以更高效的利用dom。
            if (sameVnode(vnodeToMove, newStartVnode)) {
              // patch vnodeToMove和newStartVnode
              patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue)
              // 清除
              oldCh[idxInOld] = undefined
              // 如果removeOnly是false，则将找到的和newStartVnodej具有相同的key的Vnode，叫vnodeToMove.elm
              // 移动到oldStartVnode.elm之前
              canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)

            // 如果key相同，但是节点不相同，则创建一个新的节点
            } else {
              // same key but different element. treat as new element
              createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
            }
          }

          // 右移
          newStartVnode = newCh[++newStartIdx]
        }
      }

  ```


- **总结：**
  - 当数据发生改变时，订阅者watcher就会调用patch给真实的DOM打补丁
  - 通过isSameVnode进行判断，相同则调用patchVnode方法
  - patchVnode做了以下操作：
    - 找到对应的真实dom，称为el
    - 如果都有都有文本节点且不相等，将el文本节点设置为Vnode的文本节点
    - 如果oldVnode有子节点而VNode没有，则删除el子节点
    - 如果oldVnode没有子节点而VNode有，则将VNode的子节点真实化后添加到el
    - 如果两者都有子节点，则执行updateChildren函数比较子节点
  - updateChildren主要做了以下操作：
    - 设置新旧VNode的头尾指针
    - 新旧头尾指针进行比较，循环向中间靠拢，根据情况调用patchVnode进行patch重复流程、调用createElem创建一个新节点，从哈希表寻找 key一致的VNode 节点再分情况操作




#### 6.模板编译前置知识点-with语法

- **概念：**使用 `with` 后，能改变 `{}` 内自由变量的查找方式，将 `{}` 内自由变量当作 `obj` 的属性来查找：

  ```js
  with(obj) {
    console.log(obj.a)
    console.log(obj.b)
    console.log(obj.c) 
  }
  ```


- **with在vue中的应用** ：**Vue**在模板编译中用到了**with** 语法

- 插值

  ```js
  // 插值
  const template = `<p>{{message}}</p>`

  with(this){
    return _c('p',[_v(_s(message))])
  }
  /* 在vue源码中可以搜到各种函数的名称
  	这个 this 就是 vm = new Vue()  实例
    很类似于 h('p', {}, [...]) 函数
    _c 是 createElement 用于生成vnode
    _v 是 createTextVNode 用于生成文本的vnode
    _s 是 toString，以防message是true，则能转换为字符串
  */
  ```


- 表达式

  ```js
  // 表达式
  const template = `<p>{{flag ? message : 'no message found'}}</p>`

  with(this){
    return _c('p',[_v(_s(flag ? message : 'no message found'))])
  }
  ```


- 属性和动态属性

  ```js
  const template = `
     <div id="div1" class="container">
       <img :src="imgurl" />
     </div>
   `

  with(this){
    return _c('div',
      {staticClass:"container",attrs:{"id":"div1"}},
      [
        _c('img',{attrs:{"src":imgurl}})])
  }

  ```


- 条件

  ```js
   const template = `
     <div>
         <p v-if="flag === 'a'">A</p>
         <p v-else>B</p>
     </div>
   `
   
   with(this){
     return 
          _c('div',[(flag === 'a')?_c('p',[_v("A")]):_c('p',[_v("B")])])
   }
  ```


- 循环

  ```js
   const template = `  
     <ul>
       <li v-for="item in list" :key="item.id">{{item.title}}</li>
     </ul>
   `
   
   with (this) {
      return
        _c('ul', _l((list), function (item) { 
        return _c('li', { key: item.id }, [_v(_s(item.title))]) 
      }), 0)
    }
  ```


- 事件

  ```js
   const template = `
     <button @click="clickHandler">submit</button>
  `
   
   with(this){
     return 
      _c('button',{on:{"click":clickHandler}},[_v("submit")])
   }
  ```


- v-model

  ```js
  const template = `<input type="text" v-model="name" />`

  with(this){
    return 
        _c('input',{directives:[{name:"model",rawName:"v-model",
        value:(name),expression:"name"}],attrs:{"type":"text"},domProps:{"value":(name)},
        on:{"input":function($event){if($event.target.composing)return;
          name=$event.target.value}}})
  }
  ```

  ​

#### 7.vue模板编译

- **vue模板被编译成以下内容：**

  - Vue 的模板不是 html，因为它有指令、插值、JS 表达式，能实现判断、循环；
  - html 是标签语言，只有 JS 才能实现判断、循环。


  - 因此，模板一定是转换为某种 JS 代码，即编译模板。


- Vue模板编译过程：

  - 模板编译为render函数，执行render函数返回`vnode`
  - 基于`vnode`在执行`patch和diff`
  - 使用`webpack`时，`vue-loader`会在开发环境下编译模板

  ​

#### 8.vue组件中使用render代替template

- 用`template`来定义Vue组件

  ```js
  Vue.component('heading', {
    template: `<h3><a name="headerId" href="#headerId">this is a tag</a></h3>`
  })
  ```


- 使用`render`函数来定义Vue组件，需要定义一个函数，参数为`createElement`,`createElement` 类似于前面介绍的编译后的函数体，它的第一个参数是标签，第二个参数是属性（可不写），第三个参数是子元素。

  ```js
  Vue.component('heading', {
    render: function(createElement) {
      return createElement(
        'h' + this.level,
        [
          createElement('a', {
            attrs: {
              name: 'headerId',
              href: '#' + 'headerId'
            }
          }, 'this is a tag')
        ]
      )
    }
  })
  ```

  ​

#### 9.Vue组件的渲染和更新

- **渲染过程**
  - 解析模板为render函数
  - 触发响应式，监听data属性的getter,serrer
  - 执行render函数，生成`VNode，path(elem,vnode)`


- **更新过程：**
  - 修改了data,触发了setter
  - 重新执行render函数，生成`VNode，``patch(ele,.vnode)`
  - 再执行`patch(vnode,newVnode)`


#### 10.vnode 之于 Vue 的作用

- 因为 **JS 引擎是单线程**的，而直接频繁的操作和渲染 DOM 时，又是比较消耗性能的，所以由此引发的思考就是：能不能用 JS 先进行一系列的计算和整合，再 patch 到 真实的DOM 上，这样极大地利用了 JS 执行速度快的优势，也能减少直接操作 DOM，从而达到能够很好地提高性能的这样一个目的。然后 vnode就诞生了
- vnode 是用 JS 对象来模拟的一个 DOM 树结构，在真实的 DOM 节点发生改变时，将改变映射到vnode，然后利用 diff 算法和key对vnode进行比对，比对出最小改动最小的 DOM，最后异步渲染将改变最终对应到真实的 DOM 上，即以最小的成本来更新 DOM。
- Vue 利用 vnode这样的一个 JS 对象，一方面对 DOM 结构有了一份 JS 解析，一方面在接收到 DOM 修改时，生成一份新的 vnode，Vue 能够利用这份新 vnode 与旧 vnode 进行比对，在直接修改 DOM 之前，做一层拦截和过滤，帮助 Vue 在渲染 DOM 时，提高了性能。



### 三、Vue真题

#### 1.v-for为何使用key

- vue中列表循环需加:key="唯一标识" 唯一标识可以是item里面id index等，因为vue组件高度复用增加Key可以标识组件的唯一性，为了更好地区别各个组件 key的作用主要是为了高效的更新虚拟DOM（**key的作用主要是为了高效的更新虚拟DOM** ）


- **核心思想：**
  - vue更新已渲染过的元素时，默认使用就地复用策略（如果该元素dom未修改，则复用之前的元素，否则重新渲染这一项）
  - key的作用是唯一标识组件元素，辅助判断新旧vdom节点在逻辑上是不是同一个对象，是否有更新



#### 2.组件data为何是函数

- 当一个组件被定义时，data必须声明为返回一个初始数据对象的函数，因为组件可能被用来创建多个实例。**如果data仍然是一个纯粹的对象，则所有的实例将共享引用同意个数据对象。**通过提供data函数，每次创建一个实例后，我们能够调用data函数，从而**返回初始数据的一个全新副本**数据对象。

  ```
  1.是为了在重复创建实例的时候避免共享同一数据造成的数据污染　　

  2.写函数的原因是为了保证这个对象是独立的，如果可以保证对象是惟一的,也可以不写函数直接写对象。
  ```


- 为什么放在对象里面会造成数据污染但是放在方法里面就不会造成数据污染了呢？**this的指向在函数定义的时候是确定不了的，只有在函数执行的时候才能确定this到底指向谁，实际上this指向调用它的对象。又有一个疑问了，明明指向的是同一个方法为什么会有不一样的呢？难道克隆了一个？答案：定义在构造函数内部的方法,会在它的每一个实例上都克隆这个方法;定义在构造函数的`prototype`属性上的方法会让它的所有示例都共享这个方法,但是不会在每个实例的内部重新定义这个方法引用。**



#### 3.何时需要使用beforeDestroy

- 可能在当前页面中使用了`$on`方法，那需要在组件销毁前解绑。
- 清除自己定义的定时器
- 解除事件的绑定 `scroll mousemove ....`



#### 4.diff算法时间复杂度

- 如果不做优化的话，原`diff`算法需要经过新旧vdom遍历以及排序，时间复杂度为`O(n3)`,优化之后的时间复杂度为`O(n)`


- vue做了以下优化：
  - 只比较同一层级，不跨级比较
  - tag不相同，则会直接删除重建，不再深度比较
  - tag和key都相同，则认为其为相同节点，不再深度比较


- 这里的n指的是页面的VDOM节点数，这个不太严谨。如果更严谨一点，我们应该应该假设 变化之前的节点数为m，变化之后的节点数为n。
- React 和 Vue 做优化的前提是“放弃了最优解“，本质上是一种权衡，有利有弊。


- React 和 Vue 做的假设是：
  - 检测VDOM的变化只发生在同一层
  - 检测VDOM的变化依赖于用户指定的key


- 如果变化发生在不同层或者同样的元素用户指定了不同的key或者不同元素用户指定同样的key， React 和 Vue都不会检测到，就会发生莫名其妙的问题。


- 但是React 认为， 前端碰到上面的第一种情况概率很小，第二种情况又可以通过提示用户，让用户去解决，因此 这个取舍是值得的。没有牺牲空间复杂度，却换来了在大多数情况下时间上的巨大提升。明智的选择！



#### 5.vue常见性能优化

- **编码优化:**
  - 不要将所有的数据都放在data中，data中的数据都会增加getter和setter，会收集对应的 watcher，这样就会降低性能。
  - vue 在 v-for 时给每项元素绑定事件需要用事件代理，节约性能。
  - 单页面采用keep-alive缓存组件。
  - 尽可能拆分组件，来提高复用性、增加代码的可维护性，减少不必要的渲染。因为组件粒度最细，改组件的数组，它只会渲染当前的组件。
  - v-if 当值为false时内部指令不会执行，具有阻断功能，很多情况下使用v-if替代v-show，合理使用if和show。
  - key 保证唯一性，不要使用索引 ( vue 中diff算法会采用就地复用策略)。
  - data中的所以数据都会被劫持，所以将数据尽可能扁平化，如果数据只是用来渲染可以使用Object.freeze，可以将数据冻结起来，这样就不会增加getter和setter
  - 合理使用路由懒加载、异步组件。
  - 尽量采用runtime运行时版本。
  - 数据持久化的问题，使用防抖、节流进行优化，尽可能的少执行和不执行。


- **加载性能：**
  - 使用第三方插件实现按需加载( babel-plugin-component )
  - 滚动到可视区域动态加载 
  - 图片懒加载 


- **用户体验：**
  - app-skeleton 骨架屏
  - app-shell app壳
  - pwa 可以实现H5的离线缓存，使用servicewor


- **SEO 优化：**
  - 预渲染插件 prerender-spa-plugin，可以把我们代码提前运行起来，最后将代码保存下来，缺陷就是不实时。
  - 服务端渲染 ssr


- **打包优化：**
  - 使用 cdn 的方式加载第三方模块
  - 多线程打包 happypack
  - 抽离公共文件 splitChunks
  - sourceMap 生成


- **缓存和压缩：**
  - 客户端缓存、服务端缓存
  - 服务端 gzip 压缩



beforeCreate -> 使用 setup() 
created -> 使用 use setup() 
beforeMount -> onBeforeMount 
mounted -> onMounted 
beforeUpdate -> onBeforeUpdate 
updated -> onUpdated 
beforeDestroy-> onBeforeUnmount 
destroyed -> onUnmounted 
activated -> onActivated 
deactivated -> onDeactivated 
errorCaptured -> onErrorCaptured
