### 一、文件结构分析

vue版本：2.6.14

文件结构如下所示：

![屏幕截图 2022-06-28 181034](E:\笔记\源代码\img\屏幕截图 2022-06-28 181034.png)

**dist**为构建输出文件，即运行时的那一份。我们只需要关注src目录下的文件即可。

进入到**src**内部：
**compiler：**编译相关模块，也就是template模板转换为render函数的地方；
**core：**核心模块，vue的初始化、整个生命周期都在这里实现；
**platforms：**平台化模块，分为web和weex，而我们只需要关注web即可；
**server**：服务端渲染模块。
**sfc：**对单文件组件的处理模块。
**shared：**一些公用的工具方法。

在上述的文件中，我们需要重点关注的只有：**compiler、core、platforms、shared。**

#### **1.shared全局方法**

**参考：**https://blog.csdn.net/qq_25324335/article/details/84948363

工具函数对应的是`src/shared/`下的`util.js`，这些函数都是全局通用的。

![](E:\笔记\源代码\img\屏幕截图 2022-06-29 084832.png)



**代码讲解：**

##### （1）emptyObject

```
export const emptyObject = Object.freeze({});
```

- **作用**：创建一个不可修改的对象

- **拓展**：对象的三个安全级别

  - 对象的创建方式：字面量创建，new（）创建

    ```
    var object1 = {name:'Jerry'} // 字面量创建
    var object2 = new Objcet{ // new关键字创建
    	name:'Jerry',
    	age:18,
    	null:'you'
    }
    ```


  - 对象属性访问：. 访问，[]访问

    ```
    object1.name  //Jerry
    object2['null'] // 'you' []通常用在.访问不到属性的场景中，例如属性名null不符合js命名规范，通	                 过.访问不到null属性，这时就可以使用[]访问
    ```

  - 对象属性

    **概念**：ECMAScript中定义了描述了对象属性特征的属性，由于是内部属性，所以该规范把它们放在                    了两对方括号中。该内部属性分为两类，一类是数据属性，一类是访问器属性。

    - **数据属性**

      **概念：**数据值包含一个数据值的位置。有以下四个描述其行为的特性：

      **[[Configurable]]**：能否使用delete删除该属性；能否修改属性的特性；能否把属性修改为访问器属性。
      **[[Enumerable]]：**能否通过for...in循环到该属性。
      **[[Writable]]：**该属性是否可以修改
      **[[Value]]：**该属性的值。
      直接在对象上定义的属性，[[value]]的值会赋值为相应的值，而其它三个特性都会赋值为true。
      ![20180410171016703](E:\笔记\源代码\img\20180410171016703.png)

      当修改某个属性的特征时，可以使用` Object.defineProperty()`定义这个属性，该方法接受三		 	           个参数，第一个参数为属性所在的对象，第二个参数为属性名称，第三个参数为属性的**描述符对象，描述符对象的属性必须是configurable,enumerable,writable,value中的一个或多个。**

      ```
      Object.defineProperty(person,'school',{value:'shancai'})
      person => {school:'shancai'}
      ```

      用这种方式定义的属性，[[writable]],[[configuarble]],[[enumerable]]如果不设置的话，会默认给false，也就是说，上面的person的school属性，现在是只读，不可枚举，不可删除的。如果要强制的修改或者删除它的话，在严格模式下会报错，否则会忽略你这种“违规”操作。

      **补充：**可以使用对象的方法Object.keys()查看[[enumerable]]，这方法接收一个对象实例，返回该实例的所有可枚举（enumerable为true）的属性的字符串数组。

      **综上所述：**使用Object.defineProperty()定义的属性，如果不显示的设置该属性的描述符的相应属性，那么除value外都默认为false；而直接在对象上定义的属性，除value外默认都是true。

    - **访问器属性**

      **概念：**访问器属性不包含数据值，但是提供了getter和setter用于对该属性进行读写（getter和setter都不是必须的）。访问器属性也包含下面四个特性：

      **[[Configurable]]：**能否使用delete删除该属性；能否修改属性的特性；能否把属性修改为数据属性。
      **[[Enumerable]]：**能否通过for...in循环到该属性。
      **[[Get]]：**读取属性时调用的函数。默认undefined。
      **[[Set]]：**给属性赋值时调用的函数。默认undefined。
      访问器属性不能直接在对象上定义，所以要使用对象方法定义，常见的方法如下：

      ![20180410195123166](E:\笔记\源代码\img\20180410195123166.png)

      上面的数据属性和访问器属性都是一次定义一个属性，倘若一个对象的属性特别多那么就会造成代码臃肿，所以js提供了一个可以一次定义多个属性的方法：Object.defineProperties()。用法也十分简单：

      ![20180410201303528](E:\笔记\源代码\img\20180410201303528.png)

  **对象保护：**

  - **概念：**ES5提供了下面几个级别的保护对象的方法，将对象设置为**防篡改对象**，限制你的操作。

    - #### 不可扩展对象（preventExtensions）

      **概念：**默认情况下的对象都是可扩展的，就是可以动态的给对象添加属性和方法。但是通过Object.preventExtensions方法可以改变这个行为，你就不可以添加了。

      ```
      Object.preventExtensions(对象) //可以删除、修改，不可以添加
      ```

    - #### 密封对象（seal）

      **概念：**密封对象在不可扩展对象的基础上，还将对象中已有属性的[[configurable]]属性设置为false，这下你连删除属性的权力都没有了，而且不能对数据属性和访问器属性进行转换了（参考上面[[configurable]]为false时的情况）。现在你只可以修改已有属性的值，通过使用Object.seal()方法：

      ```
      Object.seal(对象) //不可以添加、删除，只能修改
      ```

    - #### 冻结对象（freeze）

      **概念：**使用Object.freeze()操作对象以后，那么这个对象只能看

      ```
      Object.freeze(对象) //不可以删除、添加、修改
      ```

  - **查看对象状态：**ES5也提供了三种对应的对象检查的方法，用于检查对象是不是被怎么样了。这三个方法分别是：Object.isExtensible()，Object.isSealed()，Object.isFrozen()

    **注意：**一旦把对象设置为防篡改的，就无法撤销了。换句话说，这种操作是不可逆的。并且设置为防篡改对象后，你再去试图对应的修改它，非严格模式下会静默失败，严格模式下会报错

