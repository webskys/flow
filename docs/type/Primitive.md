# 原始类型
（Primitive Types）

 > `Javascript`的原始值的类型

`Javascript`有许多不同的原始类型：

 - Boolean
 - String
 - Number
 - null
 - undefined ( 在`Flow`类型中为`void`)
 - Symbol (ES6, `Flow`暂不支持)

原始类型在语言中既可作为字面量值出现。

```javascript
true;
"hello";
3.14;
null;
undefined;
```

也可作为包装对象出现。

```javascript
new Boolean(false);
new String("world");
new Number(42);
Symbol("foo");
```

字面量值的类型都是小写的

```javascript
// @flow
function method(x: number, y: string, z: boolean) {
  // ...
}

method(3.14, "hello", true);
```

包装对象的类型是大写的（像他们的构造函数一样，首字母大写）

```javascript
// @flow
function method(x: Number, y: String, z: Boolean) {
  // ...
}

method(new Number(42), new String("world"), new Boolean(false));
```
包装对象类型很少被使用。

## Booleans

在 `Javascript` 中 `Booleans`是 `true`和`false`，`Flow`里的`boolean`类型也接受这两个值

```javascript
// @flow
function acceptsBoolean(value: boolean) {
  // ...
}

acceptsBoolean(true);  // Works!
acceptsBoolean(false); // Works!
acceptsBoolean("foo"); // Error!
```

`Javascript`会隐式的把其他类型的值转换为 `boolean`。

```javascript
if (42) {} // 42 => true
if ("") {} // "" => false
```

`Flow`理解这些转换，并且允许这些转换作为 `if` 语句或其他表达式的一部分。

`Boolean`类型需要明确地转换`non-booleans`，你可以使用 `Boolean(x)` 或者 `!!x` 。

```javascript
// @flow
function acceptsBoolean(value: boolean) {
  // ...
}

acceptsBoolean(0);          // Error!
acceptsBoolean(Boolean(0)); // Works!
acceptsBoolean(!!0);        // Works!
```

记住：`boolean` 和 `Boolean` 是不同的。

 - `boolean` 是程序中的一个字面量 `true` 、 `false`或者是一个比较表达式的结果值，例如：`a === b`
 - `Boolean`是一个`包装对象`。`new Boolean(x)`

`new Boolean(x)`与 `Boolean(x)`也是不同的哟，前者返回一个基本包装对象，后者返回类型转换字面量结果。

## Numbers

与许多其他语言不同，`Javascript`中数值只有一种类型，这些值像 `42` 或者`3.14`表现出来，在`Javascript`也认为`NaN`和`Infinity`是数值类型，`number`类型捕获所有`Javascript`认为的数值。

```javascript
// @flow
function acceptsNumber(value: number) {
  // ...
}

acceptsNumber(42);       // Works!
acceptsNumber(3.14);     // Works!
acceptsNumber(NaN);      // Works!
acceptsNumber(Infinity); // Works!
acceptsNumber("foo");    // Error!
```
注意：`number`和`Number`是不同的类型

 - `number` 是指一个字面量的值，例如：`42` 、 `3.14` 或者是一个表达式返回的数值`parseFloat(x)`。
 - `Number`是一个基本包装类型，`new Number(x)`。

当然，`new Number(x)`与`Number(x)`也是不同的，前者返回包装对象，后则是一个数值转换，返回一个字面号值。

## Strings

在`Javascript`里`string`是像`"foo"`，在`Flow`里`string`类型也也接受这些值。

```javascript
// @flow
function acceptsString(value: string) {
  // ...
}

acceptsString("foo"); // Works!
acceptsString(false); // Error!
```
`JavaScript` 通过连接其它类型的值将其隐式转换为 `string` 类型

```javascript
"foo" + 42; // "foo42"
"foo" + {}; // "foo[object Object]"
```

当拼接成字符串时，`Flow`仅仅接受 `string` 和`number`。

