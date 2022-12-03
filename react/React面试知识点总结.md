## React面试知识点总结

### 一、React使用

#### 1.React事件为何bind this

- 原因：react中的dom是虚拟DOM，JSX是`React.createElement(component, props, ...children)` 的语法糖，自`onClick = {function}` 中的`onClick`本身就是一个"中间变量", `this.handleClick`又作为一个callback传给另一个函数, 这时候this就丢失了

- react合成事件：

  - 所有事件挂载到document上

  - event不是原生的，是合成事件对象

  - 和Vue以及DOM事件不同

    ```js
    event.preventDefault() // 阻止默认行为
    event.stopPropagation() // 阻止冒泡
    console.log('target', event.target) // 指向当前元素，即当前元素触发
    console.log('current target', event.currentTarget) // 指向当前元素，假象！！！

    // 注意，event 其实是 React 封装的。可以看 __proto__.constructor 是 SyntheticEvent 组合事件
    console.log('event', event) // 不是原生的 Event ，原生的 MouseEvent
    console.log('event.__proto__.constructor', event.__proto__.constructor)

    // 原生 event 如下。其 __proto__.constructor 是 MouseEvent
    console.log('nativeEvent', event.nativeEvent)
    console.log('nativeEvent target', event.nativeEvent.target)  // 指向当前元素，即当前元素触发
    console.log('nativeEvent current target', event.nativeEvent.currentTarget) // 指向 document ！！！

    // 1. event 是 SyntheticEvent ，模拟出来 DOM 事件所有能力
    // 2. event.nativeEvent 是原生事件对象
    // 3. 所有的事件，都被挂载到 document 上
    // 4. 和 DOM 事件不一样，和 Vue 事件也不一样
    ```


- 为什么要使用合成事件机制
  - 更好的兼容性和跨平台
  - 挂载到document上，减少内存消耗，避免频繁解绑
  - 方便事件的统一管理



#### 2.React事件系统（ React 事件和 DOM 事件的区别）

- React的事件系统沿袭了事件委托的思想。在 React 中，除了少数特殊的不可冒泡的事件（比如媒体类型的事件）无法被事件系统处理外，绝大部分的事件都不会被绑定在具体的元素上，而是统一被绑定在页面的 document 上。**当事件在具体的 DOM 节点上被触发后，最终都会冒泡到 document 上，document 上所绑定的统一事件处理程序会将事件分发到具体的组件实例**。

- **注意：在react17以后，绝大部分的事件都不会被绑定在具体的元素上，而是统一被绑定在页面的 root节点上，而不是document**

  ------

##### （1）React合成事件

- 合成事件是React自定义的事件对象，他符合W3C规范，在顶层磨平了不同浏览器的差异，**在上层面向开发者暴露统一的、稳定的、与 DOM 原生事件相同的事件接口**。开发者们由此便不必再关注烦琐的兼容性问题，可以专注于业务逻辑的开发。

  **虽然合成事件并不是原生 DOM 事件，但它保存了原生 DOM 事件的引用**。当你需要访问原生 DOM 事件对象时，可以通过合成事件对象的 **e.nativeEvent** 属性获取到它

##### （2）React事件工作流

- **事件的绑定**

  - 事件的绑定是在组建的挂载阶段完成的，具体来说，**是在 completeWork 中完成的** ，最终注册到 document 上的是一个**统一的事件分发函数** ，**其实就是 dispatchEvent**。

    > completeWork 内部有三个关键动作：**创建** DOM 节点（createInstance）、将 DOM 节点**插入**到 DOM 树中（appendAllChildren）、为 DOM 节点**设置属性**（finalizeInitialChildren）。


- **事件的触发**
  - 事件触发的本质是对 dispatchEvent 函数的调用


- **事件回调的收集与执行**
  - **循环收集符合条件的父节点，存进 path 数组中**
  - **模拟事件在捕获阶段的传播顺序，收集捕获阶段相关的节点实例与回调函数**
  - **模拟事件在冒泡阶段的传播顺序，收集冒泡阶段相关的节点实例与回调函数**



#### 3.手写发布订阅模式**