##### （2）isUndef

```
export  function  isUndef (v:  any):  boolean %checks {
  return  v  ===  undefined  ||  v  ===  null
}
```

- **作用：**检查一个值是不是没有定义，这里检查了它是`undefined`或者`null`类型，满足其一就表示它已定义，返回true。

##### （3）isDef

```
export  function  isDef (v:  any):  boolean %checks {
  return  v  !==  undefined  &&  v  !==  null
}
```

- **作用：**检查一个值是不是定义了，必须同时满足它不是`undefined`类型且不是`null`类型。

##### （4）isTrue

```
export  function  isTrue (v:  any):  boolean %checks {
  return  v  ===  true
}
```

- **作用：**检查一个值是不是true

##### （5）isFalse

```
export  function  isFalse (v:  any):  boolean %checks {
  return  v  ===  false
}
```

- **作用：**检查一个值是不是false

##### （6）isPrimitive

```
// Check if value is primitive
export  function  isPrimitive (value:  any):  boolean %checks {
  return (
    typeof  value  ===  'string'  
    typeof  value  ===  'number'  
    typeof  value  ===  'symbol'  
    typeof  value  ===  'boolean'
  )
}
```

- **作用：**检查一个值的数据类型是不是简单类型(字符串/数字/symbol/布尔)
- **拓展：**js中共有7种数据类型: `Number`,`Undefined`,`Null`,`String`,`Boolean`,`Object`,`Symbol`

##### （7）isObject

```
export  function  isObject (obj:  mixed):  boolean %checks {
  return  obj  !==  null  &&  typeof  obj  ===  'object'
}
```

