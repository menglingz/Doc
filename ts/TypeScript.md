## TypeScript

### 一、基础类型

- TypeScript支持与JavaScript几乎相同的数据类型，此外还提供了实用的枚举类型方便我们使用

#### 1.布尔值

- 最基本的数据类型就是简单的true/false值，在javascript和typescript中叫做`boolean`

  ```typescript
  let isDone:boolean = false
  ```

#### 2.数字

- 和JavaScript一样，TypeScript里的所有数字都是浮点数。 这些浮点数的类型是 `number`。 除了支持十进制和十六进制字面量，TypeScript还支持ECMAScript 2015中引入的二进制和八进制字面量

  ```typescript
  let decLiteral:number = 6
  let hexLiteral: number = 0xf00d;
  let binaryLiteral: number = 0b1010;
  let octalLiteral: number = 0o744;
  ```

#### 3.字符串

- 像其它语言里一样，typescript使用 `string`表示文本数据类型。 和JavaScript一样，可以使用双引号（ `"`）或单引号（`'`）表示字符串

  ```typescript
  let name:string = 'bob'
  name = 'smitth'
  ```


- 在typescript中还可以使用模板字符串，它可以定义多行文本和内嵌表达式，这种字符串是被反引号包围，并且以`${ str }`这种形式嵌入表达式

  ```typescript
  let name :string = 'Fene'
  let age:number = 37
  let sentence:string = `hllo,my name is ${name}
  I'll be ${ age + 1 } years old next month.`

  <!-- 等价于 -->
  let sentence: string = "Hello, my name is " + name + ".\n\n" +
      "I'll be " + (age + 1) + " years old next month.";
  ```

#### 4.数组

- TypeScript像JavaScript一样可以操作数组元素。 有两种方式可以定义数组。 第一种，可以在元素类型后面接上 `[]`，表示由此类型元素组成的一个数组：

  ```typescript
  let list:number[] = [1,2,3]
  ```


- 第二种方式是使用数组泛型，`Array<元素类型>`：

  ```typescript
  let list:Array<number> = [1,2,3]
  ```

#### 5.元组 Tuple

- 元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为 `string`和`number`类型的元组

  ```typescript
  let x: [string, number];
  // Initialize it
  x = ['hello', 10]; // OK
  // Initialize it incorrectly
  x = [10, 'hello']; // 报错
  ```


- 当访问一个已知索引的元素，会得到正确的类型

  ```typescript
  console.log(x[0].substr(1)); // OK
  console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
  ```


- 当访问一个越界的元素，会使用联合类型替代

  ```typescript
  x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
  console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
  x[6] = true; // Error, 布尔不是(string | number)类型
  ```

#### 6.枚举

- `enum`类型是对javascript标准类型的一个补充。 像C#等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字。

  ```typescript
  enum Color {Red, Green, Blue}
  let c: Color = Color.Green;

  var Color;
  (function (Color) {
      Color[Color["Red"] = 0] = "Red";
      Color[Color["Green"] = 1] = "Green";
      Color[Color["Blue"] = 2] = "Blue";
  })(Color || (Color = {}));
  var c = Color.Green
  ```


- 默认情况下，从`0`开始为元素编号。 你也可以手动的指定成员的数值。 例如，我们将上面的例子改成从 `1`开始编号：

  ```typescript
  enum Color {Red = 1, Green, Blue}
  let c: Color = Color.Green;

  var Color
  ;(function (Color) {
    Color[(Color['Red'] = 1)] = 'Red'
    Color[(Color['Green'] = 2)] = 'Green'
    Color[(Color['Blue'] = 3)] = 'Blue'
  })(Color || (Color = {}))
  var c = Color.Green
  ```


- 全部采用手动赋值

  ```typescript
  enum Color {Red = 1, Green = 2, Blue = 4}
  let c: Color = Color.Green;

  { '1': 'Red', '2': 'Green', '4': 'Blue', Red: 1, Green: 2, Blue: 4 } // Color
  ```


- 枚举类型提供的一个便利是你可以由枚举的值得到它的名字。 例如，我们知道数值为2，但是不确定它映射到Color里的哪个名字，我们可以查找相应的名字：

  ```typescript
  enum Color {Red = 1, Green, Blue}
  let colorName: string = Color[2];

  console.log(colorName);  // 显示'Green'因为上面代码里它的值是2
  ```

#### 7.Any

- 有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 `any`类型来标记这些变量：

  ```typescript
  let notSure: any = 4;
  notSure = "maybe a string instead";
  notSure = false; // okay, definitely a boolean
  ```


- 在对现有代码进行改写的时候，`any`类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为 `Object`有相似的作用，就像它在其它语言中那样。 但是 `Object`类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法：

  ```typescript
  let notSure: any = 4;
  notSure.ifItExists(); // okay, ifItExists might exist at runtime
  notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

  let prettySure: Object = 4;
  prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
  ```


- 当你只知道一部分数据的类型时，`any`类型也是有用的。 比如，你有一个数组，它包含了不同的类型的数据：

  ```typescript
  let list:any[] = [1,true,'free']
  list[1] = 100
  ```

#### 8.Void

- `void`类型像是与`any`类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是 `void`：

  ```typescript
  function warnUser():void {
    console.log('this is a warning message')
  }
  ```


- 声明一个void类型的变量没有什么大用，因为void类型的变量只能赋值为undefined和null

  ```typescript
  let unusable:void = undefined
  ```

#### 9.Null和Undefined

- TypeScript里，`undefined`和`null`两者各自有自己的类型分别叫做`undefined`和`null`。 和 `void`相似，它们的本身的类型用处不是很大：

  ```typescript
  let u: undefined = undefined;
  let n: null = null;
  ```


- 默认情况下null和undefined是所有类型的子类型，就是说你可以把null和unde赋值给number类型的变量。
- 然而，当你指定了`--strictNullChecks`标记，`null`和`undefined`只能赋值给`void`和它们各自。 这能避免 *很多*常见的问题。 也许在某处你想传入一个 `string`或`null`或`undefined`，你可以使用联合类型`string | null | undefined`。

#### 10.Never

- Never类型表示的是那些永不存在的值的类型。例如， `never`类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 

- 变量也可能是 `never`类型，当它们被永不为真的类型保护所约束时。`never`类型是任何类型的子类型，也可以赋值给任何类型；然而，*没有*类型是`never`的子类型或可以赋值给`never`类型（除了`never`本身之外）。 即使 `any`也不可以赋值给`never`。

- 下面是一些返回`never`类型的函数：

  ```typescript
  // 返回never的函数必须存在无法达到的终点
  function error(message: string): never {
      throw new Error(message);
  }

  // 推断的返回值类型为never
  function fail() {
      return error("Something failed");
  }

  // 返回never的函数必须存在无法达到的终点
  function infiniteLoop(): never {
      while (true) {
      }
  }
  ```

#### 11.Object

- `object`表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型。使用`object`类型，就可以更好的表示像`Object.create`这样的API。例如：

  ```typescript
  declare function create(o: object | null): void;

  create({ prop: 0 }); // OK
  create(null); // OK

  create(42); // Error
  create("string"); // Error
  create(false); // Error
  create(undefined); // Error
  ```

#### 12.类型断言

- 通过*类型断言*这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须的检查。

- 类型断言有两种形式。 其一是“尖括号”语法：

  ```typescript
  let someValue: any = "this is a string";

  let strLength: number = (<string>someValue).length;
  ```


- 另一个为`as`语法：

  ```typescript
  let someValue: any = "this is a string";

  let strLength: number = (someValue as string).length;
  ```


- 两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，当你在TypeScript里使用JSX时，只有 `as`语法断言是被允许的。




### 二、接口

- typescript的核心规则之一是对值所具有的结构进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

#### 1.接口基础

```typescript
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

