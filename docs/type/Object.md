# 对象类型
Object Types

 > 把不同类别的对象作为类型

在`Javascript`中对象有很多不同的用法，为了支持所有不同的用例，这里也有许多不同的方法来对他们类型化。


## 语法

对象类型尽可能多地匹配`Javascript`中对象的语法。使用花括号`{}`，名值对使用冒号`:`，并用逗号`,`分隔开不同名值对。

```javascript
// @flow
var obj1: { foo: boolean } = { foo: true };
var obj2: {
  foo: number,
  bar: boolean,
  baz: string,
} = {
  foo: 1,
  bar: true,
  baz: 'three',
};
```

 > 注意：以前对象类型名值对使用分号`;`分隔，虽然这种语法仍然有效，但你应该使用逗号`,`。

## 对象类型可选的属性

在`Javascript`中访问一个不存在的属性得到`undefined`,这是`Javascript`程序常见错误的来源，因上`Flow`将这些转换为类型错误。

```javascript
// @flow
var obj = { foo: "bar" };
// $ExpectError
obj.bar; // Error!
```

如果你有一个对象有时没有某个属性，可以在对象类型属性名的后面添加一个问号(`?`)使这个属性变为可选属性`optional property`。

```javascript
// @flow
var obj: { foo?: boolean } = {};

obj.foo = true;    // Works!
// $ExpectError
obj.foo = 'hello'; // Error!
```

除了授受设置的类型外，这些可选属性还可以是`void`或直接省略这个属性，但是不能为`null`。

```javascript
// @flow
function acceptsObject(value: { foo?: string }) {
  // ...
}

acceptsObject({ foo: "bar" });     // Works!
acceptsObject({ foo: undefined }); // Works!
// $ExpectError
acceptsObject({ foo: null });      // Error!
acceptsObject({});                 // Works!
```

## 对象类型推断

`Flow`根据它们使用情况有两种方法去推断字面量对象的类型。

### 密封对象

当你创建具有属性的对象，你在`Flow`里创建了一个密封对象类型。这些密封对象知道你声明的所有属性和根据这些属性值知道他们的类型。

`Javasctipt`控制对象的权限

**防止扩展**，不可增加新的属性
**密封**，不可增加新的属性，不可删除属性或修改属性的特性，但是值仍然可以被修改（`writable`为`true`））
**冻结**，不可增加新的属性，不可删除和修改属性的特殊，不可修改值

```
// @flow
var obj = {
  foo: 1,
  bar: true,
  baz: 'three'
};

var foo: number  = obj.foo; // Works!
var bar: boolean = obj.bar; // Works!
// $ExpectError
var baz: null    = obj.baz; // Error!
var bat: string  = obj.bat; // Error!
```

当对象是密封的，`Flow`将不允许你为他们添加新的属性。

```javascript
// @flow
var obj = {
  foo: 1
};

// $ExpectError
obj.bar = true;    // Error!
// $ExpectError
obj.baz = 'three'; // Error!
```

这里的解决方案可能是将对象转为未密封对象`unsealed object`。

### 未密封对象

当你创建没有任何属性的对象，你在`Flow`里就创建了一个未密封对象`unsealed object`，这些未密封对象不知道它们的属性，并允许你添加新属性。

```javascript
// @flow
var obj = {};

obj.foo = 1;       // Works!
obj.bar = true;    // Works!
obj.baz = 'three'; // Works!
```

属性的推断类型变成了你设置的属性。

```javascript
// @flow
var obj = {};
obj.foo = 42;
var num: number = obj.foo;
```

#### 未密封对象重新赋值

如果你给一个未密封对象的属性重新赋值跟`let`或`var`声明变量相似，默认情况下`Flow`将给他所有赋值可能的类型。


```javascript
// @flow
var obj = {};

if (Math.random()) obj.prop = true;
else obj.prop = "hello";

// $ExpectError
var val1: boolean = obj.prop; // Error!
// $ExpectError
var val2: string  = obj.prop; // Error!
var val3: boolean | string = obj.prop; // Works!
```

有时，重新赋值后`Flow`能够推算出（确定性）属性的类型，在这种情况下，`Flow`将给属性这个确定的类型。