- **作用：** 对象的快速检查-这主要是用来将对象从简单值中区分出来
- **拓展：** 为什么要检查一下obj !== null呢？因为虽然在js中Null与Object是两种数据类型，但是使用typeof操符号的结果是一样的,即 typeof null === 'object', 所以这里要先排除null值的干扰

##### （8）toRawType

```
const  _toString  =  Object.prototype.toString

export  function  toRawType (value:  any):  string {
  return  _toString.call(value).slice(8, -1)
}
```

- 作用: 获取一个值的原始类型字符串


- 拓展: 在任何值上调用Object原生的toString方法，都会返回一个[object NativeConstructorName]格式的字符串。每个类在内部都有一个[[Class]]的内部属性，这个属性就指定了这个NativeConstructorName的名称。例如:

  ```
  Object.prototype.toString.call([]) // "[object Array]"
  Object.prototype.toString.call(1) // "[object Number]"
  ```

**所以上述 toRawType 函数实际上是把 `[object Number]`这个字符串做了截取，返回的是类型值，如 `"Number", "Boolean", "Array"`等**

##### （9）isPlainObject

```
// Strict object type check. Only returns true for plain JavaScript objects.
export  function  isPlainObject (obj:  any):  boolean {
  return  _toString.call(obj) ===  '[object Object]'
}
```

- **作用**: 严格的类型检查，只有在是简单js对象返回true

- **拓展**: 为什么特意加一句 `Only returns true for plain JavaScript objects.`呢？因为有一些值，虽然也属于js中的对象，但是有着更精确的数据类型，比如：

  ```
  Object.prototype.toString.call([]) // "[object Array]"
  Object.prototype.toString.call(()=>{}) // "[object Function]"
  Object.prototype.toString.call(null) // "[object Null]" 
  ...
  ```

##### （10）isRegExp

```
export  function  isRegExp (v:  any):  boolean {
  return  _toString.call(v) ===  '[object RegExp]'
}
```

- **作用**: 检查一个值是不是正则表达式

- **拓展**: 正则表达式不是对象吗？为什么不能直接使用`typeof`操作符检查呢? 这主要是处于兼容不同浏览器的考虑：

  ```
  typeof /s/ === 'function'; // Chrome 1-12 Non-conform to ECMAScript 5.1
  typeof /s/ === 'object'; // Firefox 5+ Conform to ECMAScript 5.1
  ```

##### （11）isValidArrayIndex

```
// Check if val is a valid array index.
export  function  isValidArrayIndex (val:  any):  boolean {
  const  n  =  parseFloat(String(val))
    return  n  >=  0  &&  Math.floor(n) ===  n  &&  isFinite(val)
}
```

- **作用**: 检查一个值是不是合法的数组索引，要满足：非负数, 整数, 有限大
- **拓展**: 这主要是检查外来的值作为数组的索引的情况。

##### （12）toString

```
// Convert a value to a string that is actually rendered.
export  function  toString (val:  any):  string {
  return  val  ==  null
      ?  '' 
      :  typeof  val  ===  'object'
         ?  JSON.stringify(val, null, 2)
         :  String(val)
}
```

- **作用**: 把一个值转换成可以渲染的字符串。
- **拓展:**
  - 第一个判断用的是 `==` 而不是 `===`， 所以当值为`undefined`或者`null`时都会返回空串
  - JSON.stringify接收三个参数，分别是要序列化的值, 要序列化的属性， 序列化所需的空格

##### （13）toNumber

```
// Convert a input value to a number for persistence.If the conversion fails, return original string.
export  function  toNumber (val:  string):  number  |  string {
  const  n  =  parseFloat(val)
  return  isNaN(n) ?  val  :  n
}
```

- **作用**: 把一个值转换成数字，转化成功，返回数字，否则原样返回。

##### （14）makeMap