- 类型检查器会查看`printLabel`的调用。 `printLabel`有一个参数，并要求这个对象参数有一个名为`label`类型为`string`的属性。 需要注意的是，我们传入的对象参数实际上会包含很多属性，但是编译器只会检查那些必需的属性是否存在，并且其类型是否匹配。

- 面我们重写上面的例子，这次使用接口来描述：必须包含一个`label`属性且类型为`string`：

  ```typescript
  interface LabelledValue {
    label: string;
  }

  function printLabel(labelledObj: LabelledValue) {
    console.log(labelledObj.label);
  }

  let myObj = {size: 10, label: "Size 10 Object"};
  printLabel(myObj);
  ```


- `LabelledValue`接口就好比一个名字，用来描述上面例子里的要求。 它代表了有一个 `label`属性且类型为`string`的对象。 需要注意的是，我们在这里并不能像在其它语言里一样，说传给 `printLabel`的对象实现了这个接口。我们只会去关注值的外形。 只要传入的对象满足上面提到的必要条件，那么它就是被允许的。
- 类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是对的就可以。

#### 2.可选属性

- 接口里的属性不全都是必需的，有些是只在某些条件下存在，或者根本不存在。可选属性在应用“option bags”模式时很常用，即给函数传入的参数对象中只有部分属性赋值了。

  ```typescript
  interface SquareConfig{
    color?: string;
    width?: number;
  }

  function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
      newSquare.color = config.color;
    }
    if (config.width) {
      newSquare.area = config.width * config.width;
    }
    return newSquare;
  }

  let mySquare = createSquare({color: "black"});
  ```


