# ES6

## 一、ES5及JS补充

```js
ECMAscript javaScript
js   bom  dom  ECMAScript
ECMA是js 核心语法标准
js 是ECMA的语法体现
```

### 1.ES5下的严格模式

```js
相比之下比以前的限制性更强
IE6-9不能支持严格模式;
严格模式的作用？为什么设立严格模式？
1.消除js语法的一些不合理、不严谨，减少一些怪异行为;
2.消除代码行为的不安全处，保证代码运行安全;
3.提高编辑器效率，增加运行速度;
4.为未来的版本做好铺垫;

```

### 2.使用严格模式

```js
将"use strict" 放在脚本文件的第一行，则整个脚本都是以严格模式运行;
将"use strict" 放在脚本文件的不是第一行，则无效;继续以正常的模式运行;
```

### 3.应用场景

```js
"use strict"
var globalVal  =100;
console.log(globalVal);


function fn()
{
    'use strict'
    var globalVal  =200;
    console.log(globalVal);
}
fn()
严格模式与正常模式没有区别

严格模式失效的情况
1."use strict" 前面有代码
2."use strict"前面有空语句   ;
```

### 4.注意事项

```js
JS弱类型语言，不使用var 声明的变量会默认转为全局变量，严格模式下，报错
"use strict"
age = 10;//var  age; age =10;
console.log(age);//age is not defined

局部模式
function fn()
{
'use strict'
los = 200;
console.log(los);
}
fn();
全局for循环
"use strict"
for(i=1;i<=10;i++)
{
    console.log(i);
}
函数有重名的参数报错
'use strict'
function fn(a,a)
{
    alert(a);
}
fn()

arguments严格定义为参数，不在于形参绑定
function fn(a)
{
    arguments[0] =2 ;
    alert(a);
}
fn(1);
函数调用，传参为1，函数内部通过agruments修改为2，弹2

'use strict'
function fn(a)
{
    arguments[0] =2 ;
    alert(a);
}
fn(1);
不可修改

'use strict'
function fn(a) {
    arguments[0] = 2;
    alert(arguments[0]);//2
}
fn(1);
严格模式下中仍然被修改为2.

```

### 5.eval()函数

```js
作用：计算javascript字符串，并把它把他当做脚本代码来执行
如果参数是表达式,evel()函数将执行表达式
如果参数是js语句，执行js语句
语法：
	eval(string)//必须，要计算的字符串
```

### 6.eva()用法

```js
如果str执行结果是一个值，返回这个值，否则是undefined
console.log(eval("var a =1"));//undefined 声明变量a，并且赋值1
console.log(eval("2+3"));//5执行加计算，并返回这个值

eval("test()")//1 执行test函数
function test()
{
    console.log(1);
}
console.log(eval("{b:2}"));//2
// 当你声明一个对象，并且想反悔这个对象时，需要在对象外面嵌套小括号
console.log(eval("({b:2})"));//{b：2}
```

### 7.eval()函数的作用域

```c
eval函数并不会创建一个新的作用域，并且它的作用域就是eval所在的作用域
主流浏览器有什么？有几个？ IE 火狐  谷歌  Safari  欧朋
function a()
{
    eval("var x = 1");
    console.log(x);
}
a();
console.log(x);
但是有时候需要设置成全局,window.eval()
function a() {
    window.eval("var x = 1");
    console.log(x);
}
a();
console.log(x);
```

### 8.兼容所有浏览器

```js
function a()
{
    if(window.exceScript)
    {
        window.exceScript("var x =1")
    }
    else{
        window.eval("var x =1")
    }
    console.log(x);
}

a();
console.log(x);
```

### 9.严格模式下eval()函数

```js
js作用两种   全局作用域   函数作用域（局部作用域）   严格模式下，带来了第三种 eval()作用域
//严格模式
"use strict"
var a = 10;
eval("var a =20;console.log(a)")
console.log(a);
//非严格模式下,会污染全局，严格模式下,会形成自己独立的作用域
```

### 10.eval()操作不生效

```js
'use strict'
var  eval = 3;//变量名使用
var obj = {};
obj.eval = 1;//对象属性名
obj.a =eval;//对象属性值
for(var eval in obj){}//变量
function  eval(){}//函数名
function a(eval){}//函数参数
```

### 11.JSON对象

```js
JSON  对象两个方法，并且JSON通过用于客户端（前端）与服务器端交换数据.

JSON.stringify()  的作用是将js对象转换为JSON字符串
JSON.parse()   可以将我的JSON字符串转化为一个对象

......
var arr = [1,2,3];
console.log(JSON.stringify(arr));//'[1,2,3]'
// console.log(typeof arr);
console.log(typeof JSON.stringify(arr));//string

......
var str = '[1,2,3]';
console.log(JSON.parse(str));//'[1,2,3]'
console.log(typeof str);
console.log(typeof JSON.parse(str));//string

//在使用JSON.parse()需要注意，将JSON字符串转化为对象
//你的字符串必须符合JSON格式，也就是键值对的形式，且需要用双引号包含
var  json = '{"name":"1","age":"18"}';
console.log(JSON.parse(json));
```

