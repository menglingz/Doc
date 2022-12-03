## Vue3

### 一、API风格

- Vue的组件可以按两种不同的风格书写：**选项式API**和**组合式API**

#### 1.选项式API

- 使用选项式API，我们可以用包含多个选项的对象来描述组件的逻辑，例如 `data`、`methods` 和 `mounted`。选项所定义的属性都会暴露在函数内部的 `this` 上，它会指向当前的组件实例。

  ```vue
  <script>
  export default {
    // data() 返回的属性将会成为响应式的状态
    // 并且暴露在 `this` 上
    data() {
      return {
        count: 0
      }
    },

    // methods 是一些用来更改状态与触发更新的函数
    // 它们可以在模板中作为事件监听器绑定
    methods: {
      increment() {
        this.count++
      }
    },

    // 生命周期钩子会在组件生命周期的各个不同阶段被调用
    // 例如这个函数就会在组件挂载完成后被调用
    mounted() {
      console.log(`The initial count is ${this.count}.`)
    }
  }
  </script>

  <template>
    <button @click="increment">Count is: {{ count }}</button>
  </template>
  ```




#### 2.组合式API（Composition API）

- 通过组合式API，我们可以使用导入的API函数来描述组件的逻辑。在单文件组件中，组合式API通常会与 `setup`搭配使用。这个 `setup` attribute 是一个标识，告诉 Vue 需要在编译时进行一些处理，让我们可以更简洁地使用组合式 API。比如，`<script setup>` 中的导入和顶层变量/函数都能够在模板中直接使用。

  ```vue
  <script setup>
  import { ref, onMounted } from 'vue'

  // 响应式状态
  const count = ref(0)

  // 用来修改状态、触发更新的函数
  function increment() {
    count.value++
  }

  // 生命周期钩子
  onMounted(() => {
    console.log(`The initial count is ${count.value}.`)
  })
  </script>

  <template>
    <button @click="increment">Count is: {{ count }}</button>
  </template>
  ```




### 二、响应式基础

#### 1.声明响应式状态

##### （1）`reactive()`

- **reactive（）：**使用`reactive`函数创建一个响应式对象或数组

  ```js
  import { reactive } from 'vue'

  const state = reactive({ count: 0 })
  ```


- 在Vue3中，响应式状态其实是`JavaScript Proxy`,其行为表现与一般对象类似。不同之处在于Vue能够跟踪对响应式对象属性的访问与更改操作

- 在组件模板中使用响应式状态，需要在`setup()`函数中定义并返回

  ```js
  import { reactive } from 'vue'

  export default {
    setup() {
      const state = reactive({ count: 0 })

      function increment() {
        state.count++
      }

      // 不要忘记同时暴露 increment 函数
      return {
        state,
        increment
      }
    }
  }
  ```

  ```vue
  <button @click="increment">
    {{ state.count }}
  </button>
  ```


- **`reactive()` 的局限性** ：`reactive()` API 有两条限制

  - 仅对对象类型有效（对象、数组和 `Map`、`Set` 这样的集合类型)），而对 `string`、`number` 和 `boolean` 这样的原始类型 无效。

  - 因为 Vue 的响应式系统是通过属性访问进行追踪的，因此我们必须始终保持对该响应式对象的相同引用。这意味着我们不可以随意地“替换”一个响应式对象，因为这将导致对初始引用的响应性连接丢失：

    ```js
    let state = reactive({ count: 0 })

    // 上面的引用 ({ count: 0 }) 将不再被追踪（响应性连接已丢失！）
    state = reactive({ count: 1 })
    ```


- 这也意味着当我们将响应式对象的属性赋值或解构至本地变量时，或是将该属性传入一个函数时，我们会失去响应性
- 为了解决这个问题，引入了`ref()`

##### （2）`ref（）`

- `reactive()` 的种种限制归根结底是因为 JavaScript 没有可以作用于所有值类型的 “引用” 机制。为此，Vue 提供了一个 `ref（`）) 方法来允许我们创建可以使用任何值类型的响应式 **ref**

  ```js
  import { ref } from 'vue'

  const count = ref(0)
  ```


- `ref()` 将传入参数的值包装为一个带 `.value` 属性的 ref 对象，使用ref包装的数据时，需要使用`.value`获取

  ```js
  const count = ref(0)

  console.log(count) // { value: 0 }
  console.log(count.value) // 0

  count.value++
  console.log(count.value) // 1
  ```


- 和响应式对象的属性类似，ref 的 `.value` 属性也是响应式的。同时，当值为对象类型时，会用 `reactive()` 自动转换它的 `.value`

- 不同于`reactive`，在`ref`中，一个包含对象类型值的ref可以响应式地替换整个对象，**重新替换后也具有响应性**

  ```js
  const objectRef = ref({ count: 0 })

  // 这是响应式的替换
  objectRef.value = { count: 1 }
  ```


- 同样，与`reactive`不同的是，ref 被传递给函数或是从一般对象上被解构时，也不会丢失响应性

  ```js
  const obj = {
    foo: ref(1),
    bar: ref(2)
  }

  // 该函数接收一个 ref
  // 需要通过 .value 取值
  // 但它会保持响应性
  callSomeFunction(obj.foo)

  // 仍然是响应式的
  const { foo, bar } = obj
  ```


- 简言之，`ref()` 让我们能创造一种对任意值的 “引用”，并能够在不丢失响应性的前提下传递这些引用

  ​

#### 2.`<script setup>`

- 在 `setup()` 函数中手动暴露大量的状态和方法非常繁琐。幸运的是，我们可以通过使用构建工具来简化该操作。当使用单文件组件（SFC）时，我们可以使用 `<script setup>` 来大幅度地简化代码。

- `<script setup>` 中的顶层的导入和变量声明可在同一组件的模板中直接使用。你可以理解为模板中的表达式和 `<script setup>` 中的代码处在同一个作用域中，**从而不需要在setup函数中返回数据**。

  ```vue
  <script setup>
  import { reactive } from 'vue'

  const state = reactive({ count: 0 })

  function increment() {
    state.count++
  }
  </script>

  <template>
    <button @click="increment">
      {{ state.count }}
    </button>
  </template>
  ```




#### 3.DOM更改时机

- 当响应式状态进行更新时，DOM会自动更新。然而，DOM的更新并不是同步的，相反，Vue将缓冲它们知道更新周期的“下个时机”以确保无论你进行了多少次状态更改，每个组件都只需要更新一次

- 这就会导致更新完响应式数据后，DOM还没有进行更新，在有的情况下会导致错误

- **解决：**若要等待一个状态改变后的 DOM 更新完成，你可以使用nextTick（）这个全局 API

  ```js
  import { nextTick } from 'vue'

  function increment() {
    state.count++
    nextTick(() => {
      // 访问更新后的 DOM
    })
  }
  ```




#### 4.深层响应性

- 在Vue3中，状态都是默认深层响应式的（与Vue2不同，Vue2默认响应顶层对象，而且也不监听数据的变化），这意味着及时在更改深层次的对象或数组，你的改动也能被检测到。

  ```js
  import { reactive } from 'vue'

  const obj = reactive({
    nested: { count: 0 },
    arr: ['foo', 'bar']
  })

  function mutateDeeply() {
    // 以下都会按照期望工作
    obj.nested.count++
    obj.arr.push('baz')
  }
  ```

##### （1）创建浅层响应式对象

- 浅层响应式对象：它们仅在顶层具有响应性，使用`shallowReactive()`进行声明

- 和 `reactive()` 不同，这里没有深层级的转换：一个浅层响应式对象里只有根级别的属性是响应式的。属性的值会被原样存储和暴露，这也意味着值为 ref 的属性**不会**被自动解包了。

  ```js
  const state = shallowReactive({
    foo: 1,
    nested: {
      bar: 2
    }
  })

  // 更改状态自身的属性是响应式的
  state.foo++

  // ...但下层嵌套对象不会被转为响应式
  isReactive(state.nested) // false

  // 不是响应式的
  state.nested.bar++
  ```

##### （2）响应式代理 vs 原始对象

- `reactive()` 返回的是一个原始对象的`Proxy`，它和原始对象是不相等的

  ```js
  const raw = {}
  const proxy = reactive(raw)

  // 代理对象和原始对象不是全等的
  console.log(proxy === raw) // false
  ```


- 在Vue3中，只有代理对象是响应式的，原始对象不具有响应性，更改原始对象不会触发更新。因此，使用 Vue 的响应式系统的最佳实践是 **仅使用你声明对象的代理版本**。

- 为了保证访问代理对象的一致性，在Vue3中，对同一个原始对象调用 `reactive()` 会总是**返回同样的代理对象**，而对一个已存在的代理对象调用 `reactive()` 会**返回其本身**：

  ```js
  // 在同一个对象上调用 reactive() 会返回相同的代理
  console.log(reactive(raw) === proxy) // true

  // 在一个代理上调用 reactive() 会返回它自己
  console.log(reactive(proxy) === proxy) // true
  ```


- 这个规则对嵌套对象也适用。依靠深层响应性，响应式对象内的嵌套对象依然是代理

  ```js
  const proxy = reactive({})

  const raw = {}
  proxy.nested = raw

  console.log(proxy.nested === raw) // false
  ```




#### 5.ref解包

##### （1）ref在模板中的解包

- 当`ref`在模板中作为顶层属性被访问时，他们会被自动解包，所以不需要使用`.value`

  ```vue
  <script setup>
  import { ref } from 'vue'

  const count = ref(0)

  function increment() {
    count.value++
  }
  </script>

  <template>
    <button @click="increment">
      {{ count }} <!-- 无需 .value -->
    </button>
  </template>
  ```


- **注意：**仅当ref时模板渲染上下文的顶级属性时才适用自动解包。 例如， foo 是顶层属性，但 object.foo 不是

  ```js
  const object = { foo: ref(1) }
  // 不会渲染数据，会渲染一个对象，因为 object.foo 是一个 ref 对象
  {{ object.foo + 1 }}

  // 解决：
  const { foo } = object
  // 渲染结果为2
  {{ foo + 1 }}
  ```


- **注意：**如果一个 ref 是文本插值（即一个 `{{ }}` 符号）计算的最终值，它也将被解包,这只是文本插值的一个方便功能，相当于 `{{ object.foo.value }}`。

  ```js
  const object = { foo: ref(1) }
  // 渲染结果为1
  {{ object.foo }}
  ```

