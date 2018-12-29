# 变量类型

 > 声明变量时添加类型

当你声明一个新变量的时候，可以选择指定它的类型。

在`JavaScript` 中有三种方法声明变量：

 - `var`：声明一个变量，可以选择分配一个值
 - `let`：声明一个块级作用域变量，可以选择分配一个值
 - `const`：声明一个块级作用域的常量，分配的值不可以改变

在`Flow`里，这些声明被分成两组：

 - `let`和`var` ：声明的变量可以重新分配新值
 - `const`：声明的变量不允许重新分配值

```javascript
var varVariable = 1;
let letVariable = 1;
const constVariable = 1;

varVariable = 2;   // Works!
letVariable = 2;   // Works!
// $ExpectError
constVariable = 2; // Error!
```

## const

因为一个`const`变量不允许后面重新分配值，所以相对简单。

`Flow`可以根据所分配值来推断其类型，或者你也可以用一个类型来提供其变量的类型。

```javascript
// @flow
const foo /* : number */ = 1;
const bar: number = 2;
```

## var 和 let

因为`var`和`let`声明的变量可以重新分配新值，这里有一些你需要了解的相当规则，

跟`const`相似，`Flow`可以根据所分配值来推断其类型，或者你也可以用一个类型来提供其变量的类型。

```javascript
// @flow
var fooVar /* : number */ = 1;
let fooLet /* : number */ = 1;
var barVar: number = 2;
let barLet: number = 2;
```

当提供了一个类型，你可以重新分配其值，但这个值必须始终兼容其类型。

```javascript
// @flow
let foo: number = 1;
foo = 2;   // Works!
// $ExpectError
foo = "3"; // Error!
```

当你没有提供一个类型，如果重新分配新值时推断出的类型将是两个中的一个。

## 变量重新赋值

默认情况下，当你为一个变量重新赋值时，`Flow`将赋予变量所有可能的类型。

```javascript
let foo = 42;

if (Math.random()) foo = true;
if (Math.random()) foo = "hello";

let isOneOf: number | boolean | string = foo; // Works!
```

有时`Flow`能够算出（准确）一个变量分配新值后的类型，这种情况下，`Flow`将给他这个已知的类型

```javascript
// @flow
let foo = 42;
let isNumber: number = foo; // Works!

foo = true;
let isBoolean: boolean = foo; // Works!

foo = "hello";
let isString: string = foo; // Works!
```

如果语句、函数或其他条件执行代码会阻止`Flow`精确地算出变量类型。

```javascript
// @flow
let foo = 42;

function mutate() {
  foo = true;
  foo = "hello";
}

mutate();

// $ExpectError
let isString: string = foo; // Error!
```

以上代码最后一段改成如下就不会报错了

```javascript
let isString: string | number | boolean = foo;
```