- 在 `color` 和 `width` 属性后面加一个 `?` （英文问号）就表示该属性是可选的属性了

- 带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个`?`符号。

- 可选属性的好处之一是可以对可能存在的属性进行预定义，好处之二是可以捕获引用了不存在的属性时的错误。 比如，我们故意将 `createSquare`里的`color`属性名拼错，就会得到一个错误提示：

  ```typescript
  interface SquareConfig {
    color?: string;
    width?: number;
  }

  function createSquare(config: SquareConfig): { color: string; area: number } {
    let newSquare = {color: "white", area: 100};
    if (config.clor) {
      // Error: Property 'clor' does not exist on type 'SquareConfig'
      newSquare.color = config.clor;
    }
    if (config.width) {
      newSquare.area = config.width * config.width;
    }
    return newSquare;
  }

  let mySquare = createSquare({color: "black"});
  ```

#### 3.只读属性

- 一些对象属性只能在对象刚刚创建的时候修改其值。你可以在属性名前用readonly来指定只读属性

  ```typescript
  interface Point{
    readonly x: number;
    readonly y: number;
  }
  ```


- 你可以通过赋值一个对象字面量来构造一个`Point`。 赋值后， `x`和`y`再也不能被改变了。

  ```typescript
  let p1: Point = { x: 10, y: 20 };
  p1.x = 5; // error!
  ```


- TypeScript具有`ReadonlyArray<T>`类型，它与`Array<T>`相似，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改：

  ```typescript
  let a: number[] = [1, 2, 3, 4];
  let ro: ReadonlyArray<number> = a;
  ro[0] = 12; // error!
  ro.push(5); // error!
  ro.length = 100; // error!
  a = ro; // error!
  ```


- 上面代码的最后一行，可以看到就算把整个`ReadonlyArray`赋值到一个普通数组也是不可以的。 但是你可以用类型断言重写：

  ```typescript
  a = ro as number[];
  ```


- **`readonly` vs `const` ：**最简单判断该用`readonly`还是`const`的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用 `const`，若做为属性则使用`readonly`。

#### 4.额外的属性检查

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

- 注意传入`createSquare`的参数拼写为*colour*而不是`color`。 在JavaScript里，这会默默地失败。

- 然而，TypeScript会认为这段代码可能存在bug。 对象字面量会被特殊对待而且会经过 *额外属性检查*，当将它们赋值给变量或作为参数传递的时候。 如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。

  ```typescript
  // error: 'colour' not expected in type 'SquareConfig'
  let mySquare = createSquare({ colour: "red", width: 100 });
  ```


- 绕开这些检查很简单；最简便的方法是使用类型断言

  ```typescript
  let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
  ```


- 然而，最佳方式是能欧添加一个字符串索引签名，前提是你能够确定这个对象可能具有某些作为特殊用途使用的额外属性。 如果 `SquareConfig`带有上面定义的类型的`color`和`width`属性，并且*还会*带有任意数量的其它属性，那么我们可以这样定义它：

  ```typescript
  interface SquareConfig{
    color?: string;
    width?: number;
    [propName: string]: any
  }
  ```


- 最后一种跳过这些检查的方法就是将这个对象赋值给另一个变量：因为`squareOptions`不会经过额外属性检查，所以编译器不会报错。

  ```typescript
  let squareOptions = { colour: "red", width: 100 };
  let mySquare = createSquare(squareOptions);
  ```

