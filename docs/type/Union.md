# 联合类型

 > 值的类型可能是众多不同类型之一

有时，创建一个是其他类型集合之一的类型是非常有用的，例如，你可以编写一个函数，这个函数接受一组原始值类型，为此，`Flow`支持联合类型。

```javascript
// @flow
function toStringPrimitives(value: number | boolean | string) {
  return String(value);
}

toStringPrimitives(1);       // Works!
toStringPrimitives(true);    // Works!
toStringPrimitives('three'); // Works!

// $ExpectError
toStringPrimitives({ prop: 'val' }); // Error!
// $ExpectError
toStringPrimitives([1, 2, 3, 4, 5]); // Error!
```

## 语法

联合类型用一个竖线`|`把任意数量的类型连接起来。

```javascript
Type1 | Type2 | ... | TypeN
```

你也可以添加一个引导竖线，这在拆分联合类型为多行是非常有用。

```javascript
type Foo =
  | Type1
  | Type2
  | ...
  | TypeN
```

一个联合类型中的每个成员可以是任何类型，甚至是其他联合类型。

```javascript
type Numbers = 1 | 2;
type Colors = 'red' | 'blue'

type Fish = Numbers | Colors;
```

## 一个进，所有出

当调用接受联合类型的函数时，我们必须传递这些类型中某一类型，但是，在我们函数里面，我们必需处理所有这些可能的类型。

让我们重新编写函数，处理每个单独的类型。

```javascript
// @flow
// $ExpectError
function toStringPrimitives(value: number | boolean | string): string { // Error!
  if (typeof value === 'number') {
    return String(value);
  } else if (typeof value === 'boolean') {
    return String(value);
  }
}
```

你会注意到，如果我们不能处理每个值可能的类型，`Flow`将给我们一个错误。

## 联合与细化

当你有一个联合类型的值，将其拆分开，分别处理每个单独类型通常是很有用的。在`Flow`里使用联合类型，你可以把这个值细化到单一类型。

例如，如果我们有一个`number`、`boolean`、`string`联合类型的值，我们可以使用`Javascript`里的`typeof`操作符单独对待`number`类型的情况

```javascript
// @flow
function toStringPrimitives(value: number | boolean | string) {
  if (typeof value === 'number') {
    return value.toLocaleString([], { maximumSignificantDigits: 3 }); // Works!
  }
  // ...
}
```

通过检测值的类型和测试是否为一个数值，让`Flow`就明白块(`{}`)里那个值只允许是一个数值。

### 不相交联合

`Flow`联合类型里有一个特别的类型被称为不相交联合，它可以在细化里使用。这些不相交联合由任意数量的对象类型组成，每个对象类型都由一个单独属性所标记。

例如，假如我们有一个处理请求服务器后的响应函数，当请求成功后，我们将返回一个带`success`属性的对象，这个属性为`true`和一个我们己更新的值。

```javascript
{ success: true, value: false };
```

当请求失败，我们将返回一个对象，对象属性`success`为`false`和带有错误描述的`error`属性。

```javascript
{ success: false, error: 'Bad request' };
```

我们可以尝试用一个对象类型来表达这两个对象，然而，很快我们就遇到这样的问题，我们知道基于`success`属性存在的是那些属性，但`Flow`却不会知道。

```javascript
// @flow
type Response = {
  success: boolean,
  value?: boolean,
  error?: string
};

function handleResponse(response: Response) {
  if (response.success) {
    // $ExpectError
    var value: boolean = response.value; // Error!
  } else {
    // $ExpectError
    var error: string = response.error; // Error!
  }
}
```

试图把这两个单独的类型整合到一个里只会给我们带来麻烦。

取于代之，如果你创建一个这两对象类型的联合类型，`Flow`将能够明白基于`success`属性应该使用那个对象。

```javascript
// @flow
type Success = { success: true, value: boolean };
type Failed  = { success: false, error: string };

type Response = Success | Failed;

function handleResponse(response: Response) {
  if (response.success) {
    var value: boolean = response.value; // Works!
  } else {
    var error: string = response.error; // Works!
  }
}
```

### 精确类型的不相交联合

不相交联合需要你使用一个单独的属性去区分每个对象类型，不能通过不同的属性区分两个不同的对象。

```javascript
// @flow
type Success = { success: true, value: boolean };
type Failed  = { error: true, message: string };

function handleResponse(response:  Success | Failed) {
  if (response.success) {
    // $ExpectError
    var value: boolean = response.value; // Error!
  }
}
```

这是因为在`Flow`里传递一个属性比期望的属性多的对象是可以的(因为宽度子类型化)。

 - 宽度子类型化（`width subtyping`）：给记录增加更多的域。
 - 深度子类型化（`depth subtyping`）：把超类型（`supertype`）的域替换为域的子类型。这仅能用于只读（`immutable`）记录。

```javascript
// @flow
type Success = { success: true, value: boolean };
type Failed  = { error: true, message: string };

function handleResponse(response:  Success | Failed) {
  // ...
}

handleResponse({
  success: true,
  error: true,
  value: true,
  message: 'hi'
});
```

除非这些对象在某种程度上相互冲突，否则就没有办法区分它们。

但是，要解决这个问题，您可以使用精确的对象类型。

```javascript
// @flow
type Success = {| success: true, value: boolean |};
type Failed  = {| error: true, message: string |};

type Response = Success | Failed;

function handleResponse(response: Response) {
  if (response.success) {
    var value: boolean = response.value;
  } else {
    var message: string = response.message;
  }
}
```

有了精确的对象类型，我们就不可能有额外的属性，所以对象之间会发生冲突，我们就能区分出哪个是哪个。