##### （2）ref在响应式对象中的解包

- 当一个ref被嵌套在一个响应式的对象中，座位属性被访问或被更改时，他会自动解包

  ```js
  const count = ref(0)
  const state = reactive({
    count
  })

  console.log(state.count) // 0

  state.count = 1
  console.log(count.value) // 1
  ```


- 将一个新的ref赋值给一个关联了已有ref的属性，那么它会替代旧的ref

  ```js
  const otherCount = ref(2)

  state.count = otherCount
  console.log(state.count) // 2
  // 原始 ref 现在已经和 state.count 失去联系
  console.log(count.value) // 1
  ```


- 注意：只有当嵌套在一个**深层响应式对象**内时，才会发生`ref`解包。当其作为**浅层响应式对象**的属性被访问时不会解包。

##### （3）数组和集合类型的ref解包

- 当`ref`作为响应式数组或像`Map`这种原生集合类型的元素被访问时，不会进行解包

  ```js
  const books = reactive([ref('Vue 3 Guide')])
  // 这里需要 .value
  console.log(books[0].value)

  const map = reactive(new Map([['count', ref(0)]]))
  // 这里需要 .value
  console.log(map.get('count').value)
  ```

##### （4）响应性语法糖

- 处于实验性阶段，可以省去使用 `.value` 的麻烦

  ```js
  const count = ref(0)
  // 不需要使用.value
  count.value ++  ==>  count++
  ```



### 三、计算属性

#### 1.计算属性基础

- 计算属性用来描述**依赖响应式状态**的复杂逻辑

  ```js
  <script setup>
  import { reactive, computed } from 'vue'
  const author = reactive({
    name: 'John Doe',
    books: [
      'Vue 2 - Advanced Guide',
      'Vue 3 - Basic Guide',
      'Vue 4 - The Mystery'
    ]
  })

  // 一个计算属性 ref
  const publishedBooksMessage = computed(() => {
    return author.books.length > 0 ? 'Yes' : 'No'
  })
  </script>
  ```


- **注意：** `computed()` 方法期望接受一个getter函数，返回值为一个计算属性ref，可以通过计算属性函数名访问计算结果。**计算属性 ref 也会在模板中自动解包，因此在模板表达式中引用时无需添加 `.value`。**

  ```js
  <template>
    <span>{{ publishedBooksMessage }}</span>
  </template>
  ```


- **注意：**Vue的计算属性会自动**追踪响应式依赖**，它会检测到 `publishedBooksMessage` 依赖于 `author.books`，所以当 `author.books` 改变时，任何依赖于 `publishedBooksMessage` 的绑定都会同时更新。**即如果数据不是响应式的，无论其怎么变，计算属性结果都不会响应更新。**



#### 2.计算属性 vs 方法

- 在Vue的表达式中电泳一个函数也会获得和计算属性相同的结果，若我们将同样的函数定义为一个方法而不是计算属性，两种方式在结果上确实是完全相同的，然而，不同之处在于**计算属性值会基于其响应式依赖被缓存**。一个计算属性仅会在其响应式依赖更新时才重新计算。**这意味着只要 `author.books` 不改变，无论多少次访问 `publishedBooksMessage` 都会立即返回先前的计算结果，而不用重复执行 getter 函数。**

- 例：下面的计算属性永远不会更新，因为`Date.now()` 并不是一个响应式依赖

  ```js
  const now = computed(() => Date.now())
  ```


- **相比之下，方法调用总是会在重渲染发生时再次执行函数。**



#### 3.可写计算属性

- 计算属性**默认是只读的** ，当你尝试修改一个计算属性时，你会收到一个运行时警告。只在某些特殊场景中你可能才需要用到“可写”的属性，你可以通过**同时提供 getter 和 setter 来创建**：

  ```js
  <script setup>
  import { ref, computed } from 'vue'

  const firstName = ref('John')
  const lastName = ref('Doe')

  const fullName = computed({
    // getter
    get() {
      return firstName.value + ' ' + lastName.value
    },
    // setter
    set(newValue) {
      // 注意：我们这里使用的是解构赋值语法
      [firstName.value, lastName.value] = newValue.split(' ')
    }
  })
  </script>
  ```


- 此时，运行 `fullName.value = 'John Doe'` 时，setter 会被调用而 `firstName` 和 `lastName` 会随之更新。



#### 4.注意事项

##### （1）getter不应有副作用

- 计算属性的 getter 应只做计算而没有任何其他的副作用，这一点非常重要，请务必牢记。举例来说，**不要在 getter 中做异步请求或者更改 DOM**！一个计算属性的声明中描述的是如何根据其他值派生一个值。因此 getter 的职责应该仅为计算和返回该值。

##### （2）避免直接修改计算属性值

- 从计算属性返回的值是派生状态。可以把它看作是一个“临时快照”，每当源状态发生变化时，就会创建一个新的快照。更改快照是没有意义的，因此计算属性的返回值应该被视为只读的，并且永远不应该被更改——应该更新它所依赖的源状态以触发新的计算。



### 四、Class 与 Style 绑定

#### 1.绑定class

##### （1）绑定对象

- **单个值：**

  ```vue
  <div :class="{ active: isActive }"></div>
  ```

  -  `active` 是否存在取决于数据属性 `isActive` 的真假值

- **多个值：**

  ```vue
  const isActive = ref(true)
  const hasError = ref(false)

  <div
    class="static"
    :class="{ active: isActive, 'text-danger': hasError }">
  </div>

  // 渲染的结果
  <div class="static active"></div>
  ```


- **直接绑定一个对象：**

  ```js
  const classObject = reactive({
    active: true,
    'text-danger': false
  })

  <div :class="classObject"></div>
  ```


- **绑定计算属性：**

  ```js
  const isActive = ref(true)
  const error = ref(null)

  const classObject = computed(() => ({
    active: isActive.value && !error.value,
    'text-danger': error.value && error.value.type === 'fatal'
  }))

  <div :class="classObject"></div>
  ```

##### （2）绑定数组

- **绑定一个数组：**

  ```js
  const activeClass = ref('active')
  const errorClass = ref('text-danger')

  <div :class="[activeClass, errorClass]"></div>

  // 渲染结果
  <div class="active text-danger"></div>
  ```


- **使用三元表达式：**

  ```vue
  <div :class="[isActive ? activeClass : '', errorClass]"></div>
  ```

  - `errorClass` 会一直存在，但 `activeClass` 只会在 `isActive` 为真时才存在。


- **数组中嵌套对象：**

  ```js
  <div :class="[{ active: isActive }, errorClass]"></div>
  ```

##### （3）组件中绑定

- 对于只有一个根元素的组件，当你使用了 `class` attribute 时，这些 class 会被添加到根元素上，并与该元素上已有的 class 合并。

  ```js
  <!-- 子组件模板 -->
  <p class="foo bar">Hi!</p>

  <!-- 在使用组件时 -->
  <MyComponent class="baz boo" />
   
   // 最终渲染结果
  <p class="foo bar baz boo">Hi</p>
  ```


- class 的绑定也是同样的

  ```js
  <!-- 子组件模板 -->
  <p class="foo bar">Hi!</p>

  <!-- 在使用组件时 -->
  <MyComponent :class="{ active: isActive }" />

  // isActive为真时，渲染结果
  <p class="foo bar active">Hi</p>
  ```


- 使用$attrs接受绑定class

  ```js
  <!-- MyComponent 模板使用 $attrs 时 -->
  <p :class="$attrs.class">Hi!</p>
  <span>This is a child component</span>

  <!-- 在使用组件时 -->
  <MyComponent class="baz" />

  // isActive为真时，渲染结果
  <p class="baz">Hi!</p>
  <span>This is a child component</span>
  ```

  ​

#### 2.绑定内联style

- **绑定对象：**`:style` 支持绑定 JavaScript 对象值，对应的时html元素的style属性

  ```js
  const activeColor = ref('red')
  const fontSize = ref(30)

  <div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
  ```

  - 尽管推荐使用 camelCase，但 `:style` 也支持 kebab-cased 形式的 CSS 属性 key (对应其 CSS 中的实际名称)

  ```vue
  <div :style="{ 'font-size': fontSize + 'px' }"></div>
  ```

  - 直接绑定一个对象

  ```js
  const styleObject = reactive({
    color: 'red',
    fontSize: '13px'
  })

  <div :style="styleObject"></div>
  ```

  - 同样的，如果样式对象需要更复杂的逻辑，也可以使用返回样式对象的计算属性。


- **绑定数组：**还可以给 `:style` 绑定一个包含多个样式对象的数组。这些对象会被合并后渲染到同一元素上

  ```js
  <div :style="[baseStyles, overridingStyles]"></div>
  ```


- **自动前缀：**当你在 `:style` 中使用了需要浏览器特殊前缀的 CSS 属性时，Vue 会自动为他们加上相应的前缀。Vue 是在运行时检查该属性是否支持在当前浏览器中使用。如果浏览器不支持某个属性，那么将测试加上各个浏览器特殊前缀，以找到哪一个是被支持的。


- **样式多值：**可以对一个样式属性提供多个 (不同前缀的) 值

  ```vue
  <div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
  ```

  **数组仅会渲染浏览器支持的最后一个值。在这个示例中，在支持不需要特别前缀的浏览器中都会渲染为 `display: flex`。**



### 五、条件渲染

1. **`v-if:`**

   - `v-if`指令用于条件性地渲染一块内容，这个内容只有`v-if`绑定的值为真时才会被渲染

     ```vue
     <h1 v-if="awesome">Vue is awesome!</h1>
     ```

2. **`v-else:`**

   - 可以使用 `v-else` 为 `v-if` 添加一个“else 区块”

     ```vue
     <h1 v-if="awesome">Vue is awesome!</h1>
     <h1 v-else>Oh no 😢</h1>
     ```


   - **注意：**一个 `v-else` 元素必须跟在一个 `v-if` 或者 `v-else-if` 元素后面，否则它将不会被识别

3. **`v-else-if：`**

   - `v-else-if` 提供的是相应于 `v-if` 的“else if 区块”。它可以连续多次重复使用

     ```vue
     <div v-if="type === 'A'">
       A
     </div>
     <div v-else-if="type === 'B'">
       B
     </div>
     <div v-else-if="type === 'C'">
       C
     </div>
     <div v-else>
       Not A/B/C
     </div>
     ```


   - **注意：**和 `v-else` 类似，一个使用 `v-else-if` 的元素必须紧跟在一个 `v-if` 或一个 `v-else-if` 元素后面。