### 12.页面跳转应用

```js
location.herf = `跳转的页面？参数1&参数2`
location.search;//截取地址栏？及？以后的内容
识别中文   decodeURL()     encodeURL()

var str = decodeURI(location.search);//截取地址栏？及？以后的内容
var arr = str.slice(1).split("&");
// console.log();
document.write(arr[0].slice(arr[0].indexOf('=')+1) + ",欢迎您")
var str1 = JSON.stringify(arr);
console.log(str1);
```

### 13.Object扩展

#### Object.create(prototype,descritors)

```js
作用：指定对象为原型创建的新对象，为新对象指定新的属性，并做描述
第一个参数：将新对象为原型   第二个参数:为新对象添加新的属性，包含描述性的属性
value:属性值
writeable:是否可以修改新的属性，默认false.不可修改
configurable:是否可以删除新的属性 ，默认false，不可删除
ennumerable:是否可以枚举，默认false ，不可枚举


创建新对象，使现有的对象来提供新对象的proto
var Person = {
    flag: false,
    print: function () {
        console.log(1);
    }
}
//使用create基于原有对象person创建一个对象

var me = Object.create(Person);
me.name = "xiali";
me.flag = true;
me.print();

console.log(me);

var person1 = {"name":"yl","age":18};
var person2 = Object.create(person1,
{
    sex:{
        value:'男',
    },
    sex2:{
        value:'男'
    },
    des:{
        value:"这真的是男的"
    }
})



var person1 = {"name":"yl","age":18};
var person2 = Object.create(person1,
{
    sex:{
        value:'男',
        writable:true,//可修改的
        configurable:true,//可删除
        enumerable:true //是否可枚举  是否可遍历（循环）
    },
    sex2:{
        value:'男'
    },
    des:{
        value:"这真的是男的"
    }
})
person2.sex = "nv";
delete person2.sex;

for(var attr in person2)
{
    console.log(attr,person2[attr]);
}
console.log(person2.sex);
console.log(person2.sex2);
console.log(person2);
```

### 14.数组扩展

```js
迭代方法。又称高阶函数(IE8以上支持)
```

#### （1）forEach()

```js
没有返回值，就是一个简单的循环，可以代替for循环
格式：
	数组.forEach(function(数组元素,数组下标,数组本身)
    {
        操作数组的代码块;
    })
var arr = [9,5,2,7];
arr.forEach(function(item,index,arr)
{
    console.log(item,index,arr);
})
console.log("***************");
for(var i=0;i<arr.length;i++)
{
    console.log(arr[i],i,arr);
}
```

#### （2）map

```js
返回一个新数组
返回的数组和原数组是一一对应的关系
格式：
	数组.map(function(数组元素,数组下标,数组本身)
var arr = [9,5,2,7];
var brr = arr.map(function(item,index,arr)
{
    console.log(item,index,arr);
    return item*2;
})
console.log(brr);
console.log(arr);
```

#### （3） filter

```js
针对数组元素做某些判断,满足条件的元素，会组成一个新数组,并返回
//map:对数组中每一项运行的数组，成立会返回true，不成立false
对数组中每一项给定的函数，返回该函数会返回true的项组成的数组
格式：
	数组.filter(function(数组元素,数组下标,数组本身){
        操作数组的代码块;
    }

var arr = [9,5,2,7];
var brr = arr.filter(function(item,index,arr)
{
    console.log(item,index,arr);
    return item >=5;
})
console.log(brr);
```

#### （4）every

```js
针对数组元素做某些判断,满足条件的元素，返回结果为true
结果都为true,返回值为true，如果有一个是fasle.返回值就是false
格式：
	数组.every(function(数组元素,数组下标,数组本身){
        操作数组的代码块;
    }
             
var arr = [9,5,2,7];
var brr = arr.every(function(item,index,arr)
{
	console.log(item,index,arr);
	return item >=1;
})
console.log(brr);
```

#### （5）some

```js
针对数组元素做判断，如果有一个是true，则返回true
格式：
	数组.some(function(数组元素,数组下标,数组本身){
        操作数组的代码块;
    }
            
 var arr = [9,5,2,7];
var brr = arr.some(function(item,index,arr)
{
	console.log(item,index,arr);
	return item >=10;
})
console.log(brr);
```

#### （6）区别

```js
forEach()  循环遍历数组
map()      是对数组元素做操作，并返回一个新的数组
filter()   数组元素满足某一条件，并以数组的形式，返回满足条件的元素
every()    针对数组元素操作,只有所有的元素满足某一个条件,返回值是true,只要有一个不满足，返回fasle
some()     针对数组元素做操作,只要有满足条件的元素，那么就形成一个数组并返回
```



## 二、ES6简介

