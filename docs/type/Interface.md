# 接口类型

类（Classes ）在`Flow`里是名义类型，意思是当你有两个单独的类你不能使用其中一个去替换另一个，即使他们有相同的属性和方法。

```javascript
// @flow
class Foo {
  serialize() { return '[Foo]'; }
}

class Bar {
  serialize() { return '[Bar]'; }
}

// $ExpectError
const foo: Foo = new Bar(); // Error!
```

然而，你可以使用`interface`以便声明一个你所期望的类的结构。

```javascript
// @flow
interface Serializable {
  serialize(): string;
}

class Foo {
  serialize() { return '[Foo]'; }
}

class Bar {
  serialize() { return '[Bar]'; }
}

const foo: Serializable = new Foo(); // Works!
const bar: Serializable = new Bar(); // Works!
```

你也可以使用`implements`告诉`Flow`你想一个类去匹配一个接口(interface),这可以预防你在编辑类时造成的不兼容的修改。

```javascript
// @flow
interface Serializable {
  serialize(): string;
}

class Foo implements Serializable {
  serialize() { return '[Foo]'; } // Works!
}

class Bar implements Serializable {
  // $ExpectError
  serialize() { return 42; } // Error!
}
```

也可以使用`implements`实现多个接口。

```javascript
class Foo implements Bar, Baz {
  // ...
}
```

## 语法

使用`interface`关键字创建接口，在其后面紧跟它的名字和定义其类型的内容块`{}`

```javascript
interface MyInterface {
  // ...
}
```

这个块`{}`的语法跟跟对象类型语法一样，并有对象类型的所有特性。

### 接口的方法

你可以用遵循对象添加方法一样的语法给接口`interfaces`添加方法。

```javascript
interface MyInterface {
  method(value: string): number;
}
```

### 接口的属性

你可以用给对象添加属性一样的语法来给接口`interfaces`添加属性。

```javascript
interface MyInterface {
  property: string;
}
```

接口的属性也可以是可选的。

```javascript
interface MyInterface {
  property?: string;
}
```

### 把接口当作maps

你可以用对象类型相同的方法来创建索引器属性`indexer properties`。

```javascript
interface MyInterface {
  [key: string]: number;
}
```

## 接口泛型

接口也可以有他们自己的泛型

```javascript
interface MyInterface<A, B, C> {
  property: A;
  method(val: B): C;
}
```

接口泛型是参数化的。当你使用一个接口的时候，你需要为他们每个泛型传递参数。

```javascript
// @flow
interface MyInterface<A, B, C> {
  foo: A;
  bar: B;
  baz: C;
}

var val: MyInterface<number, boolean, string> = {
  foo: 1,
  bar: true,
  baz: 'three',
};
```

## 属性变化 (只读和只写)

默认情况下，一个接口的属性是不变的，但是你也可以添加修饰使他们变为`协变`(read-only) 或者`逆变` (write-only).

```javascript
interface MyInterface {
  +covariant: number;     // read-only
  -contravariant: number; // write-only
}
```

### 协变属性 (read-only)

你可以添加一个加号`+`在属性名前，使其成为协变属性

```javascript
interface MyInterface {
  +readOnly: number | string;
}
```

这样允许您传递更具体的类型来代替该属性。

```javascript
// @flow
// $ExpectError
interface Invariant {  property: number | string }
interface Covariant { +readOnly: number | string }

var value1: Invariant = { property: 42 }; // Error!
var value2: Covariant = { readOnly: 42 }; // Works!
```

由于**协方差**(`covariance`)的工作原理，在使用时协变属性也变为只读了，这将比普通属性更有用。

```javascript
// @flow
interface Invariant {  property: number | string }
interface Covariant { +readOnly: number | string }

function method1(value: Invariant) {
  value.property;        // Works!
  value.property = 3.14; // Works!
}

function method2(value: Covariant) {
  value.readOnly;        // Works!
  // $ExpectError
  value.readOnly = 3.14; // Error!
}
```

### 逆变属性 (write-only)

你可以添加一个减号`-`在属性名前，使其成为逆变属性

```javascript
interface InterfaceName {
  -writeOnly: number;
}
```

这样允许您传递不太具体的类型来代替该属性。

```javascript
// @flow
interface Invariant     {  property: number }
interface Contravariant { -writeOnly: number }

var numberOrString = Math.random() > 0.5 ? 42 : 'forty-two';

// $ExpectError
var value1: Invariant     = { property: numberOrString };  // Error!
var value2: Contravariant = { writeOnly: numberOrString }; // Works!
```

由于逆变的工作原理，逆变属性在使用时也变为只写了，这将比正常普通属性更有用。

```javascript
interface Invariant     {   property: number }
interface Contravariant { -writeOnly: number }

function method1(value: Invariant) {
  value.property;        // Works!
  value.property = 3.14; // Works!
}

function method2(value: Contravariant) {
  // $ExpectError
  value.writeOnly;        // Error!
  value.writeOnly = 3.14; // Works!
}
```