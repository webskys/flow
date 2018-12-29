# 泛型

 > 使用泛型添加抽象（多态）类型

泛型（有时称为多态类型）是一种把类型抽象出来的方法（抽象类型）。

假如编写如下`identity`函数，它返回任何被传入的参数。

```javascript
function identity(value) {
  return value;
}
```

我们试图给这个函数一个特定的类型会遇到许多麻烦，因为它可能是任何类型的值。

```javascript
function identity(value: string): string {
  return value;
}
```

然而我们可以在函数里创建一个泛型（或者多态类型），并用它代替其他类型。

```javascript
function identity<T>(value: T): T {
  return value;
}
```

泛型可以用于函数、函数类型、类、类型别名和接口。

>警告：`Flow`不会推断泛型类型。如果您希望某些东西具有泛型类型，请对其进行类型注解，否则，`Flow`可能会推断出比您预期多态性要少的类型。

在下面的例子中，我们忘记给`identity`恬当地注释为泛型，所以当我们将其分配`func`的时候会遇到麻烦。另一方面，`genericIdentity`有恬当地类型，我们就可以像期望的一样使用它。

```javascript
// @flow

type IdentityWrapper = {
  func<T>(T): T
}

function identity(value) {
  return value;
}

function genericIdentity<T>(value: T): T {
  return value;
}

// $ExpectError
const bad: IdentityWrapper = { func: identity }; // Error!
const good: IdentityWrapper = { func: genericIdentity }; // Works!
```

## 语法

在语法中出现泛型类型的地方有很多。

### 函数使用泛型

在函数中可以在参数列表的前面添加一个参数的类型列表`<T>`来创建一个泛型。

你也可以在函数中其他可以添加类型的地方使用泛型（参数或者返回值）。

```javascript
function method<T>(param: T): T {
  // ...
}

function<T>(param: T): T {
  // ...
}
```

### 函数类型使用泛型

函数类型可以跟普通函数一样的方法创建一个泛型，在函数类型参数列表之门添加一个类型参数列表`<T>`。

```javascript
<T>(param: T) => T
```

然后将其用作自己的类型。

```javascript
function method(func: <T>(param: T) => T) {
  // ...
}
```

### 类使用泛型

类可以通过把类型参数列表放置在类的主体之前来创建泛型。

```javascript
class Item<T> {
  // ...
}
```

在类中可以添加其他类型相同的地方，你都能使用泛型（属性类型和方法参数/返回类型）

```javascript
class Item<T> {
  prop: T;

  constructor(param: T) {
    this.prop = param;
  }

  method(): T {
    return this.prop;
  }
}
```

### 类型别名使用泛型

```javascript
type Item<T> = {
  foo: T,
  bar: T,
};
```

### 接口使用泛型

```javascript
interface Item<T> {
  foo: T,
  bar: T,
}
```

### 类型参数

您可以在调用中直接为它们的泛型提供可调用实际类型参数。

```javascript
//@flow
function doSomething<T>(param: T): T {
  // ...
  return param;
}

doSomething<number>(3);
```

你也可以在一个`new`表达式里直接给类类型泛型参数

```javascript
//@flow
class GenericClass<T> {}
const c = new GenericClass<number>();
```

如果你仅仅想指定一些类型参数，你可以使用`_`让`Flow`为你推断一个类型

```javascript
//@flow
class GenericClass<T, U, V>{}
const c = new GenericClass<_, number, _>()
```

> 警告：由于性能的原因，我们推荐在参数里尽量使用具体的注释参数。`_`并不是不安全，而且比明确地指定类型参数要慢。

## 行为

### 类似于变量

泛型的行为除了用于类型很像变量或函数参数，只要他们在作用域内，任何时候都能使用它们。

```javascript
function constant<T>(value: T): () => T {
  return function(): T {
    return value;
  };
}
```

### 任意数量

在你的类型参数列表里，可以根据需要尽可能多地使用泛型，并根据需要为他们命名。

```javascript
function identity<One, Two, Three>(one: One, two: Two, three: Three) {
  // ...
}
```

### 跟踪值

当使用一个泛型类型值时，`Flow`将跟踪这个值并确保不会将其替换为其他值

```javascript
// @flow
function identity<T>(value: T): T {
  // $ExpectError
  return "foo"; // Error!
}

function identity<T>(value: T): T {
  // $ExpectError
  value = "foo"; // Error!
  // $ExpectError
  return value;  // Error!
}
```

`Flow`跟踪你使用泛型传递的一个值具体类型，以便稍后使用。

```javascript
// @flow
function identity<T>(value: T): T {
  return value;
}

let one: 1 = identity(1);
let two: 2 = identity(2);
// $ExpectError
let three: 3 = identity(42);
```

### 添加类型

类似于`mixed`,泛型有一个未知（`unknown`）类型，不允许你将泛型当作一个特定的类型使用。

```javascript
// @flow
function logFoo<T>(obj: T): T {
  // $ExpectError
  console.log(obj.foo); // Error!
  return obj;
}
```