### 1.let基本用法

ES6增加了let 命令，用来声明变量。它的用法 和var类似，但是let声明的变量。只有let所在的代码块有效

```js
{
    let b = 2;
    var a =1;
}

console.log(a);
console.log(b);
表明 let声明的变量只在它所在的代码块有效
```

for循环的计数器，就很适合let

```js
for(let i =0;i<=10;i++)
{
    console.log(i);
}
console.log(i);//i is not defined
```

```js
var a = [];
for(var i=0;i<10;i++)
{
    a[i] = function()
    {
        console.log(i);
    };
}
a[6]()//10
i var ,在全局有效，全局只有一个i,log(i)  的i 指向就是全局i  所有的i 指向全局。
```

如果每一轮的i都是重新声明的，怎么知道上一轮循环的值，从而计算出本轮的值？

js引擎内部会记住上一轮的值，初始化本轮变量i时，就是在上一轮的基础上进行计算的

```js
特别之处，设置循环变量的那部分是一个父作用域,而循环题内部是一个单独字作用域

for (let i = 0; i < 3; i++) {
    let i='a';
    console.log(i);//3个a
}
```

### 2.不存在变量提升

var 命令会发生变量提升的现象

```js
// var 的情况
console.log(foo);//undefined
var foo =2;
// let情况
console.log(bar);
let bar =2;//Cannot access 'bar' before initialization

变量foo用var声明，会发生变量提升，也就是说脚本在运行之前,foo就已经存在了，但是没有值，所以是undefined

bar 用 let 命令，没有变量提升，表明在声明之前，bar 不存在，那么你要用的bar话,给你报错
```

### 3.暂时性死区

只要块级作用域内存在let命令,它所声明的变量就绑定在了这个区域，不会再受外部影响

```js
var temp = 123;
if(true)
{
	temp = 'abc';  // ReferenceError: Cannot access 'temp' before initialization
    let temp;
}
存在全局变量temp,但是跨级作用域内let声明了一个temp,导致后者绑定在了这个块级作用域下。
```

ES6明确规定,如果我的区块中存在let 和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用，凡是在声明之前就用，就会报错。



在代码块内，使用 let命令声明变量之前，该变量都不可用。语法上称：暂时性死区(temporal dead zone)

简称  ：TDZ

```js
if(true)
{
    //TDZ开始
    tmp = 'abc';//
    console.log(tmp);//ReferenceError
    let tmp;//TDZ结束
    console.log(tmp);//undefined
    tmp = 123;
    console.log(tmp);//123
}
在 let 命名声明变量tmp之前。都是属于变量tmp的“死区“

```

TDZ也就意味着typeof 不再是百分百安全的操作

```js
typeof(x)   typeof x
let x
x在let声明之前属于x的TDZ,只要用，就报错，
```

作为比较，如果变量没有被声明，不会报错typeof

```js
 console.log(typeof x);//undefined
```

TDZ隐藏比较深

```js
// function bar( x=y, y =2)//y是变量
// {
//     return [x,y];
// }

// bar()
调用bar 函数 之所以报错，因为参数x默认等于另一个参数y,此时y没有声明，属于TDZ
```

如果y的默认值是x,就不会报错，因为此时的x已经声明了

```js
function bar(x=2,y=x)//y是变量
{
    return [x,y];
}
bar();//2  2
```

```js
var x = x; //不报错
let x = x;//ReferenceError: Cannot access 'x' before initialization

使用let声明变量时，只要变量在还没有声明前使用，就会报错，
```

ES6规定TDZ和let 、const语句，不在出现变量提升，主要减少运行时的小错误，防止变量声明前就使用

**TDZ总结:**

​	TDZ本质就是,只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行出现，才能获取和使用该变量。

### 4.不允许重复声明

let不允许在相同作用域下，重复声明。

```js
//报错
function fn()
{
    let a  =1;
    let a =2;

}
//报错
function fn()
{
    var a  =1;
    let a =2;

}
//报错
function fn(a)
{
    let a;
}
fn();//
function fn(a) {
    { 
        let a;
    }
}
fn();//不报错
```

### 5.块级作用域

5.1为什么需要块级作用域？只有全局和函数。没有块级。不方便

#### （1）第一种场景

内层变量可能会覆盖外层变量

```js
var tmp = new Date();
        function fn()
        {
            console.log(tmp);//变量提升
            if(false)
            {
                var tmp = "hello";
            }
        }
        fn()//undefined
变量提升，导致内部的tmp变量覆盖了外部的temp变量
```

#### （2）第二种场景

用来计数的循环变量泄露为全局变量

```js
var  s = "hello";
for(var i = 0;i<s.length;i++)
{
    console.log(s[i]);
}

console.log(i);
```

### 6.ES6中的块级作用域

**let 就是为了js新增块级作用域**

```js
function fn()
{
    let n = 5;
    if(1)
    {
        let n =10;
    }
    console.log(n);//5
}

都声明了n，运行输出5，表示内层与外层代码块不会相互影响，如果var定义n.最后10
```