```
// Make a map and return a function for checking if a key is in that map.

export  function  makeMap (
  str:  string,
  expectsLowerCase?:  boolean
): (key:  string) =>  true  |  void {
  // 先创建一个map，用来存放每一项的数据
  const  map  =  Object.create(null)
  // 获取元素的集合
  const  list:  Array<string> =  str.split(',')
  // 遍历元素数组，以 键=元素名， 值=true 的形式，存进map中
  // 即如果 str = 'hello, world', 那么这个循环后
  // map = { 'hello': true, 'world': true}
  for (let  i  =  0; i  <  list.length; i++) {
    map[list[i]] =  true
  }
  // 如果需要小写(expectsLowerCase = true)的话，就将 val 的值转换成小写再检查； 否则直接检查
  // 这里的检查，就是检查传入的 val(键), 是不是在map中，在的话会根据该键名，找到对应的值(true)
  // 这里返回的是一个函数，该函数接收一个待检查的键名称，返回查找结果(true/undefined)
  return  expectsLowerCase
      ?  val  =>  map[val.toLowerCase()]
      :  val  =>  map[val]
}
```

- **作用**: 生成一个map，返回一个函数来检查一个键是不是在这个map中。详解见注释。
- **拓展：**
  - 函数柯里化
  - 闭包

##### （15）isBulitInTag

```
// Check if a tag is a built-in tag.
export  const  isBuiltInTag  =  makeMap('slot,component', true)
```

- **作用**: 检查一个标签是不是vue的内置标签

- **解析**: 从上面的**makeMap**函数可以知道，这个isBuiltInTag是一个函数,接收一个值作为查找的键名，返回的是查找结果。通过这个`makeMap('slot,component', true)`处理后，map的值已经变成

  ```
  {
  	"slot": true,
  	"component": true
  }
  ```

  **而且是检查小写，也就是说`"Slot", "Component"`等格式的都会返回true;**

##### （16）isReversedAttribute

```
// Check if a attribute is a reserved attribute.
export  const  isReservedAttribute  =  makeMap('key,ref,slot,slot-scope,is')
```

- **作用**: 检查一个属性是不是vue的保留属性。同上。

##### （17）remove

```
// Remove an item from an array
export  function  remove (arr:  Array<any>, item:  any):  Array<any> |  void {
  if (arr.length) {
    const  index  =  arr.indexOf(item)
    if (index  >  -1) {
      return  arr.splice(index, 1)
    }
  }
}

```

- **作用**: 移除数组的某一项。

##### （18）hasOwn

```
// Check whether the object has the property.
const  hasOwnProperty  =  Object.prototype.hasOwnProperty
export  function  hasOwn (obj:  Object  |  Array<*>, key:  string):  boolean {
  return  hasOwnProperty.call(obj, key)
}
```

- **作用:** 检查一个对象是否包含某属性, 这个属性不包括原型链上的属性(toString之类的)。
- **拓展:** 这里要思考的是，为什么不直接调用对象上的hasOwnProperty方法，反而要找对象原型上的呢？原因其实很简单，因为对象上的hasOwnProperty是可以被改写的，万一被重写了方法就无法实现这种检查了。

##### （19）cached

```
// Create a cached version of a pure function.
export  function  cached<F:  Function> (fn:  F):  F {
  // const cache = {}
  // 这个const相当于一个容器，盛放着 key-value 们
  const  cache  =  Object.create(null)
  // 返回的是一个函数表达式cacheFn
  return (function  cachedFn (str:  string) {
    const  hit  =  cache[str]
    // 如果命中了,那么拿缓存值; 反之就是第一次执行计算,调用函数fn计算并且装入 cache 容器
    // cache 容器中的键值对， key是函数入参， value是函数执行结果
    return  hit  || (cache[str] =  fn(str))
  }: any)
}

```

- **作用:** 为一个纯函数创建一个缓存的版本。
- **解析:** 因为一个纯函数的返回值只跟它的参数有关，所以我们可以将入参作为key，返回值作为value缓存成key-value的键值对，这样如果之后还想获取之前同样参数的计算结果时，不需要再重新计算了，直接获取之前计算过的结果就行。
- **拓展:** 纯函数：一个函数的返回结果只依赖于它的参数，并且在执行过程里面没有副作用，则该函数可以称为纯函数。

##### （20）camelize