4. **`<template>上的v-if`**

   - 可以在一个 `<template>` 元素上使用 `v-if`，这只是一个不可见的包装器元素，最后渲染的结果并不会包含这个 `<template>` 元素

     ```vue
     <template v-if="ok">
       <h1>Title</h1>
       <p>Paragraph 1</p>
       <p>Paragraph 2</p>
     </template>
     ```


   - **注意：**`v-else` 和 `v-else-if` 也可以在 `<template>` 上使用

5. **`v-show：`**

   -  `v-show` 会在 DOM 渲染中保留该元素；`v-show` 仅切换了该元素上名为 `display` 的 CSS 属性

     ```vue
     <h1 v-show="ok">Hello!</h1>
     ```


   - **注意：**`v-show` 不支持在 `<template>` 元素上使用，也不能和 `v-else` 搭配使用

6. **`v-if` 和 `v-show`**

   - `v-if` 是“真实的”按条件渲染，因为它确保了在切换时，条件区块内的事件监听器和子组件都会被销毁与重建。
   - `v-if` 也是**惰性**的：如果在初次渲染时条件值为 false，则不会做任何事。条件区块只有当条件首次变为 true 时才被渲染。
   - 相比之下，`v-show` 简单许多，元素无论初始条件如何，始终会被渲染，只有 CSS `display` 属性会被切换。
   - 总的来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。因此，如果需要频繁切换，则使用 `v-show` 较好；如果在运行时绑定条件很少改变，则 `v-if` 会更合适

7. **`v-if` 和 `v-for`**

   - 当 `v-if` 和 `v-for` 同时存在于一个元素上的时候，`v-if` 会首先被执行，`v-if` 比 `v-for` 的优先级更高。这意味着 `v-if` 的条件将无法访问到 `v-for` 作用域内定义的变量别名

   ​

### 六、列表渲染

1. **`v-for`与数组：**

   - 使用 `v-for` 指令基于一个数组来渲染一个列表。`v-for` 指令的值需要使用 `item in items` 形式的特殊语法，其中 `items` 是源数据的数组，而 `item` 是迭代项的**别名**

     ```vue
     const items = ref([{ message: 'Foo' }, { message: 'Bar' }])

     <li v-for="item in items">
       {{ item.message }}
     </li>
     ```


   - 在`v-for`语句中可以完整的访问父作用域内的属性和方法，`v-for` 也支持使用可选的第二个参数表示当前项的位置索引

     ```vue
     const parentMessage = ref('Parent')
     const items = ref([{ message: 'Foo' }, { message: 'Bar' }])

     <li v-for="(item, index) in items">
       {{ parentMessage }} - {{ index }} - {{ item.message }}
     </li>
     ```


   - 多层嵌套的`v-for`，作用域的工作方式和函数的作用域很类似，每个`v-for`作用域都可以访问到父级作用域

     ```vue
     <li v-for="item in items">
       <span v-for="childItem in item.children">
         {{ item.message }} {{ childItem }}
       </span>
     </li>
     ```


   - **注意：可以使用 `of` 作为分隔符来替代 `in`，这更接近 JavaScript 的迭代器语法**

     ```vue
     <div v-for="item of items"></div>
     ```

2. **`v-for`与对象：**

   - 也可以使用 `v-for` 来遍历一个对象的所有属性。遍历的顺序会基于对该对象调用 `Object.keys()` 的返回值来决定

     ```vue
     const myObject = reactive({
       title: 'How to do lists in Vue',
       author: 'Jane Doe',
       publishedAt: '2016-04-10'
     })

     <ul>
       <li v-for="value in myObject">
         {{ value }}
       </li>
     </ul>
     ```


   - 可以提供第二个参数表示属性名（key）

     ```vue
     <li v-for="(value, key) in myObject">
       {{ key }}: {{ value }}
     </li>
     ```


   - 第三个参数表示位置索引

     ```vue
     <li v-for="(value, key, index) in myObject">
       {{ index }}. {{ key }}: {{ value }}
     </li>
     ```

3. **在`v-for`里使用范围值：**

   - `v-for` 可以直接接受一个整数值。在这种用例中，会将该模板基于 `1...n` 的取值范围重复多次

     ```vue
     <span v-for="n in 10">{{ n }}</span>
     ```


   - **注意：**此处 `n` 的初值是从 `1` 开始而非 `0`

4.  **`<template>`上的v-for**

   - 与模板上的 `v-if` 类似，你也可以在 `<template>` 标签上使用 `v-for` 来渲染一个包含多个元素的块

     ```vue
     <ul>
       <template v-for="item in items">
         <li>{{ item.msg }}</li>
         <li class="divider" role="presentation"></li>
       </template>
     </ul>
     ```

5. **通过key管理状态：**

   - Vue 默认按照“就地更新”的策略来更新通过 `v-for` 渲染的元素列表。当数据项的顺序改变时，Vue 不会随之移动 DOM 元素的顺序，而是就地更新每个元素，确保它们在原本指定的索引位置上渲染

   - 为了给 Vue 一个提示，以便它可以跟踪每个节点的标识，从而重用和重新排序现有的元素，你需要为每个元素对应的块提供一个唯一的 `key` attribute

     ```vue
     <div v-for="item in items" :key="item.id">
       <!-- 内容 -->
     </div>
     ```


   - 当你使用 `<template v-for>` 时，`key` 应该被放置在这个 `<template>` 容器上

     ```vue
     <template v-for="todo in todos" :key="todo.name">
       <li>{{ todo.name }}</li>
     </template>
     ```


   - **注意：** 在任何可行的时候为 `v-for` 提供一个 `key` attribute，除非所迭代的 DOM 内容非常简单 (例如：不包含组件或有状态的 DOM 元素)，或者你想有意采用默认行为来提高性能。`key` 绑定的值期望是一个基础类型的值，例如字符串或 number 类型。不要用对象作为 `v-for` 的 key

6. **组件上使用`v-for` ：**

   - 可以直接在组件上使用 `v-for`，和在一般的元素上使用没有区别 (别忘记提供一个 `key`)

     ```vue
     <MyComponent v-for="item in items" :key="item.id" />
     ```


   - 但不会将任何数据传递给组件，组件有自己独立的作用域，为了将迭代后的数据传递给组件中，我们还需传递props

     ```vue
     <MyComponent
       v-for="(item, index) in items"
       :item="item"
       :index="index"
       :key="item.id"
     />
     ```

7. **数组变化侦测**

   - Vue3能够侦听响应式数组的变更方法，并在它们被调用时触发相关的更新，包括：
     - `push()`
     - `pop()`
     - `shift()`
     - `unshift()`
     - `splice()`
     - `sort()`
     - `reverse()`


   - **替换一个数组：**变更方法，顾名思义，就是会对调用它们的原数组进行变更。相对地，也有一些不可变 (immutable) 方法，例如 `filter()`，`concat()` 和 `slice()`，这些都不会更改原数组，而总是**返回一个新数组**。当遇到的是非变更方法时，我们需要将旧的数组替换为新的：

     ```js
     // `items` 是一个数组的 ref
     items.value = items.value.filter((item) => item.message.match(/Foo/))
     ```



### 七、生命周期

- 每个 Vue 组件实例在创建时都需要经历一系列的初始化步骤，比如设置好数据侦听，编译模板，挂载实例到 DOM，以及在数据改变时更新 DOM。在此过程中，它也会运行被称为生命周期钩子的函数，让开发者有机会在特定阶段运行自己的代码。

#### 1.onMounted（）

- 注册一个回调函数，在组件挂在完成后执行

  ```vue
  <script setup>
  import { ref, onMounted } from 'vue'

  const el = ref()

  onMounted(() => {
    el.value // <div>
  })
  </script>

  <template>
    <div ref="el"></div>
  </template>
  ```


- 这个生命周期钩子函数通常用于执行需要访问组件所渲染的DOM树相关的副作用
- **注意：这个钩子在服务器端渲染期间不会被调用**



#### 2.onUpdated（）

- 注册一个回调函数，在组件因为响应式状态变更而更新其DOM树之后被调用

  ```vue
  <script setup>
  import { ref, onUpdated } from 'vue'

  const count = ref(0)

  onUpdated(() => {
    // 文本内容应该与当前的 `count.value` 一致
    console.log(document.getElementById('count').textContent)
  })
  </script>

  <template>
    <button id="count" @click="count++">{{ count }}</button>
  </template>
  ```


- **注意：**
  - 父组件的更新钩子将在其子组件的更新钩子之后被调用
  - 这个生命周期函数会在组件的任意DOM更新后被调用，这些更新可能有不同的状态变更导致的，如果你需要在某个特定的状态更改后访问更新后的 DOM，请使用`nextTick()`作为替代
  - **这个钩子在服务器端渲染期间不会被调用**
  - **不要在 updated 钩子中更改组件的状态，这可能会导致无限的更新循环**



#### 3.onUnmounted（）

- 注册一个回调函数，在组件实例被卸载之后调用

  ```vue
  <script setup>
  import { onMounted, onUnmounted } from 'vue'

  let intervalId
  onMounted(() => {
    intervalId = setInterval(() => {
      // ...
    })
  })

  onUnmounted(() => clearInterval(intervalId))
  </script>
  ```


- **注意：**
  - 当一个组件的所有子组件都已经被卸载或者其相关的响应式作用都已经停止时被视为已卸载
  - 经常用于在这个钩子中手动清理一些副作用，例如计时器、DOM 事件监听器或者与服务器的连接
  - **这个钩子在服务器端渲染期间不会被调用**



#### 4.onBeforeMount（）

- 注册一个钩子，在组件被挂载之前被调用


- 当这个钩子被调用时，组件已经完成了其响应式状态的设置，但还没有创建 DOM 节点。它即将首次执行 DOM 渲染过程。
- **注意： 这个钩子在服务器端渲染期间不会被调用。**



#### 5.onBeforeUpdate（）

- 注册一个钩子，在组件即将因为响应式状态变更而更新其DOM树之前被调用
- 这个钩子可以用来在 Vue 更新 DOM 之前访问 DOM 状态。在这个钩子中更改状态也是安全的
- **注意： 这个钩子在服务器端渲染期间不会被调用**



#### 6.onBeforeUnmount（）