**ES6允许跨级作用域任意嵌套**

```js
{{{{{{{
    let a =  "hello";}
    console.log(a);
}}}}}}
每一层都是独立的作用域，第六层没办法读取第七层的内容
```

**内层作用域定义变量可以和外层重复**

```js
{{{{{{{
    let a =  "hello";}
    let a = 1;
}}}}}}
```

```js
ITFE写法
(function()
{
    var tp = 'aaa';
}());

块级作用域的写法

{
    let tmp = '';
}
```

### 7.块级作用域与函数声明

es5规定，函数只能在顶层作用域和函数作用域之间声明，不能在跨级作用域声明

```js
if(1)
{
    function fn()
    {
        console.log(1);
    }
    fn();
}

try
{
    function f(){}
}catch(e)
{
    //...
}


ES6引入块级作用域，明确允许在块级作用域中声明函数，块级作用域，函数声明语句的行为，类似let,b不可引用
function fn()
{
    console.log("out");
}
(function()
{
    if(0)
    {
        //重复声明一次函数fn
        function fn()
        {
            console.log("in");
        }

    }
    fn()
}());

function fn(){console.log("out");}

(function ()
{
    function fn(){console.log("in");}
    if(0)
    {
		//原来的函数，经过声明提升，到了函数头部，所以可以打印in
    }
    fn()
}())

```



### 8.const命令

#### 8.1基本用法

const声明的是一个只读常量，一旦声明，常量的值就不能改变

```js
const PI = 3.1415926;

//PI 3.1415926
PI = 3;
//Assignment to constant variable

const声明的变量不可改变,const一旦声明变量，就必须立即初始化，不能留到以后赋值
const bar;//Missing initializer in const declaration

const的作用域与let 相同，只在声明所在的块级作用域内有效

if(1)
{
    const mm = 5;
}
console.log(mm);
//ReferenceError: mm is not defined

const 命令声明的常量也不提升，同样存在TDZ,只能在声明位置后使用
if(1)
{
	console.log(aa);//ReferenceError
    const aa = 5;
}
const 与 let 都不可以重复声明

var  m = "aaa";
let ag =18;
const m ="bbbb";
const ag = 19;


```

#### 8.2本质

`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，`const`只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。

```js
const foo = {};
//为对象添加属性
foo.prop = 123;

//将foo 指向另一个对象，报错
foo = {}//TypeError: Assignment to constant variable.


const a = [];
a.push("aa");//
a.length = 0;

a = ["hekk"]
常量a是一个数组，本身可写，但是你另一个数组赋值给a,报错



如果真的将对象冻结，应该使用Object.freeze方法
const foo = Object.freeze({});
常规模式下,不起作用;
严格模式下,报错
foo.prop =123;
```

### 9.解构赋值

ES6允许我们按照一定的模式，将数组或对象中的数据提取出来，赋值给变量；这个过程叫解构赋值

#### 9.1数组的解构赋值

对于数组，一直都是用下标提取，存储有序，基于有序的情况下，才可以提取

对于对象，我们是key 提取，按照hash结构，基于hash特征，就是按照key value形式

```js
let arr = [1,2,3,4];
let stu = {"name":"xiaoli","age":18};

// es5的提取方法
console.log(arr[0]);  //通过下标
console.log(stu.name);//key
console.log(stu['name']);
//es5的变量赋值


let a =1;
let b =2;
let c =3;

es6允许写成     let  [a,b,c] = [1,2,3]
按照对应位置，对变量进行赋值
属于模式匹配 ，只要等号两边的模式相同，左边的就会被赋值

let [foo,[bar],baz] = [1,[2],3];
console.log(foo,bar,baz);//1 2 3


let [foo,[[bar],baz]] = [1,[[2],3]];
console.log(foo,bar,baz);//1 2 3


let  [,,third] = ["foo","bar","baz"];
console.log(third);//baz


let [x,,y] =[1,,3];
console.log(x,y); 


let [x,y,...z] = ['a'];

console.log(x,y,z);//如果解构不成功，变量的值就是undefined

let  [head,...tail] = [1,2,3,4,5,6,7];
console.log(head,tail);

不完全解构
let  [x,y] = [1,2,3,6,4];
console.log(x,y);//1 2

let [a,[b],d] = [1,[2,3,4],9];
console.log(a,b,d);//1  2  9

如果等号右边的不是数组
//报错
let  [foo] = 1;
let  [foo] = false;
let  [foo] = NaN;
let  [foo]  = undefined;
let  [foo] = null;
let  [foo] = {}

默认值

// let [foo = true] = [];
// console.log(foo);//true

// let [x,sex = 'n'] = ['a'];
// console.log(x,sex);  //   a   n
let [x,y= 'b'] = ['a',undefined];
console.log(x,y);//a b
注意：ES6内部使用 === 判断一个位置是不是有值,所以说只有数组成员严格等于undefined 默认值才会生效