```
// Camelize a hyphen-delimited string.
const  camelizeRE  = /-(\w)/g
export  const  camelize  =  cached((str:  string):  string  => {
  return  str.replace(camelizeRE, (_, c) =>  c  ?  c.toUpperCase() :  '')
})
```

- **作用**: 将连字符`-`连接的字符串转化成驼峰标识的字符串

- **解析**: 可以通过此例来感受下上面的`cached`函数的作用，因为cache的参数, 一个转换连字符字符串的函数，是一个很纯很纯的函数，所以可以把它缓存起来。

  例：

  ```
  camelize('hello-world') // 'HelloWorld'
  ```


##### （21）capitalize

```
// Capitalize a string.
export  const  capitalize  =  cached((str:  string):  string  => {
  return  str.charAt(0).toUpperCase() +  str.slice(1)
})
```

- **作用**: 将一个字符串的首字母大写后返回，也是可以被缓存的函数。

  例：

  ```
  capitalize('hello') // 'Hello'
  ```

##### （22）hyphenate

```
// Hyphenate a camelCase string.
const  hyphenateRE  = /\B([A-Z])/g
export  const  hyphenate  =  cached((str:  string):  string  => {
  return  str.replace(hyphenateRE, '-$1').toLowerCase()
})
```

- **作用**: 将一个驼峰标识的字符串转换成连字符`-`连接的字符串

  例：

  ```
  hyphenate('HelloWorld') // 'hello-world'
  ```

##### （23）polyfillBind

```
function  polyfillBind (fn:  Function, ctx:  Object):  Function {
  function  boundFn (a) {
    // 获取函数参数个数
    // 注意这个arguments是boundFn的，不是polyfillBind的
    const  l  =  arguments.length
    // 如果参数不存在，直接绑定作用域调用该函数
    // 如果存在且只有一个，那么调用fn.call(ctx, a), a是入参
    // 如果存在且不止一个，那么调用fn.apply(ctx, arguments)
    return  l
      ?  l  >  1 ? 
      fn.apply(ctx, arguments)
      :  fn.call(ctx, a)
      :  fn.call(ctx)
  }
  boundFn._length  =  fn.length
  return  boundFn
}
```

- **作用:** bind 函数的简单的 polyfill。因为有的环境不支持原生的 bind， 比如: PhantomJS 1.x。技术上来说我们不需要这么做，因为现在大多数浏览器都支持原生bind了，但是移除这个吧又会导致在PhantomJS 1.x 上的代码出错，所以为了向后兼容还是留住。


- **拓展:** call与apply的区别，call接受参数是一个一个接收，apply是作为数组来接收。

##### （24）nativeBind

```
function  nativeBind (fn:  Function, ctx:  Object):  Function {
  return  fn.bind(ctx)
}
```

- **作用**: 原生的bind。

##### （25）bind

```
export  const  bind  =  Function.prototype.bind
  ?  nativeBind
  :  polyfillBind
```

- **作用**: 导出的bind函数，如果浏览器支持原生的bind则用原生的,否则使用polyfill版的bind。

##### （26）toArray

```
// Convert an Array-like object to a real Array.
export  function  toArray (list:  any, start?:  number):  Array<any> {
  // start为开始拷贝的索引，不传的话默认0，代表整个类数组元素的的转换
  start  =  start  ||  0
  let  i  =  list.length  -  start
  const  ret:  Array<any> =  new  Array(i)
  while (i--) {
    ret[i] =  list[i  +  start]
  }
  return  ret
}
```

- **作用**: 将类数组的对象转换成一个真正的数组。实际上就是一个元素逐一复制到另一个数组的过程。

##### （27）extend

```
// Mix properties into target object.
export  function  extend (to:  Object, _from:  ?Object):  Object {
  for (const  key  in  _from) {
    to[key] =  _from[key]
  }
  return  to
}
```

- **作用**: 将属性混入到目标对象中，返回被增强的目标对象。

##### （28）toObject

```
// Merge an Array of Objects into a single Object.
export  function  toObject (arr:  Array<any>):  Object {
  // 定义一个目标对象
  const  res  = {}
  // 遍历对象数组
  for (let  i  =  0; i  <  arr.length; i++) {
    if (arr[i]) {
      // 遍历这个对象，将属性都拷贝到res中
      extend(res, arr[i])
    }
  }
  return  res
}
```

