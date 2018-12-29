# 交集类型

有时，创建一个属于所有其它类型集合的类型是非常有用的。例如，你可能想编写某个接受一个对象的函数，这个对象是其它对象类型的组合。为此，Flow支持交集类型。

```javascript
// @flow
type A = { a: number };
type B = { b: boolean };
type C = { c: string };

function method(value: A & B & C) {
  // ...
}

// $ExpectError
method({ a: 1 }); // Error!
// $ExpectError
method({ a: 1, b: true }); // Error!
method({ a: 1, b: true, c: 'three' }); // Works!
```

## 语法

交集类型是用`&`符号连接的任意数量类型。

```javascript
Type1 & Type2 & ... & TypeN
```

你也可以添加一个引导的连接符`&`把交集类型拆分成多行也是很有用的。

```javascript
type Foo =
  & Type1
  & Type2
  & ...
  & TypeN
```

交集类型中的每个成员可以是任何类型，甚至是另一个交集类型。

```javascript
type Foo = Type1 & Type2;
type Bar = Type3 & Type4;

type Baz = Foo & Bar;
```

## 所有进，一个出

交集类型与联合类型相反，当调用一个接受交集类型的函数，我们必须传入所有这些类型，但是在我们的函数内部，我们只需要把它当作这些类型中的任何一种对待即可。

```javascript
// @flow
type A = { a: number };
type B = { b: boolean };
type C = { c: string };

function method(value: A & B & C) {
  var a: A = value;
  var b: B = value;
  var c: C = value;
}
```

即使我们把个值仅当其中的一种类型对待，我们也不会得到一个错误，因为它满足了所有类型。


## 不可能的交集类型

使用交集类型时，创建一个在运行时无法创建的类型是可能的。交集类型允许我们组合任意类型集，即使是一些相互冲突的类型。

例如，你可以创建一个数字和字符串的交集

```javascript
// @flow
type NumberAndString = number & string;

function method(value: NumberAndString) {
  // ...
}

// $ExpectError
method(3.14); // Error!
// $ExpectError
method('hi'); // Error!
```

你不可能创建一个即是数值又是字符串的值，但是您可以创建一个这样的类型。创建这样的类型没有实际的用途，它是交集类型工作方式的副作用。

## 对象类型的交集

当你创建一个对象类型的交集，它们所有的属性就合并在了一起。

例如，当你使用两个拥有不同属性集的对象创建一个交集，它的结果就成了拥有所有这些属性的对象

```javascript
// @flow
type One = { foo: number };
type Two = { bar: boolean };

type Both = One & Two;

var value: Both = {
  foo: 1,
  bar: true
};
```

当有同名的属性重叠时，它也会创建属性类型的交集。

例如，如果你合并两个对象，他们有一个叫`prop`的属性名，一个的类型是`number`，另一个的类型是`boolean`，这个属性合并后的结果是`number`和`boolean`类型的交集。

```javascript
// @flow
type One = { prop: number };
type Two = { prop: boolean };

type Both = One & Two;

// $ExpectError
var value: Both = {
  prop: 1 // Error!
};
```