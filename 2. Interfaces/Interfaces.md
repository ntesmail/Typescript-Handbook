# 接口

Typescript 的一个核心原则是做类型检查时只对值的 shape 进行检查。这种原则也被叫做 [duck typing](http://zh.wikipedia.org/zh-cn/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B) 或者 [structural subtyping](http://zh.wikipedia.org/wiki/%E5%AD%90%E7%B1%BB%E5%9E%8B)。在 Typescript 中，接口被用于描述这种类型，也用于在你的代码中以及与外部代码之间定义约定。

###  我们的第一个接口

请看一个简单的例子：

```ts
function printLabel(labelledObj: {label: string}) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

类型检查器会检查对 printLabel 的调用。printLabel 方法接收一个对象作为参数，并要求这个对象带有一个 string 类型的属性 label。请注意除了 label 外，我们传给 printLabel 方法的参数 myObj 含有更多的属性，但是编译器只会要求 label 属性存在并且是正确的类型。

下面我们使用接口来完成这个要求：

```ts
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

接口 LabelledValue 用于完成前面提到的要求。我们并不像其他语言那样需要printLabel的参数实现这个接口，只需要这个参数符合限定的 shape。

同样需要指出的是类型检查器不会对属性出现的顺序继续宁限制，只对属性以及属性的类型有要求。

### 可选属性

可选属性在类似 option bags 的模式中很流行，在这类模式中，用户只需要传入一个带有部分默认属性的对象。

请看下面的例子：

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

你可以用一个问号来表示当前属性为可选属性。可选属性的好处是通过描述所有可能存在的属性，来捕获哪些不应该存在的属性。例如我们错误输入了一个传给createSquare方法的参数属性，我们就会得到一个错误提示：

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor;  // Type-checker can catch the mistyped name here
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});  
```

### 函数类型

接口可以用于描述各种 shape 的 Javascript 对象。除了带有属性的对象之外，接口同样可以用于描述函数类型。

函数类型的声明包含对函数的参数列表和返回值的描述。

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

函数类型接口的使用和其他接口一样，下面是一个对函数类型变量赋值的例子。

```ts
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

函数类型接口并不要求参数的名称与接口定义一致，例如将上面的例子写为这样也是可以的：

```ts
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string) {
  var result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

函数参数的类型需要与接口中定义的完全一致，函数的返回值类型也是一样。如果上面的方法返回了一个数字或者字符串，类型检查器会警告返回值类型与接口中定义的不一致。

### 数组类型

与函数类型接口类似，我们也可以定义数组类型接口。数组类型接口包含对索引类型和返回值类型的定义。

```ts
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

我们支持的索引类型有2种：字符串和数字。在一个接口中同时支持这两种索引是允许的，但是有一个限制：数字索引返回值的类型必须为字符串索引返回值类型的子类。（译者注：通过实验，数字索引返回值的类型必须符合字符串索引返回值类型的 shape，比如下面这样也是可以的：）

```ts
interface A {
	value: number;
}

interface B {
	value: number;
	type: string;
}

interface a {
	[index: number]: B;
	[index: string]: A;
}
```

索引类型接口很适合于描述数字或者 dictionary 模式，但它要求接口中定义的其他属性必须与索引返回值的类型一致。例如，这个例子中的 length 属性不符合索引返回值的类型，类型检查器就会报错：

```ts
interface Dictionary {
  [index: string]: string;
  length: number;    // error, the type of 'length' is not a subtype of the indexer
} 
```

### Class 类型

#### 实现一个接口

在 C# 或者 Java 等语言中，接口最常见的用法是强制类实现某种约定，在 Typescript 也可以这样做。

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

你同样可以在接口中定义一个方法并在类中实现，例如下面例子中的 setTime 方法：

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口只能用于描述了类的公开部分，不包括私有部分，所以你不能使用接口来强制一个类的私有部分也包含特定的类型。

#### 类的静态部分和实例部分的区别（Difference between static/instance side of class）

当同时使用类和接口时，你最好能记住类的成员有两种类型：静态成员类型和实例成员类型。如果你创建了一个带有构造函数的接口，并尝试创建一个类来实现这个接口，就会发生一个错误：

```ts
interface ClockInterface {
    new (hour: number, minute: number);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

这是因为当类实现一个接口时，只有类的实例成员会被检查。而构造函数作为类的静态成员，是不会进行检查的。

所以，你需要通过某种方式与类的静态成员部分直接进行交流。请看下面的例子：

```ts
interface ClockStatic {
    new (hour: number, minute: number);
}

class Clock  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}

var cs: ClockStatic = Clock;
var newClock = new cs(7, 30);
```

### 接口继承

和类一样，接口之间也可以继承。通过继承可以避免在接口之间拷贝接口成员，也便于将接口拆分为不同用途的元件。

```ts
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

var square: Square;
square.color = "blue";
square.sideLength = 10;
```

一个接口可以同时继承多个其他接口，从而创建一个包含其他所有接口的合集。

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

var square: Square;
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

### 混合类型

接口可以用于描述 Javascript 中各种丰富的类型。由于 Javascript 动态和灵活的特点，你时常会遇到一个包含了以上介绍的各种类型的对象。

例如下面这个对象，它即是一个方法，也是一个对象，同时还带有额外的属性：

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

var c: Counter;
c(10);
c.reset();
c.interval = 5.0;
```

当需要与第三方 Javascript 代码相互调用时，你时常需要用到混合类型接口来完整的描述一个类型。
