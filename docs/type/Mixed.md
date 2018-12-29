# 混合类型
（Mixed Types）

 > 使用`mixed`表示未知的类型（unknown types）

一般来说，程序都到几种不同类别的类型：

## 单个类型
（A single type）

这里输入值只能是一个`number`

```javascript
function square(n: number) {
  return n * n;
}
```

## 一组不同可能类型
（A group of different possible types）

这里输入值既可以是一个 `string` 也可以是一个 `number`。

```javascript
function stringifyBasicValue(value: string | number) {
  return '' + value;
}
```

## 基于另一类型的类型
（A type based on another type）

这里，返回类型跟传入函数参数值的类型相同。

```javascript
function identity<T>(value: T): T {
  return value;
}
```

## 任意类型
（An arbitrary type that could be anything）

这里，传递的值一个未知类型，它可以是任何类型，并且函数将仍然工作。

```javascript
function getTypeOf(value: mixed): string {
  return typeof value;
}
```

这些未知类型不常见，但有时却有用。

你可以使用`mixed`代表这些值（未知类型值）。


## 只进不出
（Anything goes in, Nothing comes out）

`mixed` 将接受任何类型的值。 `Strings`, `numbers`, `objects`, `functions`等，任何东西都可以。

```javascript
// @flow
function stringify(value: mixed) {
  // ...
}

stringify("foo");
stringify(3.14);
stringify(null);
stringify({});
```

当你试图使用一个`mixed`类型的值时，首先应该算出它的真实类型是什么，否则会导致错误。

```javascript
// @flow
function stringify(value: mixed) {
  // $ExpectError
  return "" + value; // Error!
}

stringify("foo");
```

相反，必须细化它来确保这个值是某一特定类型。

```javascript
//@fleo
function stringify(value:mixed){
	if(typeof value === 'tring'){
		return '' + value
	}else{
		return ''
	}
}
stringify("foo");
```

因为`typeof value === 'string'`检查，`Flow`知道在`if`语句的里面只可能是`string`。这就是所谓的细化（`refinement`）