# 模块

本章会列举在 TypeScript 中使用模块的各种方法。我们会讲到内部和外部模块，并讨论何时应该使用以及如何使用。我们也会讲到如何使用外部模块等高级话题，以及如何处理在 TypeScript 中使用模块时会遇到的一些常见问题。

#### 第一步

让我们从一段将会贯穿于本章内容的示例程序开始。在这个程序中，我们编写了几个简单的 string 校验器，用于检查例如用户在网页上的输入或者是从外部获取的数据文件。

##### 写在一个文件中的校验器

```ts
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: StringValidator; } = {};
validators['ZIP code'] = new ZipCodeValidator();
validators['Letters only'] = new LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

#### 加入模块化

当我们加入更多的校验器时，我们需要有一种组织代码的方法来避免对象之间的命名冲突。让我们通过把对象放进模块来避免把太多名字放入全局命名空间。

在下面的例子中，我们将所有校验器相关的类型放入一个叫做 Validation 的模块中。对于需要对外界可见的接口和类，我们使用 export 来注明。相反，变量 lettersRegexp 和 numberRegexp 是实现的细节，所以它们不需要被注明为 export，从而对外界是不可见的。在文件最后的测试代码中，我们需要从模块外调用模块内部的类型，可以使用模块名称来修饰类型，比如：Validation.LettersOnlyValidator。

##### 模块化的校验器

```ts
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    var lettersRegexp = /^[A-Za-z]+$/;
    var numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 划分为多个文件

当程序规模增长时，我们需要把代码划分到多个模块中以便于维护。

下面我们会把 Validation 模块划分到多个文件中。虽然分为了多个文件，但是他们仍然可以属于统一个模块，并且作为同一个模块被使用。由于这些文件之间存在依赖，我们加上了 reference 标签来告诉编译器这些文件之间的联系。我们的测试代码没有变化。

#### 多文件的内部模块

##### Validation.ts

```ts
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

##### LettersOnlyValidator.ts

```ts
/// <reference path="Validation.ts" />
module Validation {
    var lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

##### ZipCodeValidator.ts

```ts
/// <reference path="Validation.ts" />
module Validation {
    var numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

##### Test.ts

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

当有多个文件时，我们需要确定所有编译后的文件都得到加载，有两种方法：

第一种方法，我们可以使用 --out 标志来将所有输入的文件合并输出为一个 JavaScript 文件：

```sh
tsc --out sample.js Test.ts
```

编译器会根据文件中的 reference 标签自动安排各个输入文件在输出文件中的次序。你同样也可以依次写明所有的输入文件：

```sh
tsc --out sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

在另一种方法中，我们可以将每一个输入文件单独编译，然后在网页上使用 &lt;script&gt; 标签按顺序加载每一个编译后的文件，例如：

##### MyTestPage.html（摘录）

```html
<script src="Validation.js" type="text/javascript" />
<script src="LettersOnlyValidator.js" type="text/javascript" />
<script src="ZipCodeValidator.js" type="text/javascript" />
<script src="Test.js" type="text/javascript" />
```

### 外部模块

TypeScript 也支持外部模块。外部模块有两种：node.js 和 require.js。没有使用这两种外部模块的程序可以完全依靠前面提到的内部模块来组织代码。

在外部模块中，文件之间的关系可以通过文件之间的 import 和 export 来定义。在 TypeScript 中，任何包含一个顶层 import 或者 export 的文件都会被认为是一个外部模块。

下面，我们将使用外部模块重新组织前面的例子程序。请注意我们不再使用关键词 module，每个文件本身都是一个模块，并用文件名作为标识。

import 语句替代了Reference 标签来说明模块之间的依赖关系。import 语句包含两部分：被引入模块在当前文件中的名称，以及使用 require 关键字注明的被导入模块的路径：

```ts
import someMod = require('someModule');
```

我们使用 export 关键字来表示模块的某个部分是对模块外可见的，这点与内部模块是类似的。

编译时，我们需要在命令中注明模块类型。对于 node.js，请使用 --target commonjs，对于 require.js，请使用 --target amd。例如：

```sh
tsc --module commonjs Test.ts
```

当编译时，每个外部模块都会成为一个单独的.js 文件。类似于 reference 标签，编译器会跟随 import 语句并编译依赖的文件。

##### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```
##### LettersOnlyValidator.ts

```ts
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

##### ZipCodeValidator.ts

```ts
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

##### Test.ts

```ts
import validation = require('./Validation');
import zip = require('./ZipCodeValidator');
import letters = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zip.ZipCodeValidator();
validators['Letters only'] = new letters.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

#### 外部模块的代码生成

根据编译时指定的模块类型，编译器会生成适用于 node.js (commonjs) 或 require.js (AMD) 的代码。关于生成代码中 define 和 require 的更多信息，请参考这些模块加载器的文档。

下面的例子会说明转换过程中 import 和 export 的代码都发生了什么变化。

##### SimpleModule.ts

```ts
import m = require('mod');
export var t = m.something + 1;
```

##### AMD/RequireJS SimpleModule.js

```js
define(["require", "exports", 'mod'], function(require, exports, m) {
    exports.t = m.something + 1;
});
```

##### CommmonJS/Node SimpleModule.js:

```js
var m = require('mod');
exports.t = m.something + 1;
```

### Export =

在前面的例子中，当我们使用各个 valiator 时，每个模块中只包含有一个值。在这种情况下，使用一个模块前缀就显得比较累赘。

下面，我们会使用 export = 语法来简化 Validator 的实现，从而让每个模块只 export 出一个单独的对象。这样在使用时会更简单，我们不需要通过 zip.ZipCodeValidator 来引用这个 validator，可以直接调用 zipValidator。

##### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

##### LettersOnlyValidator.ts

```ts
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
export = LettersOnlyValidator;
```

##### ZipCodeValidator.ts

```ts
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

##### Test.ts

```ts
import validation = require('./Validation');
import zipValidator = require('./ZipCodeValidator');
import lettersValidator = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zipValidator();
validators['Letters only'] = new lettersValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 别名

另一种可以简化模块调用的方法是使用类似于 import q = x.y.z 的格式来创建常用对象的短名字。请不要与加载外部模块的语法 import x = require('name') 混淆，这些短名字只是指定对象的别名。你可以对任意标识符使用这种 import语法，包括从外部模块中加载到的对象。

##### 基本别名

```ts
module Shapes {
    export module Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
var sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

请注意我们没有使用 require 关键字而是直接对导入对象的某个属性进行赋值。这与使用 var 是类似的，但好处是被导入对象的类型和命名空间会得到保留。请注意，对于值类型，import 只是创建了一个到原有标识符的引用，所以对 import 得到的变量进行修改不会对被 import 的变量造成影响。

### 可选模块加载和其他高级模块加载方案





