let [x =1] = [undefined]
console.log(x);  //1

let [y =1] =[null];
console.log(y); 
数组成员是null,默认值不生效，因为null 不严格等于undefined
```

#### 9.2对象的解构赋值

```js
let {foo,bar} = {foo:"aaa",bar:"bbb"}

console.log(foo,bar);
数组元素按次序排列的，值由位置决定  ，变量必须与属性同名，才能取得值

let {baz} = {foo:"aaa",bar:"bbb"}

console.log(baz);//undefiend

第一个例子，等号左边的次序与右边属性名不同。完全没有影响
第二个例子:变量没有与之对应的同名属性，所以等于undefined

let {foo} = {bar:'zad'};
console.log(foo);


const {log} = console;
log("hello")
将console.log 赋值给log

如果变量名和属性名不同，
let {foo:baz} = {foo:'aaa',bar:'bbb'}
console.log(baz);


let obj = {f:'hello',l:'word'};
let {f:first,l:last} = obj;
console.log(first,last);


let obj = {
    p:[
        'hello',
        {y:'word'}
    ]
};

let {p :[x,{y}]} = obj;
console.log(x,y);//hello word
p是模式。不是变量，不会被赋值。

想要p也变量赋值
let {p,p:[x,{y}]} =obj;


let {foo:{bar}} = {baz:'baz'};// Cannot read properties of undefined (reading 'bar')
等号左边对象的foo属性，对应一个子对象，该子对象的bar属性，解构时报错，因为foo等于undefined,再去子属性报错


默认值
// var {x = 3} = {};
// console.log(x);

// var {x,y =5} ={x:1}

// console.log(x,y);//1  5

// var {x:y=3} = {};

// console.log(y);//3

var {x:y=3} ={x:5}

console.log(y);//5
默认值生效的条件是：对象的属性值 严格等于 undefined

var {x=3} = {x:undefined}
x//3
var {x=3} ={x:null};
x//null




```

#### 9.3函数参数解构赋值

```js
function add([x,y])
{
    return x+y;
}
add([1,2])//3

函数add的参数表面是个数组,但是传入参数的那刻，数组参数就会被结构成变量x和y，对于函数，感受到就是x，y

函数参数的解构也可以使用默认值
function f({x=0,y=0} = {})
{
    return [x,y];
}

f({x:3,y:8})  //[3,8]
f({x:3});     //[3,0];
f({})         //[0,0];
```

#### 9.4圆括号

解构赋值方便但是不知这个式子到底是模式，是表达式。必须解析到（解析不到）等号才知道

只要可能导致歧义（错误），就不能使用原括号

##### 9.4.1不能使用的情况

```js
1.变量声明语句
let [(a)] = [1];
let {x:(c)} = {}

let ({x:c}) = {}
let {(x):c} ={}
2.函数参数
function f([(a)]){
    return a;
}

function f([z:(x)]){
    return x;
}

//全部报错
```

##### 9.4.2能使用的情况

```js
赋值语句的非模式部分，可以使用圆括号
[(b)] = [3];
({p:(d)} = {});
是赋值语句。不是声明语句
```

#### 9.5用途

##### 9.5.1交换变量的值

```js
let x  =1;
let y =2;
[x,y] = [y,x];
console.log(x,y);
简洁 易读 语法清晰
```

##### 9.5.2从函数返回对个值

```js
函数只能返回一个值,若果你要返回多个值,只能放在数组或者对象中返回
// 数组
function f()
{
    return [1,2,3];
}
let [a,b,c] = f();

// 对象
function fn()
{
    return {
        foo:1,
        bar:2
    }
}
let {foo,bar}  = fn()
```

##### 9.5.3提取JSON数据

```js
let sdu = {
    id:1,
    score:"98",
    data:[9899,7898]
};

let {id ,score,data:number} = sdu;

console.log(id,score,number);

利用解构赋值快速提取JSON数据的值;
```

### 10.模板字符串

```js
// es5的拼接
var name = "xiaoming";
var age = 18
var  sex = "男"
document.body.innerHTML = "姓名是" + name + "今年是" + age +"岁" + ",性别是"+sex;

重点：
ES6引入了模板字符串（template string）  用反引号`表示;
document.body.innerHTML = `姓名是${name},今年是${age}岁，性别是${sex}`;


普通字符串
`i lobe '\n'javascript,`
d多行字符串

`in js  yis   is
not enddd
`

 console.log(`qqqqqq
                    wwww   `)
字符串嵌入变量
// es5的拼接
var name = "xiaoming";
var age = 18
var  sex = "男"
document.body.innerHTML = `姓名是${name},今年是${age}岁，性别是${sex}`;

如果模板字符串中需要反引号，则前面需要用到反斜杠转义
let ee = `\`uo\` word`;
console.log(ee);