#### 5.函数类型

- 接口能够描述JavaScript中对象拥有的各种各样的外形。 除了描述带有属性的普通对象外，接口也可以描述函数类型。

- 为了使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

  ```typescript
  interface SearchFunc{
    (source: string, subString: string) : boolean;
  }
  ```


- 这样定义后，我们可以像使用其它接口一样使用这个函数类型的接口

  ```typescript
  let mySearch: SearchFunc;
  mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
  }
  ```


- 对于函数类型的类型检查来说，函数的参数名不需要与接口里定义的名字相匹配。 比如，我们使用下面的代码重写上面的例子：

  ```typescript
  let mySearch: SearchFunc;
  mySearch = function(src: string, sub: string): boolean {
    let result = src.search(sub);
    return result > -1;
  }
  ```


- 函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。 如果你不想指定类型，TypeScript的类型系统会推断出参数类型，因为函数直接赋值给了 `SearchFunc`类型变量。 函数的返回值类型是通过其返回值推断出来的（此例是 `false`和`true`）。 如果让这个函数返回数字或字符串，类型检查器会警告我们函数的返回值类型与 `SearchFunc`接口中的定义不匹配。

  ```typescript
  let mySearch: SearchFunc;
  mySearch = function(src, sub) {
      let result = src.search(sub);
      return result > -1;
  }
  ```

#### 6.可索引的类型

- 可以描述那些能够通过索引得到的类型，比如`a[10]`或`ageMap["daniel"]`。 可索引类型具有一个 *索引签名*，它描述了对象索引的类型，还有相应的索引返回值类型。 

  ```typescript
  interface StringArray {
    [index: number]: string;
  }

  let myArray: StringArray;
  myArray = ["Bob", "Fred"];

  let myStr: string = myArray[0];
  ```


- 上面例子里，我们定义了`StringArray`接口，它具有索引签名。 这个索引签名表示了当用 `number`去索引`StringArray`时会得到`string`类型的返回值。

- TypeScript支持两种索引签名：字符串和数字。 可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。 这是因为当使用 `number`来索引时，JavaScript会将它转换成`string`然后再去索引对象。 也就是说用 `100`（一个`number`）去索引等同于使用`"100"`（一个`string`）去索引，因此两者需要保持一致。

  ```typescript
  class Animal {
      name: string;
  }
  class Dog extends Animal {
      breed: string;
  }

  // 错误：使用数值型的字符串索引，有时会得到完全不同的Animal!
  interface NotOkay {
      [x: number]: Animal;
      [x: string]: Dog;
  }
  ```


- 字符串索引签名能够很好的描述`dictionary`模式，并且它们也会确保所有属性与其返回值类型相匹配。 因为字符串索引声明了 `obj.property`和`obj["property"]`两种形式都可以。 

  ```typescript
  interface NumberDictonary {
    [index: string]: number
    length: number;    // 可以，length是number类型
    name: string       // 错误，`name`的类型与索引类型返回值的类型不匹配
  }
  ```


- 最后，你可以将索引签名设置为只读，这样就防止了给索引赋值

  ```typescript
  interface ReadonlyStringArray {
      readonly [index: number]: string;
  }
  let myArray: ReadonlyStringArray = ["Alice", "Bob"];
  myArray[2] = "Mallory"; // error!
  ```

#### 7.类类型

##### （1）实现接口

- 与C#或Java里接口的基本作用一样，TypeScript也能够用它来明确的强制一个类去符合某种契约。

  ```typescript
  interface ClockInterface{
    currentTime: Date;
  }

  class Clock implements ClockInterface {
      currentTime: Date;
      constructor(h: number, m: number) { }
  }
  ```

- 你也可以在接口中描述一个方法，在类里实现它，如同下面的`setTime`方法一样：

  ```typescript
  interface ClockInterface {
      currentTime: Date;
      setTime(d: Date);
  }

  class Clock implements ClockInterface {
      currentTime: Date;
      setTime(d: Date) {
          this.currentTime = d;
      }
      constructor(h: number, m: number) { }
  }
  ```

##### （2）类静态部分与实例部分的区别