```js
class myEventEmitter {
  constructor() {
    // eventMap 用来存储事件和监听函数之间的关系
    this.eventMap = {};
  }
  // type 这里就代表事件的名称
  on(type, handler) {
    // hanlder 必须是一个函数，如果不是直接报错
    if (!(handler instanceof Function)) {
      throw new Error("哥 你错了 请传一个函数");
    }
    // 判断 type 事件对应的队列是否存在
    if (!this.eventMap[type]) {
      // 若不存在，新建该队列
      this.eventMap[type] = [];
    }
    // 若存在，直接往队列里推入 handler
    this.eventMap[type].push(handler);
  }
  // 别忘了我们前面说过触发时是可以携带数据的，params 就是数据的载体
  emit(type, params) {
    // 假设该事件是有订阅的（对应的事件队列存在）
    if (this.eventMap[type]) {
      // 将事件队列里的 handler 依次执行出队
      this.eventMap[type].forEach((handler, index) => {
        // 注意别忘了读取 params
        handler(params);
      });
    }
  }
  off(type, handler) {
    if (this.eventMap[type]) {
      this.eventMap[type].splice(this.eventMap[type].indexOf(handler) >>> 0, 1);
    }
  }
}
```



#### 4.Why React Hooks

- **告别难以理解的Class：**类组件有两个难点：this、生命周期，增加了学习成本，且生命周期具有不合理的逻辑规划方式
- **解决业务逻辑难以拆分的问题：**在类组件中，逻辑曾经一度与生命周期耦合在一起，在生命周期函数之中，我们会写许多个事件逻辑，这些事情之间看上去毫无关联，逻辑就像是被打散到生命周期里了一样；而Hooks可以按照逻辑上的关联划分进不同的函数组件李，我们可以在专门管理订阅的函数组件、专门处理DOM的函数组件、专门获取数据的函数组件等，Hooks能够帮助我们实现业务逻辑的耦合，避免复杂的组件和冗余的代码
- **使状态逻辑复用变得简单可行：**我们复用状态逻辑，经常使用HOC或者是Render Props这些组件设计模式，Hooks可以视作是React为解决状态逻辑复用这个问题所提供的一个原生途径，可以在既不破坏组件结构、又能够实现逻辑复用的效果
- **函数组件从设计思想上来看，更加契合React的理念：**React有一个公式 `UI = render(data) 或 UI = f(data)`




#### 5.React事件系统的设计动机是什么？

- 合成事件符合W3C规范，在底层磨平了不同浏览器的差异，在上层面向开发者暴露统一的、稳定的、与DOM原生事件相同的事件接口。开发者因此便不必在关注繁琐的底层兼容问题，可以专注于业务逻辑的开发
- 自研事件系统使React牢牢把握住了事件处理的主动权，通过自研事件系统，React能够从很大程度上干预事件的表现，使其符合自身的需求（在事件系统中处理 Fiber 相关的优先级）


- 挂载到document，减少内存消耗（17之前）



#### 6.React表单知识点串讲

- 在React中，表单会是可控组件，由内部的state控制

  ```react
  class NameForm extends React.Component {
    constructor(props) {
      super(props);
      this.state = {value: ''};
      this.handleChange = this.handleChange.bind(this);
      this.handleSubmit = this.handleSubmit.bind(this);
    }
    
    handleChange(event) {
      this.setState({value: event.target.value});
    }
    
    handleSubmit(event) {
      alert('A name was submitted: ' + this.state.value);
      event.preventDefault();
    }
    
    render() {
      return (
        <form onSubmit={this.handleSubmit}>
        	<span>名字:</span>
          <input type="text" value={this.state.value} onChange={this.handleChange} />
          <input type="submit" value="Submit" />
        </form>
      );
    }
  }
  ```