- **作用**: 将一个对象数组合并到一个单一对象中去。

  例：

  ```
  const arr = [{age: 12}, {name: 'jerry', age: 24}, {major: 'js'}]
  const res = toObject(arr)
  console.info(res) // {age: 24, name: 'jerry', major: 'js'}
  ```

##### （29）noop

```
export  function  noop (a?:  any, b?:  any, c?:  any) {}
```

- **作用**: 一个空函数。在这里插入的参数，是为了避免 Flow 使用 rest操作符(…) 产生无用的转换代码。

##### （30）no

```
// Always return false.
export  const  no  = (a?:  any, b?:  any, c?:  any) =>  false
```

- **作用**: 总是返回false

##### （31）identity

```
//  Return same value
export  const  identity  = (_:  any) =>  _
```

- **作用**: 返回自身。

##### （32）genStaticKeys

```
// Generate a static keys string from compiler modules.
export  function  genStaticKeys (modules:  Array<ModuleOptions>):  string {
  return  modules.reduce((keys, m) => {
    return  keys.concat(m.staticKeys  || [])
  }, []).join(',')
}
```

- **作用**: 从编译器模块中生成一个静态的 键 字符串。接收一个对象数组，然后取出其中的 `staticKeys` 的值(是个数组), 拼成一个`keys`的数组，再返回这个数组的字符串形式，用`,`连接的，如：`'hello,wolrd,vue'`

##### （33）looseEqual

```
// Check if two values are loosely equal - that is,
// if they are plain objects, do they have the same shape?
export  function  looseEqual (a:  any, b:  any):  boolean {
  // 如果全等，返回true
  if (a  ===  b) return  true
  const  isObjectA  =  isObject(a)
  const  isObjectB  =  isObject(b)
  if (isObjectA  &&  isObjectB) {
  // 如果 a 和 b 都是对象的话
    try {
      const  isArrayA  =  Array.isArray(a)  
      const  isArrayB  =  Array.isArray(b)
      // 如果 a 和 b 都是数组
      if (isArrayA  &&  isArrayB) {
          // 长度相等 且 每一项都相等(递归)
          return  a.length  ===  b.length  &&  a.every((e, i) => {
            return  looseEqual(e, b[i])
          })
      } else  if (!isArrayA  &&  !isArrayB) {
          // 如果 a 和 b 都不是数组
          const  keysA  =  Object.keys(a)
          const  keysB  =  Object.keys(b)
          // 两者的属性列表相同，且每个属性对应的值也相等
          return  keysA.length  ===  keysB.length  &&  keysA.every(key  => {
            return  looseEqual(a[key], b[key])
          })
      } else {
          // 不满足上面的两种都返回 false
          return  false
      }
    } catch (e) {
        // 发生异常了也返回 false
        return  false
    }
  } else  if (!isObjectA  &&  !isObjectB) {
    // 如果 a 和 b 都不是对象，则转成String来比较字符串来比较
    return  String(a) ===  String(b)
  } else {
    // 其余返回 false
    return  false
  }
}
```

- **作用**: 判断两个值是否相等。是对象吗？结构相同吗？
- **解析**: 为什么叫`loosely equal`呢？因为两个对象是不相等的，这里比较的只是内部结构和数据

##### （34）looseIndexOf

```
export  function  looseIndexOf (arr:  Array<mixed>, val:  mixed):  number {
  for (let  i  =  0; i  <  arr.length; i++) {
    if (looseEqual(arr[i], val)) return  i
  }
  return  -1
}
```

- **作用**: 返回一个元素在数组中的索引，-1表示没找到。方法很简单，就是上面`looseEqual`的一个使用场景。

##### （35）once

```
// Ensure a function is called only once.
export  function  once (fn:  Function):  Function {
  let  called  =  false
  return  function () {
    if (!called) {
      called  =  true
      fn.apply(this, arguments)
    }
  }
}
```

