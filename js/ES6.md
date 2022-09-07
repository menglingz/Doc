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



