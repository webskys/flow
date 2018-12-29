# 可能类型
（Maybe Types）

 > 表示可能存在或可能不存在的类型值

在`javascript`代码中引入"可选"的值是非常常见的，这样你可以选择省略这个值或传递`null`。

在使用`Flow`的时候你可以为那些值使用“可能类型”（`Maybe Types`），可能类型在其他类型的前面加上问号`?`作为前缀就像`?number`这样的修饰符。

可能类型接受提供的类型以及`null`或`undefined`。所以`?number` 的意思是 `number`, `null`, `undefined`。

```javascript
// @flow
function acceptsMaybeNumber(value: ?number) {
  // ...
}

acceptsMaybeNumber(42);        // Works!
acceptsMaybeNumber();          // Works!
acceptsMaybeNumber(undefined); // Works!
acceptsMaybeNumber(null);      // Works!
acceptsMaybeNumber("42");      // Error!
```

函数参数不传值实质是传了`nudefined`。

在对象属性的情况下，缺失的属性跟显明定义为`undefined`是不同的。

```javascript
// @flow
function acceptsMaybeProp({ value }: { value: ?number }) {
  // ...
}

acceptsMaybeProp({ value: undefined }); // Works!
acceptsMaybeProp({});                   // Error!
```

（虽然在`Javascript`中读取对象不存在的属性得到的是`undefined`）

如果想允许缺失的属性，使用可选属性（`optional property`）语法，将问题`?`添加到冒号的前面（属性名的后面）。这两种语法可以结合使用成为一个可选的可能类型，例如：`{ value?: ?number }`

## 细化

假设我们有一个 `?number`类型,如果我们想作为 `number`使用这个值，我们首先需要先检查这个值不是`null`或`nudefined`。

```javascript
// @flow
function acceptsMaybeNumber(value: ?number) {
  if (value !== null && value !== undefined) {
    return value * 2;
  }
}
```

你可以使用单一的`!= null`来简化既不为`null` 也不为 `undefined`的两个检测。

```javascript
// @flow
function acceptsMaybeNumber(value: ?number) {
  if (value != null) {
    return value * 2;
  }
}
```

你也可以反过来说，使用之前确定这个值是一个`number`类型

```javascript
// @flow
function acceptsMaybeNumber(value: ?number) {
  if (typeof value === 'number') {
    return value * 2;
  }
```

然而，类型细化可能丢失。例如，对象属性类型细化后再调用了一个函数，这个细化将失效。查阅`细化失效`文档可以看到更多说明，理解`Flow`为什么用这种方式运行并且你如何避免这些常见的陷阱。

```javascript
// @flow
function otherMethod() { /* ... */ }

function method(value: { prop?: string }) {
  if (value.prop) {
    otherMethod();
    // $ExpectError
    value.prop.charAt(0);
  }
}
```

为什么会这样的原因是：我们不知道`otherMethod()`这个函数会不会对我们的值做什么。假设下面的情景：

```javascript
// @flow
var obj = { prop: "test" };

function otherMethod() {
  if (Math.random() > 0.5) {
    delete obj.prop;
  }
}

function method(value: { prop?: string }) {
  if (value.prop) {
    otherMethod();
    // $ExpectError
    value.prop.charAt(0);
  }
}

method(obj);
```