```javascript
// @flow
var obj = {};

obj.prop = true;
obj.prop = "hello";

// $ExpectError
var val1: boolean = obj.prop; // Error!
var val2: string  = obj.prop; // Works!
```

当`Flow`越来越聪明的时候，它将在更多的情况推算出属性的类型。

#### 查找未知属性

 > 在未密封的对象上查找未知属性是不安全的。

未密封的对象允许任何时候写入新属性。`Flow`确保读取与写入相兼容，但是不能确保写入发生在读取之前（按执行顺序）

这意味着从未密封的对象读取还没被写入的属性不会被检测。这是`Flow`的一个不安全的行为，这点可能会在将来得到改善。

```javascript
var obj = {};

obj.foo = 1;
obj.bar = true;

var foo: number  = obj.foo; // Works!
var bar: boolean = obj.bar; // Works!
var baz: string  = obj.baz; // Works?
```

## 精确对象类型

在`Flow`里，当传递一个拥有额外属性的且是期望对象类型的对象是安全的。

```javascript
// @flow
function method(obj: { foo: string }) {
  // ...
}

method({
  foo: "test", // Works!
  bar: 42      // Works!
});
```

 > 注：这是因为`宽子类型化`

有时禁用这个行为只允许设置的那些特定属性是非常有用的，为此，`Flow`支持精确对象类型。

```javascript
{| foo: string, bar: number |}
```

与常规对象类型不同，传递一个具有额外属性的对象到精确对象类型是无效的。

```javascript
// @flow
var foo: {| foo: string |} = { foo: "Hello", bar: "World!" }; // Error!
```

精确对象类型的交集可能并不像你期望的那样工作，如果你需要拼结精确对象类型，得使用对象类型扩展（ES6里扩展运算符`...`）。

```javascript
// @flow

type FooT = {| foo: string |};
type BarT = {| bar: number |};

type FooBarFailT = FooT & BarT;
type FooBarT = {| ...FooT, ...BarT |};

const fooBarFail: FooBarFailT = { foo: '123', bar: 12 }; // Error!
const fooBar: FooBarT = { foo: '123', bar: 12 }; // Works!
```

## 对象作为 maps

在新版本的`Javascript`标准中包含了`Map`类，但是把`objects`作为`maps`仍然很常见，在这种使用情况下，一个对象很有可能在整个生命周期中添加或检索属性，而且，属性键甚至可能不是可知静态的，所以写出一种类型注解几乎是不可能的。

对于这样的对象，`Flow `提供了一个特殊的属性类别叫做属性索引器（`indexer property`），一个属性索引器允许使用与索引器键类型匹配的任何键进行读写。

```javascript
// @flow
var o: { [string]: number } = {};
o["foo"] = 0;
o["bar"] = 1;
var foo: number = o["foo"];
```

一个索引器可以根据文件目的随意命名：

```javascript
// @flow
var obj: { [user_id: number]: string } = {};
obj[1] = "Julia";
obj[2] = "Camille";
obj[3] = "Justin";
obj[4] = "Mark";
```

当一个对象类型有一个索引器属性，访问属性被假定为有这个注解的类型，即使对象在运行时该插糟并没有值。确保属性访问是安全，这是程序员的职责，就像访问数组元素一样。

```javascript
var obj: { [number]: string } = {};
obj[42].length;// No type error, but will throw at runtime
```

属性索引器可以与命名的属性混合使用

```javascript
// @flow
var obj: {
  size: number,
  [id: number]: string
} = {
  size: 0
};

function add(id: number, name: string) {
  obj[id] = name;
  obj.size++;
}
```

## 对象类型

有时，写一个接受任意对象的类型是很有用的，为此你应该向下面一样写`{}`：

有时用`{}`编写接受任意类型是非常有用的

```javascript
function method(obj: {}) {
  // ...
}
```

然而，如果你想跳出类型检测又不想用`any`的方法转变成任何类型，你可以使用`Object`来代替。`Object`是不安全的，应该避免。

例如，以下代码不会报告任何错误

```javascript
function method(obj: Object) {
  obj.foo = 42;               // Works.
  let bar: boolean = obj.bar; // Works.
  obj.baz.bat.bam.bop;        // Works.
}

method({ baz: 3.14, bar: "hello" });
```
