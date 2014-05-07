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


