- 注册一个钩子，在组件实例被卸载之前调用
- 当这个钩子被调用时，组件实例依然还保有全部的功能
- **注意： 这个钩子在服务器端渲染期间不会被调用**



#### 7.onErrorCaputred（）

- 注册一个钩子，再捕获后代组件传递的错误时被调用
- 这个钩子带有三个实参：**错误对象、触发该错误的组件实例，以及一个说明错误来源类型的信息字符串**
- 可以在 `errorCaptured()` 中更改组件状态来为用户显示一个错误状态。注意不要让错误状态再次渲染导致本次错误的内容，否则组件会陷入无限循环。
- **错误传递规则：**
  - 默认情况下，所有的错误都会被发送到应用级的`app.config.errorHandler`， 这样这些错误都能在一个统一的地方报告给分析任务
  - 如果组件的继承链或组件链上存在多个 `errorCaptured` 钩子，对于同一个错误，这些钩子会被按从底至上的顺序一一调用。这个过程被称为“向上传递”，类似于原生 DOM 事件的冒泡机制
  - 如果 `errorCaptured` 钩子本身抛出了一个错误，那么这个错误和原来捕获到的错误都将被发送到 `app.config.errorHandler`
  - `errorCaptured` 钩子可以通过返回 `false` 来阻止错误继续向上传递。即表示“这个错误已经被处理了，应当被忽略”，它将阻止其他的 `errorCaptured` 钩子或 `app.config.errorHandler` 因这个错误而被调用



#### 8.onRenderTracked（）

- 注册一个调试钩子，当组件渲染过程中追踪到响应式依赖时调用
- **注意：这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用**



#### 9.onRenderTriggered（）

- 注册一个调试钩子，当响应式依赖的变更触发了组件渲染时调用
- **注意：这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用**



#### 10.onActivated（）

- 注册一个回调函数，若组件实例是`<keepAlive>` 缓存树的一部分，当组件被插入到 DOM 中时调用。
- **注意： 这个钩子在服务器端渲染期间不会被调用**



#### 11.onDeactivated（）

- 注册一个回调函数，若组件实例是`<keepAlive>` 缓存树的一部分，当组件DOM 中被移除时调用
- **注意： 这个钩子在服务器端渲染期间不会被调用**



#### 12.onServerPrefetch（）

- 注册一个异步函数，在组件实例在服务器上被渲染之前调用

- 如果这个钩子返回了一个 Promise，服务端渲染会在渲染该组件前等待该 Promise 完成。这个钩子仅会在服务端渲染中执行，可以用于执行一些仅存在于服务端的数据抓取过程

  ```vue
  <script setup>
  import { ref, onServerPrefetch, onMounted } from 'vue'

  const data = ref(null)

  onServerPrefetch(async () => {
    // 组件作为初始请求的一部分被渲染
    // 在服务器上预抓取数据，因为它比在客户端上更快。
    data.value = await fetchOnServer(/* ... */)
  })

  onMounted(async () => {
    if (!data.value) {
      // 如果数据在挂载时为空值，这意味着该组件
      // 是在客户端动态渲染的。将转而执行
      // 另一个客户端侧的抓取请求
      data.value = await fetchOnClient(/* ... */)
    }
  })
  </script>
  ```




### 八、侦听器

- 在组合式 API 中，我们可以使用`watch`函数在每次响应式状态发生变化时触发回调函数

  ```vue
  <script setup>
  import { ref, watch } from 'vue'

  const question = ref('')
  const answer = ref('Questions usually contain a question mark. ;-)')

  // 可以直接侦听一个 ref
  watch(question, async (newQuestion, oldQuestion) => {
    if (newQuestion.indexOf('?') > -1) {
      answer.value = 'Thinking...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        answer.value = (await res.json()).answer
      } catch (error) {
        answer.value = 'Error! Could not reach the API. ' + error
      }
    }
  })
  </script>

  <template>
    <p>
      Ask a yes/no question:
      <input v-model="question" />
    </p>
    <p>{{ answer }}</p>
  </template>
  ```

#### 1.侦听数据源类型

- `watch` 的第一个参数可以是不同形式的“数据源”：它可以是**一个 ref (包括计算属性)**、**一个响应式对象**、**一个 getter 函数**、或**多个数据源组成的数组**

  ```js
  const x = ref(0)
  const y = ref(0)

  // 单个 ref
  watch(x, (newX) => {
    console.log(`x is ${newX}`)
  })

  // getter 函数
  watch(
    () => x.value + y.value,
    (sum) => {
      console.log(`sum of x + y is: ${sum}`)
    }
  )

  // 多个来源组成的数组
  watch([x, () => y.value], ([newX, newY]) => {
    console.log(`x is ${newX} and y is ${newY}`)
  })
  ```


- **注意：** 不能直接侦听响应式对象的属性值，需要用一个返回该属性的 getter 函数

  ```js
  // 提供一个 getter 函数
  watch(
    () => obj.count,
    (count) => {
      console.log(`count is: ${count}`)
    }
  )
  ```


 

#### 2.深层侦听器

- 直接给 `watch()` 传入一个响应式对象，会**隐式地创建一个深层侦听器——该回调函数在所有嵌套的变更时都会被触发**

  ```js
  const obj = reactive({ count: 0 })

  watch(obj, (newValue, oldValue) => {
    // 在嵌套的属性变更时触发
    // 注意：`newValue` 此处和 `oldValue` 是相等的
    // 因为它们是同一个对象！
  })

  obj.count++
  ```

- 一个返回响应式对象的 getter 函数，只有在返回不同的对象时，才会触发回调

  ```js
  watch(
    () => state.someObject,
    () => {
      // 仅当 state.someObject 被替换时触发
    }
  )
  ```

- 也可以给上面这个例子显式地加上 `deep` 选项，强制转成深层侦听器

  ```js
  watch(
    () => state.someObject,
    (newValue, oldValue) => {
      // 注意：`newValue` 此处和 `oldValue` 是相等的
      // *除非* state.someObject 被整个替换了
    },
    { deep: true }
  )
  ```



#### 3.watchEffect（）

- `watch（）`是懒执行的：仅当数据源变化时，才会执行回调。但在某些场景中，我们希望在创建侦听器时，立即执行一遍回调。这时可以使用`watchEffect（）`

  ```js
  watchEffect(async () => {
    const response = await fetch(url.value)
    data.value = await response.json()
  })
  ```


- `watchEffect（）`会立即执行一遍回调函数，如果这是函数产生了副作用，Vue会自动追踪副作用的依赖关系，自动分析出响应源，每当 `url.value` 变化时，回调会再次执行


- **`watch` vs `watchEffect`** ：`watch` 和 `watchEffect` 都能响应式地执行有副作用的回调。它们之间的主要区别是追踪响应式依赖的方式
  - `watch` 只追踪明确侦听的数据源。它不会追踪任何在回调中访问到的东西。另外，仅在数据源确实改变时才会触发回调。`watch` 会避免在发生副作用时追踪依赖，因此，我们能更加精确地控制回调函数的触发时机。
  - `watchEffect`，则会在副作用发生期间追踪依赖。它会在同步执行过程中，自动追踪所有能访问到的响应式属性。这更方便，而且代码往往更简洁，但有时其响应性依赖关系会不那么明确。



#### 4.回调的触发时机

- 当你更改了响应式状态，他可能会**同时触发Vue组件更新和侦听器回调。**

- 默认情况下，用户创建的侦听器回调，都会在Vue组件**更新之前被调用** 。这意味着你在侦听器回调中访问的DOM将是被Vue更改之前的状态

- **如果你想在侦听器回调中能够访问被Vue更新之后的DOM，你需要指明 `flush: 'post'` 选项**

  ```js
  watchEffect(callback, {
    flush: 'post'
  })
  ```


- **后置刷新的`watchEffect（）`有个更方便的别名`watchPostEffect（）`**

  ```js
  import { watchPostEffect } from 'vue'

  watchPostEffect(() => {
    /* 在 Vue 更新后执行 */
  })
  ```



#### 5.停止侦听器

- 在`setup（）`或`<script setup>`中**用同步语句创建的侦听器，会自动绑定到宿主组件实例上，并且会在宿主组件被卸载时自动停止**。因此，在大多数情况下（同步创建），你无需关心怎么停止一个侦听器


- 但关键点是，侦听器必须是同步语句创建，**如果用异步回调创建一个侦听器，那么他不会绑定到当前组件上，你必须手动停止它**，以放内存泄露

  ```js
  <script setup>
  import { watchEffect } from 'vue'

  // 它会自动停止
  watchEffect(() => {})

  // ...这个则不会！
  setTimeout(() => {
    watchEffect(() => {})
  }, 100)
  </script>
  ```


- 要手动停止一个侦听器，请调用`watch`或`watchEffect（）`返回的函数

  ```js
  const unwatch = watchEffect(() => {})

  // ...当该侦听器不再需要时
  unwatch()
  ```


- 注意：需要一步创建侦听器的情况很少，请尽可能选择同步创建。如果需要等待一些异步数据，你可以使用条件式的侦听逻辑

  ```js
  // 需要异步请求得到的数据
  const data = ref(null)

  watchEffect(() => {
    if (data.value) {
      // 数据加载后执行某些操作...
    }
  })
  ```

  ​



### 九、模板引用

- 在某些情况下，我们仍需要直接访问底层DOM元素，要实现这一点，可以使用特殊的 `ref` attribute

  ```vue
  <input ref="input">
  ```


- `ref`是一个特殊的attribute，和`v-for`中的`key`类似，它允许我们在一个特定的DOM元素或子组件实例被挂载后，获得对他的直接引用。这可能很有用，比如说在组件挂载时将焦点设置到一个 input 元素上，或在一个元素上初始化一个第三方库



#### 1.访问模板引用

- 为了通过组合式API获得该模板引用，需要声明一个同名的`ref`

  ```js
  <script setup>
  import { ref, onMounted } from 'vue'

  // 声明一个 ref 来存放该元素的引用
  // 必须和模板里的 ref 同名
  const input = ref(null)

  onMounted(() => {
    input.value.focus()
  })
  </script>

  <template>
    <input ref="input" />
  </template>
  ```


- 如果不使用 `<script setup>`，需确保从 `setup()` 返回 `ref`

  ```js
  export default {
    setup() {
      const input = ref(null)
      // ...
      return {
        input
      }
    }
  }
  ```