您可以细化类型，但泛型仍然允许传入任何类型。

```javascript
// @flow
function logFoo<T>(obj: T): T {
  if (obj && obj.foo) {
    console.log(obj.foo); // Works.
  }
  return obj;
}

logFoo({ foo: 'foo', bar: 'bar' });  // Works.
logFoo({ bar: 'bar' }); // Works. :(
```

相反，您可以像使用函数参数那样向泛型添加类型。

```javascript
// @flow
function logFoo<T: { foo: string }>(obj: T): T {
  console.log(obj.foo); // Works!
  return obj;
}

logFoo({ foo: 'foo', bar: 'bar' });  // Works!
// $ExpectError
logFoo({ bar: 'bar' }); // Error!
```

通过这种方式，您可以保持泛型的行为，同时只允许使用某些类型。

```javascript
// @flow
function identity<T: number>(value: T): T {
  return value;
}

let one: 1 = identity(1);
let two: 2 = identity(2);
// $ExpectError
let three: "three" = identity("three");
```

### 充当界限

```javascript
// @flow
function identity<T>(val: T): T {
  return val;
}

let foo: 'foo' = 'foo';           // Works!
let bar: 'bar' = identity('bar'); // Works!
```

在`Flow`里，大部分时间当你将一个类型传递给给另一个类型时会丢失原始类型。因此，当你传递一个具体类型到一个不太具体类型时，`Flow`会忘记"forgets"它曾经是更具体的类型。

```javascript
// @flow
function identity(val: string): string {
  return val;
}

let foo: 'foo' = 'foo';           // Works!
// $ExpectError
let bar: 'bar' = identity('bar'); // Error!
```

泛型允许您在添加约束时保留更具体的类型，这样，泛型上的类型就充当了界限（`bounds`）

```javascript
// @flow
function identity<T: string>(val: T): T {
  return val;
}

let foo: 'foo' = 'foo';           // Works!
let bar: 'bar' = identity('bar'); // Works!
```

注意，当你有一个值在泛型界限内，你不能将其作为更具体的类型来使用

```javascript
// @flow
function identity<T: string>(val: T): T {
  let str: string = val; // Works!
  // $ExpectError
  let bar: 'bar'  = val; // Error!
  return val;
}

identity('bar');
```

### 参数化泛型

泛型有时允许你像传递函数参数一样将一个类型传递进去，这又叫作参数化泛型（或者参数多态性）。

例如，参数化的一个具有泛型的类型别名。当你使用它时必需提供类型参数。

```javascript
type Item<T> = {
  prop: T,
}

let item: Item<string> = {
  prop: "value"
};
```

你可以认为这像传递参数给函数，只是返回值是一个你可以使用的类型。

类`Classes`(当作为一个类型使用时)、类型别名和接口都需要你传递类型参数。函数和函数类型没有参数化泛型

#### Classes

```javascript
// @flow
class Item<T> {
  prop: T;
  constructor(param: T) {
    this.prop = param;
  }
}

let item1: Item<number> = new Item(42); // Works!
// $ExpectError
let item2: Item = new Item(42); // Error!
```

#### Type Aliases

```javascript
// @flow
type Item<T> = {
  prop: T,
};

let item1: Item<number> = { prop: 42 }; // Works!
// $ExpectError
let item2: Item = { prop: 42 }; // Error!
```

#### Interfaces

```javascript
// @flow
interface HasProp<T> {
  prop: T,
}

class Item {
  prop: string;
}

(Item.prototype: HasProp<string>); // Works!
// $ExpectError
(Item.prototype: HasProp); // Error!
```

#### 参数化泛型默认值

你也可以为参数化泛型提供默认值就像一个函数的参数一样

```javascript
type Item<T: number = 1> = {
  prop: T,
};

let foo: Item<> = { prop: 1 };
let bar: Item<2> = { prop: 2 };
```

当使用类型时，必须始终包括括号`<>`（就像函数调用的括号一样）。

### 变异标记

你也可以通过变异标记指定泛型的子类型行为。默认情况下，泛型表现为不变的，但是你可以在声明时添加一个加号（`+`）使他们表现为协变，或者使用一个减号（`-`）使他们表现为逆变。更多关于`Flow`里变异的信息请查看 `变异文档`

变异标记能具体解决你打算怎么使用泛型，给`Flow`提供更明确的类型检测的能力。例如，你可能想要这种关系：

```javascript
//@flow
type GenericBox<+T> = T;

var x: GenericBox<number> = 3;
(x: GenericBox<number| string>);
```

上面的这个例子不使用`+`变异标记是不可能完成的。

```javascript
//@flow
type GenericBoxError<T> = T;

var x: GenericBoxError<number> = 3;
(x: GenericBoxError<number| string>); // number | string is not compatible with number.
```

注意，如果你用变异标记注解你的泛型，`Flow`将检查以确保那些类型仅仅出现在对那个变异标记有意思的位置上。例如，你不能声明一个协变的泛型参数并把它用在逆变的地方。

```javascript
//@flow
type NotActuallyCovariant<+T> = (T) => void;
```