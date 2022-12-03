# Vue2

### 一、组件之间的通信

如图所示：A,B,C三个组件依次嵌套

![](https://img-blog.csdnimg.cn/20210422160322929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1pZUzEwMDAw,size_16,color_FFFFFF,t_70#pic_center)

#### 1.父子组件之间的通信

- 父 —> 子：`A to B` 通过`props`的方式向子组件传递
- 子 —> 父：`B to A` 通过在B组件中`$emit`，A组件中`v-on`的方式实现
- 通过设置全局`Vue`x共享状态，通过`compted`计算属性和`commit mutation`的方式实现数据的获取和更新，已达到父子组件通信的目的
- `Vue Event Bus`，使用`Vue`的实例，实现事件的监听和发布，实现组件之间的传递

#### 2.跨多级组件之间的通信

- 如上图，A 组件与 C 组件之间属于跨多级的组件嵌套关系，以往两者之间如需实现通信，往往通过以下方式实现：
  - 借助B组件的中转，从上到下使用`props`依次传递，从下到上使用`$emit`事件的传递，达到跨级组件通信的效果
  - 借助`Vuex`的全局状态共享
  - `Vue Event Bus`，使用Vue的实例实现事件的监听和发布，实现组件之间的通信
- 第一种通过 props 和 $emit 的方式，使得组件之间的业务逻辑臃肿不堪，B组件在其中仅仅充当的是一个中转站的作用；
  第二种 Vuex 的方式，某些情况下似乎又有点大材小用的意味（仅仅是想实现组件之间的一个数据传递，并非数据共享的概念）；
  第三种情况的使用在实际的项目操作中发现，如不能实现很好的事件监听与发布的管理，往往容易导致数据流的混乱，在多人协作的项目中，不利于项目的维护；
- `$attrs` 以及 `$listeners` 的出现解决的就是第一种情况的问题，B 组件在其中传递 `props` 以及事件的过程中，不必在写多余的代码，仅仅是将 `$attrs` 以及 `$listeners` 向上或者向下传递即可。


##### （1）inheritAttrs

- 默认值为true，继承所有的父组件的属性（除去组件内props绑定的属性）用在子组件的实例上
- 简单的说，`inheritAttrs：true` 继承除 `props` 之外的所有属性；`inheritAttrs：false` 只继承 `class style` 属性

##### （2）$attrs

- 继承所有的父组件属性（除了 prop 传递的属性、class 和 style ），一般用在子组件的子元素上

  ```vue
  <child-b v-bind="$attrs"></child-b>
  ```

##### （3）$listeners

- `$ listeners` 属性，它是一个对象，里面包含了作用在这个组件上的所有监听器，你就可以配合 `v-on="$listeners"` 将所有的事件监听器指向这个组件的某个特定的子元素。（相当于子组件继承父组件的事件）

  ```vue
  <child-b v-on="$listeners"></child-b>
  ```

##### （4）.sync修饰符

- 有些情况下，我们可能需要对一个prop进行双向绑定，不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以变更父组件，且在父组件和子组件两侧都没有明显的变更来源。

- 因此，推荐以`update:myPropName`的模式触发事件取而代之

  ```vue
  <!-- 父组件中 -->
  <child-sync v-show="isShow" @update:isShow="val => isShow=val"></child-sync>

  <!-- 子组件中 -->
  <input type="text" @click="upIsShow" value="点我隐身" />
  export default {
    methods: {
      upIsShow() {
        this.$emit('update:isShow', false)
      }
    }
  }
  ```


  - 为了方便起见， vue为这种模式提供了一种缩写，即`.sync`修饰符

    `:isShow.sync="isShow"` 相当于是 `@update:isShow="bol => isShow=bol"` 的语法糖

    ```vue
    <!-- 父组件中 -->
    <child-sync v-show="isShow" :isShow.sync="isShow"></child-sync>

    <!-- 子组件中 -->
    <input type="text" @click="upIsShow" value="点我隐身" />
    export default {
      methods: {
        upIsShow() {
          this.$emit('update:isShow', false)
        }
      }
    }
    ```


  - 注意：

    - 注意带有 `.sync` 修饰符的 `v-bind` **不能**和表达式一起使用 (例如 `v-bind:title.sync=”doc.title + ‘!’”` 是无效的)。取而代之的是，你只能提供你想要绑定的 property 名，类似 `v-model`。

    - 当我们用一个对象同时设置多个 prop 的时候，也可以将这个 `.sync` 修饰符和 `v-bind` 配合使用：

      ```vue
      <text-document v-bind.sync="doc"></text-document>
      ```

      这样会把 `doc` 对象中的每一个 property (如 `title`) 都作为一个独立的 prop 传进去，然后各自添加用于更新的 `v-on` 监听器。


    将 `v-bind.sync` 用在一个字面量的对象上，例如 `v-bind.sync=”{ title: doc.title }”`，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。
### 二、插槽

- 概念：**插槽**（Slot）是`vue`为组件的封装者提供的能力，允许开发者在封装组件是，把不确定的、希望由用户指定的部分定义为插槽
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

#### 1.后备内容

- 有时为一个插槽设置具体的后备（也就是默认的）内容是很有用的，他只会在没有提供内容的时候被渲染，如果提供内容，则这个提供的内容会被渲染从而取代后背内容

  ```vue
  <!-- 没有提供内容，slot标签内的内容会被渲染 -->
  <submit-button></submit-button>  // 父组件
  <button type="submit">Submit</button>  // 子组件

  <!-- 提供内容，slot标签内的内容不会被渲染，而会渲染提供的内容 -->
  <submit-button>Save</submit-button>  // 父组件
  <button type="submit">Submit</button>  // 子组件
  ```

#### 2.具名插槽

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

#### 3.作用域插槽

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

#### 4.解构插槽prop

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

#### 5.动态插槽名

- 动态指令参数可以用在`v-slot`上，来定义动态的插槽名

  ```vue
  <base-layout>
    <template v-slot:[dynamicSlotName]>
      ...
    </template>
  </base-layout>
  ```

### 三、动态组件

#### 1.is属性

- 在两种情况下要使用`is`
  - 需要频繁更换当前这个组件。比如在首页的时候，点击不同的tab使用的不同的组件
  - 遇到html的固定语法，想在里面使用组件但是没办法使用（例如，table 下面固定的tr，seelct下option，ul下li）


- 频繁更换当前这个组件

  ```vue
  <!-- 组件会在 `组件名` 改变时改变 -->
  <div :is="组件名变量"></div>
  ```


- 解决html的固定写法

  - 在某些元素内部，有特定的包裹元素规范，不能包裹其他元素。比如`ul`内只能包含`li`，而不能使用自定义组件。也就是说，这样是不允许的

    ```vue
    <ul>
    	<li is="my-component"></li>
    </ul>
    ```

#### 2.keep-alive

- 在一个多标签的界面中使用`is`attribute来切换不同的组件时这些组建的状态不会保存，这是因为在每次切换新组件的时候，`Vue`都创建了一个新的`currentTabComponent`

- 为了在切换组件的时候可以保存组件的状态，可以使用`<keep-alive>` 元素将其动态组件包裹起来

  ```vue
  <!-- 失活的组件将会被缓存！-->
  <keep-alive>
    <component v-bind:is="currentTabComponent"></component>
  </keep-alive>
  ```


- 注意这个 `<keep-alive>` 要求被切换到的组件都有自己的名字，不论是通过组件的 `name` 选项还是局部/全局注册

#### 3.异步组件

- 在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器中加载一个模块，为了简化，Vue允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue只有在这个组件需要被渲染的时候才会触发这个工厂函数，且会把结果缓存起来供未来重新渲染

  ```js
  Vue.component('async-example', function (resolve, reject) {
    setTimeout(function () {
      // 向 `resolve` 回调传递组件定义
      resolve({
        template: '<div>I am async!</div>'
      })
    }, 1000)
  })
  ```


- 这个工厂函数会收到一个resolve回调，这个回调函数会在你从服务器中得到组件定义的时候被调用，可以将步组件和 webpack的code-splitting功能一起配合使用

  ```js
  Vue.component('async-webpack-example', function (resolve) {
    // 这个特殊的 `require` 语法将会告诉 webpack
    // 自动将你的构建代码切割成多个包，这些包
    // 会通过 Ajax 请求加载
    require(['./my-async-component'], resolve)
  })
  ```


- 处理加载状态：异步组件工厂函数也可以返回一个如下格式的对象

  ```js
  const AsyncComponent = () => ({
    // 需要加载的组件 (应该是一个 `Promise` 对象)
    component: import('./MyComponent.vue'),
    // 异步组件加载时使用的组件
    loading: LoadingComponent,
    // 加载失败时使用的组件
    error: ErrorComponent,
    // 展示加载时组件的延时时间。默认值是 200 (毫秒)
    delay: 200,
    // 如果提供了超时时间且组件加载也超时了，
    // 则使用加载失败时使用的组件。默认值是：`Infinity`
    timeout: 3000
  })
  ```

### 四、访问元素、组件、程序化事件

#### 1.$root

- `$root`可以访问到根组件，在每个 `new Vue` 实例的子组件中，其根实例可以通过 `$root` property 进行访问

  ```js
  <!-- 根组件 -->
  new Vue({
    data() {
      return {
        loading: true,
        num: 1
      }
    },
    methods: {
      add() {
        this.num++
      }
    },
    render: (h) => h(App)
  }).$mount('#app')

  <!-- 其余组件中 -->
  mounted() {
      console.log(this.$root.loading)   // true
      console.log(this.$root.num)  // num + 1
    }
  ```

#### 2.$parent

- `$parent`  可以用来从一个子组件访问父组件的实例。它提供了一种机会，可以在后期随时触达父级组件，以替代将数据以 prop 的方式传入子组件的方式

  ```js
  <!-- 父组件 -->
  methods: {
      add() {
        this.$root.num++
      }
    }
  <!-- 子组件 -->
  <button @click="$parent.add">++</button>  // num+ 1
  ```

#### 3.$refs

- ref：如果在普通的 `DOM` 元素上使用，引用指向的就是 `DOM` 元素；如果用在`子组件`上，引用就指向`组件`实例

- `this.$refs` 返回了一个对象，包含当前页面的所有 `ref`引用名称

- 注意：需要在DOM渲染完成后才可以使用，因此不能再created（）中使用，需要在mounted（）中使用

  ```vue
  <base-slot ref="child"></base-slot>base-slot>

  mounted() {
      console.log(this.$refs.child.user)  // 访问子组件的user
    }
  ```

#### 4.依赖注入

```vue
<google-map>
  <google-map-region v-bind:shape="cityBoundaries">
    <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
  </google-map-region>
</google-map>
```

- 在这个组件里，所有 `<google-map>` 的后代都需要访问一个 `getMap` 方法，以便知道要跟哪个地图进行交互。不幸的是，使用 `$parent` property 无法很好的扩展到更深层级的嵌套组件上。这也是依赖注入的用武之地，它用到了两个新的实例选项：`provide` 和 `inject`

- `provide` 选项允许我们指定我们想要**提供**给后代组件的数据/方法

  ```js
  <!-- <google-map> 内部的 getMap 方法 -->
  provide: function () {
    return {
      getMap: this.getMap
    }
  }
  ```


- 然后在任何后代组件里，都可以使用`inject`选项来接受指定的我们想要添加在这个实例上的`property`

  ```vue
  inject: ['getMap']
  ```


- 相比 `$parent` 来说，这个用法可以让我们在*任意*后代组件中访问 `getMap`，而不需要暴露整个 `<google-map>` 实例。这允许我们更好的持续研发该组件，而不需要担心我们可能会改变/移除一些子组件依赖的东西。同时这些组件之间的接口是始终明确定义的
- 实际上，可以把依赖注入看作一部分“大范围有效的 prop”，除了：
  - 祖先组件不需要知道哪些后代组件使用它提供的 property
  - 后代组件不需要知道被注入的 property 来自哪里


- 依赖注入还是有负面影响的。它将你应用程序中的组件与它们当前的组织方式耦合起来，使重构变得更加困难。同时所提供的 property 是非响应式的。这是出于设计的考虑，因为使用它们来创建一个中心化规模化的数据跟使用`$root`做这件事都是不够好的。如果你想要共享的这个 property 是你的应用特有的，而不是通用化的，或者如果你想在祖先组件中更新所提供的数据，那么这意味着你可能需要换用一个像 [Vuex](https://github.com/vuejs/vuex) 这样真正的状态管理方案了

#### 5.程序化的事件侦听器

##### （1）$emit

- `vm.$emit( event, […args] )`:触发当前实例上的事件。附加参数都会传给监听器回调，如果没有参数，形式为`vm.$emit(event)`

  ```vue
  <!-- 父组件 -->
  <template>
  <div>
    <div>父组件的toCity{{toCity}}</div>
    <train-city @showCityName="updateCity" :sendData="toCity"></train-city>
   </div>
  <template>
    <script>
      import TrainCity from "./train-city";
      export default {
        name:'index',
        components: {TrainCity},
        data () {
          return {
            toCity:"北京"
          }
        },
        methods:{
          updateCity(data){//触发子组件城市选择-选择城市的事件
            this.toCity = data.cityname;//改变了父组件的值
            console.log('toCity:'+this.toCity)
          }
        }
      }
    </script>
   
  <!-- 子组件 -->
    methods:{
    	select(val) {
    		let data = {
    			cityname: val
    		};
    		this.$emit('showCityName',data);//select事件触发后，自动触发showCityName事件
    	}
    }
  ```

##### （2）$on,once,off

- 通过 `$on(eventName, eventHandler)` ：侦听一个事件

- 通过 `$once(eventName, eventHandler)` ：一次性侦听一个事件

- 通过 `$off(eventName, eventHandler)` ：停止侦听一个事件

  ```vue
  // 一次性将这个日期选择器附加到一个输入框上
  // 它会被挂载到 DOM 上。
  mounted: function () {
    // Pikaday 是一个第三方日期选择器的库
    this.picker = new Pikaday({
      field: this.$refs.input,
      format: 'YYYY-MM-DD'
    })
  },
  // 在组件被销毁之前，
  // 也销毁这个日期选择器。
  beforeDestroy: function () {
    this.picker.destroy()
  }
  ```


- 这里有两个潜在的问题：
  - 它需要在这个组件实例中保存这个 `picker`，如果可以的话最好只有生命周期钩子可以访问到它。这并不算严重的问题，但是它可以被视为杂物。
  - 我们的建立代码独立于我们的清理代码，这使得我们比较难于程序化地清理我们建立的所有东西


- 你应该通过一个程序化的侦听器解决这两个问题:

```vue
mounted: function () {
  var picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
  })

  this.$once('hook:beforeDestroy', function () {
    picker.destroy()
  })
}
```

#### 6.循环引用

##### （1）递归组件

- 组件是可以在它们自己的模板中调用自身的。不过它们只能通过 `name` 选项来做这件事：

  ```vue
  name: 'unique-name-of-my-component'
  ```


- 当你使用 `Vue.component` 全局注册一个组件时，这个全局的 ID 会自动设置为该组件的 `name` 选项

- 稍有不慎，递归组件就可能导致无限循环：

  ```vue
  name: 'stack-overflow',
  template: '<div><stack-overflow></stack-overflow></div>'
  ```

##### （2）组件之间的循环引用

- `$options`：获取、调用data外定义的属性

  ```vue
  <script>
    export default {
      data() {
        return {
        };
      },
      //在data外面定义的属性和方法通过$options可以获取和调用
      name: "options_test",
      age: 18,
      testOptionsMethod() {
        console.log("hello options");
      },
      created() {  
        console.log(this.$options.name);  // options_test
        console.log(this.$options.age);  // 18
        this.$options.testOptionsMethod();  // hello options
      },
  </script>
  ```


- 假设你需要构建一个文件目录树，像访达或资源管理器那样的。你可能有一个 `<tree-folder>` 组件，还有一个 `<tree-folder-contents>` 组件，模板如下：

  ```vue
  <!-- <tree-folder> -->
  <p>
    <span>{{ folder.name }}</span>
    <tree-folder-contents :children="folder.children"/>
  </p>

  <!-- <tree-folder-contents> -->
  <ul>
    <li v-for="child in children">
      <tree-folder v-if="child.children" :folder="child"/>
      <span v-else>{{ child.name }}</span>
    </li>
  </ul>
  ```


- 仔细观察这两个组件，会发现这些组件在渲染树中互为对方的后代和祖先，这是一个悖论！当通过`Vue.component` 全局注册组件的时候，这个悖论会被自动解开；但如果使用一个模块系统依赖/导入组件（webpack）会遇到一个错误

- 原因是：我们先把两个组件称为 A 和 B。模块系统发现它需要 A，但是首先 A 依赖 B，但是 B 又依赖 A，但是 A 又依赖 B，如此往复。这变成了一个循环，不知道如何不经过其中一个组件而完全解析出另一个组件。

- 为了解决这个问题，我们需要给模块系统提供一个点，那里“A *反正*是需要 B 的，但是我们不需要先解析 B。”

- 在我们的例子中，把 `<tree-folder>` 组件设为了那个点。我们知道那个产生悖论的子组件是 `<tree-folder-contents>` 组件，所以我们会等到生命周期钩子 `beforeCreate` 时去注册它：

  ```vue
  beforeCreate: function () {
  	this.$options.components.TreeFolderContents = require('./tree-folder-		contents.vue').default
  }
  ```


- 或者，在本地注册组件的时候，你可以使用 webpack 的异步 `import`：

  ```vue
  components: {
    TreeFolderContents: () => import('./tree-folder-contents.vue')
  }
  ```

#### 7.模板定义的替代品

##### （1）内联模板

- 当 `inline-template` 这个特殊的 attribute 出现在一个子组件上时，这个组件将会使用其里面的内容作为模板，而不是将其作为被分发的内容。这使得模板的撰写工作更加灵活。

  ```vue
  <my-component inline-template>
    <div>
      <p>These are compiled as the component's own template.</p>
      <p>Not parent's transclusion content.</p>
    </div>
  </my-component>
  ```


- 内联模板需要定义在 Vue 所属的 DOM 元素内

##### （2）X-Template

- 另一个定义模板的方式是在一个 `<script>` 元素中，并为其带上 `text/x-template` 的类型，然后通过一个 id 将模板引用过去

  ```vue
  <script type="text/x-template" id="hello-world-template">
    <p>Hello hello hello</p>
  </script>

  Vue.component('hello-world', {
    template: '#hello-world-template'
  })
  ```


- x-template 需要定义在 Vue 所属的 DOM 元素外

### 五、进入/离开过渡效果

- 概述：Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。
  - 在 CSS 过渡和动画中自动应用 class
  - 可以配合使用第三方 CSS 动画库，如 Animate.css
  - 在过渡钩子函数中使用 JavaScript 直接操作 DOM
  - 可以配合使用第三方 JavaScript 动画库，如 Velocity.js

#### 1.单元素/组件的过渡

- Vue提供了transition的封装组件在下列情形中，可以给任何元素和组件添加进入/离开过渡
  - 条件渲染 (使用 `v-if`)
  - 条件展示 (使用 `v-show`)
  - 动态组件
  - 组件根节点


- 当插入或删除包含在 `transition` 组件中的元素时，Vue 将会做以下处理：
  - 自动嗅探目标元素是否应用了 CSS 过渡或动画，如果是，在恰当的时机添加/删除 CSS 类名。
  - 如果过渡组件提供了JavaScript钩子函数，这些钩子函数将在恰当的时机被调用。
  - 如果没有找到 JavaScript 钩子并且也没有检测到 CSS 过渡/动画，DOM 操作 (插入/删除) 在下一帧中立即执行。(注意：此指浏览器逐帧动画机制，和 Vue 的 `nextTick` 概念不同)

##### （1）过渡的类名

- 在进入/离开的过渡中，会有 6 个 class 切换
  - `v-enter`：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
  - `v-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
  - `v-enter-to`：**2.1.8 版及以上**定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 `v-enter` 被移除)，在过渡/动画完成之后移除。
  - `v-leave`：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
  - `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
  - `v-leave-to`：**2.1.8 版及以上**定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 `v-leave` 被删除)，在过渡/动画完成之后移除。


- 对于这些在过渡中切换的类名来说，如果你使用一个没有名字的 `<transition>`，则 `v-` 是这些类名的默认前缀。如果你使用了 `<transition name="my-transition">`，那么 `v-enter` 会替换为 `my-transition-enter`。
- `v-enter-active` 和 `v-leave-active` 可以控制进入/离开过渡的不同的缓和曲线

##### （2）CSS动画

- CSS 动画用法同 CSS 过渡，区别是在动画中 `v-enter` 类名在节点插入 DOM 后不会立即删除，而是在 `animationend` 事件触发时删除

  ```css
  .bounce-enter-active {
    animation: bounce-in .5s;
  }
  .bounce-leave-active {
    animation: bounce-in .5s reverse;
  }
  @keyframes bounce-in {
    0% {
      transform: scale(0);
    }
    50% {
      transform: scale(1.5);
    }
    100% {
      transform: scale(1);
    }
  }
  ```


- 补充：Velocity.js是一个简单易用，高性能且功能丰富的轻量级JavaScript动画库，它拥有颜色动画，转换动画（transforms）、循环、缓动、SVG动画、和滚动动画等特色功能，支持Chaining链式动画，当一个元素连续应用多个velocity()时，动画以队列的方式执行

  ```js
  <!-- 下载 -->
  npm install velocity-animate@beta
  <!-- 引入 -->
  import  Velocity from 'velocity-animate'
  <!-- 使用 -->
  methods:{
    beforeEnter(el){
      el.style.apacity= 0 //透明度
      el.style.transformOrigin='left'//设置旋转元素的基点位置
      el.style.color='red'//颜色
    },
      enter(el,done){
        Velocity(el,{opacity:1,fontSize:'1.4em'})
        Velocity(el,{fontSize: '1em'},{complete:done})
      },
        leave(el,done){
          Velocity(el,{translateX:'15px',rotateZ:'50deg'},{duration:300})
          Velocity(el,{rotateZ:'100deg'},{loop:2},{duration: 300})
          Velocity(el,{rotateZ:'45deg',translateY:'30px',translateX: '30px',opacity: 0 },{complete: done})
        }
  }
  ```

  ​

##### （3）自定义过渡类名

- 可以通过以下 attribute 来自定义过渡类名：
  - `enter-class`
  - `enter-active-class`
  - `enter-to-class` (2.1.8+)
  - `leave-class`
  - `leave-active-class`
  - `leave-to-class` (2.1.8+)


- 他们的优先级高于普通的类名，这对于 Vue 的过渡系统和其他第三方 CSS 动画库，如 Animate.css结合使用十分有用

  ```html
  <link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

  <div id="example-3">
    <button @click="show = !show">
      Toggle render
    </button>
    <transition
      name="custom-classes-transition"
      enter-active-class="animated tada"
      leave-active-class="animated bounceOutRight"
    >
      <p v-if="show">hello</p>
    </transition>
  </div>
  ```

##### （4）同时使用过渡和动画

- Vue 为了知道过渡的完成，必须设置相应的事件监听器。它可以是 `transitionend` 或 `animationend`，这取决于给元素应用的 CSS 规则。如果你使用其中任何一种，Vue 能自动识别类型并设置监听
- 但是，在一些场景中，你需要给同一个元素同时设置两种过渡动效，比如 `animation` 很快的被触发并完成了，而 `transition` 效果还没结束。在这种情况中，你就需要使用 `type` attribute 并设置 `animation` 或 `transition` 来明确声明你需要 Vue 监听的类型。

##### （5）显性地定义过渡持续时间

- 在很多情况下，Vue 可以自动得出过渡效果的完成时机。默认情况下，Vue 会等待其在过渡效果的根元素的第一个 `transitionend` 或 `animationend` 事件。然而也可以不这样设定，比如，我们可以拥有一个精心编排的一系列过渡效果，其中一些嵌套的内部元素相比于过渡效果的根元素有延迟的或更长的过渡效果。

- 在这种情况下你可以用 `<transition>` 组件上的 `duration` prop 定制一个显性的过渡持续时间 (以毫秒计)

  ```vue
  <transition :duration="1000">...</transition>
  ```


- 也可以定制进入和移出的持续时间

  ```vue
  <transition :duration="{ enter: 500, leave: 800 }">...</transition>
  ```

##### （5）JavaScript钩子

- 可以在 attribute 中声明 JavaScript 钩子

  ```vue
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:after-enter="afterEnter"
    v-on:enter-cancelled="enterCancelled"

    v-on:before-leave="beforeLeave"
    v-on:leave="leave"
    v-on:after-leave="afterLeave"
    v-on:leave-cancelled="leaveCancelled"
  >
    <!-- ... -->
  </transition>
  ```


  // ...
  methods: {
    // --------
    // 进入中
    // --------
    
    beforeEnter: function (el) {
      // ...
    },
    // 当与 CSS 结合使用时
    // 回调函数 done 是可选的
    enter: function (el, done) {
      // ...
      done()
    },
    afterEnter: function (el) {
      // ...
    },
    enterCancelled: function (el) {
      // ...
    },
    
    // --------
    // 离开时
    // --------
    
    beforeLeave: function (el) {
      // ...
    },
    // 当与 CSS 结合使用时
    // 回调函数 done 是可选的
    leave: function (el, done) {
      // ...
      done()
    },
    afterLeave: function (el) {
      // ...
    },
    // leaveCancelled 只用于 v-show 中
    leaveCancelled: function (el) {
      // ...
    }
  }
  ```


- 这些钩子函数可以结合 CSS `transitions/animations` 使用，也可以单独使用
- 当只用 JavaScript 过渡的时候，**在 enter 和 leave 中必须使用 done 进行回调**。否则，它们将被同步调用，过渡会立即完成
- 推荐对于仅使用 JavaScript 过渡的元素添加 `v-bind:css="false"`，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响

##### （6）初始渲染的过渡

- 可以通过 `appear` attribute 设置节点在初始渲染的过渡

  ```vue
  <transition appear>
    <!-- ... -->
  </transition>
  ```


- 这里默认和进入/离开过渡一样，同样也可以自定义 CSS 类名

  ```vue
  <transition
    appear
    appear-class="custom-appear-class"
    appear-to-class="custom-appear-to-class" (2.1.8+)
    appear-active-class="custom-appear-active-class"
  >
    <!-- ... -->
  </transition>
  ```


- 也可以自定义钩子函数

  ```vue
  <transition
    appear
    v-on:before-appear="customBeforeAppearHook"
    v-on:appear="customAppearHook"
    v-on:after-appear="customAfterAppearHook"
    v-on:appear-cancelled="customAppearCancelledHook"
  >
    <!-- ... -->
  </transition>
  ```

#### 2.多个元素的过渡

##### （1）v-if

- 对于原生标签可以使用 `v-if`/`v-else`

  ```vue
  <transition>
    <table v-if="items.length > 0">
      <!-- ... -->
    </table>
    <p v-else>Sorry, no items found.</p>
  </transition>
  ```


- 使用多个 `v-if` 的多个元素的过渡可以重写为绑定了动态 property 的单个元素过渡

  ```vue
  <transition>
    <button v-if="docState === 'saved'" key="saved">
      Edit
    </button>
    <button v-if="docState === 'edited'" key="edited">
      Save
    </button>
    <button v-if="docState === 'editing'" key="editing">
      Cancel
    </button>
  </transition>

  <!-- 可以重写为 -->
  <transition>
    <button v-bind:key="docState">
      {{ buttonMessage }}
    </button>
  </transition>

  // ... 利用计算属性
  computed: {
    buttonMessage: function () {
      switch (this.docState) {
        case 'saved': return 'Edit'
        case 'edited': return 'Save'
        case 'editing': return 'Cancel'
      }
    }
  }
  ```

##### （2）过渡模式

- `<transition>` 有一个默认行为 - 进入和离开同时发生，而同时生效的进入和离开的过渡不能满足所有要求，所以 Vue 提供了**过渡模式**
  - `in-out`：新元素先进行过渡，完成之后当前元素过渡离开。
  - `out-in`：当前元素先进行过渡，完成之后新元素过渡进入

#### 3.多个组件的过渡

- 多个组件的过渡只需要使用动态组件

  ```vue
  <transition name="component-fade" mode="out-in">
    <component v-bind:is="view"></component>
  </transition>

  new Vue({
    el: '#transition-components-demo',
    data: {
      view: 'v-a'
    },
    components: {
      'v-a': {
        template: '<div>Component A</div>'
      },
      'v-b': {
        template: '<div>Component B</div>'
      }
    }
  })
  ```

#### 4.列表过渡

- 怎么同时渲染整个列表，比如使用 `v-for`？在这种场景中，使用 `<transition-group>` 组件
  - 不同于 `<transition>`，它会以一个真实元素呈现：默认为一个 `<span>`。你也可以通过 `tag` attribute 更换为其他元素。
  - 过渡模式不可用，因为我们不再相互切换特有的元素。
  - 内部元素**总是需要**提供唯一的 `key` attribute 值。
  - CSS 过渡的类将会应用在内部的元素中，而不是这个组/容器本身

##### （1）列表的进入/离开过渡

- 进入和离开的过渡使用之前一样的css类名

  ```vue
  <div id="list-demo" class="demo">
    <button v-on:click="add">Add</button>
    <button v-on:click="remove">Remove</button>
    <transition-group name="list" tag="p">
      <span v-for="item in items" v-bind:key="item" class="list-item">
        {{ item }}
      </span>
    </transition-group>
  </div>
  ```

  ```css
  .list-item {
    display: inline-block;
    margin-right: 10px;
  }
  .list-enter-active, .list-leave-active {
    transition: all 1s;
  }
  .list-enter, .list-leave-to
  /* .list-leave-active for below version 2.1.8 */ {
    opacity: 0;
    transform: translateY(30px);
  }
  ```


- 这个例子有个问题，当添加和移除元素的时候，周围的元素会瞬间移动到他们的新布局的位置，而不是平滑的过渡过去

##### （2）列表的排序过渡

- `<transition-group>` 组件还有一个特殊之处。不仅可以进入和离开动画，还可以改变定位。要使用这个新功能只需了解新增的 **v-move class**，它会在元素的改变定位的过程中应用。像之前的类名一样，可以通过 `name` attribute 来自定义前缀，也可以通过 `move-class` attribute 手动设置

  ```vue
  <div id="flip-list-demo" class="demo">
    <button v-on:click="shuffle">Shuffle</button>
    <transition-group name="flip-list" tag="ul">
      <li v-for="item in items" v-bind:key="item">
        {{ item }}
      </li>
    </transition-group>
  </div>
  ```

  ```css
  .flip-list-move {
    transition: transform 1s;
  }
  ```


- 补充：推荐一个洗牌的工具 lodash

  ```js
  <!-- 下载 -->
  npm i lodash
  <!-- 引入 -->
  import _ from 'lodash
  <!-- 使用 -->
  this.items = _.shuffle(this.items)
  ```

#### 5.可复用的过渡

- 过渡可以通过Vue的组件系统实现复用。要创建一个可复用过渡组件，你需要做的就是将`<transition>` 或者 `<transition-group>` 作为根组件，然后将任何子组件放置在其中就可以了。

  ```vue
  Vue.component('my-special-transition', {
    template: '\
      <transition\
        name="very-special-transition"\
        mode="out-in"\
        v-on:before-enter="beforeEnter"\
        v-on:after-enter="afterEnter"\
      >\
        <slot></slot>\
      </transition>\
    ',
    methods: {
      beforeEnter: function (el) {
        // ...
      },
      afterEnter: function (el) {
        // ...
      }
    }
  })
  ```


- 推荐使用函数式组件声明

  ```vue
  Vue.component('my-special-transition', {
    functional: true,
    render: function (createElement, context) {
      var data = {
        props: {
          name: 'very-special-transition',
          mode: 'out-in'
        },
        on: {
          beforeEnter: function (el) {
            // ...
          },
          afterEnter: function (el) {
            // ...
          }
        }
      }
      return createElement('transition', data, context.children)
    }
  })
  ```

#### 6.动态过渡

- 在 Vue 中即使是过渡也是数据驱动的，动态过渡最基本就是通过`name` attribute 来绑定动态值

  ```vue
  <transition v-bind:name="transitionName">
    <!-- ... -->
  </transition>
  ```


- 当你想用 Vue 的过渡系统来定义的 CSS 过渡/动画在不同过渡间切换会非常有用,所有过渡 attribute 都可以动态绑定，但我们不仅仅只有 attribute 可以利用，还可以通过事件钩子获取上下文中的所有数据，因为事件钩子都是方法。这意味着，根据组件的状态不同，你的 JavaScript 过渡会有不同的表现
- 最后，创建动态过渡的最终方案是组件通过接受 props 来动态修改之前的过渡

### 六、混入

- 混入（Mixin）提供了一种非常灵活的方式，来分发Vue组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项


- 使用`Vue.extends`或者`mixins: [mixin]`

  ```js
  // 定义一个混入对象
  var myMixin = {
    created: function () {
      this.hello()
    },
    methods: {
      hello: function () {
        console.log('hello from mixin!')
      }
    }
  }

  // 定义一个使用混入对象的组件
  var Component = Vue.extend({
    mixins: [myMixin]
  })

  var component = new Component() // => "hello from mixin!"
  ```

  ​

#### 1.选项合并

- 当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。比如，数据对象在内部会进行递归合并，并在发生冲突时以组件数据优先。

  ```js
  var mixin = {
    data: function () {
      return {
        message: 'hello',
        foo: 'abc'
      }
    }
  }

  new Vue({
    mixins: [mixin],
    data: function () {
      return {
        message: 'goodbye',
        bar: 'def'
      }
    },
    created: function () {
      console.log(this.$data)
      // => { message: "goodbye", foo: "abc", bar: "def" }
    }
  })
  ```


- 同名钩子函数将合并为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子之前调用

  ```js
  var mixin = {
    created: function () {
      console.log('混入对象的钩子被调用')
    }
  }

  new Vue({
    mixins: [mixin],
    created: function () {
      console.log('组件钩子被调用')
    }
  })

  // => "混入对象的钩子被调用"
  // => "组件钩子被调用"
  ```


- 值为对象的选项，比如 `methods`、`components` 和 `directives`，都将被额并未同一个对象。两个对象键名冲突时，取组件对象的键值对（后面覆盖前面）

  ```js
  var mixin = {
    methods: {
      foo: function () {
        console.log('foo')
      },
      conflicting: function () {
        console.log('from mixin')
      }
    }
  }

  var vm = new Vue({
    mixins: [mixin],
    methods: {
      bar: function () {
        console.log('bar')
      },
      conflicting: function () {
        console.log('from self')
      }
    }
  })

  vm.foo() // => "foo"
  vm.bar() // => "bar"
  vm.conflicting() // => "from self"
  ```


- 注意：`Vue.extend()` 也使用同样的策略进行合并。

#### 2.全局混入

- 混入也可以进行全局注册。**使用时应注意：一旦使用全局混入，它将影响每一个之后创建的Vue实例。**

- 使用`Vue.mixin`进行全局混入

  ```js
  // 为自定义的选项 'myOption' 注入一个处理器。
  Vue.mixin({
    created: function () {
      var myOption = this.$options.myOption
      if (myOption) {
        console.log(myOption)
      }
    }
  })

  new Vue({
    myOption: 'hello!'
  })
  // => "hello!"
  ```

#### 3.自定义选项合并策略

- 自定义选项将使用默认策略，即简单地覆盖以优质。如果想让自定义选项以自定义逻辑合并，可以向`Vue.config.optionMergeStrategies` 添加一个函数：

  ```js
  <!-- 自定义myOption选项制定合并策略 -->
  Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
    // 返回合并后的值
  }
  ```


- 对于多数值为对象的选项，可以使用与`methods` 相同的合并策略：

  ```js
  var strategies = Vue.config.optionMergeStrategies
  strategies.myOption = strategies.methods
  ```

### 七、自定义指令

- Vue允许自定义指令，在Vue2.0中，代码复用和抽象的主要形式是组件.然而，有的情况下，仍然需要对普通DOM元素进行底层操作，这时就需要使用自定义指令

#### 1.全局注册指令

- 在`main.js`中使用`Vue.directive`命令注册

  ```js
  // 注册一个全局自定义指令 `v-focus`
  Vue.directive('focus', {
    // 当被绑定的元素插入到 DOM 中时……
    inserted: function (el) {
      // 聚焦元素
      el.focus()
    }
  })
  ```

#### 2.局部注册

- 在当前组件内部使用directives属性注册

  ```js
  directives: {
    focus: {
      // 指令的定义
      inserted: function (el) {
        el.focus()
      }
    }
  }
  ```


- 自定义指令的使用对于全局注册还是局部注册效果一致

  ```vue
  <input v-focus>
  ```

#### 3.钩子函数

- 一个指令定义对象可以提供如下几个钩子函数（均为可选）

  - `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
  - `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
  - `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新


  - `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
  - `unbind`：只调用一次，指令与元素解绑时调用。

##### （1）函数简写

- 在很多时候，可能想在不同的钩子函数上触发相同行为，比如在 `bind` 和 `update` 时触发相同行为，而不关心其他的钩子，这是就可以进行函数简写

  ```js
  Vue.directive('color-swatch', function (el, binding) {
    el.style.backgroundColor = binding.value
  })
  ```

##### （2）对象字面量

- 如果指令需要多个值，可以和窜入一个`JavaScript`对象字面量，记住，指令函数能够接受所有合法的`JavaScript`表达式

  ```js
  <div v-demo="{ color: 'white', text: 'hello!' }"></div>

  Vue.directive('demo', function (el, binding) {
    console.log(binding.value.color) // => "white"
    console.log(binding.value.text)  // => "hello!"
  })
  ```

#### 4.钩子函数参数

- 钩子函数会被传入以下几个参数
  - `el`：指令所绑定的元素，可以用来直接操作 DOM。
  - `binding`：一个对象，包含以下 property：
    - `name`：指令名，不包括 `v-` 前缀。
    - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
    - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
    - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
    - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
    - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。
  - `vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://v2.cn.vuejs.org/v2/api/#VNode-%E6%8E%A5%E5%8F%A3) 来了解更多详情。
  - `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。


- **除了 `el` 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的`dataset`来进行**

#### 5.动态指令参数

- 指令的参数可以是动态的，例：`v-mydirective:[argument]="value"` ，这使得自定义指令可以在应用中被灵活使用

- 例如：编写一个自定义指令将元素固定定位在浏览器的一个方向上，将方向作为动态参数，绑定的动态参数可以根据`binding.arg`获取

  ```js
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>

  Vue.directive('pin', {
    bind: function (el, binding, vnode) {
      el.style.position = 'fixed'
      var s = (binding.arg == 'left' ? 'left' : 'top')
      el.style[s] = binding.value + 'px'
    }
  })

  new Vue({
    el: '#dynamicexample',
    data: function () {
      return {
        direction: 'left'
      }
    }
  })
  ```

### 八、渲染函数&JSX

- Vue推荐在绝大多数情况下使用模板来创建你的HTML，然而在一些场景中，可能会需要JavaScript的完全编程的能力，这是可以使用渲染函数，他比模板更接近编译器

- 比如，当开始写一个只能通过 `level` prop 动态生成标题 (heading) 的组件时，你可能很快想到这样实现：

  ```js
  <script type="text/x-template" id="anchored-heading-template">
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-else-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-else-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-else-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-else-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-else-if="level === 6">
      <slot></slot>
    </h6>
  </script>
  ```


- 这里用模板并不是最好的选择：不但代码冗长，而且在每一个级别的标题中重复书写了 `<slot></slot>`，在要插入锚点元素时还要再次重复。

- 来尝试使用 `render` 函数重写上面的例子：

  ```js
  Vue.component('anchored-heading', {
    render: function (createElement) {
      return createElement(
        'h' + this.level,   // 标签名称
        this.$slots.default // 子节点数组
      )
    },
    props: {
      level: {
        type: Number,
        required: true
      }
    }
  })
  ```


- 补充：向组件中传递不带`v-slot` 指令的子节点时，比如 `anchored-heading` 中的 `Hello world!`，这些子节点被存储在组件实例中的 `$slots.default`

#### 1.节点、树以及虚拟DOM

- 在深入渲染函数之前，了解浏览器的工作原理非常重要
- 当浏览器读取到一段HTML代码时，它会建立一个DOM节点树来保持追踪所有内容。每个元素都是一个节点，每段文字也是一个节点，甚至注释也是节点。一个节点就是页面的一部分，就像家谱树一样，每个节点都可以有孩子节点和父亲节点
- 在Vue中，只需要告诉Vue你希望在页面上的HTML是什么，可以是在一个模板中，也可以是在渲染函数中，在两种情况下，Vue都会自动保持页面的更新

##### （1）虚拟DOM

- Vue通过建立一个虚拟DOM来追踪自己要如何改变真实DOM

  ```js
  return createElement('h1', this.blogTitle)
  ```


- `createElement` 到底会返回什么呢？其实不是一个*实际的* DOM 元素。它更准确的名字可能是 `createNodeDescription`，因为它所包含的信息会告诉 Vue 页面上需要渲染什么样的节点，包括及其子节点的描述信息。我们把这样的节点描述为“虚拟节点 (virtual node)”，也常简写它为“**VNode**”。“虚拟 DOM”是我们对由 Vue 组件树建立起来的整个 VNode 树的称呼。
- **注意：VNode必须唯一，组件树中的所有 VNode 必须是唯一的**

##### （2）`createElement`参数

- 如何在 `createElement` 函数中使用模板中的那些功能？这里是 `createElement` 接受的参数：

  ```js
  // @returns {VNode}
  createElement(
    // {String | Object | Function}
    // 一个 HTML 标签名、组件选项对象，或者
    // resolve 了上述任何一种的一个 async 函数。必填项。
    'div',

    // {Object}
    // 一个与模板中 attribute 对应的数据对象。可选。
    {
      // (详情见下一节)
    },

    // {String | Array}
    // 子级虚拟节点 (VNodes)，由 `createElement()` 构建而成，
    // 也可以使用字符串来生成“文本虚拟节点”。可选。
    [
      '先写一些文字',
      createElement('h1', '一则头条'),
      createElement(MyComponent, {
        props: {
          someProp: 'foobar'
        }
      })
    ]
  )
  ```

##### （3）深入数据对象

- 有一点要注意：正如 `v-bind:class` 和 `v-bind:style` 在模板语法中会被特别对待一样，它们在 VNode 数据对象中也有对应的顶层字段。该对象也允许你绑定普通的 HTML attribute，也允许绑定如 `innerHTML` 这样的 DOM property (这会覆盖 `v-html` 指令)

  ```js
  {
    // 与 `v-bind:class` 的 API 相同，
    // 接受一个字符串、对象或字符串和对象组成的数组
    'class': {
      foo: true,
      bar: false
    },
    // 与 `v-bind:style` 的 API 相同，
    // 接受一个字符串、对象，或对象组成的数组
    style: {
      color: 'red',
      fontSize: '14px'
    },
    // 普通的 HTML attribute
    attrs: {
      id: 'foo'
    },
    // 组件 prop
    props: {
      myProp: 'bar'
    },
    // DOM property
    domProps: {
      innerHTML: 'baz'
    },
    // 事件监听器在 `on` 内，
    // 但不再支持如 `v-on:keyup.enter` 这样的修饰器。
    // 需要在处理函数中手动检查 keyCode。
    on: {
      click: this.clickHandler
    },
    // 仅用于组件，用于监听原生事件，而不是组件内部使用
    // `vm.$emit` 触发的事件。
    nativeOn: {
      click: this.nativeClickHandler
    },
    // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`
    // 赋值，因为 Vue 已经自动为你进行了同步。
    directives: [
      {
        name: 'my-custom-directive',
        value: '2',
        expression: '1 + 1',
        arg: 'foo',
        modifiers: {
          bar: true
        }
      }
    ],
    // 作用域插槽的格式为
    // { name: props => VNode | Array<VNode> }
    scopedSlots: {
      default: props => createElement('span', props.text)
    },
    // 如果组件是其它组件的子组件，需为插槽指定名称
    slot: 'name-of-slot',
    // 其它特殊顶层 property
    key: 'myKey',
    ref: 'myRef',
    // 如果你在渲染函数中给多个元素都应用了相同的 ref 名，
    // 那么 `$refs.myRef` 会变成一个数组。
    refInFor: true
  }
  ```

#### 2.使用JavaScript代替模板功能

##### （1）v-if 和 v-for

```vue
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```

- 可以在渲染函数中用 JavaScript 的 `if`/`else` 和 `map` 来重写

```js
Vue.component('MyUl', {
  data() {
    return {
      items: [
        { name: 'a', id: 1 },
        { name: 'b', id: 2 },
        { name: 'c', id: 3 },
        { name: 'd', id: 4 }
      ]
    }
  },

  render: function (createElement) {
    console.log(this)
    if (this.items.length) {
      return createElement(
        'ul',
        this.items.map(function (item) {
          return createElement('li', item.name)
        })
      )
    } else {
      return createElement('p', 'No items found.')
    }
  }
})
```

##### （2）v-model

- 渲染函数中没有与 `v-model` 的直接对应——你必须自己实现相应的逻辑

  ```js
  Vue.component('MyModel', {
    data() {
      return {
        value: ''
      }
    },
    render: function (createElement) {
      const self = this
      return createElement('div', [
        createElement('input', {
          domProps: {
            value: self.value
          },
          on: {
            input: function (event) {
              // self.$emit('input', event.target.value)
              self.value = event.target.value
            }
          }
        }),
        createElement('span', {
          domProps: {
            innerHTML: self.value
          }
        })
      ])
    }
  })
  ```

##### （3）事件&按键修饰符

- 对于 `.passive`、`.capture` 和 `.once` 这些事件修饰符，Vue 提供了相应的前缀可以用于 `on`：

  | 修饰符                              | 前缀   |
  | -------------------------------- | ---- |
  | `.passive`                       | `&`  |
  | `.capture`                       | `!`  |
  | `.once`                          | `~`  |
  | `.capture.once` 或`.once.capture` | `~!` |

  ```js
  on: {
    '!click': this.doThisInCapturingMode,
    '~keyup': this.doThisOnce,
    '~!mouseover': this.doThisOnceInCapturingMode
  }
  ```


- 对于所有其它的修饰符，私有前缀都不是必须的，因为你可以在事件处理函数中使用事件方法

  | 修饰符                                    | 处理函数中的等价操作                               |
  | -------------------------------------- | ---------------------------------------- |
  | `.stop`                                | `event.stopPropagation()`                |
  | `.prevent`                             | `event.preventDefault()`                 |
  | `.self`                                | `if (event.target !== event.currentTarget) return` |
  | 按键：`.enter`, `.13`                     | `if (event.keyCode !== 13) return` (对于别的按键修饰符来说，可将 `13` 改为另一个按键码 |
  | 修饰键：`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (将 `ctrlKey` 分别修改为 `altKey`、`shiftKey` 或者 `metaKey`) |