```javascript
// @flow
"foo" + "foo"; // Works!
"foo" + 42;    // Works!
"foo" + {};    // Error!
"foo" + [];    // Error!
```

你必须明确地将其它类型转换为字符串。要转换，你可以使用`String()`方法或使用其他转换为字符串的方法。

```javascript
// @flow
"foo" + String({});     // Works!
"foo" + [].toString();  // Works!
"" + JSON.stringify({}) // Works!
```

记住`string`和`String`是不同的类型。

 - `string` 是指一个字面量的值，例如：`'"foo"'` 、 `'"Hello world"'` 或者是一个表达式返回的数值`"" + 42`。
 - `String`是一个基本包装类型，`new String(x)`。

`String()`与`new String()`也是不同的，前者返回转换后的字符串字面量，后者返回的是一个字符串包装对象。

## null and void

`javascript`有`null`和`undefined`两个值，`Flow`把他们作为单独的类型：`null`和`void`(代表`undefined`)。

```javascript
// @flow
function acceptsNull(value: null) {
  /* ... */
}

function acceptsUndefined(value: void) {
  /* ... */
}

acceptsNull(null);      // Works!
acceptsNull(undefined); // Error!
acceptsUndefined(null);      // Error!
acceptsUndefined(undefined); // Works!
```

`null` 和 `void` 也会出再在其他类型。

## 可能的类型
（Maybe types）

可能类型代表这个位置的值是可选的，你可以在类型前添加一个问号`?`来创建可选类型，例如：`?string` 或者 `?number`。

除了`?type`中的`type`这个类型值外，可能类型也可以是 `null` 或者 `void`。

```javascript
// @flow
function acceptsMaybeString(value: ?string) {
  // ...
}

acceptsMaybeString("bar");     // Works!
acceptsMaybeString(undefined); // Works!
acceptsMaybeString(null);      // Works!
acceptsMaybeString();          // Works!
```

上面例子中如果调用函数不带参数其实质是参数为`undefined`。

## 可选的对象类型属性
（Optional object properties）

**对象类型**可以有可选的属性，在那里，该属性名后面有一个问号`?`。

```javascript
{ propertyName?: string }
```

除了设置的`type`外，可选属性既可以是`void`也可以省略整个属性。但是，这个属性不能为`null`。

```javascript
// @flow
function acceptsObject(value: { foo?: string }) {
  // ...
}

acceptsObject({ foo: "bar" });     // Works!
acceptsObject({ foo: undefined }); // Works!
acceptsObject({ foo: null });      // Error!
acceptsObject({});                 // Works!
```

## 可选的函数参数
（Optional function parameters）

函数可以有可选的参数，在那里，该参数名后面有一个问号`?`。

```javascript
function method(param?: string) { /* ... */ }
```

除了设置的类型外，可选参数既可以是`void`也可以直接省略这个参数。但是，这个参数不能是`null`

```javascript
// @flow
function acceptsOptionalString(value?: string) {
  // ...
}

acceptsOptionalString("bar");     // Works!
acceptsOptionalString(undefined); // Works!
acceptsOptionalString(null);      // Error!
acceptsOptionalString();          // Works!
```

其实省略不传这个参数就是这个参数为`undefined`(在`Flow`里为`void`)。

## 带默认值的函数参数
（Function parameters with defaults）

函数参数可以有默认值，这是`ECMAScript 2015`的特性。

```javascript
function method(value: string = "default") { /* ... */ }
```

除了设置的类型外，带默认值参数既可是是`void`也可以省略这个参数。但是，这个参数不为`null`

```javascript
// @flow
function acceptsOptionalString(value: string = "foo") {
  // ...
}

acceptsOptionalString("bar");     // Works!
acceptsOptionalString(undefined); // Works!
acceptsOptionalString(null);      // Error!
acceptsOptionalString();          // Works!
```

省略不传这个参数就是这个参数为`undefined`(在`Flow`里为`void`)。

## Symbol

`Symbols`暂时不被`Flow`所被支持。
