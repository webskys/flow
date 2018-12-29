# Typeof Types

 > 获取值的内部类型

 `Javascript`有`typeof`操作符，返回一个描述值的字符串。

```javascript
typeof 1 === 'number'
typeof true === 'boolean'
typeof 'three' === 'string'
```

然而，它是有限的，这个字符串只是关于类型的描述。

```javascript
typeof { foo: true } === 'object'
typeof { bar: true } === 'object'
typeof [true, false] === 'object'
```

在`Flow`里，有一个类似的`typeof`操作符，但是它要强大得多。

## 语法

`Flow`中的`typeof`操作符返回给定值的类型，并作为类型使用

```javascript
// @flow
let num1 = 42;
let num2: typeof num1 = 3.14;     // Works!
// $ExpectError
let num3: typeof num1 = 'world';  // Error!

let bool1 = true;
let bool2: typeof bool1 = false;  // Works!
// $ExpectError
let bool3: typeof bool1 = 42;     // Error!

let str1 = 'hello';
let str2: typeof str1 = 'world'; // Works!
// $ExpectError
let str3: typeof str1 = false;   // Error!
```

你可以对任意值使用`typeof`操作。

```javascript
// @flow
let obj1 = { foo: 1, bar: true, baz: 'three' };
let obj2: typeof obj1 = { foo: 42, bar: false, baz: 'hello' };

let arr1 = [1, 2, 3];
let arr2: typeof arr1 = [3, 2, 1];
```

## 继承自推理

`Flow`会对你的代码执行各种各样的推断，因此你不必为任何东西都添加类型注解。通常，推断会避免妨碍你的同时任然会防止你引入错误。

但是，当你使用`typeof`时,你得到一个`Flow`的推断结果并把它作为一个类型断言。虽然这很有用，但也会可能导致一些意外的结果。

例如，当您在`Flow`中使用字面量值时，它们推断出类型就是它所属的原始类型。因此，数字`42`被推断出的`number`类型。当你使用`Typeof`时就可以看到这一点。

```javascript
// @flow
let num1 = 42;
let num2: typeof num1 = 3.14;    // Works!

let bool1 = true;
let bool2: typeof bool1 = false; // Works!

let str1 = 'hello';
let str2: typeof str1 = 'world'; // Works!
```

然而，这仅仅发生在类型推断的情况下。如果你指定了字面量类型，`typeof`就会使用这个类型。

```javascript
// @flow
let num1: 42 = 42;
// $ExpectError
let num2: typeof num1 = 3.14;    // Error!

let bool1: true = true;
// $ExpectError
let bool2: typeof bool1 = false; // Error!

let str1: 'hello' = 'hello';
// $ExpectError
let str2: typeof str1 = 'world'; // Error!
```

## 继承自其他类型

在`Flow`里有很多不同的类型，其中一些类型的表现行为跟其他的不一样，这些差异对于该特定类型是意义的，但对于其它类型则没用。

当你使用`typeof`,你将插入另一个类型及其所有行为。这点使得`typeof`看起来跟不使用它时前后矛盾。

例如，如果你对一个类使用`typeof`，你需要记住类类型是名义类型，而不是结构类型。因此两个具有相同精确模型的类类型被认为是不相等的。

```javascript
// @flow
class MyClass {
  method(val: number) { /* ... */ }
}

class YourClass {
  method(val: number) { /* ... */ }
}

// $ExpectError
let test1: typeof MyClass = YourClass; // Error!
let test2: typeof MyClass = MyClass;   // Works!
```
