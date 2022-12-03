### 一、JS使用

**文档分为三种层：** 

- 结构层  -- html 
- 样式层  -- css  
- 行为层  -- js（javascript）

##### 1.行内

- 在开始标签内，写入js方法，后面跟上表达式(数值和运算符组合成的

  ```js
  <div onclick="alert('弹窗')"></div>
  ```

##### 2.内部

- 是在html页面内，写入script双标签，所有的代码写在这个双标签之间，可以存在页面当中的任何位置，但是，要求放在body结束标签之前，或者是head的结束标签之前，其他的位置，并不推荐使用；

  ```js
  <script>
      alert('内部的使用方式')
  </script>
  ```

##### 3.外部

- 在外部新建一个js文件，扩展名是 .js 需要使用script双标签的src属性，链接外部的js 文件中，直接写js代码，不需要script双标签；

  ```js
  <script src="./js/index.js"></script>
  ```

**总结：**

1、不推荐使用行内js , 因为可读性或者可维护性较差 

2、如果是小型的项目，推荐使用内部  

3、如果是大型的项目，推荐使用外部，可以少量的使用内部 

4、如果script双标签，是用来实现内部js的，那就不能在用于外部js的链接，反之亦然



### 二、变量

##### 1.概念

- 变量就是用来存储值的一个容器
- 变量一定是先声明(定义和赋值)后使用  
- 声明变量使用关键字 var (es5)

##### 2.变量的声明规则

- 以字母，下划线，$开头 

- 由字母，下划线，$开头，数字组成  

- 不能使用关键字和保留字 

- 见名知意 -- 语义化  

- 遵循驼峰命名法 --  变量名从第二个单词开始，首字母要大写  

  ```js
  var a; // 声明了 但是没有值 结果是 undefined
  /* 1、声明一个变量，并赋值 */
  var num = 10;
  console.log(num + 10);

  /* 2、 声明多个变量，并同时赋值，每个变量之间使用逗号隔开 */
  var num1 = 10, num2 = 20, num3 = 30; 
  console.log(num1, num2, num3);

  /* 3、声明多个变量，并分开赋值 */
  var sum1,sum2,sum3; 
  sum1 = 100
  sum2 = 200 
  sum3 = 300
  console.log(sum1, sum2,sum3);
  ```



### 三、事件

##### 1.概念

通过鼠标或者键盘触发，引发一些列的元素的行为就是事件 ；

事件三步骤：

- 事件源 -- 也就是哪个元素发生事件  ---  获取元素  
- 发生什么事件 -- 把事件绑定给元素  
- 干什么事   -- 事件触发后，会引发什么行为

##### 2.事件种类 

```js
onclick: 单击事件 

onmouseover: 鼠标经过  
onmouseout: 鼠标离开  

onmouseenter: 鼠标经过
onmouseleave: 鼠标离开   

onmousedown: 按下 
onmouseup: 抬起 

onmousemove: 移动  

ondblclick：双击事件 

oncontextmenu: 右键
```



### 四、JS操作

##### 1.文本操作

- 概念：操作双标签的文本内容或者单标签（input）的单独文本属性；


- 双标签：

  ```js
  获取：元素.innerHTML
  设置：元素.innerHTML = '新内容'
  //innerHTML是支持标签的
  ```

  ```js
  获取：元素.innerText
  设置：元素.innerText = ’新内容‘
  // innerText不支持标签
  ```


- 单标签：

  ```js
  获取内容：元素.value 
  设置内容：元素.value = '新内容'
  ```

##### 2.样式操作

```js
// 单独修改
元素.style.样式名 = 样式值
```

- 只要是js操作的样式，一定是行内样式
- 如果样式是复合词，需要遵循驼峰命名法 

```js
// 批量设置
元素.style.cssText = "width:100px; height:100px;"
// 会发生覆盖，用+解决
box.style.cssText += 'border:2px solid red'
// 会出现兼容问题，在样式的前面加一个分号 解决非标准浏览器的兼容问题
box.style.cssText += ';border:2px solid red'
```

##### 3.属性操作

- 原有属性：标签元素本身自带的属性，比如：id class  src  href  

  ```js
  // 获取属性
  元素.属性名  // 元素.id 
  // 设置属性
  元素.属性名 = 新属性 // box.id = 'box1234567'
  ```

  - 如果一个属性，赋值给了一个变量，那么就要把点的方式转换成方括号的方式 

  ```js
  var a = 'id'
  console.log(box[a]); 
  console.log(box['id']); 
  ```


- 非原有属性:程序员为了实现某些效果，自己添加的就是非原有属性

  ```js
  <div index="1"></div>  // index为非原有属性
  div.index = undefined // 获取不到非原有属性，也获取不到class（保留字）
  ```

  - 操作class

    ```js
    元素.className  // 获取类名 
    元素.className = '新的类名'  // 设置类名
    ```

    - 注意：className是可以覆盖原有的类名的  


  - 新增方法classLIst

    ```js
    box.classList.add() // 增加一个类 
    box.classList.remove() // 删除一个类 
    ```

    ​

### 五、this指向

定义：this是一个直柄元素，this的具体指向决定于它的词法环境

js是一门动态语言



### 六、函数

- 定义：就是一段代码块，可以被重复调用执行的代码块

##### 1.声明方式

- 字面量

  ```js
  var add = function() {
      // 代码块
  }

  // 代码
  var add = function(){
      console.log('字变量的方式');
  }
  ```


- 关键字

  ```js
  function add(){
      // 代码块
  }

  // 代码
  function num(){
      console.log('关键字的方式');
  }
  ```

**注意：函数，要先声明，后调用，不调用的函数是不执行的**  

**调用的方式：函数名 + ()  比如 add()**  

##### 2.函数的调用方式

- 普通调用

  ```js
  function add(){
      console.log('关键字的方式');
  }
  add(); // 调用
  ```

  ​

- 事件中的调用

  ```js
   <div onclick="add">1231231</div>
  //注意：在行内js中的事件中调用函数，也是 函数名 + () 
  ```

  ​

- 函数中调用函数

  ```js
  var add = function(){
      console.log('字变量的方式');
  }

  function num(){
      add()
  }
  num()
  ```

##### 3.函数的参数

- 参数有两种；实参和形参 


- **注意：参数其实一个省略了  var  关键字的变量**


- 实参：就是实际的数据，是确切存在的一个值  --- 在函数调用的时候，在小括号内传入的值 


- 形参：形式上的参数，是一个代表词 ，不知道确切的值，需要实参传入，才知道是什么值  -- 在函数声明的时候，在函数名后面的小括号中，写入的参数

  ```js
  function add (x, y) { // 形参
      console.log();
  }
  add(10,20); // 实参

  // 传值的方式，实参和形参是一一对照的方式传值 
  ```


- 参数的类型：参数可以传入任何类型，也可以接收任何类型

  ```js
  function add (a,b,c,d,e,f,r,g) { // 形参
      console.log(a,b,c,d,e,f,r,g);
  }
  add(10,'string',true,[],{},function(){},null,undefined); // 实参
  ```


- 特殊情况：
  - 实参的个数多余形参的个数多余出来的实参值，会丢失 ；
  - 形参的个数大于实参的个数如果多出来的形参，没有接收到实参传过来的值，并且参与了运算

##### 4.arguments

- 定义：是实参的一个集合，是以伪数组的形式存在的，有长度有下标 

  ```js
  function fn(){
  console.log(arguments); // 是实参的集合
  console.log(arguments[0]);
  }
  fn('好家伙',1,2,3,45,6)
  ```

##### 5.函数的封装

```js
// 封装的原则：当代码大部分相同，把相同的代码保留一份
// 把不同的变成参数

// 1-30
// 1-50
// 1-100

function add(a) {
    var num = 0;
    for (var i = 1; i <= a; i++) {
        num += i;
    }
    console.log(num);
}
add(30)
add(50)
add(100)
add(300)
add(3000)
```

##### 6.return返回值

- 概念：把一个值，返回给函数本身，那么函数本身会存储这个值，所以，可以在函数外使用一个变量来接收这个值  

  ```js
  function add () {
      var a = 10;
      // 返回值 -- 是把值返回给函数本身
      return a
  }

  console.log(add()); // 10 add() = 10

  //    也可以使用一个变量来接收
  var num = add();
  console.log(num);
  ```


- **什么时候使用返回值？**

  当函数内部( 花括号之间 )的值需要在外部使用或者展示，就需要使用返回值进行返回 


- **return是有阻断代码的作用的**

  ```js
  function fn(){
      console.log('今天天气好好啊');
      return
      console.log('小芳，今天约吗？'); // 不执行
  }
  fn()
  ```

##### 7.break和continue

- break: 也是有阻断代码的作用，条件满足，后面条件即使满足，也不会执行  


- continue：也是有阻断代码的作用，但是阻断的是当前的条件，后续的条件满足，会继续执行

  ```js
  for (var i = 1; i < 10; i++) {
      if (i == 6) {
          // break
          continue
      }
      console.log(i);
  }
  ```

##### 8.自执行函数

- 自执行函数就是匿名函数，也就是需要自己调用自己  

  ```js
  (function () {
      console.log('匿名函数');
  })()
  ```


- 可以把代码形成模块化，也就是把很多的功能都放在这个自执行函数中，使用返回值实现 

- 防止全局变量的污染，以及代码块之间混执行

- 问题：如果自执行函数的上一行代码，没有分号结束符，那么，上一行代码，会被当做函数体调用，所以会报错  

- 为了防止报错，一般在自执行函数前面，接一个符号 

  ```js
  var a = 1
  ~(function () {
      var a = 10
      console.log(a);
  })()

  console.log(a);
  ```

##### 9.获取样式

- 内部的样式和外部的样式 是无法获取的，只有通过js设置的样式 或者行内 才可以获取

  ```js
   div.style.width = '200px'
   console.log(div.style.width);
  ```


- **标准浏览器和非标准浏览器获取样式**

  ```js
  // 标准浏览器
  /* 语法：
  window.AbortController(元素).属性名 */
  console.log(window.getComputedStyle(div).width);

  // 非标准
  // 语法：
  // 元素.currentStyle.属性名
  console.log(div.currentStyle.width);
  ```

  ​

- **获取非行间样式的封装**

  ```js
  // 浏览器的能力检测
  /*  if (window.getComputedStyle) {
       // 标准浏览器
       console.log(window.getComputedStyle(div).width);
   } else {
       // 非标准
       console.log(div.currentStyle.width);
   }
  */
  // 封装
  /* 
  element: 元素
  attr:属性
  */
  function getStyle (element, attr) { // element = div  attr = 'width'
      if (window.getComputedStyle) {
          // 标准浏览器
           return window.getComputedStyle(element)[attr];
      } else {
          // 非标准
          return element.currentStyle[attr];
      }
  }
  ```

  ​


### 七、DOM

##### 1.节点类型

- 1 ： 元素节点 
- 2 ：属性节点 
- 3 ：文本节点  

##### 2.获取元素的节点名称，节点属性，节点类型

- 节点名称：元素.nodeName ---- 注意：获取到的元素名称是大写的  
- 节点类型：元素.nodeType ---注意：一共是12种，常用的是 1 ，2， 3， 其余的作为了解就可以了 
- 节点属性：元素.nodeVlaue -- 注意：必须使用getAttributeNode设置，否则是获取到的  

```js
var nameAttr = document.getElementById('name').getAttributeNode('name') 

console.log(jinan.nodeType);  // 1 -- 元素节点  
console.log(jinan.nodeName);  // LI -- 名字是大写的 
console.log(jinan.nodeValue); // null 

console.log('---------------------------');

console.log(nameAttr.nodeType);  // 2 -- 属性节点  
console.log(nameAttr.nodeName);  // name 
console.log(nameAttr.nodeValue); // username

console.log('***下面是获取原有属性和非原有属性的方法 -- 获取、设置、删除');
var box = document.getElementById('box');
console.log(box.id);
console.log(box.a);
```

##### 3.获取、设置、删除原有属性和非原有属性

```js
// getAttribute: 获取原有属性和非原有属性
console.log(box.getAttribute('a'));
console.log(box.getAttribute('id'));

// setAttribute： 设置原有属性和非原有属性
box.setAttribute('a','456');
box.setAttribute('id','div');
```

   // removeAttribute： 移除原有属性和非原有属性
   box.removeAttribute('a');
   box.removeAttribute('id');
  ```

  ##### 4.获取节点

  - 获取子节点

  ```js
  // 获取所有的子节点 --- 包含了 文本节点，属性节点，元素节点 返回值是一个数组
  var childs = oUl.childNodes; 

  // 获取所有的子元素节点  -- 只包含元素节点，没有其他的节点，返回值是一个数组
  console.log(oUl.children); 
  ```

  - 获取第一个子节点 

  ```js
  // 获取第一个子节点 
  console.log(oUl.firstChild); 
  // 获取第一个子元素节点
  console.log(oUl.firstElementChild);
  ```

  - 获取最后一个子节点

  ```js
  // 获取最后一个子节点
  console.log(oUl.lastChild);
  // 获取最后一个子元素节点
  console.log(oUl.lastElementChild);
  ```

  - 获取直接父元素节点的

  ```js
  // 获取直接的父节点 -- parentNode 
  var  span = document.querySelector('.span')
  console.log(span.parentNode);
  ```

  总结： 

-   1、childNodes： 获取所有的子节点 --- 包含了 文本节点，属性节点，元素节点 返回值是一个数组，所有用来获取需要的子元素节点，需要知道前面有几个文本节点，也就是需要算下标值 

-   2、children： // 获取所有的子元素节点  -- 只包含元素节点，没有其他的节点，返回值是一个数组，是比较直接的，**推荐使用的**

-   3、firstChild：获取第一个子节点， 需要知道前面有几个文本节点，也就是需要算下标值 **不推荐使用**

-   4、firstElementChild： 获取第一个子元素节点，但是有兼容性，**不推荐使用**

-   5、lastChild： 获取最后一个子节点，需要知道前面有几个文本节点，也就是需要算下标值 **不推荐使用**

-   6、lastElementChild：获取最后一个子元素节点，但是有兼容性，**不推荐使用**


  ```js
  console.log(oUl.nextSibling);
  console.log(oUl.previousSibling);

  console.log(oUl.nextElementSibling);
  console.log(oUl.previousElementSibling);
  ```

  ##### 4.创建节点

  ```js
  /*   语法：
  document.createElement('元素名称')
  */

  var p = document.createElement('p'); 
  var a = document.createElement('a'); 
  a.href = 'http://www.baidu.com'
  a.innerHTML = '百度'
  ```

  ##### 5.添加节点

  ```js
  // 添加节点 -- 在父元素的末尾追加节点
  /*  
  语法：
  父元素.appendChild(需要添加的节点) 
  */
  document.body.appendChild(a)
  ```

  ##### 6.插入节点

  ```js
  //插入节点 -- 在某个元素之前加入节点
  /* 
  语法：
      父元素.insertBefore(newChild, refChild)
      newChild: 新的一个元素，也就是需要插入的元素 
      refChild：参考元素，也就是在哪个元素的前面插入
   */

  document.body.insertBefore(a, document.body.children[0])
  ```

  ##### 7.留言板

  ```js
  // 获取元素
  var btn = document.querySelector('button');
  var text = document.querySelector('textarea');
  var ul = document.querySelector('ul');
  /* 
          事件发生后：
          1、把textarea获取到赋值给 li元素 
          2、先去创建一个li元素 
          3、把创建的li元素给 ul 
   */
  btn.onclick = function () {

      if (text.value == '') {
          alert('请输入内容');
          return false
      } else {
          var li = document.createElement('li'); // 创建元素 
          li.innerHTML = text.value
          // 添加元素
          ul.insertBefore(li, ul.children[0])

          // 清空
          text.value = ''
      }
  }
  ```

  ##### 8.删除节点

  ```js
  /*
  语法：
  父元素.removeChild()方法从node节点中删除一个子节点，返回值是被删除的节点
  */

  btn.onclick = function(){
      // 有没有元素
      if(ul.children.length == 0){
          this.disabled = true
      }else {
          ul.removeChild(ul.children[0])
          var num = ul.removeChild(ul.children[0]);
          console.log(num);
      }
  }
  ```

  ##### 9.替换节点

  ```js
  // 语法
  父元素.replaceChild(新的节点，被替换的节点)
  // 父元素.replaceChild(新的节点，被替换的节点)
  document.body.replaceChild(p,span)
  ```

  ##### 10.复制节点

  ```js
  // 语法： 
  // node.cloneNode()  
  // 如果没有参数，只复制元素节点，如果参数true 复制节点的同时，也会复制节点里面的内容


  console.log(span.cloneNode()); // 只复制元素节点
  console.log(div.cloneNode(true)); // 复制元素和内容
  ```

  ​

###  八、BOM

- BOM就是浏览器对象模型，提供了内容与浏览器窗口进行交互的对象，核心是window  


- BOM 的标准是浏览器厂商制定，具有兼容性  

##### 1.BOM的构成

dom  localtion  navigation history  ......... 

##### 2.window对象中的常见事件

```js
// 让js代码最后加载
window.onload = function(){

}

// 调整窗口大小的事件
window.onresize = function () {
    console.log('变化了');
    if (window.innerWidth <= 800) {
        box.style.width = '600px'
    } else {
        box.style.width = '300px'
    }
}

// 打开一个窗口
window.open('https://www.baidu.com','_self')
```

##### 3.绑定事件

- 传统的方式给元素绑定事件

```js
// 绑定事件
元素.onclick = function() {}

// 解绑事件
元素.onclick = null
```

代码实列

```js
box.onclick = function () {
    console.log('我是红色的');
    // 解绑事件
    // box.onclick = null;
}
// 注意：同一个元素执行同一种事件，执行不同的事件函数，后面的事件会覆盖前面的事件
```

- 注册监听的方式给元素绑定事件

```js
/* 
语法： 
元素.addEventListener(事件名称，事件函数，布尔值)
事件名称：这个事件名称是不带的 on 的 比如onclick应该写成 click  是一个字符串
事件函数：是一个匿名函数，事件触发后执行的函数 
布尔值：值有true和false，代表的是元素是否支持冒泡 或者是捕获
*/

// 代码使用
box.addEventListener('click', function(){
    console.log('我是新绑定事件的方式');
})
// 开心就好啊

// 解绑事件 
/*  
 语法：
 元素.removeEventListener(事件名称，事件函数)
 事件名称：这个事件名称是不带的 on 的 比如onclick应该写成 click  是一个字符串
 事件函数：是一个匿名函数，事件触发后执行的函数 
*/

box.addEventListener('click', move) // 注意：如果事件函数被独立，调用的时候不带小括号

function move () {
    console.log('我是新绑定事件的方式');
    // 解绑事件
    box.removeEventListener('click',move)
}
```

非标准浏览器

```js
// 非标准浏览器
// 事件名称是需要带的 on 的 
box.attachEvent('onclick', move)


function move () {
    console.log(123456);

    // 解绑事件
    box.detachEvent('onclick', move)
}
```

绑定事件的兼容处理

```js
getEvent(box, 'click', function () {
    console.log('处理兼容');
})
// 绑定事件的兼容
function getEvent (obj, eventName, callback) {
    if (obj.addEventListener) {
        // 标准浏览器
        obj.addEventListener(eventName, callback, false)
    } else {
        // 非标准浏览器
        obj.attachEvent('on' + eventName, callback)
    }
}
```

解绑事件的兼容处理

```js
getEvent(box, 'click',move)

function move() {
    console.log('处理兼容');
    remEvent(box,'click',move)
} 

// 解绑事件的兼容
 function remEvent (obj, eventName, callback) {
    if (obj.removeEventListener) {
        // 标准浏览器
        obj.removeEventListener(eventName, callback)
    } else {
        // 非标准浏览器
        obj.detachEvent('on' + eventName, callback)
    }
}
```

##### 4.元素偏移量 offset系列

- offset就是偏移的意思，可以使用这个方法动态的获取到元素的位置(偏移)，元素的大小等  
- 1、获取到元素距离带有定位属性的父元素的位置
- 2、获取元素本身的大小 
- 3、返回值是不带的单位的

```js
element.offsetParent: 返回作为该元素有定位的父元素，如果父元素没有定位，返回的是到body
element.offsetTop:返回的是有定位属性的父元素的上方距离
element.offsetLeft:返回的是有定位属性的父元素左边的距离
element.offsetWidth:返回的是元素自身的宽度(padding、边框、内容区域的宽度) 返回值不带单位
element.offsetHeight:返回的是元素自身的高度(padding、边框、内容区域的高度) 返回值不带单位
```

```js
console.log(div.offsetParent);
console.log(div.offsetTop);
console.log(div.offsetLeft);
console.log(div.offsetWidth);
console.log(div.offsetHeight);
```

总结：

- ##### **offset**

  - offset可以得到任意样式中的样式值 
  - offset系列获取到的数值是没有单位的 
  - offsetWidth包含： padding + border + width
  - offsetWidth等属性是只读属性，只能获取不能赋值 


- **style**
  - style只能得到行内样式表中的样式值  
  - style.width获取到数值是带单位 
  - style.width不包含padding和border 
  - style.width可读可写

结论：想要获取元素的大小位置，使用offset是比较合适  

##### 5.client系列

- ##### 就是客户端的意思，可以获取到元素可视区域的相关值，得到元素的边框的大小，元素的大小等 

```js
element.clientTop: 返回元素上边框的大小 
element.clientLeft: 返回元素左边框的大小 
element.clientWidth: 返回元素的宽度（padding,内容区域的宽度，不包含边框）返回值不带单位
element.clientHeight: 返回元素的高度（padding,内容区域的高度，不包含边框）返回值不带单位
```

```js
console.log(box.clientTop);
console.log(div.clientTop);

console.log(box.clientLeft);
console.log(div.clientLeft);

console.log(box.clientWidth);

// body 和 html 
console.log(document.body.clientWidth); // body
console.log(document.documentElement.clientHeight); // html  

//    处理兼容
var clientTop = document.documentElement.clientWidth || document.body.clientWidth
console.log(clientTop);
```

##### 6.scroll系列

- 就是滚动的意思，可以获取元素的大小，以及滚动距离  

```js
element.scrollTop: 返回被卷去的上边距离，返回值不带单位
element.scrollLeft: 返回被卷去的左侧距离，返回值不带单位
element.scrollWidth: 返回自身实际的宽度，不包含边框，返回值不带单位
element.scrollHeight: 返回自身实际的高度，不包含边框，返回值不带单位
```

```js
// 注意： 当需要获取滚动条滚动的距离的时候，需要使用onscroll事件监听，否则的获取不到 

// 事件监听
window.onscroll = function () {
    // 获取值
    var top = document.documentElement.scrollTop || document.body.scrollTop
    console.log(top);
}

// 赋值 
// 注意： 获取值是一起获取，但是赋值，需要分开赋值
document.documentElement.scrollTop = 500
document.body.scrollTop = 500
```

```js
var box = document.querySelector('div')

window.onscroll = function () {
    var top = document.documentElement.scrollTop || document.body.scrollTop
    if (top >= 800) {
        box.style.display = 'block'
    }

    // 添加点击事件
    box.onclick = function () {
        var time = setInterval(function () {
            var top = document.documentElement.scrollTop || document.body.scrollTop

            document.documentElement.scrollTop -= 50
            document.body.scrollTop -= 50

            if (top <= 0) {
                console.log(1);
                clearInterval(time)
            }
        }, 50)
    }
}
```



##### 7.事件高级

- **什么是事件**

  用户操作上由某种行为产生的操作，比如点击触发事件，回车搜素(键盘事件)


- **事件对象**

  事件触发后，根事件相关的一系列信息数据的集合会在底层当中，被存放到一个实参中，这个实参就是事件对象

  在页面中，定义事件函数的时候，需要在事件中函数设置形参，来接收这个实参，就可以使用事件对象了  

  1、谁绑定了事件 

  2、鼠标触发事件，可以得到鼠标相关的信息，比如：位置 

  3、键盘触发事件，可以得到键盘的信息，比如：按下了哪个键 

```js
btn.onclick = function(event){
    // 注意：一般会使用 e 或者 event 
    console.log(event);
    // 事件对象的兼容
    // 注意：非标准浏览器，只能使用window.event  
    console.log(window.event);

    // 兼容
    var event = event || window.event 
}
```

##### 8.常用的属性和方法

```js
// 常用的属性和方法
console.log(event.target); // 事件源 -- 发生事件的元素 -- 标准浏览器
console.log(event.srcElement);
var target = event.target || event.srcElement

// 鼠标距离浏览器左上角的距离
console.log(event.clientX); 
console.log(event.clientY);

// 鼠标到屏幕左上角的距离
console.log(event.screenX);
console.log(event.screenY);

// 鼠标距离文档左上角的距离(包含被卷出去的距离)
console.log(event.pageX);
console.log(event.pageY);
```

##### 9.event.target 和 this的区别

- this是事件绑定的元素
- e.target是事件触发的元素

##### 10.DOM的事件流

- html标签中是互相嵌套的，可以将元素想象成一个盒子，document就是最大的盒子 
- 当点击最里面的div的时候，同时你也点击了div的父元素，甚至是整个页面 
- 事件流分为三个阶段：
  - 1、当从最外面开始找目标元素的过程，是从最不具体的元素，到具体的某一个元素，这个过程是捕获过程  
  - 2、找到对应的元素，并触发了相应的事件，这个是目标阶段  
  - 3、当从最里面，点击了这个元素触发事件，从最具体的元素，到最不具体的元素，这个过程是冒泡阶段  

##### 11.事件冒泡

- 当一个元素接收到事件，会把接收到的事件依次传递给父元素，一直到window(document)

```js
div[0].onclick = function(){
            console.log(this.id);
        }
        div[1].onclick = function(){
            console.log(this.id);
        }
        div[2].onclick = function(){
            console.log(this.id);
        }

        document.body.onclick = function(){
            console.log('body');
        }

```

##### 12.阻止冒泡

- 指的是子元素接收到事件后，阻止子元素在给父元素传播事件 

```js
标准浏览器：事件对象.stopPropagation(); 
IE浏览器（非标准）: 事件对象.cancelBubble = true 
```

- 兼容处理

```js
// 阻止冒泡兼容
/* 
@params： stopPropagation： 标准
@params： cancelBubble 非标准
*/
function stopPropagation(e){
    if(e.stopPropagation){
        e.stopPropagation()
    }else{
        e.cancelBubble = true
    }
}
```

##### 13.捕获阶段

```js
div[0].addEventListener('click', fn, false)
div[1].addEventListener('click', fn, true)
div[2].addEventListener('click', fn, true)
// 把布尔值false变成true，就是在捕获阶段触发
```

##### 14.事件的默认行为

- 事件的默认行为，就是元素的特殊操作，比如超链接


- **取消默认行为**

```js
// 元素.事件名 = 事件函数 -- 传统的方式添加事件 
return false 

// 元素.addEventListener() 
事件对象.preventDefault() 

// 元素.attachEvent()  
事件对象.returnValue = false; 
```

```js
// 兼容
function preventDefault(e){
    if(e.preventDefault){
        e.preventDefault()
    }else{
        e.returnValue = false
    }
}
```





### 九、jQuery

##### 1.选择器

**（1）基本选择器**

- - 定义：通过jQuery选择器获取的元素都是一个jQuery的元素集合（伪数组）， css选择器的写法都可以

  ```js
  console.log($('ul > li'));
  console.log($('li'));
  console.log($('li#text'));
  console.log($('li.a'));
  console.log($('ul > li:nth-child(odd)'));
  ```

**（2）特殊选择器**

- 定义：在基本选择器的基础上加上jQuery独立封装的一些内容 => 参照伪类的方式来使用
- 语法  
  - :first  第一个
  - :last   最后一个
  - :eq(索引)指索引的第几个
  - :odd    指索引的奇数个
  - :even   指索引数为偶数的
  - :empty  获取内容为空的元素
  - :checked    获取选中的状态的元素

```js
console.log($('ul > li:first'));
console.log($('ul > li:last'));
console.log($('ul > li:eq(1)'));
console.log($('ul > li:odd'));
console.log($('p:empty'));
console.log($('input:checked').length);
```

##### 2.筛选器

- 定义：就是一个一个的方法，对选择器获取的元素进行二次筛选

- 语法：

  first()  ：第一个

  last()  ：最后一个

  eq(n)  ：索引第n个

  next()  ：下一个

  nextAll()  ： 后面的所有兄弟元素 ，括号中写条件 选中后面的所有符合条件的兄弟元素

  nextUntil()  ：后面的所有兄弟元素直到（中的选择器），不包含条件对应的这个元素 // 不写条件就和nextAll一样，把后面的兄弟元素都选上

  prev()  ：获取上一个兄弟元素

  prevAll()   ：获取上面的所有兄弟元素 ，获取到的数组顺序是按照从下到上的，依次获取前面的每一个兄弟元素

  prevUntil()  ：获取上面的所有兄弟元素知道括号中选择到的元素，括号中也是写选择器条件，获取的也不包含该条件对应的元素

  parent()  ：获取的是亲父级元素

  parents()   ：获取所有父级，直到html

  parentsUntil()   ：不写条件和parents()获取所有父级，写选择器条件，那就获取到父级中截止到该条件的父级

  siblings()   ：获取除了自己以外的所有兄弟元素

  find()  ：元素集合.find(选择器)  // 在该元素的所有后代中查找所有符合条件的元素

  .index() ： 元素集合.index()，返回该元素在html结构中的索引

##### 3.操作元素样式

- 语法： css()
- 形式1：获取样式
  - 语法：元素集合.css('样式名')
  - 返回值：元素集合内的第一个元素的指定样式的值，行内和非行内都能拿到


-  形式2：设置样式
  - 语法：元素集合.css('样式名','样式值')
  - 取值数值px单位可以省略，纯数字的取值可以不放在引号里，比如透明度


- 形式3：批量设置样式
  -  语法：元素集合.css({对象，你要用添加所有样式  样式名:样式值,样式名:样式值,...})




### 十、函数柯里化

- currying，把接收多个参数的函数变为接收单一单一参数的函数，并且返回接收剩余参数而且返回结果的新函数


- 例：利用函数柯里化实现`let a = add(1)(2)(3)()`

```js
const add=(a)=>{
  let current=a;
  let adder=function(b:any){
    if(b){
      current=current+b;
      return adder;
    }
    return current;
  }
  return adder;
}
```

### 十一、bind函数

- bind函数最常见的用法是绑定函数的上下文，除此之外，bind函数还有两个特殊的用法，一个是柯里化，一个是绑定构造函数无效。

  - 柯里化

    ```js
    var test = function (b) {
      console.log(this.a + b)
    }
    // 如果直接执行test，最终打印的是10.
    var bindTest1 = test.bind({ a: 20 })
    bindTest1()  // 10
    bindTest1(10) // 30
    // 这里的bind是个柯里化的函数
    var bindTest2 = test.bind({ a: 20 }, 10)
    bindTest2()  // 30
    ```


  - 绑定构造函数无效

    - 其实准确的来说，bind并不是对构造函数无效，只是对new的时候无效，如果直接执行构造函数，那么还是有效的。

    ```js
    var a = 10
    var Test = function (a) {
      console.log(this.a)
    }
    var bindTest = Test.bind({ a: 20 })
    bindTest() // 20
    // 在new的时候，Test中的this并没有指向bind中的对象
    new bindTest()
    ```


- 实现一个简单的bind

  - 由于是在函数上调用bind，所以bind方法肯定存在于Function.prototype上面

  - 其次bind函数要有改变上下文的作用，我们想一想，怎么才能改变上下文？没错，就是call和apply方法。

  - 然后还要可以柯里化，还好这里只是简单的柯里化，我们只要在bind中返回一个新的函数，并且将前后两次的参数收集起来就可以做到了。

  - 这里我们还需要考虑一下，怎么做才能让在new的时候无效，而其他时候有效？我们需要在returnFunc里面的apply第一个参数进行判断，如果是用new调用构造函数的时候应该传入函数本身，否则才应该传入context。

    ```js
    Function.prototype.bind = function() {
      var args = arguments || [];
      var context = args[0];
      var func = this;
      var thisArgs = Array.prototype.slice.call(args, 1);
      var returnFunc = function() {
        Array.prototype.push.apply(thisArgs, arguments);
        // 最关键的一步，this是new returnFunc中创建的那个新对象，此时将其传给func函数，其实相当于做了new操作最后一步（执行构造函数）
        return func.apply(this instanceof func ? this : context, thisArgs);
      }
      returnFunc.prototype = new func()
      return returnFunc
    }
    ```

### 十二、数组打平（扁平化）

- 数组的扁平化其实就是将一个[嵌套](https://so.csdn.net/so/search?q=%E5%B5%8C%E5%A5%97&spm=1001.2101.3001.7020)多层的数组 `array`（嵌套可以是任何层数）转换为只有一层的数组

##### 1.方法一：普通的递归实现

- 普通的递归思路很容易理解，就是通过循环递归的方式，一项一项地去遍历，如果每一项还是一个数组，那么就继续往下遍历，利用递归程序的方法，来实现数组的每一项的连接。

  ```js
  const a = [1, [2, [3, [4, 5]]]];
  const flatten = (arr) => {
    let result = [];
    for (let i = 0; i < arr.length; i++) {
      if (Array.isArray(arr[i])) {
        result = result.concat(flatten(arr[i]));
      } else {
        result.push(arr[i]);
      }
    }
    return result;
  };
  console.log(flatten(a));
  ```

##### 2.方法二：利用reduce函数迭代

```js
const flatten = (arr) => {
  return arr.reduce((prev,next)=>{
    return prev.concat(Array.isArray(next)?flatten(next):next)
  },[]);
};
```

##### 3.方法三：扩展运算符实现

```js
const flatten = (arr) => {
  while(arr.some(item=>Array.isArray(item))){
    arr = [].concat(...arr);
  }
  return arr;
};
```

##### 4.方法四：split和toString共同处理

- 我们也可以通过 `split`和 `toString` 两个方法，来共同实现数组扁平化，由于数组会默认带一个 `toString`的方法，所以可以把数组直接转换成逗号分隔的字符串，然后再用 `split`方法把字符串重新转换为数组

  ```js
  const flatten = (arr) => {
    return arr.toString().split(',')
  };
  ```

##### 5.方法五：调用ES6中的flat

- 我们还可以直接调用 `ES6` 中的 `flat`方法，可以直接实现数组扁平化

  ```js
  const flatten = (arr) => {
    return arr.flat(Infinity)
  };
  ```

##### 6.方法六：正则和JSON方法共同处理

- 采用了将 `JSON.stringify` 的方法先转换为字符串，然后通过正则表达式过滤掉字符串中的数组的方括号，最后再利用 `JSON.parse` 把它转换成数组。

  ```js
  const flatten = (arr) => {
    let str = JSON.stringify(arr);
    str = str.replace(/(\[|\])/g,'');
    str = `[${str}]`;
    return JSON.parse(str);
  };
  ```

  ​






  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

  ​

