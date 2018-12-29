# 子集与子类型


## 什么是子类型?

像`number`, `boolean`和`string`类型描述的是可能值的集合，一个`number`描述每个可能的数字，所以一个单独的数字(例如42)就是一个`number`类型的`子类型`。

如果我们想知道一种类型是否是另一种类型的子类型，我们需要查找出这两种类型的所有可能值并算出另一种类型是否有这个值的子集。

例如，如果我们有一个`TypeA`,它描述了1到3的数字。还有一个`TypeB`,它描述了1到5的数字。`TypeA`被视为`TypeB`的子类型，因为`TypeA`是`TypeB`的子集。

```javascript
type TypeA = 1 | 2 | 3;
type TypeB = 1 | 2 | 3 | 4 | 5;
```

`TypeLetters`描述了字符串“A”, “B”, “C”。`TypeNumbers`描述了数字1, 2, 3。他们都不是另一类型的子类型，因为他们各自包含了完全不同的值的集合。

```javascript
type TypeLetters = "A" | "B" | "C";
type TypeNumbers =  1  |  2  |  3;
```

最后，如果我们有一个描述1到3的`TypeA`,和一个描述3到5的`TypeB`,这两个都不是其中另一个的子类型。尽管他们都有3且都描述数字，但他们每个都包含了另一个没有的单独的项。

```javascript
type TypeA = 1 | 2 | 3;
type TypeB = 3 | 4 | 5;
```

## 子类型用在什么时候?

`Flow`所做的大部分工作是比较类型。

例如，为了确定你是正确地调用了一个函数，`Flow`需要把你传递的实参跟函数期望的参数进行比较。

这通常意味着要算出你传递的值是不是你所期望的值的子类型。

所以，如果我编写一个期望传入数字1到5的函数，它将接受参数的任何子类型集。

```javascript
// @flow
function f(param: 1 | 2 | 3 | 4 | 5) {
  // ...
}

declare var oneOrTwo: 1 |  2; // Subset of the input parameters type.
declare var fiveOrSix: 5 | 6; // Not a subset of the input parameters type.

f(oneOrTwo); // Works!
// $ExpectError
f(fiveOrSix); // Error!
```

## 复杂类型的子类型

`Flow`不仅仅比较原始值的集合，它也需要能够比较对象、函数和在语言中出现在的其他每个类型。

### 对象的子类型

你可以通过他们的键来比较两个对象。如果一个对象包含了另一对象的所有键，则这个对象是另一对象的子类型。

例如，我们有一个`ObjectA`对象，包含了`foo`键，`ObjectB`对象包含了`foo`键和`bar`键，则`ObjectB`可能是`ObjectA`的子类型。

```javascript
// @flow
type ObjectA = { foo: string };
type ObjectB = { foo: string, bar: number };

let objectB: ObjectB = { foo: 'test', bar: 42 };
let objectA: ObjectA = objectB; // Works!
```

但是，我们也需要比较值的类型，如果两个对象都有一个`foo`的键，但是一个是`number`另一人是`string`,则其中一个不会是另一个的子类型

```javascript
// @flow
type ObjectA = { foo: string };
type ObjectB = { foo: number, bar: number };

let objectB: ObjectB = { foo: 1, bar: 2 };
// $ExpectError
let objectA: ObjectA = objectB; // Error!
```

如果对象上的值又是其他对象，我们就必须将他们再进行比较。我们需要递归地比较每个值，直到我们能够确定是否为子类型

### 函数的子类型

函数的子类型规则更难懂。迄今为止，我们相信如果`B`包含`A`中所有可以的值，则`A`是`B`的子类型。对于函数，不清晰如何应用这种关系。简而言之，你可以认为，如果函数类型`A`可以用于任何期望使用函数类型`B`的地方，则`A`是`B`的子类型。

假设我们有一个函数类型和一些函数，哪些函数可以安全地在期望给定函数类型的代码中使用？

```javascript
type FuncType = (1 | 2) => "A" | "B";

let f1: (1 | 2) => "A" | "B" | "C" = (x) => /* ... */
let f2: (1 | null) => "A" | "B" = (x) => /* ... */
let f3: (1 | 2 | 3) => "A" = (x) => /* ... */
```

 - `f1`能返回一个`C`值，但`FuncType`不会，所以`f1`的代码使用`FuncType`是不安全的，它的类型不是`FuncType`的子类型。
 - `f2`不能操作`FuncType`里所有实参(`2`),所以`FuncType`的代码用于`f2`是不安全的，它的类型也不是`FuncType`的子类型。
 - `f3`能授受所有`FuncType`的实参，并且仅仅返回`FuncType`也能返回的值，因此它的类型也是`FuncType`的子类型

总之，函数子类型规则是这样的：当且仅当`B`的输入是`A`的超集，并且`B`的输出是`A`的子集，这个函数类型`B`就是函数类型`A`的子类型。这个子类型必须至少接受跟父类相同的输入且必须最多返回跟父类相同的输出。（子类型的输入要大于等于父类，输出要小于等于父类）

决定如何应用子类型输入与输出规则的是变异，这是下一节的主题。