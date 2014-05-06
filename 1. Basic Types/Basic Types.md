基本类型
--

为了让程序更有用，我们需要兼容几种最基本的数据单元：数字，字符串，结构，布尔值等等。在 TypeScript 中，我们支持和 Javascript 几乎一样多的类型，并且新增了实用的枚举类型。

### Boolean 布尔值
最基础的数据类型就是简单的 true/false ，在 Javascript 和 TypeScript （以及其他语言）中被称作是 "boolean"。

```ts
var isDone: boolean = false;
```

### Number 数字
和 Javascript 一样，在 TypeScript 中所有的数字都是浮点值。这些浮点值的类型为 "number"。

```ts
var height: number = 6;
```

### String 字符串

另一个在用 Javascript 编写网页和服务端代码时很基础的部分是处理文本数据。和其他语言一样，我们使用 "string" 类型来表示那些文本数据。和 Javascript 一样，TypeScript 也使用双引号或单引号来围绕字符串数据。

```ts
var name: string = "bob";
name = 'smith';
```


### Array 数组

TypeScript 和 Javascript 一样，允许你使用数组。数组类型的定义可以有两种写法。第一种写法，你在数组元素类型后面添加‘[]’来表示这是一个该类型的数组。

```ts
var list:number[] = [1, 2, 3];
```

第二种写法使用一种通用的数组类型表示，Array<数组元素类型>

```ts
var list:Array<number> = [1, 2, 3];
```

### Enum 枚举

枚举是在 Javascript 标准的类型基础上增加的一个有用的类型。类似 C# 等语言，枚举可以给数字集合里的值一些友好的命名。

```ts
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```

默认情况下，枚举从 0 开始给它们的成员赋值。你也可以自己设置成员的值。比如，我们从 1 开始给成员赋值：

```ts
enum Color {Red = 1, Green, Blue};
var c: Color = Color.Green;
```

甚至可以给所有的枚举成员手动赋值：

```ts
enum Color {Red = 1, Green = 2, Blue = 4};
var c: Color = Color.Green;
```

枚举的一个有用的特征是你可以根据值找到枚举里对应的名称。比如，我们可以通过下面的方式获取 Color 枚举中 值为 2 的颜色名称：

```ts
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2];

alert(colorName);
```

### Any 任何类型

在编写应用程序的时候，我们可能需要描述未知数据变量的类型。这些变量的值来自于动态的内容，比如来自用户或者第三方的库。在这些场景下，我们想要关闭类型检查，并让这些值通过编译检查。在这种情况下，我们给这些变量赋予 "any" 类型：

```ts
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

"any" 类型是一个兼容已有的 Javascript 代码的强大的解决方案，允许你在编译的时候优雅的开启或关闭类型检查。

如果你知道这些变量的部分类型，但也许不是全部，这个时候使用 "any" 类型就很方便。比如，你有一个拥有不同类型元素的数组：

```ts
var list:any[] = [1, true, "free"];

list[1] = 100;
```

### Void 空

也许在某种角度来看，和任何类型对立的就是 void，没有任何类型。你可以经常在没有返回值的函数的类型里见到。

```ts
function warnUser(): void {
    alert("This is my warning message");
}
```