- **注意：** 只可以在组件挂在后才能访问模板引用。如果你想在模板的表达式上访问input，在初次渲染时会是null（初次渲染时这个元素不存在）



#### 2.v-for中的模板引用

- 当在v-for中使用模板引用时，对应的ref中包含的值是一个数组，他将在元素被挂载后包含对应整个列表的所有元素

  ```js
  <script setup>
  import { ref, onMounted } from 'vue'

  const list = ref([
    /* ... */
  ])

  const itemRefs = ref([])

  onMounted(() => console.log(itemRefs.value))
  </script>

  <template>
    <ul>
      <li v-for="item in list" ref="itemRefs">
        {{ item }}
      </li>
    </ul>
  </template>
  ```



#### 3.函数模板引用

- `ref` attribute 还可以绑定为一个函数，会**在每次组件更新时都被调用**。该函数会收到元素引用作为其第一个参数

  ```vue
  <input :ref="(el) => { /* 将 el 赋值给一个数据属性或 ref 变量 */ }">
  ```


- **注意：**这里需要使用动态的 `:ref` 绑定才能够传入一个函数。当绑定的元素被卸载时，函数也会被调用一次，此时的 `el` 参数会是 `null`。你当然也可以绑定一个组件方法而不是内联函数



#### 4.组件上的ref

- 模板引用也可以被用在一个子组件上，这种情况下引用中获得的值是组件实例

  ```js
  <script setup>
  import { ref, onMounted } from 'vue'
  import Child from './Child.vue'

  const child = ref(null)

  onMounted(() => {
    // child.value 是 <Child /> 组件的实例
  })
  </script>

  <template>
    <Child ref="child" />
  </template>
  ```


- 如果一个子组件使用的是选项式API或没有使用 `<script setup>`，被引用的组件实例和该子组件的`this`完全一致，这意味着父组件对子组件的每一个属性和方法都有完全的访问权。这使得在父组件和子组件之间创建紧密耦合的实现细节变得很容易，当然也因此，应该只在绝对需要时才使用组件引用。大多数情况下，你应该首先使用标准的 props 和 emit 接口来实现父子组件交互。

- 有一个例外的情况，使用了 `<script setup>` 的组件是**默认私有**的：一个父组件无法访问到一个使用了 `<script setup>` 的子组件中的任何东西，除非子组件在其中通过 `defineExpose` 宏显式暴露

  ```js
  <script setup>
  import { ref } from 'vue'

  const a = 1
  const b = ref(2)

  defineExpose({
    a,
    b
  })
  </script>
  ```





### 十、组件

#### 1.组件注册

##### （1）全局注册

- 使用Vue组件实例的 `app.component()` 方法，让组件在当前 Vue 应用中全局可用

  ```js
  import { createApp } from 'vue'

  const app = createApp({})

  app.component(
    // 注册的名字
    'MyComponent',
    // 组件的实现
    {
      /* ... */
    }
  )
  ```


- 如果使用单文件组件，你可以注册被导入的 `.vue` 文件

  ```js
  import MyComponent from './App.vue'

  app.component('MyComponent', MyComponent)
  ```


- `app.component()` 方法可以被链式调用
- 全局注册的组件可以在此应用的任意组件的模板中使用，所有的子组件也可以使用全局注册的组件，这意味着这三个组件也都可以在*彼此内部*使用
- 全局注册具有以下几个问题：
  - 全局注册，但并没有被使用的组件无法在生产打包时被自动移除 (也叫“tree-shaking”)。如果你全局注册了一个组件，即使它并没有被实际使用，它仍然会出现在打包后的 JS 文件中。
  - 全局注册在大型项目中使项目的依赖关系变得不那么明确。在父组件中使用子组件时，不太容易定位子组件的实现。和使用过多的全局变量一样，这可能会影响应用长期的可维护性

##### （2）局部注册

- 局部注册的组件需要在使用它的父组件中显式导入，并且只能在该父组件中使用。它的优点是使组件之间的依赖关系更加明确，并且对 tree-shaking 更加友好。

- 在使用 `<script setup>` 的单文件组件中，导入的组件可以直接在模板中使用，无需注册

  ```js
  <script setup>
  import ComponentA from './ComponentA.vue'
  </script>

  <template>
    <ComponentA />
  </template>
  ```


- 如果没有使用 `<script setup>`，则需要使用 `components` 选项来显式注册

  ```js
  import ComponentA from './ComponentA.js'

  export default {
    components: {
      ComponentA
    },
    setup() {
      // ...
    }
  }
  ```


- 对于每个 `components` 对象里的属性，它们的 key 名就是注册的组件名，而值就是相应组件的实现。上面的例子中使用的是 ES2015 的缩写语法，等价于

  ```js
  export default {
    components: {
      ComponentA: ComponentA
    }
    // ...
  }
  ```


- **注意：局部注册的组件在后代组件中并不可用**

##### （3）组件名格式

- 推荐使用 PascalCase 作为组件名的注册格式，这是因为：
  - PascalCase 是合法的 JavaScript 标识符。这使得在 JavaScript 中导入和注册组件都很容易，同时 IDE 也能提供较好的自动补全。
  - `<PascalCase />` 在模板中更明显地表明了这是一个 Vue 组件，而不是原生 HTML 元素。同时也能够将 Vue 组件和自定义元素 (web components) 区分开来


- 为了方便，Vue 支持将模板中使用 kebab-case 的标签解析为使用 PascalCase 注册的组件。这意味着一个以 `MyComponent` 为名注册的组件，在模板中可以通过 `<MyComponent>` 或 `<my-component>` 引用。这让我们能够使用同样的 JavaScript 组件注册代码来配合不同来源的模板



#### 2.Props

##### （1）Props声明

- 组件需要显示地声明他所接受的props，这样Vue才能知道外部传入的哪些是props，哪些是透传 attribute

- **在使用`<script setup>` 的单文件组件中，props 可以使用 `defineProps()` 宏来声明：**

  ```js
  <script setup>
  const props = defineProps(['foo'])
  console.log(props.foo)
  </script>
  ```


- **在没有使用 `<script setup>` 的组件中，prop 可以使用 `props`选项来声明：**

  ```js
  export default {
    props: ['foo'],
    setup(props) {
      // setup() 接收 props 作为第一个参数
      console.log(props.foo)
    }
  }
  ```


- **注意：**传递给 `defineProps()` 的参数和提供给 `props` 选项的值是相同的，两种声明方式背后其实使用的都是 prop 选项


- 除了使用字符串数组来声明prop外，还可以使用对象的形式：

  ```js
  // 使用 <script setup>
  defineProps({
    title: String,
    likes: Number
  })

  // 非 <script setup>
  export default {
    props: {
      title: String,
      likes: Number
    }
  }
  ```


- 以对象形式声明的每个属性，key是prop的名称，而值是该prop预期类型的构造函数。对象形式的 props 声明不仅可以一定程度上作为组件的文档，而且如果其他开发者在使用你的组件时传递了错误的类型，也会在浏览器控制台中抛出警告

  ```typescript
  <script setup lang="ts">
  defineProps<{
    title?: string
    likes?: number
  }>()
  </script>
  ```

##### （2）prop的细节

- **Prop名字格式：**如果一个 prop 的名字很长，应使用 camelCase 形式，因为它们是合法的 JavaScript 标识符，可以直接在模板的表达式中使用，也可以避免在作为属性 key 名时必须加上引号

  ```js
  defineProps({
    greetingMessage: String
  })
  ```


- **动态prop：** `<BlogPost :title="post.title" />`


- **任何类型的值都可以作为props的值传递**

- **使用一个对象绑定多个prop：**如果你想要将一个对象的所有属性都当作 props 传入，你可以使用没有参数的`v-bind`，即只使用 `v-bind` 而非 `:prop-name`

  ```js
  const post = {
    id: 1,
    title: 'My Journey with Vue'
  }

  <BlogPost v-bind="post" />
  // 等价于
  <BlogPost :id="post.id" :title="post.title" />
  ```

##### （3）单向数据流

- 所有的props都遵循着**单向绑定**的原则，props因父组件的更新而变化，自然地将新的状态向下流往子组件，而不会逆向传递。这就**避免了子组件以外修改父组件的状态的情况**。
- 另外，每次父组件更新后，所有的字组件中的props都会被更新到最新值，这就意味着你**不应该在子组件中去更改一个 prop**。若你这么做了，Vue 会在控制台上向你抛出警告

##### （4）更改对象/数组类型的props

- 当对象或数组作为 props 被传入时，虽然子组件无法更改 props 绑定，但仍然**可以**更改对象或数组内部的值。这是因为 JavaScript 的对象和数组是按引用传递，而对 Vue 来说，禁止这样的改动虽然可能，但有很大的性能损耗，比较得不偿失。
- 这种更改的主要缺陷是它允许了子组件以某种不明显的方式影响父组件的状态，可能会使数据流在将来变得更难以理解。在最佳实践中，你应该尽可能避免这样的更改，除非父子组件在设计上本来就需要紧密耦合。在大多数场景下，子组件应该**抛出一个事件来通知父组件做出改变**。

##### （5）prop校验

- Vue组件可以更细致的声明对传入的props的校验要求，要声明对 props 的校验，你可以向 `defineProps()` 宏提供一个带有 props 校验选项的对象

  ```js
  defineProps({
    // 基础类型检查
    // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
    propA: Number,
    // 多种可能的类型
    propB: [String, Number],
    // 必传，且为 String 类型
    propC: {
      type: String,
      required: true
    },
    // Number 类型的默认值
    propD: {
      type: Number,
      default: 100
    },
    // 对象类型的默认值
    propE: {
      type: Object,
      // 对象或数组的默认值
      // 必须从一个工厂函数返回。
      // 该函数接收组件所接收到的原始 prop 作为参数。
      default(rawProps) {
        return { message: 'hello' }
      }
    },
    // 自定义类型校验函数
    propF: {
      validator(value) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].includes(value)
      }
    },
    // 函数类型的默认值
    propG: {
      type: Function,
      // 不像对象或数组的默认，这不是一个工厂函数。这会是一个用来作为默认值的函数
      default() {
        return 'Default function'
      }
    }
  })
  ```