- 在React中，select标签并不适用selected属性，而是在根select标签上使用value属性

  ```react
  class FlavorForm extends React.Component {
    constructor(props) {
      super(props);
      this.state = {value: 'coconut'};

      this.handleChange = this.handleChange.bind(this);
      this.handleSubmit = this.handleSubmit.bind(this);
    }

    handleChange(event) {
      this.setState({value: event.target.value});
    }

    handleSubmit(event) {
      alert('你喜欢的风味是: ' + this.state.value);
      event.preventDefault();
    }

    render() {
      return (
        <form onSubmit={this.handleSubmit}>
            <span>选择你喜欢的风味:</span>
            <select value={this.state.value} onChange={this.handleChange}>
              <option value="grapefruit">葡萄柚</option>
              <option value="lime">酸橙</option>
              <option value="coconut">椰子</option>
              <option value="mango">芒果</option>
            </select>
          <input type="submit" value="提交" />
        </form>
      );
    }
  }
  ```



#### 7.React父子组件通讯

##### （1）父 — 子

- **props：在将父组件的state作为子组件的props传入实现父向子之间的通信**

  ```react
  const Son = ({ name }) => {
      return <div>{name}</div>;
  };

  const Father = () => {
      const [name, setName] = useState('Jack');
      return (
          <>
              <Son name={name} />
          </>
      );
  };
  ```


- **createRef：**父组件通过React.createRef()创建Ref，保存在实例属性myRef上。父组件中，渲染子组件时，定义一个Ref属性，值为刚创建的myRef。父组件调用子组件的myFunc函数，传递一个参数，子组件接收到参数，打印出参数。参数从父组件传递给子组件，完成了父组件向子组件通信。

  ```react
  class Son extends Component {
      myFunc(name) {
          console.log(name);
      }
      render() {
          return <></>;
      }
  }

  // 父组件
  export default class Father extends Component {
      constructor(props) {
          super(props);
          // 创建Ref，并保存在实例属性myRef上
          this.myRef = React.createRef();
      }

      componentDidMount() {
          // 调用子组件的函数，传递一个参数
          this.myRef.current.myFunc('Jack');
      }
      render() {
          return (
              <>
                  <Son ref={this.myRef} />
              </>
          );
      }
  }
  ```

##### （2）子 — 父

- **props：**父组件在自身state内声明一个函数作为子组件的props传入，在子组件内部触发这个函数将想要传递的值作为参数传入并在父组件内接收

  ```react
  const Son = ({ setCount }) => {
      return <button onClick={() => setCount(count => count + 1)}>点击+1</button>;
  };

  const Father = () => {
      const [count, setCount] = useState(0);
      return (
          <>
              <div>计数值：{count}</div>
              <Son setCount={setCount} />
          </>
      );
  };
  ```


- **事件冒泡：**利用事件冒泡**机制，点击子组件的`button`按钮，事件会冒泡到父组件身上，触发父组件的`onClick`函数，打印出`Jack`。点击的是子组件，父组件执行函数，完成了子组件向父组件通信

  ```react
  const Son = () => {
      return <button>点击</button>;
  };

  const Father = () => {
      const sayName = name => {
          console.log(name);
      };
      return (
          <div onClick={() => sayName('Jack')}>
              <Son />
          </div>
      );
  };
  ```



#### 8.setState为何使用不可变值



#### 9.setState是同步还是异步

- setState本身并不具备绝对的同步/异步概念，可能是同步也可能是异步，react会有一个上下文环境，在同步环境中，setState处于react的上下文中，react会监控动作合并，所以setState()是异步的。而在异步环境中，react实际上已经脱离了react的上下文环境。所以setState()是同步执行的
- setState()在react中一开始之所以被设定为异步触发，是因为每当触发setState时react需要去更新视图。通过异步更新视图来合并在短时间内的大量setState动作。这样，当我们在短时间内多次的调用setState时，视图只会被更新一次


- 总结：
  - setState 只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的。
  - setState的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。
  - setState 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次 setState ， setState 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 setState 多个不同的值，在更新时会对其进行合并批量更新。



#### 10.setState何时会合并state

- setstate 异步更新，多次setstate函数调用产生的效果会合并



#### 11.什么场景需要用React Portals

- Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案
- 一个 portal 的典型用例是当父组件有 `overflow: hidden` 或 `z-index` 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框



### 二、Redux

#### 1.redux概念

