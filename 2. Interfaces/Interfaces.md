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