使用模板字符串，表示多行字符串
div.innerHTML = data.map(function(item)
                        {
    return `<li>${item</li>`
})
div.innerHTML = `
    <ul>
    	<li>qq</li>
    	<li>ss</li>
    </ul>
    `.trim()

trim 消除空格或者换行

大括号内部可以放入任意js表达式，可以运算，以及引用对象属性
let x  =1;
let y  =2;

`${x}+${y} = ${x+y}`;//相当于  "1+2 =3";

模板字符串中可以调用函数
function fn()
{
    return "helo"
}
`foo${fn()} bar`
//foo helo bar

//报错 pas声明
let msg = `hello ${pad}word`;

模板字符串的嵌套

const  temp = `
		<table>
    	${dd.map(function(item)
                {
            	return `
					<tr><td>${add.fi}</td></tr>
			`
        })}
    </table>
`
```

### 11.对象扩展

##### 11.1属性的简洁表式

```js
es6允许在大括号里面,直接写入变量和函数，作为对象的属性和方法
const foo = "bar";
const baz = {foo};
baz   //{foo:"bar"}

等价于    
const  baz = {foo:foo}

```

##### 11.2属性名表达式

js定义对象的属性，两种方式

```js
1.obj.foo = true;

2.obj['a'+'bc'] = 1234;

方法1，直接用标识作为属性名
方法2，用表达式作为属性名
```

### 12.箭头函数

ES6新增一种函数：Arrow Function(箭头函数)

#### 12.1基本用法

```js
ES6允许使用"箭头"  (=>) 定义函数
var f = v =>v;

等价于
var  f = function(v)
{
	return v;
}
如果箭头函数不需要参数或者需要多个参数，就使用圆括号代表参数

var f = ()=>5;
等价于
var f = function()
{
    return 5;
}


var sum=(num1,num2) => num1+num2;
等价于
var sum = function(num1,num2)
{
return num1+num2;
}

如果箭头函数的代码块对于一条语句,就要用大括号包含起来，并且使用 return 语句返回

var sum = (num1,num2) =>{return num1 + num2};


[1,2,4].map(function(item)
           {
    return item*2;
})

//箭头函数
[1,2,3].map(item => item*2)


var result = aa.sort(functin(a,b)
{
	return a-b;
})

//箭头函数
var result = aa.sort((a,b) =>a-b);

```

#### 12.2注意事项

1.函数体内的this对象,就是定义时所在的对象。而不是使用时所在的对象（而不是谁调用指向谁）

2.不可当做构造函数，不能使用new,否则会报错

3.不可以使用arguments对象，该对象在函数体内不存在，如果要用。使用rest参数。

#### 12.3箭头函数中的this

```js
箭头函数是没有自己的this ,箭头函数里面的this会继承上级的this,
箭头函数中的this就是外层代码块的this

普通函数
var fn1  =()=>
{
    console.log(this)//window
}
fn1()
事件
var box = ....id('box');
box.onclick = ()=>
{
    console.log(this);//window
}

对象
var  x =11;
var obj1 = {
    x:22,
    say1:()=>{
        console.log(this.x);//11
        console.log(this);window
    }
}
obj1.say1();//11


var x = 11;
   var obj  ={
    x:22,
    methods:
    {
        x:33,
        say2:()=>{
            console.log(this.x);//11
            console.log(this);//window
        }
    }
   }
   obj.methods.say2() 


定时器

function foo(){
    setTimeout(() => {
        console.log("id",this.id);//21
    }, 100);
}
var id = 21;
foo();

构造函数
function Dog(name,age)
{
    this.name  =name;
    this.age = age;
    this.eat = function()
    {
        return ()=>{
            console.log(this);//Dog
        }

    };
    this.ye= ()=>{
        console.log(this);//Dog
    }
}
var a = new Dog("xiaoli",20);
console.log(a);//
a.eat()
a.ye();


var Per = (name) =>{
    this.name = name;
}
var p = new Per("aadd");
console.log(p.name);


箭头函数的作用:将我的this从"动态"变成了"静态";
不适合的场景：
1.定义对象方法，且该方法内部包括this
const cat = {
        lives:9,
        jump:()=>{
            this.lives--;
        }
    }
    cat.jump();

    console.log(cat.lives);

cat.jump()方法是一个箭头函数，调用cat.jump()时，如果是普通函数，该方法内部this指向cat.如上述代码一样，this指向的window,因为对象不构成单独的作用域。

2.是需要动态this的
 var btn = document.getElementById("btn");
    btn.addEventListener("click",()=>{

        this.classList.add("active");
    })

btn监听的是一个箭头函数，导致里面的this就是全局对象，
如果改为普通函数。this会动态指向被点击的按钮对象

3.函数体很复杂，或者有大量的读写操作，也不应该用箭头函数。而是用普通函数。增加代码可读性。



```

#### 12.4箭头函数的嵌套

```js
function insert(value)
{
    return {into :function (array)
    {
        return {after:function(afterValue){
            array.splice(array.indexOf(afterValue) + 1,0,value)
            return array;
        }}
    }}
}