- 例如，下面使用所有修饰符

  ```js
  on: {
    keyup: function (event) {
      // 如果触发事件的元素不是事件绑定的元素
      // 则返回
      if (event.target !== event.currentTarget) return
      // 如果按下去的不是 enter 键或者
      // 没有同时按下 shift 键
      // 则返回
      if (!event.shiftKey || event.keyCode !== 13) return
      // 阻止 事件冒泡
      event.stopPropagation()
      // 阻止该元素默认的 keyup 事件
      event.preventDefault()
      // ...
    }
  }
  ```

##### （4）插槽

- 可以通过`this.$slots`访问静态插槽的内容，每个插槽都是一个 VNode 数组

  ```vue
  Vue.component('MySlot', {
    render: function (createElement) {
  	// 相当于结构为`<div><slot></slot></div>`
      return createElement('div', this.$slots.default)
    }
  })

  <MySlot>
    <template #default>
      <h3>hello world</h3>
  	1223
    </template>
  </MySlot>
  ```

- 通过 `this.$scopedSlots`访问作用域插槽，每个作用域插槽都是一个返回若干 VNode 的函数

  ```vue
  Vue.component('MyScopeSlot', {
    data() {
      return {
        message: 'hello world'
      }
    },
    render: function (createElement) {
      // 相当于结构为`<div><slot :text="message"></slot></div>`
      return createElement('div', [
        this.$scopedSlots.default({
          text: this.message
        })
      ])
    }
  })
  ```