- **注意：**
  - 所有 prop 默认都是可选的，除非声明了 `required: true`。
  - 除 `Boolean` 外的未传递的可选 prop 将会有一个默认值 `undefined`。
  - `Boolean` 类型的未传递 prop 将被转换为 `false`。你应该为它设置一个 `default` 值来确保行为符合预期。
  - 如果声明了 `default` 值，那么在 prop 的值被解析为 `undefined` 时，无论 prop 是未被传递还是显式指明的 `undefined`，都会改为 `default` 值。


- 当 prop 的校验失败后，Vue 会抛出一个控制台警告 (在开发模式下)
- 校验选项中的 `type` 可以是下列这些原生构造函数：
  - `String`
  - `Number`
  - `Boolean`
  - `Array`
  - `Object`
  - `Date`
  - `Function`
  - `Symbol`


- 另外，`type` 也可以是自定义的类或构造函数，Vue 将会通过 `instanceof` 来检查类型是否匹配。例如下面这个类：

  ```js
  class Person {
    constructor(firstName, lastName) {
      this.firstName = firstName
      this.lastName = lastName
    }
  }

  defineProps({
    author: Person
  })
  ```


- 为了更贴近原生 boolean attributes 的行为，声明为 `Boolean` 类型的 props 有特别的类型转换规则

  ```js
  defineProps({
    disabled: Boolean
  })

  <!-- 等同于传入 :disabled="true" -->
  <MyComponent disabled />

  <!-- 等同于传入 :disabled="false" -->
  <MyComponent />
  ```



#### 3.组件事件

##### （1）触发与监听事件

- **在子组件的模板表达式中直接使用`$emit`方法触发自定义事件**

  ```vue
  <button @click="$emit('someEvent')">click me</button>
  ```


- **在父组件中通过v-on（@）来监听事件**

  ```js
  <MyComponent @some-event="callback" />
  ```


- 像组件与prop一样，事件的名字也提供了自动的格式转换（触发了一个以 camelCase 形式命名的事件，但在父组件中可以使用 kebab-case 形式来监听）

##### （2）事件参数

- 可以给子组件`$emit`触发自定义事件时传入一个额外的参数并在监听事件中接收

  ```vue
  // 父组件
  <MyButton @increase-by="(n) => count += n" />

  // 子组件
  <button @click="$emit('increaseBy', 1)">
    Increase by 1
  </button>
  ```

##### （3）声明触发的事件

- 组件要触发的事件可以显示地通过`defineEmits（）`来声明。在 `<template>` 中使用的 `$emit` 方法不能在组件的 `<script setup>` 部分中使用，但 `defineEmits()` 会返回一个相同作用的函数供我们使用

  ```js
  <script setup>
  const emit = defineEmits(['inFocus', 'submit'])

  function buttonClick() {
    emit('submit')
  }
  </script>
  ```


- `defineEmits()` 宏**不能**在子函数中使用。如上所示，它必须直接放置在 `<script setup>` 的顶级作用域下.

- 如果使用 `setup` 函数而不是 `<script setup>`，则事件需要通过 `emits` 选项来定义，`emit` 函数也被暴露在 `setup()` 的上下文对象上

  ```js
  export default {
    emits: ['inFocus', 'submit'],
    setup(props, ctx) {
      ctx.emit('submit')
    }
  }

  // 解构emit
  export default {
    emits: ['inFocus', 'submit'],
    setup(props, { emit }) {
      emit('submit')
    }
  }
  ```


-  `emits` 选项还支持对象语法，它允许我们对触发事件的参数进行验证

  ```js
  <script setup>
  const emit = defineEmits({
    submit(payload) {
      // 通过返回值为 `true` 还是为 `false` 来判断
      // 验证是否通过
    }
  })
  </script>
  ```


- 搭配 TypeScript 使用 `<script setup>`

  ```typescript
  <script setup lang="ts">
  const emit = defineEmits<{
    (e: 'change', id: number): void
    (e: 'update', value: string): void
  }>()
  </script>
  ```

##### （4）事件校验

- 为事件添加校验，那么事件可以被赋值为一个函数，接受的参数就是抛出事件时传入 `emit` 的内容，返回一个布尔值来表明事件是否合法。

  ```js
  <script setup>
  const emit = defineEmits({
    // 没有校验
    click: null,

    // 校验 submit 事件
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  })

  function submitForm(email, password) {
    emit('submit', { email, password })
  }
  </script>
  ```

#### 4.V-model展开

##### （1）使用自定义事件

- **在表单上：**

  ```js
  <input v-model="searchText" />
    
  // 等价于
    
  <input
    :value="searchText"
    @input="searchText = $event.target.value"
  />
  ```


- **在组件上：**

  ```js
  <CustomInput
    :modelValue="searchText"
    @update:modelValue="newValue => searchText = newValue"
  />
  ```


- **注意：如果没有单独设置，必须使用modelValue**

- `<CustomInput>` 组件内部需要做两件事：

  1. 将内部原生 `input` 元素的 `value` attribute 绑定到 `modelValue` prop
  2. 输入新的值时在 `input` 元素上触发 `update:modelValue` 事件

  ```js
  <script setup>
  defineProps(['modelValue'])
  defineEmits(['update:modelValue'])
  </script>

  <template>
    <input
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    />
  </template>
  ```

##### （2）使用计算属性

- 另外一种方式是使用一个可写的，同时具有 getter 和 setter 的计算属性。`get` 方法需返回 `modelValue` prop，而 `set` 方法需触发相应的事件

  ```js
  <script setup>
  import { computed } from 'vue'

  const props = defineProps(['modelValue'])
  const emit = defineEmits(['update:modelValue'])

  const value = computed({
    get() {
      return props.modelValue
    },
    set(value) {
      emit('update:modelValue', value)
    }
  })
  </script>

  <template>
    <input v-model="value" />
  </template>
  ```

##### （3）v-model的参数

- 默认情况下，v-model在组件上都是使用 `modelValue` 作为 prop，并以 `update:modelValue` 作为对应的事件。我们可以通过给 `v-model` 指定一个参数来更改这些名字：

  ```vue
  // 父组件
  <MyComponent v-model:title="bookTitle" />

  // 子组件
  <script setup>
  defineProps(['title'])
  defineEmits(['update:title'])
  </script>

  <template>
    <input
      type="text"
      :value="title"
      @input="$emit('update:title', $event.target.value)"
    />
  </template>
  ```

##### （4）处理`v-model`修饰符

- 组件的 `v-model` 上所添加的修饰符，可以通过 `modelModifiers` prop 在组件内访问到，它的默认值是一个空对象

  ```js
  // 父组件
  <MyComponent v-model.capitalize="myText" />
    
  // 子组件
  <script setup>
  const props = defineProps({
    modelValue: String,
    modelModifiers: { default: () => ({}) }
  })

  const emit = defineEmits(['update:modelValue'])

  function emitValue(e) {
    let value = e.target.value
    if (props.modelModifiers.capitalize) {
      value = value.charAt(0).toUpperCase() + value.slice(1)
    }
    emit('update:modelValue', value)
  }
  </script>

  <template>
    <input type="text" :value="modelValue" @input="emitValue" />
  </template>
  ```


- **注意：对于又有参数又有修饰符的 `v-model` 绑定，生成的 prop 名将是 `arg + "Modifiers"`**

  ```vue
  // 父组件
  <MyComponent v-model:title.capitalize="myText">
    
  // 子组件
  const props = defineProps(['title', 'titleModifiers'])
  defineEmits(['update:title'])

  console.log(props.titleModifiers) // { capitalize: true }
  ```

#### 5.透传Attributes

##### （1）Attributes继承

- **透传attribute**指的是传递给一个组件，却没有被该组件声明为`props`或`emits`的attribute或者`v-on`事件监听器，常见的是`class，style，id`


- 当一个组件以单个元素作为根作渲染时，透传的attribute会自动被添加到根元素上

  ```vue
  <!-- <MyButton> 的模板 -->
  <button>click me</button>

  <MyButton class="large" />

  // 最后结果
  <button class="large">click me</button>
  ```


- 如果一个子组件的根元素已经有了 `class` 或 `style` attribute，它会和从父组件上继承的值合并

  ```vue
  <!-- <MyButton> 的模板 -->
  <button class="btn">click me</button>

  <MyButton class="large" />

  // 最后结果
  <button class="btn large">click me</button>
  ```


- **v-on**监听器继承也有同样的规则

  ```vue
  <MyButton @click="onClick" />
  ```


- 深层组件继承

  ```vue
  <!-- <MyButton/> 的模板，只是渲染另一个组件 -->
  <BaseButton />
  ```

  此时 `<MyButton>` 接收的透传 attribute 会直接继续传给 `<BaseButton>`。

  请注意：

  1. 透传的 attribute 不会包含 `<MyButton>` 上声明过的 props 或是针对 `emits` 声明事件的 `v-on` 侦听函数，换句话说，声明过的 props 和侦听函数被 `<MyButton>`“消费”了。
  2. 透传的 attribute 若符合声明，也可以作为 props 传入 `<BaseButton>`。



##### （2）禁用Attributes继承

- 在组件选项中设置 `inheritAttrs: false`关闭attribute继承

- 如果你使用了 `<script setup>`，你需要一个额外的 `<script>` 块来书写这个选项声明

  ```js
  <script>
  // 使用普通的 <script> 来声明选项
  export default {
    inheritAttrs: false
  }
  </script>

  <script setup>
  // ...setup 部分逻辑
  </script>
  ```


- 最常见的需要禁用 attribute 继承的场景就是 attribute 需要应用在根节点以外的其他元素上。通过设置 `inheritAttrs` 选项为 `false`，你可以完全控制透传进来的 attribute 被如何使用。

- 这些透传进来的 attribute 可以在模板的表达式中直接用 `$attrs` 访问到

  ```vue
  <div class="btn-wrapper">
    <button class="btn" v-bind="$attrs">click me</button>
  </div>
  ```


- 这个 `$attrs` 对象包含了除组件所声明的 `props` 和 `emits` 之外的所有其他 attribute，例如 `class`，`style`，`v-on` 监听器等等
- **注意：**
  - 和 props 有所不同，透传 attributes 在 JavaScript 中保留了它们原始的大小写，所以像 `foo-bar` 这样的一个 attribute 需要通过 `$attrs['foo-bar']` 来访问。
  - 像 `@click` 这样的一个 `v-on` 事件监听器将在此对象下被暴露为一个函数 `$attrs.onClick`。



##### （3）多根节点的Attributes继承