- **什么是redux：**redux是一个管理状态（数据）的容器，提供了可预测的状态管理
- **什么是可预测的状态管理：**数据在什么时候，因为什么，发生了什么改变，都是可以控制和追踪的，就称之为可预测的状态管理
- **为什么要使用redux：**
  - React是通过数据驱动视图更新的，React负责更新界面，而Redux负责管理数据
  - 默认情况下我们可以在每个组件中管理自己的状态，但是现在前端应用程序变得越来越复杂，状态之间可能存在依赖管理，一个状态的变化会引起另一个状态的改变
  - 因此在应用程序复杂的时候，状态在很么时候改变，因为什么改变，发生了什么改变，就会变得非常难以控制和追踪，此时可以使用Redux


- **Redux核心概念：**
  - 通过store来保存数据
  - 通过action来修改数据
  - 通过reducer来关联store和action

#### 2.Redux三大原则

- **单一数据源：**
  - 整个应用程序的state只存储在一个store中
  - Redux并没有强制让我们不能创造多个store，但是那样做并不利于数据的维护
  - 单一的数据源可以让整个应用程序的state变得方便维护、追踪、修改


- **state是只读的：**
  - 唯一修改state的方法银锭是触发action，不要试图在其他地方通过任何的方式来修改state
  - 这样就确保了View或网络请求都不能直接修改state，它们只能通过action来描述自己想要如何修改stat；
  - 这样可以保证所有的修改都被集中化处理，并且按照严格的顺序来执行，所以不需要担心race condition（竟态）的问题


- **使用纯函数来执行修改：**
  - 通过reducer将 旧state和 action联系在一起，并且返回一个新的State：
  - 随着应用程序的复杂度增加，我们可以将reducer拆分成多个小的reducers，分别操作不同state tree的一部分
  - 但是所有的reducer都应该是纯函数，不能产生任何的副作用

#### 3.Redux单向数据流