```vue
<MyScopeSlot>
  <template #default="data">
	<h3>meng</h3>
	<div>{{ data }}</div>
  </template>
</MyScopeSlot>
```
  ```


- 如果要用渲染函数向子组件中传递作用域插槽，可以利用 VNode 数据对象中的 `scopedSlots` 字段，他不同于`$scopedSlots`，它是存在于render数据对象中的一个对象

  ```vue
  Vue.component('MyChildScopeSlot', {
    render(_c) {
      // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
      return _c('div', [
        _c('childSlot', {
          scopedSlots: {
            default(prop) {
              // 对应 v-slot:default="prop"
              return _c('span', prop)
            },
            content(prop) {
              // 对应 v-slot:default="prop"
              return prop
            },
            other(prop) {
              // 对应 v-slot:default="prop"
              return prop
            }
          }
        })
      ])
    }
  })
  Vue.component('childSlot', {
    // eslint-disable-next-line space-before-function-paren
    render(_c) {
      let def = this.$slots.default
      def || (def = '我是默认插槽的后备内容')

      let con = this.$slots.content
      con || (con = '我是具名插槽 content 的后备内容')

      let oth = this.$slots.other
      oth || (oth = '我是具名插槽 other 的后备内容')
      return _c('div', ['我是子组件!', _c('br'), def, _c('br'), con, _c('br'), oth])
    }
  })

  <MyChildScopeSlot></MyChildScopeSlot>
  ```

#### 3.JSX

- 在模板简单的情况下，但对应的render函数却很麻烦

```js
createElement(
  'anchored-heading', {
    props: {
      level: 1
    }
  }, [
    createElement('span', 'Hello'),
    ' world!'
  ]
)