- 和单根节点组件有所不同，有着多个根节点的组件没有自动attribute透传行为。如果 `$attrs` 没有被显式绑定，将会抛出一个运行时警告

  ```vue
  <CustomLayout id="custom-layout" @click="changeValue" />
  ```

- 如果 `<CustomLayout>` 有下面这样的多根节点模板，由于 Vue 不知道要将 attribute 透传到哪里，所以会抛出一个警告

  ```vue
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
  ```


- 如果 `$attrs` 被显式绑定，则不会有警告

  ```vue
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
  ```



##### （4）在JavaScript中访问透传Attributes

- 可以在 `<script setup>` 中使用 `useAttrs()` API 来访问一个组件的所有透传 attribute

  ```js
  <script setup>
  import { useAttrs } from 'vue'

  const attrs = useAttrs()
  </script>
  ```


- 如果没有使用 `<script setup>`，`attrs` 会作为 `setup()` 上下文对象的一个属性暴露

  ```js
  export default {
    setup(props, ctx) {
      // 透传 attribute 被暴露为 ctx.attrs
      console.log(ctx.attrs)
    }
  }
  ```


- **注意：**虽然这里的 `attrs` 对象总是反映为最新的透传 attribute，但它并不是响应式的 (考虑到性能因素)。你不能通过侦听器去监听它的变化。如果你需要响应性，可以使用 prop。或者你也可以使用 `onUpdated()` 使得在每次更新时结合最新的 `attrs` 执行副作用



### 十一、依赖注入

- 通常情况下，当我们需要从父组件向子组件传递数据时，会使用 `props`。想象一下这样的结构：有一些多层级嵌套的组件，形成了一颗巨大的组件树，而某个深层的子组件需要一个较远的祖先组件中的部分数据。在这种情况下，如果仅使用 props 则必须将其沿着组件链逐级传递下去，这会非常麻烦
- `provide` 和 `inject` 可以帮助我们解决这一问题。 一个父组件相对于其所有的后代组件，会作为**依赖提供者**。任何后代的组件树，无论层级有多深，都可以**注入**由父组件提供给整条链路的依赖。

#### 1.Provide（提供）

- 为后代提供数据，需要使用到`provide（）`函数

  ```js
  <script setup>
  import { provide } from 'vue'

  provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
  </script>
  ```


- 如果不使用 `<script setup>`，请确保 `provide()` 是在 `setup()` 同步调用的

  ```js
  import { provide } from 'vue'

  export default {
    setup() {
      provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
    }
  }
  ```


- `provide()` 函数接收两个参数。

  - 第一个参数被称为**注入名**，可以是一个字符串或是一个 `Symbol`。后代组件会用注入名来查找期望注入的值。一个组件可以多次调用 `provide()`，使用不同的注入名，注入不同的依赖值。

  - 第二个参数是提供的值，值可以是任意类型，包括响应式的状态，比如一个 ref

    ```js
    import { ref, provide } from 'vue'

    const count = ref(0)
    provide('key', count)
    ```

#### 2.应用层Provide

- 除了在一个组件中提供依赖，我们还可以在整个应用层面提供依赖

  ```js
  import { createApp } from 'vue'

  const app = createApp({})

  app.provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
  ```


- 在应用级别提供的数据在该应用内的所有组件中都可以注入

#### 3.Inject（注入）

- 要注入上册组件提供的数据，需要使用`inject（）`函数

  ```js
  <script setup>
  import { inject } from 'vue'

  const message = inject('message')
  </script>
  ```


- 如果提供的值是一个 ref，注入进来的会是该 ref 对象，而**不会**自动解包为其内部的值。这使得注入方组件能够通过 ref 对象保持了和供给方的响应性链接

- 如果没有使用 `<script setup>`，`inject()` 需要在 `setup()` 内同步调用

  ```js
  import { inject } from 'vue'

  export default {
    setup() {
      const message = inject('message')
      return { message }
    }
  }
  ```

#### 4.注入默认值

- 默认情况下，`inject` 假设传入的注入名会被某个祖先链上的组件提供。如果该注入名的确没有任何组件提供，则会抛出一个运行时警告

- 如果在注入一个值时不要求必须有提供者，那么我们应该声明一个默认值，和 props 类似

  ```js
  const value = inject('message', '这是默认值')
  ```


- 在一些场景中，默认值可能需要通过调用一个函数或初始化一个类来取得。为了避免在用不到默认值的情况下进行不必要的计算或产生副作用，我们可以使用工厂函数来创建默认值

  ```js
  const value = inject('key', () => new ExpensiveClass())
  ```

#### 5.和响应式数据配合使用

- 当提供 / 注入响应式的数据时，**建议尽可能将任何对响应式状态的变更都保持在供给方组件中**。这样可以确保所提供状态的声明和变更操作都内聚在同一个组件内，使其更容易维护

- 有的时候，我们可能需要在注入方组件中更改数据。在这种情况下，我们推荐在供给方组件内声明并提供一个更改数据的方法函数

  ```js
  <!-- 在供给方组件内 -->
  <script setup>
  import { provide, ref } from 'vue'

  const location = ref('North Pole')

  function updateLocation() {
    location.value = 'South Pole'
  }

  provide('location', {
    location,
    updateLocation
  })
  </script>
  ```

  ```js
  <!-- 在注入方组件 -->
  <script setup>
  import { inject } from 'vue'

  const { location, updateLocation } = inject('location')
  </script>

  <template>
    <button @click="updateLocation">{{ location }}</button>
  </template>
  ```


- 如果你想确保提供的数据不能被注入方的组件更改，你可以使用 `readonly`来包装提供的值

  ```js
  <script setup>
  import { ref, provide, readonly } from 'vue'

  const count = ref(0)
  provide('read-only-count', readonly(count))
  </script>
  ```



#### 6.使用Symbol作为注入名

- 如果你正在构建大型的应用，包含非常多的依赖提供，或者你正在编写提供给其他开发者使用的组件库，建议最好使用 Symbol 来作为注入名以避免潜在的冲突

- 我们通常推荐在一个单独的文件中导出这些注入名 Symbol

  ```js
  // keys.js
  export const myInjectionKey = Symbol()
  ```

  ```js
  // 在供给方组件中
  import { provide } from 'vue'
  import { myInjectionKey } from './keys.js'

  provide(myInjectionKey, { /*
    要提供的数据
  */ });
  ```

  ```js
  // 注入方组件
  import { inject } from 'vue'
  import { myInjectionKey } from './keys.js'

  const injected = inject(myInjectionKey)
  ```



### 十二、异步组件

#### 1.基本用法

- 在大型项目中，我们可能需要拆分应用为更小的块，并仅在需要时再从服务器加载相关组件。这时需要使用异步组件，Vue 提供了`defineAsyncComponent`方法来实现此功能

  ```js
  import { defineAsyncComponent } from 'vue'

  const AsyncComp = defineAsyncComponent(() => {
    return new Promise((resolve, reject) => {
      // ...从服务器获取组件
      resolve(/* 获取到的组件 */)
    })
  })
  // ... 像使用其他一般组件一样使用 `AsyncComp`
  ```


- `defineAsyncComponent` 方法接收一个返回 Promise 的加载函数。这个 Promise 的 `resolve` 回调方法应该在从服务器获得组件定义时调用。你也可以调用 `reject(reason)` 表明加载失败

- ES模块动态导入也会返回一个 Promise，所以多数情况下我们会将它和 `defineAsyncComponent` 搭配使用。类似 Vite 和 Webpack 这样的构建工具也支持此语法 (并且会将它们作为打包时的代码分割点)，因此我们也可以用它来导入 Vue 单文件组件

  ```js
  import { defineAsyncComponent } from 'vue'

  const AsyncComp = defineAsyncComponent(() =>
    import('./components/MyComponent.vue')
  )
  ```


- 与普通组件一样，异步组件可以使用 `app.component()` 全局注册

  ```js
  app.component('MyComponent', defineAsyncComponent(() =>
    import('./components/MyComponent.vue')
  ))
  ```


- 也可以直接在父组件中定义

  ```js
  <script setup>
  import { defineAsyncComponent } from 'vue'

  const AdminPage = defineAsyncComponent(() =>
    import('./components/AdminPageComponent.vue')
  )
  </script>

  <template>
    <AdminPage />
  </template>
  ```



#### 2.加载与错误状态

- 异步操作不可避免地会涉及到加载和错误状态，因此 `defineAsyncComponent()` 也支持在高级选项中处理这些状态

  ```js
  const AsyncComp = defineAsyncComponent({
    // 加载函数
    loader: () => import('./Foo.vue'),

    // 加载异步组件时使用的组件
    loadingComponent: LoadingComponent,
    // 展示加载组件前的延迟时间，默认为 200ms
    delay: 200,

    // 加载失败后展示的组件
    errorComponent: ErrorComponent,
    // 如果提供了一个 timeout 时间限制，并超时了
    // 也会显示这里配置的报错组件，默认值是：Infinity
    timeout: 3000
  })
  ```


- 如果提供了一个加载组件，它将在内部组件加载时先行显示。在加载组件显示之前有一个默认的 200ms 延迟——这是因为在网络状况较好时，加载完成得很快，加载组件和最终组件之间的替换太快可能产生闪烁，反而影响用户感受。
- 如果提供了一个报错组件，则它会在加载器函数返回的 Promise 抛错时被渲染。你还可以指定一个超时时间，在请求耗时超过指定时间时也会渲染报错组件。



#### 3.搭配Suspense使用

- 异步组件可以搭配内置的 `<Suspense>` 组件一起使用，若想了解 `<Suspense>` 和异步组件之间交互


### 十三、自定义指令

#### 1.指令介绍

- Vue 允许注册自定义的指令。一个自定义指令由一个包含类似组件生命周期钩子的对象来定义。钩子函数会接收到指令所绑定元素作为其参数

  ```js
  <script setup>
  // 在模板中启用 v-focus
  const vFocus = {
    mounted: (el) => el.focus()
  }
  </script>

  <template>
    <input v-focus />
  </template>
  ```

- 在 `<script setup>` 中，任何以 `v` 开头的驼峰式命名的变量都可以被用作一个自定义指令。

- 在没有使用 `<script setup>` 的情况下，自定义指令需要通过 `directives` 选项注册：

  ```js
  export default {
    setup() {
      /*...*/
    },
    directives: {
      // 在模板中启用 v-focus
      focus: {
        /* ... */
      }
    }
  }
  ```


