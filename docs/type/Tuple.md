# 元组类型
>把类元组数组值类型化

元组是一个列表类型，并且项目集是有限的，在`Javascript`中元组使用数组来创建。

在`Flow`中你可以使用`[type, type, type]`的语法来创建元组

```javascript
let tuple1: [number] = [1];
let tuple2: [number, boolean] = [1, true];
let tuple3: [number, boolean, string] = [1, true, "three"];
```

当你从元组中一个特定的索引处读取值，它将返回那个索引处的类型

```javascript
// @flow
let tuple: [number, boolean, string] = [1, true, "three"];

let num  : number  = tuple[0]; // Works!
let bool : boolean = tuple[1]; // Works!
let str  : string  = tuple[2]; // Works!
```

如果你试图从元组不存在的索引处读取值，它将返回一个`void`类型

```javascript
// @flow
let tuple: [number, boolean, string] = [1, true, "three"];

let none: void = tuple[3];
``

如果`Flow`不清楚你试图访问哪个索引，它将返回所有可能的类型。

```javascript
// @flow
let tuple: [number, boolean, string] = [1, true, "three"];

function getItem(n: number) {
  let val: number | boolean | string = tuple[n];
  // ...
}
```

当在元组里设置新值时，这个新值必须匹配这个索引处的类型


```javascript
// @flow
let tuple: [number, boolean, string] = [1, true, "three"];

tuple[0] = 2;     // Works!
tuple[1] = false; // Works!
tuple[2] = "foo"; // Works!

// $ExpectError
tuple[0] = "bar"; // Error!
// $ExpectError
tuple[1] = 42;    // Error!
// $ExpectError
tuple[2] = false; // Error!
```

## 严格限制长度

元组的长度又叫元数,在`Flow`中严格执行元组的长度。

### 匹配相同长度

只匹配相同长度的元组，意思是说，一个较短的元组不能用于替代较长的元组

```javascript
// @flow
let tuple1: [number, boolean] = [1, true];
// $ExpectError
let tuple2: [number, boolean, void] = tuple1; // Error!
```

当然，一个较长的元组也不能替代较短的元组

```javascript
// @flow
let tuple1: [number, boolean, void] = [1, true];
// $ExpectError
let tuple2: [number, boolean]       = tuple1; // Error!
```

### 不匹配数组类型

由于`Flow`不清楚数组的长度，因此无法将`Array<T>`类型传递给元组。


```javascript
// @flow
let array: Array<number>    = [1, 2];
// $ExpectError
let tuple: [number, number] = array; // Error!
```

当然，一个元组类型也不能传递给一个数组类型`Array<T>`，因为后面你可以用不安全的方式使元组变异。

```javascript
// @flow
let tuple: [number, number] = [1, 2];
// $ExpectError
let array: Array<number>    = tuple; // Error!
```

### 不能使用变异数组方法

你不能使用`Array.prototype`中那些变异元组的方法，只能使用那些不变异的。

```javascript
// @flow
let tuple: [number, number] = [1, 2];
tuple.join(', '); // Works!
// $ExpectError
tuple.push(3);    // Error!
```