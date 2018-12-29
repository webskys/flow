# 类型转换表达式

 > 不同类型的断言与转换

有时，没有用像函数或变量这些去断言一个类型是很有用的。为此，`Flow`支持一种行内类型转换的表达式语法，这语法有许多不同的使用方法。

## 语法

为了创建一个值`value`类型转换表达式，在`value`后添加一个冒号`:`和其类型，并使用括号`( )`把表达式包裹起来。

```javascript
(value: Type)
```

 > 注意：括号是必需的，以避免和其他语法产生歧义。

类型转换表达式可以出现在任何表达式可以出现的地方。

```javascript
let val = (value: Type);
let obj = { prop: (value: Type) };
let arr = ([(value: Type), (value: Type)]: Array<Type>);
```

这个值本身也可以是一个表达式。

```javascript
(2 + 2: number);
```

在剥离类型时，剩下的就是值

```javascript
(value: Type);

value;
```

##　类型断言

使用类型转换表达式，你可以断言值是某一类型

```javascript
// @flow
let value = 42;

(value: 42);     // Works!
(value: number); // Works!
(value: string); // Error!
```

以这种方式断言类型的工作方式与其他任何地方的类型断言相同。

## 类型转换

当你编写一个类型转换表达式时，这个表达式的结果是所提供的类型的值。如果把这个结果值保留下来，其类型就是这个新的类型。

```javascript
// @flow
let value = 42;

(value: 42);     // Works!
(value: number); // Works!

let newValue = (value: number);

// $ExpectError
(newValue: 42);     // Error!
(newValue: number); // Works!
```

## 使用类型转换表达式

 > 注意：我们将通过一个精简的例子来演示怎么使用类型转换表达式。这个例子在实践中并不是好的解决方案。

### 通过`any`类型转换

因为类型转换跟所有其他类型注解的工作方式相同，所以只能把值转换为不太具体的类型。你不能更改类型或者使它变为更具体的类型。

但是你可以使用`any`来转换为你想要的任何类型。

```javascript
let value = 42;

(value: number); // Works!
// $ExpectError
(value: string); // Error!

let newValue = ((value: any): string);

// $ExpectError
(newValue: number); // Error!
(newValue: string); // Works!
```

通过将值转换为`any`,然后你就可以将其转换为任何您想要的。

这是不安全的并且不建议这样做。但是，当您使用一个很难或不可能的值转换，并想确保结果是具有所需的类型时，它是有用的。

例如，下面这个克隆一个对象的函数。

```javascript
function cloneObject(obj) {
  const clone = {};

  Object.keys(obj).forEach(key => {
    clone[key] = obj[key];
  });

  return clone;
}
```

要为此创建一个类型是很困难的，因为我们正在基于另一个对象创建一个新对象。

如果我们通过`any`转换，可以得到一个更有用的类型。

```javascript
// @flow
function cloneObject(obj) {
  const clone = {};

  Object.keys(obj).forEach(key => {
    clone[key] = obj[key];
  });

  return ((clone: any): typeof obj); // <<
}

const clone = cloneObject({
  foo: 1,
  bar: true,
  baz: 'three'
});

(clone.foo: 1);       // Works!
(clone.bar: true);    // Works!
(clone.baz: 'three'); // Works!
```

### 通过断言来检测类型

以前，我们想验证进入`cloneObject`方法的类型是什么，可以编写以下注解：

```javascript
function cloneObject(obj: { [key: string]: mixed }) {
  // ...
}
```

但是现在我们遇到了一个问题，我们的`typeof obj`注解也会得到这个新注解，破坏了整个目的。

```javascript
// @flow
function cloneObject(obj: { [key: string]: mixed }) {
  const clone = {};
  // ...
  return ((clone: any): typeof obj);
}

const clone = cloneObject({
  foo: 1,
  bar: true,
  baz: 'three'
});

// $ExpectError
(clone.foo: 1);       // Error!
// $ExpectError
(clone.bar: true);    // Error!
// $ExpectError
(clone.baz: 'three'); // Error!
```

相反，我们能在函数里使用一个类型断言来断言类型，现在我们将验证我们的输入。

```javascript
// @flow
function cloneObject(obj) {
  (obj: { [key: string]: mixed });
  // ...
}

cloneObject({ foo: 1 }); // Works!
// $ExpectError
cloneObject([1, 2, 3]);  // Error!
```

现在，类型推断可以继续为`typeof obj`工作，它返回这个对象预期的形状。

```javascript
// @flow
function cloneObject(obj) {
  (obj: { [key: string]: mixed }); // <<

  const clone = {};
  // ...
  return ((clone: any): typeof obj);
}

const clone = cloneObject({
  foo: 1,
  bar: true,
  baz: 'three'
});

(clone.foo: 1);       // Works!
(clone.bar: true);    // Works!
(clone.baz: 'three'); // Works!
```

 > 对于上面的例子，这并不是一个恰当的解决方案，仅仅是一个使用示范，正确的解决方案是像下面这样注释函数。

```javascript
function cloneObject<T: { [key: string]: mixed }>(obj: T): $Shape<T> {
 // ...
}
```