- 也可以注册全局指令

  ```js
  const app = createApp({})

  // 使 v-focus 在所有组件中都可用
  app.directive('focus', {
    /* ... */
  })
  ```

#### 2.指令钩子函数

- 一个指令的定义对象可以提供几种钩子函数 (都是可选的)

  ```js
  const myDirective = {
    // 在绑定元素的 attribute 前
    // 或事件监听器应用前调用
    created(el, binding, vnode, prevVnode) {
      // 下面会介绍各个参数的细节
    },
    // 在元素被插入到 DOM 前调用
    beforeMount(el, binding, vnode, prevVnode) {},
    // 在绑定元素的父组件
    // 及他自己的所有子节点都挂载完成后调用
    mounted(el, binding, vnode, prevVnode) {},
    // 绑定元素的父组件更新前调用
    beforeUpdate(el, binding, vnode, prevVnode) {},
    // 在绑定元素的父组件
    // 及他自己的所有子节点都更新后调用
    updated(el, binding, vnode, prevVnode) {},
    // 绑定元素的父组件卸载前调用
    beforeUnmount(el, binding, vnode, prevVnode) {},
    // 绑定元素的父组件卸载后调用
    unmounted(el, binding, vnode, prevVnode) {}
  }
  ```

#### 3.钩子参数

- 指令的钩子会传递以下几种参数：

  - `el`：指令绑定到的元素。这可以用于直接操作 DOM。
  - `binding`：一个对象，包含以下属性。
    - `value`：传递给指令的值。例如在 `v-my-directive="1 + 1"` 中，值是 `2`。
    - `oldValue`：之前的值，仅在 `beforeUpdate` 和 `updated` 中可用。无论值是否更改，它都可用。
    - `arg`：传递给指令的参数 (如果有的话)。例如在 `v-my-directive:foo` 中，参数是 `"foo"`。
    - `modifiers`：一个包含修饰符的对象 (如果有的话)。例如在 `v-my-directive.foo.bar` 中，修饰符对象是 `{ foo: true, bar: true }`。
    - `instance`：使用该指令的组件实例。
    - `dir`：指令的定义对象。
  - `vnode`：代表绑定元素的底层 VNode。
  - `prevNode`：之前的渲染中代表指令所绑定元素的 VNode。仅在 `beforeUpdate` 和 `updated` 钩子中可用。

  ```vue
  <div v-example:foo.bar="baz">
    
  // binding
  {
    arg: 'foo',
    modifiers: { bar: true },
    value: /* `baz` 的值 */,
    oldValue: /* 上一次更新时 `baz` 的值 */
  }
  ```


- 和内置指令类似，自定义指令的参数也可以是动态的

  ```vue
  <div v-example:[arg]="value"></div>
  ```


- **简化形式：**对于自定义指令来说，一个很常见的情况是仅仅需要在 `mounted` 和 `updated` 上实现相同的行为，除此之外并不需要其他钩子。这种情况下我们可以直接用一个函数来定义指令

  ```vue
  <div v-color="color"></div>

  app.directive('color', (el, binding) => {
    // 这会在 `mounted` 和 `updated` 时都调用
    el.style.color = binding.value
  })
  ```


- 如果你的指令需要多个值，你可以向它传递一个 JavaScript 对象字面量。指令也可以接收任何合法的 JavaScript 表达式

  ```vue
  <div v-demo="{ color: 'white', text: 'hello!' }"></div>

  app.directive('demo', (el, binding) => {
    console.log(binding.value.color) // => "white"
    console.log(binding.value.text) // => "hello!"
  })
  ```

- 当在组件上使用自定义指令时，它会始终应用于组件的根节点，和透传attributes类似

  ```
  <MyComponent v-demo="test" />
  <!-- MyComponent 的模板 -->

  <div> <!-- v-demo 指令会被应用在此处 -->
    <span>My component content</span>
  </div>
  ```


- **注意：**组件可能含有多个根节点。当应用到一个多根组件时，指令将会被忽略且抛出一个警告。和 attribute 不同，指令不能通过 `v-bind="$attrs"` 来传递给一个不同的元素。总的来说，**不**推荐在组件上使用自定义指令。



### 十四、KeepAlive

- `<KeepAlive>` 是一个内置组件，它的功能是在多个组件间动态切换时缓存被移除的组件实例

  ```vue
  <!-- 非活跃的组件将会被缓存！ -->
  <KeepAlive>
    <component :is="activeComponent" />
  </KeepAlive>
  ```

#### 1.包含/排除

- `<KeepAlive>` 默认会缓存内部的所有组件实例，但我们可以通过 `include` 和 `exclude` prop 来定制该行为。这两个 prop 的值都可以是一个以英文逗号分隔的字符串、一个正则表达式，或是包含这两种类型的一个数组

  ```vue
  <!-- 以英文逗号分隔的字符串 -->
  <KeepAlive include="a,b">
    <component :is="view" />
  </KeepAlive>

  <!-- 正则表达式 (需使用 `v-bind`) -->
  <KeepAlive :include="/a|b/">
    <component :is="view" />
  </KeepAlive>

  <!-- 数组 (需使用 `v-bind`) -->
  <KeepAlive :include="['a', 'b']">
    <component :is="view" />
  </KeepAlive>
  ```


- **注意：**它会根据组件的`name`选项进行匹配，所以组件如果想要条件性地被 `KeepAlive` 缓存，就必须显式声明一个 `name` 选项

#### 2.最大缓存实例数

- 可以通过传入 `max` prop 来限制可被缓存的最大组件实例数。`<KeepAlive>` 的行为在指定了 `max` 后类似一个LRU缓存：如果缓存的实例数量即将超过指定的那个最大数量，则最久没有被访问的缓存实例将被销毁，以便为新的实例腾出空间

  ```vue
  <KeepAlive :max="10">
    <component :is="activeComponent" />
  </KeepAlive>
  ```

#### 3.缓存实例的生命周期

- 当一个组件实例从 DOM 上移除但因为被 `<KeepAlive>` 缓存而仍作为组件树的一部分时，它将变为**不活跃**状态而不是被卸载。当一个组件实例作为缓存树的一部分插入到 DOM 中时，它将重新**被激活**。

- 一个持续存在的组件可以通过`onActivated（）`和`onDeactivated（）`注册相应的两个状态的生命周期钩子

  ```js
  <script setup>
  import { onActivated, onDeactivated } from 'vue'

  onActivated(() => {
    // 调用时机为首次挂载
    // 以及每次从缓存中被重新插入时
  })

  onDeactivated(() => {
    // 在从 DOM 上移除、进入缓存
    // 以及组件卸载时调用
  })
  </script>
  ```


- **注意：**
  - `onActivated` 在组件挂载时也会调用，并且 `onDeactivated` 在组件卸载时也会调用。
  - 这两个钩子不仅适用于 `<KeepAlive>` 缓存的根组件，也适用于缓存树中的后代组件

### 十五、Suspense

- `<Suspense>` 是一个内置组件，用来在组件树中协调对异步依赖的处理。它让我们可以在组件树上层等待下层的多个嵌套异步依赖项解析完成，并可以在等待时渲染一个加载状态

- `<Suspense>` 可以等待的异步依赖有两种：

  - 带有异步 `setup()` 钩子的组件。这也包含了使用 `<script setup>` 时有顶层 `await` 表达式的组件。

    ```js
    export default {
      async setup() {
        const res = await fetch(...)
        const posts = await res.json()
        return {
          posts
        }
      }
    }

    <script setup>
    const res = await fetch(...)
    const posts = await res.json()
    </script>

    <template>
      {{ posts }}
    </template>
    ```

  - 异步组件：异步组件默认就是**“suspensible”**的。这意味着如果组件关系链上有一个 `<Suspense>`，那么这个异步组件就会被当作这个 `<Suspense>` 的一个异步依赖。在这种情况下，加载状态是由 `<Suspense>` 控制，而该组件自己的加载、报错、延时和超时等选项都将被忽略。异步组件也可以通过在选项中指定 `suspensible: false` 表明不用 `Suspense` 控制，并让组件始终自己控制其加载状态。

#### 1.加载中状态

- `<Suspense>` 组件有两个插槽：`#default` 和 `#fallback`。两个插槽都只允许**一个**直接子节点。在可能的时候都将显示默认槽中的节点。否则将显示后备槽中的节点

  ```vue
  <Suspense>
    <!-- 具有深层异步依赖的组件 -->
    <Dashboard />

    <!-- 在 #fallback 插槽中显示 “正在加载中” -->
    <template #fallback>
      Loading...
    </template>
  </Suspense>
  ```


- 在初始渲染时，`<Suspense>` 将在内存中渲染其默认的插槽内容。如果在这个过程中遇到任何异步依赖，则会进入**挂起**状态。在挂起状态期间，展示的是后备内容。当所有遇到的异步依赖都完成后，`<Suspense>` 会进入**完成**状态，并将展示出默认插槽的内容。
- 如果在初次渲染时没有遇到异步依赖，`<Suspense>` 会直接进入完成状态。
- 进入完成状态后，只有当默认插槽的根节点被替换时，`<Suspense>` 才会回到挂起状态。组件树中新的更深层次的异步依赖**不会**造成 `<Suspense>` 回退到挂起状态。
- 发生回退时，后备内容不会立即展示出来。相反，`<Suspense>` 在等待新内容和异步依赖完成时，会展示之前 `#default` 插槽的内容。这个行为可以通过一个 `timeout` prop 进行配置：在等待渲染新内容耗时超过 `timeout` 之后，`<Suspense>` 将会切换为展示后备内容。若 `timeout` 值为 `0` 将导致在替换默认内容时立即显示后备内容。

#### 2.事件

- `<Suspense>` 组件会触发三个事件：`pending`、`resolve` 和 `fallback`。
  - `pending` 事件是在进入挂起状态时触发。
  - `resolve` 事件是在 `default` 插槽完成获取新内容时触发。
  - `fallback` 事件则是在 `fallback` 插槽的内容显示时触发。

#### 3.错误处理

- `<Suspense>` 组件自身目前还不提供错误处理，不过你可以使用 `errorCaptured`选项或者 `onErrorCaptured（）`钩子，在使用到 `<Suspense>` 的父组件中捕获和处理异步错误。

### 十六、API

#### 1.应用实例API

##### （1）createApp()

- 创建一个应用实例