// [1,3]  2   [1,2,3]
let insert  =(value)=>({into:(array)=>({after:(afterValue)=>{
    array.splice(array.indexOf(afterValue)+1,0,value);
    return array;
}})})
console.log(insert(2).into([1,3]).after(1));
```

##### 12.6rest参数

```js
ES6 引入rest参数（形式为...变量名），用于获取函数的多余参数，不需要arguments.
rest参数 搭配的变量是一个数组，该变量将多余的参数放入到数组中

function add(...values)
{
    let sum = 0;
    for(var val of values)
    {
        sum+=val;
    }
    return sum;
}
console.log(add(1,2,3,4,));//10
console.log(add(1,2,3,4,5,6,7));//28

```

```js
利用rest参数改写push方法
function push(array,...items)
{
    items.forEach(function(item)
    {
        array.push(item);
        console.log(item);
    })
}

var a = [];
push(a,1,2,3,4);
```

```js
注意：rest参数之后不能再有其他参数（rest参数必须在最后一位），否则报错
function  f(a,...b,c)
{
    //
}
f()函数的长度不包括rest参数
console.log((function(a){}).length);

console.log((function(...a){}).length);
console.log((function(a,...b){}).length);
```

### 13.Symbol类型

```js
Number  String Boolean  undefined  null   Object(对象)    js第七种 数据类型
表示独一无二的值

```

##### 13.1定义Symbol

```js
Symbol 值通过Symbol 函数生成

let s = Symbol();

 let s = Symbol();
        console.log(typeof s);
变量s是Symbol类型，而不是其他类型

注意：
	Symbol 函数前不能使用new，报错，因为生成SYmbol是一个原始类型的值
    

    接受一个字符串作为参数，表示对Symbol实例的描述
    
    let s1  = Symbol("foo");
        let s2 = Symbol("bar");

        console.log(s1);
        console.log(s2);
加字符串的原因就是为了区分

        console.log(s1.toString());
        console.log(s2.toString());



注意：函数的参数只是表示对当前Symbol值的描述，因此相同的参数的Symbol函数的返回值是不相等的


let s1 = Symbol();
let s2 = Symbol();
console.log(s1===s2);//false

let s3 = Symbol("foo");
let s4 = Symbol("foo");
console.log(s3===s4);//false
```

##### 13.2作为属性名的Symbol

```js
由于每一个Symbol值都是不相等的，可以作为标识符用以对象的属性名，就能保证不会出现同名的属性。防止某一个键被不小心改写或覆盖

// 1
// let a  ={};
// a[mySymbol] ="hello";
// 2.
let a = {
    [mySymbol]:"hello"
}


console.log(a[mySymbol]);

注意：Symbol值作为对象属性名是，不能使用   .   运算符
let mySymbol = Symbol();


const  a = [];

a.mySymbol = "helo";
console.log(a[mySymbol]);//ubdefined
console.log(a["mySymbol"]);//helo

因为  .  运算符 后面总是字符串,所以不会读取mySymbol作为标识符所指代那个值，导致a的属性名实际上是字符串，而不是Symbol值



```

### 14.Set

ES6提供了新的数据结构Set,类似数组，但是成员的值都是唯一的

```js
Set本身就是一个构造函数，用来生成Set的数据结构
// const s = new Set();
// [2,3,4,5,2,3].forEach(item=>s.add(item));

// for (let i of s)
// {
//     console.log(i);
// }
let arr = [1,2,3,4,6,1,2,4];

[...new Set(arr)] //12 346

[...document.getElementsByTagName("div")] //类数组转为数组
Array.from(类数组)


let set  = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);

console.log(set);//{NaN}

NaN === NaN fasle
只加入了一个NaN,在Set内部,两个NaN相等

let set  = new Set();
set.add({})

console.log(set);

set.add({})
console.log(set);
两个空对象不相等，两个值
let set  = new Set();
set.add({})

console.log(set);
console.log(set.size);//1

set.add({})
console.log(set);
console.log(set.size);//2



Set实例的属性和方法
Set的实例属性：
			Set.prototype.constructor :构造函数，默认就是Set函数
            Set.prototype.size :返回Set实例的成员总数
Set的实例方法：
			操作方法：
            		Set.prototype.add(value) 添加某个值，返回Set结构本身
                    Set.prototype.delete(value) 删除某个值，返回布尔值，表示是否删除成功
                    Set.prototype.has(value) 返回布尔值,表示该值是否为Set的成员
                    Set.prototype.clear(value) 清除所有成员，没有返回值
                    
            遍历方法：Set.prototype.keys()	   返回键名的遍历器
            		Set.prototype.values()    返回键值的遍历器
                    Set.prototype.entries()   返回键值对的遍历器
                    Set.prototype.forEach()   使用回调函数遍历me几个成员
                                		
