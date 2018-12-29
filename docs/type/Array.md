# 数组类型

 > 把数组和其里面的元素类型化

 > 注：在`Javascript`中数组`Arrays`有时能作为元组使用，但在`Flow`里他们有不同的注解。查看元组相关的文档可以得到更多信息。

在`Javascript`中数组是一个列表形式的特殊对象，你可以用不同的方式创建数组。

```javascript
new Array(1, 2, 3); // [1, 2, 3];
new Array(3);       // [undefined, undefined, undefined]
[1, 2, 3];          // [1, 2, 3];
```

也可以创建一个数组，并在后续再给他们添加值：

```javascript
let arr = []; // []
arr[0] = 1;   // [1]
arr[1] = 2;   // [1, 2]
arr[2] = 3;   // [1, 2, 3]
```

## 数组类型

使用`Array<Type>`创建一个数组类型，这里的`Type`表示数组里面元素的类型。例如，你使用`Array<number>`来创建一个元素为数值的数组。

```javascript
let arr: Array<number> = [1, 2, 3];
```

你可以在`Array<Type>`的`Type`里面使用任何类型：

```javascript
let arr1: Array<boolean> = [true, false, true];
let arr2: Array<string> = ["A", "B", "C"];
let arr3: Array<mixed> = [1, true, "three"]
```

## 速记语法

这里也有一种短的语法形式：`Type[]`。

```javascript
//let arr: Array<number> = [1,2,3,]
let arr: number[] = [0, 1, 2, 3];
```

请注意`?Type[]`跟`?Array<T>`是相等的，跟`Array<?T>`不相等.

```javascript
// @flow
let arr0: ?number[] = undefined; // Works!
let arr1: ?number[] = null;   // Works!
let arr2: ?number[] = [1, 2]; // Works!
let arr3: ?number[] = [null]; // Error!
```

`?number[]` 表示可以接受元素为数字的数组或`null`、`undefined`。

如果使用`Array<?T>`，添加了一个括号来表示其的短语法：(?Type)[]。

```javascript
// @flow
let arr1: (?number)[] = null;   // Error!
let arr2: (?number)[] = [1, 2]; // Works!
let arr3: (?number)[] = [null]; // Works!
```

## 访问不安全

当你从一个数组中读取元素很有可能是`undefined`。你可以访问超出数组索引边界的元素，因为是稀疏数组`sparse array`这个元素可能并不存在。

例如：你可以访问超出数组边界的元素

```javascript
// @flow
let array: Array<number> = [0, 1, 2];
let value: number = array[3]; // Works.
                       // ^ undefined
```

或者访问一个稀疏数组中不存在的元素

```javascript
// @flow
let array: Array<number> = [];

array[0] = 0;
array[2] = 2;

let value: number = array[1]; // Works.
                       // ^ undefined
```

为了安全起见，`Flow`必须标记每个数组访问为可能未定义（`possibly undefined`）。

但是`Flow`并没有这样做，因为这样做对使用极不方便。所以你被迫细化访问数组元素得到的值的类型。

```javascript
let array: Array<number> = [0, 1, 2];
let value: number | void = array[1];

if (value !== undefined) {
  // number
}
```

当未来`Flow`变得更聪明后可以解决这个问题，但现在应该知道这点。

## $ReadOnlyArray<T>

`$ReadOnlyArray<T>`跟`$ReadOnly<T>`一样，它是所有数组和元组的超类，并且代表一个数组的只读视图，它不包含任何允许这种类型变异的方法（`push`、`pop`等）。

```javascript
// @flow
const readonlyArray: $ReadOnlyArray<number> = [1, 2, 3]

const first = readonlyArray[0] // OK to read
readonlyArray[1] = 20          // Error!
readonlyArray.push(4)          // Error!
```

注意，一个`$ReadOnlyArray<T>`类型的数组仍然可以有可变的元素

```
// @flow
const readonlyArray: $ReadOnlyArray<{x: number}> = [{x: 1}];
readonlyArray[0] = {x: 42}; // Error!
readonlyArray[0].x = 42; // OK
```

这里涉及到引用类型的知识，数组中第一个元素是一个对象，及保存的是这个对象的引用地址，所以改变在不改变其引用地址的情况下改变这个对象的属性是可行的。

使用`$ReadOnlyArray`代替`Array`的好处是`$ReadOnlyArray`类型的参数是协变的(`covariant `)，而`Array`类型的参数是不变的(`invariant`)。这意味着`$ReadOnlyArray<number>`是`$ReadOnlyArray<number | string>`的子类型，而`Array<number>`不是`Array<number | string>`的子类，所以`$ReadOnlyArray`用于各种数组元素的类型注释是非常有用的。

关于协变（covariance），逆变（contravariance）与不变（invariance）不明白的自行查询资料。

比如下面情景的例子：

```javascript
// @flow
const someOperation = (arr: Array<number | string>) => {
  // Here we could do `arr.push('a string')`
}

const array: Array<number> = [1]
someOperation(array) // Error!
```

因为`someOperation`函数里的参数`arr`数组是可变的（能添加、修改、删除数组里的值），`number | string`所以允许添加一个字符串元素。当外面`Array<number> = [1]`传入`someOperation`函数，`arr`添加一个字符串元素将打破`someOperation(array)`中`array`的类型结构（函数体内允许字符串而外面这个又不让用，冲突了）。这种情况通过`$ReadOnlyArray`注释参数`Flow`就不会发生这种情况并且没有错误发生。

```javascript
// @flow
const someOperation = (arr: $ReadOnlyArray<number | string>) => {
  // Nothing can be added to `arr`
}

const array: Array<number> = [1]
someOperation(array) // Works!
```

函数`someOperation`的参数`arr`是只读的，并且参数是`协变`的，上面参数只要是`<number | string>`的子类型都行，如：`Array<number>`、Array<string>。
