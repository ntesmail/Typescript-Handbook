类
--

传统的 JavaScript 里实现可以重用的组件主要依赖于函数和原型链继承，但是这让那些更习惯使用原生拥有类和继承的面向对象语言的程序员感觉有些棘手。从 ECMAScript 6，下一个版本的 Javascript 开始，JavaScript 程序员将能够使用原生的类来实现面向对象。在 TypeScript 中，我们让开发者无需等待下个版本的 Javascript，现在就开始使用这些技术进行开发，然后把代码编译成兼容所有主流浏览器和平台的 Javascript。

### 类

让我们先简单看一下基于类的实现范例：

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```

如果你使用过 C# 或者 Java，这种语法看起来应该很熟悉。我们声明了一个 'Greeter' 类。这个类有三个成员，一个叫做 'greeting' 的属性，一个构造函数和一个方法 'greet'。

你可能会注意到在类里我们引用类的某个成员的时候，我们用了前缀 'this.'。这表明它是成员访问。

最后一行，我们使用 'new' 来构造了 Greeter 类的一个实例。这会调用我们之前定义的构造函数，根据 Greeter 的特征创建一个对象，然后运行构造函数来初始化它。

### 继承

在 TypeScript 中，我们可以使用普通的面向对象的模式。当然，在基于类的编程中其中一个最基本的模式就是可以继承已有的类来创建新的类。

让我们看个例子：

```ts
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move() {
        alert("Slithering...");
        super.move(5);
    }˝
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move() {
        alert("Galloping...");
        super.move(45);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

这个例子涵盖了 TypeScript 中很大一部分继承的特性，这些特性和其他一些语言很类似。我们看到使用 'extends' 这个关键词来创建一个子类。你可以看到 'Horse' 和 'Snake' 继承了基类，并且获得了基类的特征。

这个例子也说明了可以在子类中重写基类已有的方法。这里 'Snake' 和 'Horse' 都创建了一个 'move' 方法，覆盖了从 'Animal' 继承过来的 'move'，给予子类不同的特征。

### Private/Public modifiers 私有/公有 修饰符

#### Public by default 默认 Public

你可能已经注意到上面的例子里我们不是一定需要使用 'public' 让类的成员对外可见。像 C# 这样的语言要求每个成员必须显式的标记 'public' 以表示对外可见。在 TypeScript 中，成员默认就是 public 的。

你也可以将成员标记为 private，总之你可以控制类的哪些成员对外可见。我们可以重写下前面实现的 'Animal' 类：

```ts
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

#### Understanding private 理解 private

TypeScript 是一个结构性的类型系统 ([Structural type system](http://en.wikipedia.org/wiki/Structural_type_system))。当我们比较两个类型的时候，不去理会他们从哪里来，如果这两个类型的每个成员的类型都是兼容的，那就认为这两个类型是兼容的。

当比较两个拥有私有成员的类型的时候，我们处理方式有些不同。如果两个类型认为是互相兼容的，并且他们中的一个有私有成员，那么另一个必须拥有一个来自相同声明的私有成员。

让我们看个例子来更好的理解下这是怎么回事：

```ts
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
	constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }	
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; //error: Animal and Employee are not compatible
```

在这个例子中，有 'Animal' 和 'Rhino' 两个类，其中 'Rhino' 是 'Animal' 的子类。同时，还有一个 'Employee' 类的特征看上去和 'Animal' 等同。我们创建了这些类的实例，并且尝试把其中的一个赋值给另一个看看会发生什么。因为 'Animal' 和 'Rhino' 共用在 'Animal' 里的私有成员声明 'private name: string' ，因此他们是兼容的。但是，'Employee' 则不同。当我们尝试把 'Employee' 的实例赋值给 'Animal' 时，会提示错误说类型不兼容。虽然 'Employee' 也有一个私有变量叫做 name，但是它和 'Animal' 里的 name 并不相同。

#### Parameter properties 参数属性

支持通过添加参数属性（关键词 'public' 和 'private' ）来快捷的创建和初始化类里的成员。这些属性可以让你一步完成类成员的创建和初始化。这里有一个之前示例修改后的版本。注意我们完全抛弃了 'theName'， 只是在构造函数的参数上使用缩短的 'private name: string' 来创建和初始化 'name' 成员。

```ts
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

使用 'private' 可以创建和初始化私有成员，类似的使用 'public' 可以创建和初始化公有成员。

### Accessors 存取器

TypeScript 支持使用 getters/setters 来拦截对成员变量的访问。这让你更小粒度的去控制对象上的每个成员如何被访问。

让我们把一个简单的类转化成使用 'get' 和 'set'。首先，让我们看一个没有使用 getters 和 setters 的例子。

```ts
class Employee {
    fullName: string;
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

虽然允许直接修改 fullName 看起来很方便，但是我们可以心血来潮的修改名字可能会带来麻烦。

在这个版本中，在允许修改 employee 之前，我们检查了用户是否有正确的密码。我们把可以直接修改 fullName 替换成了使用 set 方法来检查密码。同时，我们增加了一个匹配的 get 让前面的示例可以继续无缝的工作。

```ts
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }
	
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

为了证明我们的存取器现在在检查密码，我们可以修改密码来确认当密码不匹配的时候我们会得到一个没有更新权限的提示框。

注意：访问器需要你将编译器的输出配置为 ECMAScript 5。

### Static Properties 静态属性

说到这里，我们只是谈及了类的实例成员，它们只有在类初始化之后才出现。我们也可以创建类的静态成员，它们是在类本身上可见，而不是在实例上可见。在这个例子中，我们将 origin 声明为 static，因为它对于所有的网格都是一个相同的值。每个实例都可以通过类的名称来访问这个值。类似于在实例成员前面增加 'this.'，在这里，我们在静态属性的名称前增加 'Grid.'' 前缀。

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

var grid1 = new Grid(1.0);  // 1x scale
var grid2 = new Grid(5.0);  // 5x scale

alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```