- 当你操作类和接口的时候，你要知道类是具有两个类型的：静态部分的类型和实例的类型。 你会注意到，当你用构造器签名去定义一个接口并试图定义一个类去实现这个接口时会得到一个错误：

  ```typescript
  interface ClockConstructor {
      new (hour: number, minute: number);
  }

  class Clock implements ClockConstructor {
      currentTime: Date;
      constructor(h: number, m: number) { }
  }
  ```


- 这里因为当一个类实现了一个接口时，只对其实例部分进行类型检查。 constructor存在于类的静态部分，所以不在检查的范围内。

- 因此，我们应该直接操作类的静态部分。 

- 看下面的例子，我们定义了两个接口， `ClockConstructor`为构造函数所用和`ClockInterface`为实例方法所用。 为了方便我们定义一个构造函数 `createClock`，它用传入的类型创建实例。

  ```typescript
  interface ClockConstructor {
      new (hour: number, minute: number): ClockInterface;
  }
  interface ClockInterface {
      tick();
  }

  function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
      return new ctor(hour, minute);
  }

  class DigitalClock implements ClockInterface {
      constructor(h: number, m: number) { }
      tick() {
          console.log("beep beep");
      }
  }
  class AnalogClock implements ClockInterface {
      constructor(h: number, m: number) { }
      tick() {
          console.log("tick tock");
      }
  }

  let digital = createClock(DigitalClock, 12, 17);
  let analog = createClock(AnalogClock, 7, 32);
  ```

#### 8.继承接口

- 和类一样，接口也可以相互继承。 这让我们能够从一个接口里复制成员到另一个接口里，可以更灵活地将接口分割到可重用的模块里。

  ```typescript
  interface Shape {
      color: string;
  }

  interface Square extends Shape {
      sideLength: number;
  }

  let square = <Square>{};
  square.color = "blue";
  square.sideLength = 10;
  ```

- 一个接口可以继承多个接口，创建出多个接口的合成接口

  ```typescript
  interface Shape {
      color: string;
  }

  interface PenStroke {
      penWidth: number;
  }

  interface Square extends Shape, PenStroke {
      sideLength: number;
  }

  let square = <Square>{};
  square.color = "blue";
  square.sideLength = 10;
  square.penWidth = 5.0;
  ```

#### 9.混合类型

- 接口能够描述JavaScript里丰富的类型。 因为JavaScript其动态灵活的特点，有时你会希望一个对象可以同时具有上面提到的多种类型。

- 一个例子就是，一个对象可以同时做为函数和对象使用，并带有额外的属性。

  ```typescript
  interface Counter {
      (start: number): string;
      interval: number;
      reset(): void;
  }

  function getCounter(): Counter {
      let counter = <Counter>function (start: number) { };
      counter.interval = 123;
      counter.reset = function () { };
      return counter;
  }

  let c = getCounter();
  c(10);
  c.reset();
  c.interval = 5.0;
  ```

#### 10.接口继承类

- 当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。 接口同样会继承到类的private和protected成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

- 当你有一个庞大的继承结构时这很有用，但要指出的是你的代码只在子类拥有特定属性时起作用。 这个子类除了继承至基类外与基类没有任何关系。 例：

  ```typescript
  class Control {
      private state: any;
  }

  interface SelectableControl extends Control {
      select(): void;
  }

  class Button extends Control implements SelectableControl {
      select() { }
  }

  class TextBox extends Control {
      select() { }
  }

  // 错误：“Image”类型缺少“state”属性。
  class Image implements SelectableControl {
      select() { }
  }

  class Location {

  }
  ```


- 在上面的例子里，`SelectableControl`包含了`Control`的所有成员，包括私有成员`state`。 因为 `state`是私有成员，所以只能够是`Control`的子类们才能实现`SelectableControl`接口。 因为只有 `Control`的子类才能够拥有一个声明于`Control`的私有成员`state`，这对私有成员的兼容性是必需的。
- 在`Control`类内部，是允许通过`SelectableControl`的实例来访问私有成员`state`的。 实际上， `SelectableControl`接口和拥有`select`方法的`Control`类是一样的。 `Button`和`TextBox`类是`SelectableControl`的子类（因为它们都继承自`Control`并有`select`方法），但`Image`和`Location`类并不是这样的。