![](https://img-blog.csdnimg.cn/img_convert/62b8178d34a127464fcd9ceb9605a131.png)

![](https://img-blog.csdnimg.cn/img_convert/ee69423219f1c0e69427984ec299c853.png#pic_center)

#### 4.React Redux异步处理Action方案

- redux默认处理不了异步action，在redux中使用异步action需要使用中间件



#### 5.redux中间件

- 什么是redux中间件：redux提供了类似后端express的中间件概念，本质的目的是提供第三方插件的模式，自定义拦截action-reducer的过程，变为action -> middlewares -> reducer 。这种机制可以让我们改变数据流，实现如异步 action ，action 过滤，日志输出，异常报告等功能。
- 通俗来说，redux中间件就是对dispatch的功能做了扩展。

![](https://img.kancloud.cn/04/5b/045bab4731ce9f5fb0b725934ba4a1fe_1116x542.png)

### 三、react原理

#### 1.不可变数据

- 不可变数据（Immer）概念来源于函数式编程，在函数式编程中，对已初始化的变量是不可以更改的（纯函数，没有任何副作用），每次更改都要创建一个新的变量，JavaScript在底层没有实现不可变数据，需要借助第三方库来实现，Immer就是其中一个
- 为什么使用：使用不可变数据可以解决性能优化引入的问题（shouldComponentUpdate）

#### 2.虚拟DOM和diff算法

##### （1）虚拟DOM

- 对复杂的文档DOM结构，提供一种方便的工具，进行最小化地DOM操作


- 虚拟DOM是什么：虚拟DOM是利用了js的对象Object对象模型来模拟真实DOM，它的解构是一个树性结构

- 虚拟DOM是干什么的：使用js实现了DOM树，组件的HTML结构并不会直接生成DOM，而是映射成虚拟的JSDOM结构

- 可以把虚拟Dom理解成对应真实Dom的一种状态。当真实Dom发生变化后，虚拟Dom可以为我们提供这个真实Dom变化之前和变化之后的状态，我们通过对比这两个状态，即可得出真实Dom真正需要更新的部分，即可实现最小量更新。在一些比较复杂的Dom变化场景中，通过对比虚拟Dom后更新真实Dom会比直接更新真实Dom的效率高，这也就是虚拟Dom和diff算法真正存在的意义

- 虚拟Dom，其实很简单，就是一个用来描述真实Dom的对象

- 它有六个属性，sel表示当前节点标签名，data内是节点的属性，children表示当前节点的其他子标签节点，elm表示当前虚拟节点对应的真实节点（这里暂时没有），key即为当前节点的key，text表示当前节点下的文本，结构类似这样

  ```js
  let vnode = {
      sel: 'ul', 
      data: {},
      children: [ 
          {
              sel: 'li', data: { class: 'item' }, text: 'son1'
          },
          {
              sel: 'li', data: { class: 'item' }, text: 'son2'
          },    
      ],
      elm: undefined,
      key: undefined,
      text: undefined
  }
  ```

##### （2）diff

- diff算法使用来比较两个节点的
- 在采取diff算法比较新旧节点的时候，比较只会在同层级进行，不会跨层级比较

#### 3.React的batchUpdate机制

![](https://img.kancloud.cn/f9/8f/f98f808ed4acad954cef43094fc6d2d3_750x563.png)

- 如果是处于批量更新模式，也就是isBatchingUpdates为true时，不进行state的更新操作，而是将需要更新的component添加到dirtyComponents数组中。

- 如果不处于批量更新模式，则对所有队列中的更新执行batchedUpdates方法

- 何时合并：传入对象时合并，传入函数时不合并

- 同步异步：在同步环境下异步，在异步环境下同步

  ​

#### 4.React事务机制

![](https://img.kancloud.cn/70/66/7066fc61427661fb7cf2c61362238135_883x595.png)

- 对一个函数`anyMethod`进行切片包裹`wrapper`，每个`wrapper`可以实现自己的`initialize`和`close`接口，可以嵌套使用
- 事务的实现其实不难，可以简单理解为React仅仅是为方法加了前置和后置函数的钩子，并原子化执行函数

#### 5.React组件渲染和更新的过程

#### 6.React-fiber如何优化性能

- #### Fiber 架构核心：“可中断”“可恢复”与“优先级”


- `React 15`版本(Fiber以前)整个更新渲染流程分为两个部分：
  - `Reconciler`(协调器); 负责找出变化的组件
  - `Renderer`(渲染器); 负责将变化的组件渲染到页面上


- 存在的问题：**Stack Reconciler 是一个同步的递归过程**，一旦开始就"无法中断"。当层级太深或者`diff`逻辑(钩子函数里的逻辑)太复杂，导致递归更新的时间过长，`Js`线程一直卡住，那么用户交互和渲染就会产生卡顿。在这样的机制下，若 JavaScript 线程长时间地占用了主线程，那么**渲染层面的更新就不得不长时间地等待，界面长时间不更新，带给用户的体验就是所谓的“卡顿”**
- React15架构不能支撑异步更新以至于需要重构，于是React16架构改成分为三层结构：
  - `Scheduler(调度器)`：调度任务的优先级，高优任务优先进入Reconciler
  - `Reconciler(协调器)`：负责找出变化的组件
  - `Renderer(渲染器)`：负责将变化的组件渲染到页面上


- 而`React16`使用了一种新的数据结构：`Fiber`。`Virtual DOM`树由之前的从上往下的树形结构，变化为基于多向链表的"图"。更新流程从递归变成了可以中断的循环过程。每次循环都会调用`shouldYield()`判断当前是否有剩余时间
- 在这套架构模式下，更新的处理工作流变成了这样：首先，每个更新任务都会被赋予一个优先级。当更新任务抵达调度器时，高优先级的更新任务（记为 A）会更快地被调度进 Reconciler 层；此时若有新的更新任务（记为 B）抵达调度器，调度器会检查它的优先级，若发现 B 的优先级高于当前任务 A，那么当前处于 Reconciler 层的 A 任务就会被中断，调度器会将 B 任务推入 Reconciler 层。当 B 任务完成渲染后，新一轮的调度开始，之前被中断的 A 任务将会被重新推入 Reconciler 层，继续它的渲染之旅，这便是所谓“可恢复”。
  ​