- **作用**: 确保一个函数只执行一次。
- **解析**: 这里是闭包的一个运用，该函数返回的是一个函数，不过这个函数是由`called`这个变量决定的。第一次调用后这个值就被设置成`true`了，之后再调用，由于闭包的存在，`called`这个标记变量会被访问到，这时已经是`true`了，就不会在调用了。

#### 2.**compiler文件**

整个compiler的核心作用就是生成render函数。而在该模块中的重点逻辑为 HTMLParser、parse、optimization、generate。在该文件中，会存在大量的高阶函数，在阅读该模块代码的时候也是以充分学习到函数式编程的思想。以下是对几个核心文件的简单介绍：

![屏幕截图 2022-06-29 163611](E:\笔记\源代码\img\屏幕截图 2022-06-29 163611.png)

- **codengen：**主要功能是用AST生成render函数字符串；
- **directives：**存放一些指令的处理逻辑，如v-bind、v-model、v-on等；
- **parser：**主要功能是将template模板编译为AST；
- **index：**compiler的入口文件；
- **optimizer：**用来对AST做一些剪枝操作的标记处理，会在codengen和vnode的patch中用到；
- **to-function：**将codengen生成的render函数字符串用new Function的方式最终生成render函数。

#### 3.core文件

core模块为整个vue的核心模块，其中几乎包含了vue的所有核心内容。如vue实例化的选项合并，data、computed等属性的初始化，Watcher、Observer的实现、vue实例的挂载等等。内容很多，因此我们需要重点分析该模块：

![屏幕截图 2022-06-29 164014](E:\笔记\源代码\img\屏幕截图 2022-06-29 164014.png)

- components：名称取的比较让人迷惑，但其实他并不是组件创建或更新相关的模块，在其内部只存在一个keep-alive；



- glodbal-api：存在一些全局api，如extend、mixin等等，也包括assets属性（component、directive）的初始化逻辑；



- instance：core模块中的核心，也是整个vue初始化的地方。包括了各种属性、事件的初始化，以及钩子函数的调用。其中的index文件，就是vue构造函数所在。而其他的文件，就像是一个个工厂，对vue进行层层加工，即初始化参数、初始化属性和方法等等；



- observer：响应式的实现所在，也就是数据劫持、依赖添加的具体逻辑实现。在我之前的博客中经常说到的Watcher、Dep、Observer都存放在这个文件中；



- util：工具文件。各种工具函数的所在。其中nextTick函数就存放在这儿；



- vdom：也就是虚拟DOM（vonde）相关内容模块。包括普通节点vnode、component vnode、functional component等的初始化、patch函数等等。

### 二、HTMLParser

**概念：**HTMLParser是浏览器的渲染进程中一个专门负责解析HTML文件的模块。浏览器并不能直接理解和使用HTML文档，因此就需要这样一种工具，能将HTML解析为浏览器能够理解的结构，也就是常说的DOM树。基于此DOM树，渲染引擎会继续做分层、绘制、合成等操作，最终将HTML转换为页面上一帧帧的内容。

vue只是借鉴了这样一种概念。但是区别在于，对于vue来说，最终目的并不是生成页面，而是生成render函数。而vue的后续操作，比如生成vnode，就是基于render函数进行的。

而要将template模板转换为render函数，抽象语法树AST就是绕不开的一个课题。AST的应用场景非常广泛，比如浏览器对JS的编译、babel对JS代码的转换、ESLint对代码的检测，都有AST的身影。对于vue来说，Parser的作用，就是将template模板转换为一颗AST，供后续生成render函数字符串使用。vue编译模板的大致流程，可以结合下图理解：

![0f9a4bb24bb54192a283efad8f304a66](E:\笔记\源代码\img\0f9a4bb24bb54192a283efad8f304a66.png)

 在浏览器中，HTMLParser代表的就是HTML解析为AST（也就是DOM树）的过程，包含词法分析和语法分析；而在vue中，将HTMLParser的概念做了简化，只代表词法分析阶段，也就是生成Token的过程。

#### 1.AST

编程语言的编译分为三部分，分为词法分析，语法分析，代码生成。

