# 名义与结构类型

 > 名义类型与结构类型之间的差异

每个类型系统的一个重要特征是它们是结构的还是名义的，他们甚至可以混合在单个类型系统中，所以了解两者的区别相当重要。

一个类型比如像`string`、`boolean`、`object`或`class`。他们有名字和结构。像`string`或`boolean`这些原始值有一个非常简单的结构，并且只有一个名字。

像`object`或`classes`这些复杂类型，具有更复杂的结构。即使有时具有相同的结构，他们每个都有自己的名字。

一个静态类型检测器使用类型的名字和结构来将它们与其他类型进行比较，以名称检测的叫名义类型，以结构检测的叫结构类型。

## 名义类型

像`C++`、`Java`和`Swift`这些语言，有基础的名义类型系统。

```javascript
class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: string) { /* ... */ } }

let foo: Foo = new Bar(); // Error!
```

这里，当你试图将`Bar`放入期望是`Foo`的地方，可以看到一个名义类型系统错误，因为它们具有不同的名称。

## 结构类型

像`OCaml`、`Haskell`、`Elm`这些语言，有基础的结构类型系统。

```javascript
class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: string) { /* ... */ } }

let foo: Foo = new Bar(); // Works!
```

这里，可以看到一个结构类型系统的伪示例，当你试图将`Bar`放入期望是`Foo`的地方，因为它们的结构完全相同。

但是一旦你改变形状，就会导致错误。

```javascript
class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: number) { /* ... */ } }

let foo: Foo = new Bar(); // Error!
```

它可能比这更复杂一点。

我们己经演示了类的名义和结构类型，但是，这里还有其它复杂类型，比如对象、函数，它们也可以是名义类型，也可以是结构类型。此外，在同一类型系统中，它们也可以不同（前面列出的大多数语言都具有这两种语言的特性）

例如，`Flow`对于对象和函数使用结构类型，而对于类作用名义类型。

### 结构化类型的函数

当函数类型与函数比较时，它必须具有相同的结构才认为是有效的。

```javascript
// @flow
type FuncType = (input: string) => void;
function func(input: string) { /* ... */ }
let test: FuncType = func; // Works!
```

### 结构化类型的对象

当对象类型与对象比较时，它必须具有相同的结构才认为是有效的。

```javascript
type ObjType = { property: string };
let obj = { property: "value" };
let test: ObjType = obj;
```

### 名义化类型的类

当你有两个拥有相同结构的类，他们仍不被认为是相等的，因为`Flow`为类使用名义类型。

```javascript
// @flow
class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: string) { /* ... */ } }
let test: Foo = new Bar(); // Error!
```

如果你想在结构上使用类，你可以把它闪与对象混合，作为接口来实现：

```javascript
type Interface = {
  method(value: string): void;
};

class Foo { method(input: string) { /* ... */ } }
class Bar { method(input: string) { /* ... */ } }

let test: Interface = new Foo(); // Okay.
let test: Interface = new Bar(); // Okay.
```

### 混合名义和结构类型

在`Flow`里，关于混合名义和结构类型的设计方案是基于`JavaScript`中如何使用对象、函数和类来选择的。

`JavaScript`语言将面向对象思想和函数思想混合在一起，开发人员对JavaScript的使用也趋向于混合这两种思想。类（或者构造函数）更面向对象思想，函数（匿名函数）和对象更倾向于函数思想这边，开发人员同时使用这两者。

当有人编写了一个类时，相当于他们正在声明一件事物。这件事物可能和其他事物有相同的结构，但它们却有不同的用途。假设两个组件类都具有`render()`方法，这些组件可能仍然具有完全不同的用途，但是在结构类型系统中，它们被认为是完全相同的。

`Flow`选择什么对于`JavaScript`来说是自然的,并应该按照你期望的方式运作。