<anchored-heading :level="1">
  <span>Hello</span> world!
</anchored-heading>
```

- 为了解决这个问题，出现了一个`Babel`插件，用于在Vue中使用`JSX`语法，它可以让我们更接近于模板的语法上

```vue
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render: function (h) {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```

- 补充：将 `h` 作为 `createElement` 的别名是 Vue 生态系统中的一个通用惯例，实际上也是 JSX 所要求的。从 Vue 的 Babel 插件的3.4.0版本开始，我们会在以 ES2015 语法声明的含有 JSX 的任何方法和 getter 中 (不是函数或箭头函数中) 自动注入 `const h = this.$createElement`，这样你就可以去掉 `(h)` 参数了。对于更早版本的插件，如果 `h` 在当前作用域中不可用，应用会抛错

#### 4.Babel预设JSX

##### （1）基础

- 内容

  ```js
  render() {
    return <p>hello</p>
  }
  ```


- 动态内容

  ```js
  render() {
    return <p>hello { this.message }</p>
  }
  ```


- 自闭时

  ```js
  render() {
    return <input />
  }
  ```


- 有一个组件

  ```vue
  import MyComponent from './my-component'

  export default {
    render() {
      return <MyComponent>hello</MyComponent>
    },
  }
  ```

##### （2）属性/道具

- 属性

  ```js
  render() {
    return <input type="email" />
  }
  ```


- 动态绑定属性

  ```vue
  render() {
    return <input
      type="email"
      placeholder={this.placeholderText}
    />
  }
  ```


- 使用扩展运算符

  ```vue
  render() {
    const inputAttrs = {
      type: 'email',
      placeholder: 'Enter your email'
    }

    return <input {...{ attrs: inputAttrs }} />
  }
  ```

##### （3）插槽

- 命名槽

  ```vue
  render() {
    return (
      <MyComponent>
        <header slot="header">header</header>
        <footer slot="footer">footer</footer>
      </MyComponent>
    )
  }
  ```


- 作用域插槽

  ```vue
  render() {
    const scopedSlots = {
      header: () => <header>header</header>,
      footer: () => <footer>footer</footer>
    }

    return <MyComponent scopedSlots={scopedSlots} />
  }
  ```

##### （4）指令

- 指令

  ```vue
  <input vModel={this.newTodoText} />
  ```


- 带有修饰符的指令

  ```vue
  <input vModel_trim={this.newTodoText} />
  ```


- 事件指令

  ```vue
  <input vOn:click={this.newTodoText} />
  ```


- 带有参数和修饰符

  ```vue
  <input vOn:click_stop_prevent={this.newTodoText} />
  ```


- v-html

  ```vue
  <p domPropsInnerHTML={html} />
  ```

##### （5）功能组件

- 当它们是默认导出时，将返回 JSX 的箭头函数转换为功能组件

  ```vue
  export default ({ props }) => <p>hello {props.message}</p>
  ```


- 或 PascalCase 变量声明

  ```vue
  const HelloWorld = ({ props }) => <p>hello {props.message}</p>
  ```

#### 5.函数式组件

- 之前创建的锚点标题组件是比较简单，没有管理任何状态，也没有监听任何传递给它的状态，也没有生命周期方法。实际上，它只是一个接受一些 prop 的函数。在这样的场景下，我们可以将组件标记为 `functional`，这意味它无状态 (没有**响应式数据**)，也没有实例 (没有 `this` 上下文)

  ```js
  Vue.component('my-component', {
    functional: true,
    // Props 是可选的
    props: {
      // ...
    },
    // 为了弥补缺少的实例
    // 提供第二个参数作为上下文
    render: function (createElement, context) {
      // ...
    }
  })
  ```


- 注意：在 2.3.0 之前的版本中，如果一个函数式组件想要接收 prop，则 `props` 选项是必须的。在 2.3.0 或以上的版本中，你可以省略 `props` 选项，所有组件上的 attribute 都会被自动隐式解析为 prop。当使用函数式组件时，该引用将会是 HTMLElement，因为他们是无状态的也是无实例的

- 在 2.5.0 及以上版本中，如果你使用了单文件组件，那么基于模板的函数式组件可以这样声明

  ```vue
  <template functional>
  </template>
  ```

- 组件需要的一切都是通过 `context` 参数传递，它是一个包括如下字段的对象

  - `props`：提供所有 prop 的对象
  - `children`：VNode 子节点的数组
  - `slots`：一个函数，返回了包含所有插槽的对象
  - `scopedSlots`：(2.6.0+) 一个暴露传入的作用域插槽的对象。也以函数形式暴露普通插槽。
  - `data`：传递给组件的整个数据对象，作为 `createElement` 的第二个参数传入组件
  - `parent`：对父组件的引用
  - `listeners`：(2.3.0+) 一个包含了所有父组件为当前组件注册的事件监听器的对象。这是 `data.on` 的一个别名。
  - `injections`：(2.3.0+) 如果使用了 `inject`选项，则该对象包含了应当被注入的 property。


- 在添加 `functional: true` 之后，需要更新我们的锚点标题组件的渲染函数，为其增加 `context` 参数，并将 `this.$slots.default` 更新为 `context.children`，然后将 `this.level` 更新为 `context.props.level`

### 九、插件

- 插件通常用来为 Vue 添加全局功能。插件的功能范围没有严格的限制——一般有下面几种：
  1. 添加全局方法或者 property。如`vue-custom-element`
  2. 添加全局资源：指令/过滤器/过渡等。如`vue-touch`
  3. 通过全局混入来添加一些组件选项。如`vue-router`
  4. 添加 Vue 实例方法，通过把它们添加到 `Vue.prototype` 上实现。
  5. 一个库，提供自己的 API，同时提供上面提到的一个或多个功能。如`vue-router`

#### 1.使用插件

- 通过全局方法`Vue.use()`使用插件，它需要在你调用`new Vue()`启动应用之前完成

  ```js
  // 调用 `MyPlugin.install(Vue)`
  Vue.use(MyPlugin)

  new Vue({
    // ...组件选项
  })
  ```


- 也可以传入一个可选的参数

  ```js
  Vue.use(MyPlugin, { someOption: true })
  ```


- `Vue.use` 会自动阻止多次注册相同插件，届时即使多次调用也只会注册一次该插件
- Vue.js 官方提供的一些插件 (例如 `vue-router`) 在检测到 `Vue` 是可访问的全局变量时会自动调用 `Vue.use()`。然而在像 CommonJS 这样的模块环境中，你应该始终显式地调用 `Vue.use()`

#### 2.开发插件

- Vue.js 的插件应该暴露一个 `install` 方法。这个方法的第一个参数是 `Vue` 构造器，第二个参数是一个可选的选项对象

  ```js
  MyPlugin.install = function (Vue, options) {
    // 1. 添加全局方法或 property
    Vue.myGlobalMethod = function () {
      // 逻辑...
    }

    // 2. 添加全局资源
    Vue.directive('my-directive', {
      bind (el, binding, vnode, oldVnode) {
        // 逻辑...
      }
      ...
    })

    // 3. 注入组件选项
    Vue.mixin({
      created: function () {
        // 逻辑...
      }
      ...
    })

    // 4. 添加实例方法
    Vue.prototype.$myMethod = function (methodOptions) {
      // 逻辑...
    }
  }
  ```

### 十、过滤器

- Vue.js 允许你自定义过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：**双花括号插值和 v-bind 表达式** (后者从 2.1.0+ 开始支持)。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示

  ```html
  <!-- 在双花括号中 -->
  {{ message | capitalize }}

  <!-- 在 `v-bind` 中 -->
  <div v-bind:id="rawId | formatId"></div>
  ```


- 可以在组件的选项中定义本地的过滤器

  ```js
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
  ```


- 或者注册全局定义过滤器（在创建Vue实例之前定义）

  ```js
  Vue.filter('capitalize', function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  })
  ```


- 注意：当全局过滤器和局部过滤器重名时，会采用局部过滤器

- 过滤器函数总接收表达式的值 (之前的操作链的结果) 作为第一个参数

- 过滤器可以串联

  ```html
  {{ message | filterA | filterB }}
  ```


- 过滤器是 JavaScript 函数，因此可以接收参数

  ```html
  {{ message | filterA('arg1', arg2) }}
  ```

### 十一、单文件组件

- 在很多 Vue 项目中，我们使用 `Vue.component` 来定义全局组件，紧接着用 `new Vue({ el: '#container '})` 在每个页面内指定一个容器元素。


- 这种方式在很多中小规模的项目中运作的很好，在这些项目里 JavaScript 只被用来加强特定的视图。但当在更复杂的项目中，或者你的前端完全由 JavaScript 驱动的时候，下面这些缺点将变得非常明显：
  - **全局定义 (Global definitions)** 强制要求每个 component 中的命名不得重复
  - **字符串模板 (String templates)** 缺乏语法高亮，在 HTML 有多行的时候，需要用到丑陋的 `\`
  - **不支持 CSS (No CSS support)** 意味着当 HTML 和 JavaScript 组件化时，CSS 明显被遗漏
  - **没有构建步骤 (No build step)** 限制只能使用 HTML 和 ES5 JavaScript，而不能使用预处理器，如 Pug (formerly Jade) 和 Babel


- 文件扩展名为 `.vue` 的 **single-file components (单文件组件)** 为以上所有问题提供了解决方法，并且还可以使用 webpack 或 Browserify 等构建工具。

```vue
<template>
  <div></div>