4个操作方法
let s = new Set();
s.add(1).add(2).add(2);//2被加了两次

console.log(s);{1,2}
console.log(s.size);//2

console.log(s.has(2));//true
console.log(s.has(3));//fasle

s.delete(2);
console.log(s.has(2));//fasle

s.clear()
console.log(s);


4个遍历方法
遍历顺序就是插入顺序

Set结构没有键名，只有键值(键名和键值是同一个值)   keys  values方法完全一致


let s = new Set(['pear','orange','watermelon']);

for (let item of s.keys())
{
    console.log(item);
}
// pear orange watermelon
for (let item of s.values())
{
    console.log(item);
}
// pear orange watermelon

for (let item of s.entries())
{
    console.log(item);//键名等于键值
}
//['pear', 'pear']
// ['orange', 'orange']
//['watermelon', 'watermelon']


for(let item of s)
{
    console.log(item);
}
//直接省略values

let  set = new Set([1,4,9]);
set.forEach((value,key)=>console.log(key +":"+value))
// 1:1
// 4:4
// 9:9
和数组的forEach参数一样


[...]  扩展运算符  和Set结合，就可以去除数组的重复元素
 
 
 集合
 交集(Intersect)   并集(Union)    差集(Difference)
 // 并集

 let Un  = new Set([...a,...b]);
 console.log(Un);
 // 交集
 let In  = new Set([...a].filter(item=>b.has(item)));
 console.log(In);

 // 差集
 let Dn  = new Set([...a].filter(item=>!b.has(item)));
 console.log(Dn);
                                		
```



### 15、Map

#### 15.1含义与基本用法

```js
js对象，键值对的集合（Hash结构），传统上只能把字符串当做键
const  data = {}
const  oDiv = docment.getElementById('box');//<div id = "box"></div>

data[oDiv]  ="mmmmaas";
oDiv  ---->   '[object HTMLDivElement]'

为了解决此问题，Es6提供了Map数据结构，类似于对象，也是键值对的集合，但是“键”的范围不限于字符串
Map 结构    完美的体现Hash结构，Map比Objct 更合适。

const  m = new Map();
const  o = {p:"hello"};
// m.set(把...作为键)
m.set(o,'content');//设置
//获取
console.log(m.get(o));


```

Map可以接受一个数组作为参数，成员是一个个表示键值对 的数组

```js
const ss = new Map([
    ['name', 'xiaoli'],
    ['age', '18']
]
);
console.log(ss.size);//2
console.log(ss.has('name'));//true
console.log(ss.get("name"));//xiaoli
ss.delete("name");
console.log(ss.has('name'));//true


const ss = new Map([
    ['name', 'xiaoli'],
    ['age', '18']
]
);
const sss = new Map();
ss.forEach(
    ([key,vale]) => sss.set(key,value)
);
```

事实上,不只有数组，任何具有iterator 接口、且每个成员都是双元素的数组的数据结构，都可以是Map参数

Set和Map都可以生成新的Map;

```js
const set1 = new Set([
    ['foo',1],
    ['bar',2]
]);
const aa = new Map(set1);
console.log(aa.get('foo'));

const bb = new Map([['baz',3]]);
const cc = new Map(bb);
console.log(cc.get('baz'));
```

```js
const aa = new Map();
aa.set(1,'aaa').set(1,'bbb');

console.log(aa.get(1));
上述代码对键1连续赋值两次，后一次覆盖前一次的值


如果我读取一个未知的键,返回undefined
new Map().get("ahsjhjhfwjefiwj ")  //undefined
```

```js
set get  表面是针对同一个键，但是实际上是两个不同的数组实例,内存地址是不一样的。因为get无法读取该键
，返回undefined

const aa = new Map();
aa.set(['a'],5555);
console.log(aa.get['a']);

当我传入的键值是对象，再赋值键名，
当我传入的键值是['a'] ,再赋值键名
```

#### 15.2实例方法和实例属性

```js
size()  成员总数
Map.prototype.set(key,value)  
set 方法设置键名key 对应的键值为value,如果key 已经有值，则键值会被更新，否则生成新的键
 const aa = new Map();
        aa.set('aaaa',6);  //键  字符串
        aa.set(234,'ffff'); //键  数值 
        aa.set(undefined,'ddd');  //undefined

链式结构
const aa = new Map().set('aaaa',6).set(234,'ffff').set(undefined,'ddd');

Map.prototype.get(key) 读取key的值，找不到返回undefined

Map.prototype.has(key)  返回boolean ，表示键是或在当前Map中

Map.prototype.delete(key) 删除某个键，返回true,删除失败返回false

Map.prototype.clear()   清除全部成员，没有返回值

遍历方法

Map.prototype.keys()    返回健名遍历器 

Map.prototype.values()  返回键值的遍历器 

Map.prototype.entries()  返回所有成员的遍历器 

Map.prototype.forEach()  遍历Map 的所有成员


[...aaa]
Array.from(aaaa)
```