- 分词/词法分析：这个过程将有字符串组成的字符串分解为有意义的代码块，这些代码块统称为**词法单元（token）**。

  举个例子：let a = 1，这段程序通常会分解为下面这些词法单元：let、a、=、1、；空格是否被当成词法单元，取决于空格在当前语言的意义；


- 解析/词法分析：这个过程是将词法单元流转为一个由元素嵌套所组成的代表代表程序语法结构的树，这个树被称为**抽象语法树（AST）**。
- 代码生成：将AST转换为可执行代码的过程

**概念：** 由代码生成的树形结构。

**AST构建流程：**

根据浏览器中HTMLParser的大致流程，对AST构建形成初步认识；

- 将一个HTML标签分解为startTag，endTag和文本内容三部分对待；


- 创建一个栈结构，用来存放startTag；



- 遇到起始标签时入栈，遇到结束标签时出栈。每插入一个起始标签，都会以栈中上一个标签当作父节点；



- 对自闭合标签做特殊处理；



- 若出现嵌套问题，即结束标签与栈顶标签对应不上，则从栈顶一直循环出栈，直到找到相匹配的起始标签，并抛出错误；

  例：

  ```
  <div>
      <div>123</div>
      <span></span>
  </div>
  ```

  ![ddf61e58d8a64a08a75d0964fecc7b3d](E:\笔记\源代码\img\ddf61e58d8a64a08a75d0964fecc7b3d.png)

**AST编译过程：**

![v2-41788753ec9f58a1f079dce0b336fe08_r](E:\笔记\源代码\img\v2-41788753ec9f58a1f079dce0b336fe08_r.png)

AST的编译过程经过四个阶段：scanner,parser,traverse,generator

##### （1）scanner解析器进行解析

```
var company = '11'
```

例如有以上代码，在词法分析阶段会先对整个代码进行扫描生成tokens流。

- 我们会通过条件判断语句判断这个字符是 字母， "/" , "数字" , 空格 , "(" , ")" , ";" 等等。
- 如果是字母会继续往下看如果还是字母或者数字，会继续这一过程直到不是为止，这个时候发现找到的这个字符串是一个 "var"， 是一个Keyword，并且下一个字符是一个 "空格"， 就会生成{ "type" : "Keyword" , "value" : "var" }放入数组中。
- 它继续向下找发现了一个字母 'company'(因为找到的上一个值是 "var" 这个时候如果它发现下一个字符不是字母可能直接就会报错返回)并且后面是空格，生成{ "type" : "Identifier" , "value" : "company" }放到数组中。
- 发现了一个 "=", 生成了{ "type" : "Punctuator" , "value" : "=" }放到了数组中。
- 发现了'zhuanzhuan',生成了{ "type" : "String" , "value" : "zhuanzhuan" }放到了数组中。
  解析如下：

![v2-f8f7a39fb7eabb1674cd755a26caa07b_r](E:\笔记\源代码\img\v2-f8f7a39fb7eabb1674cd755a26caa07b_r.jpg)

##### （2）parser生成AST树

将tokens生成一颗AST树；例如：

```
Script {
  type: 'Program',
  body: [
    VariableDeclaration {
      type: 'VariableDeclaration',
      declarations: [Array],
      kind: 'const'
    }
  ],
  sourceType: 'script'
}
```

##### （3）travserse对AST数进行遍历，进行增删改查

对AST进行遍历更新，例如：

```
Script {
  type: 'Program',
  body: [
    VariableDeclaration {
      type: 'VariableDeclaration',
      declarations: [Array],
      kind: 'const',
      name: 'team',
      value: '大转转FE'
    }
  ],
  sourceType: 'script',
  name: 'team',
  value: '大转转FE'
}
```

##### （4）generator将更新后的AST转化为代码

将AST树编译为代码；

#### 2.vue中HTMLParser-AST

在vue的HTMLParser中，会将创建AST的任务通过钩子抛给parse函数去实现，他的核心在于通过template模板生成一个个token，并将token组装为相应的数据结构传递给parse函数即可。