</template>

<script>
export default {}
</script>

<style></style>
```

### 十二、API

#### 1.全局配置

- `Vue.config` 是一个对象，包含 Vue 的全局配置。可以在启动应用之前修改下列 property

##### （1）silent

- **类型**：`boolean`

- **默认值**：`false`

- **用法**：

  ```js
  Vue.config.silent = true
  ```


- **作用**：取消Vue所有的日至与警告

##### （2）optionMergeStrategies

- **类型**：`{ [key: string]: Function }`

- **默认值**：`{}`

- **用法**：

  ```js
  Vue.config.optionMergeStrategies._my_option = function (parent, child, vm) {
    return child + 1
  }

  const Profile = Vue.extend({
    _my_option: 1
  })

  // Profile.options._my_option = 2
  ```


- **作用**：自定义合并策略的选项，合并策略选项分别接受在父实例和子实例上定义的该选项的值作为第一个参数和第二个参数，Vue实例上下文被作为第三个参数传入

##### （3）devtools

- **类型**：`boolean`

- **默认值**：`true` (生产版为 `false`)

- **用法**：

  ```js
  // 务必在加载 Vue 之后，立即同步设置以下内容
  Vue.config.devtools = true
  ```


- **作用**：配置是否允许`vue-devtools`查代码。开发版本默认为 `true`，生产版本默认为 `false`。生产版本设为 `true` 可以启用检查。

##### （4）errorHandler

- **类型**：`Function`

- **默认值**：`undefined`

- **用法**：

  ```js
  Vue.config.errorHandler = function (err, vm, info) {
    // handle error
    // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
    // 只在 2.2.0+ 可用
  }
  ```


- **作用**：指定组件的渲染和观察期间未捕获错误的处理函数。这个处理函数被调用时，可获取错误信息和 Vue 实例

##### （5）warnHandler

- **类型**：`Function`

- **默认值**：`undefined`

- **用法**：

  ```js
  Vue.config.warnHandler = function (msg, vm, trace) {
    // `trace` 是组件的继承关系追踪
  }
  ```


- **作用**：为 Vue 的运行时警告赋予一个自定义处理函数。注意这只会在开发者环境下生效，在生产环境下它会被忽略。

##### （6）ignoreElements

- **类型**：`Array<string | RegExp>`

- **默认值**：`[]`

- **用法**：

  ```js
  Vue.config.ignoredElements = [
    'my-custom-web-component',
    'another-web-component',
    // 用一个 `RegExp` 忽略所有“ion-”开头的元素
    // 仅在 2.5+ 支持
    /^ion-/
  ]
  ```


- **作用**：须使 Vue 忽略在 Vue 之外的自定义元素 (e.g. 使用了 Web Components APIs)。否则，它会假设你忘记注册全局组件或者拼错了组件名称，从而抛出一个关于 `Unknown custom element` 的警告

##### （7）keyCodes

- **类型**：`{ [key: string]: number | Array<number> }`

- **默认值**：`{}`

- **用法**：

  ```js
  Vue.config.keyCodes = {
    v: 86,
    f1: 112,
    // camelCase 不可用
    mediaPlayPause: 179,
    // 取而代之的是 kebab-case 且用双引号括起来
    "media-play-pause": 179,
    up: [38, 87]
  }
  ```

  ```html
  <input type="text" @keyup.media-play-pause="method">
  ```

- **作用**：给 `v-on` 自定义键位别名

