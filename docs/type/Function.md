# 函数类型
Function Types

 > 把函数声明和其值类型化

函数有两个地方可以应用类型：参数（输入）和返回值（输出）。

```javascript
// @flow
function concat(a: string, b: string): string {
  return a + b;
}

concat("foo", "bar"); // Works!
// $ExpectError
concat(true, false);  // Error!
```

利用`Flow`推断，这些类型通常是可选的

```javascript
// @flow
function concat(a, b) {
  return a + b;
}

concat("foo", "bar"); // Works!
// $ExpectError
concat(true, false);  // Error!
```

有时`Flow`的推断出的类型比我们预想的要宽松得多。

```javascript
// @flow
function concat(a, b) {
  return a + b;
}

concat("foo", "bar"); // Works!
concat(1, 2);         // Works!
```

因为这原因（或其它），为重要的函数写上类型是很有用的。

## 语法

有三种形式的函数，每种都有各自细微不同的语法。

### 函数声明

这里你可以看到函数声明时添加或不添加类型的语法。

```javascript
function method(str, bool, ...nums) {
  // ...
}

function method(str: string, bool?: boolean, ...nums: Array<number>): void {
  // ...
}
```

### 箭头函数

这里你可以看到箭头函数添加或不添加类型的语法。

```javascript
let method = (str, bool, ...nums) => {
  // ...
};

let method = (str: string, bool?: boolean, ...nums: Array<number>): void => {
  // ...
};
```

### 函数作为类型

这里你可以看到编写函数类型的语法

```javascript
(str: string, bool?: boolean, ...nums: Array<number>) => void
```

也可以选择省略参数名

```javascript
(string, boolean | void, Array<number>) => void
```

也可以使用这些函数类型做一些事，例如回调函数（callback）。

```javascript
function method(callback: (error: Error | null, value: string | null) => void) {
  // ...
}
```

## 函数参数

函数参数可以有类型，通过在参数名的后面添加一个冒号`:`，在这冒号的后面紧接类型注释。

```javascript
function method(param1: string, param2: boolean) {
  // ...
}
```

## 可选参数

你也可以有可选参数，在参数名后面、冒号`:`的前面，添加一个问号`?`。

```javascript
function method(optionalValue?: string) {
  // ...
}
```

可选参数接受省略这个参数、`undefined`或者配置的类型，但是不接受`null`

```javascript
// @flow
function method(optionalValue?: string) {
  // ...
}

method();          // Works.
method(undefined); // Works.
method("string");  // Works.
// $ExpectError
method(null);      // Error!
```

可选参数（`Optional Parameters`）与可能类型（`Maybe Types`）的区别：

 - 可选的参数`Value?: string`接受缺失、`nudefined`、匹配的类型，不接受`null`
 - 可能的类型`Value: ?string`接受缺失、`null` 、`undefined`、匹配的类型


### 剩余参数

`JavaScript`也支持剩余参数，在参数的前面加上`...`来收集`arguments`参数列表末尾参数的数组（`...args`）。

你也可以使用相同的语法为剩余参数添加类型注解，但要使用数组(`Array`)。

```javascript
function method(...args: Array<number>) {
  // ...
}
```

你可以传递尽可能的参数到剩余参数。

```javascript
// @flow
function method(...args: Array<number>) {
  // ...
}

method();        // Works.
method(1);       // Works.
method(1, 2);    // Works.
method(1, 2, 3); // Works.
```

 > 注意：如果你为剩余参数添加了类型注解，其类型注解必须始终明确的是一个`Array`类型。

### 函数返回值

在函数参数列表后面使用冒号和一个类型（`:number`）也可以给函数返回值添加一个类型。

```javascript
function method(): number {
  // ...
}
```

你函数每个分支的返回值类型要确定跟函数返回值类型一致。这可以预防你在某些意外情况件下没有返回值

```javascript
// @flow
// $ExpectError
function method(): boolean {
  if (Math.random() > 0.5) {
    return true;
  }
}
```

上面代码因为在条件不满足是返回了`undefined`

### 函数里的this

在`Javascript`中，每个函数里都可以访问一个叫`this`的特殊上下文，你可以用任何你想用的上下文访问一个函数。

在`Flow`里你不能为`this`添加类型注解，并且`Flow`将检测你用来访问函数的所有上下文。

```javascript
function method() {
  return this;
}

var num: number = method.call(42);
// $ExpectError
var str: string = method.call(42);
```

### 谓词函数

 > `谓词函数` 是一个返回布尔值的函数

有时，你想将`if`语句的条件放入函数里。

```javascript
function concat(a: ?string, b: ?string): string {
  if (a && b) {
    return a + b;
  }
  return '';
}
```

但是，`Flow`将会在下面的代码标记为错误。

```javascript
function truthy(a, b): boolean {
  return a && b;
}

function concat(a: ?string, b: ?string): string {
  if (truthy(a, b)) {
    // $ExpectError
    return a + b;
  }
  return '';
}
```

你可以使用`%checks`注解把`truthy`函数变为一个谓词函数来修复这个问题。像这样：

```javascript
function truthy(a, b): boolean %checks {
  return !!a && !!b;
}

function concat(a: ?string, b: ?string): string {
  if (truthy(a, b)) {
    return a + b;
  }
  return '';
}
```

谓词函数( `Predicate Functions` )的主体需要是表达式(即不支持声明局部变量)。但是在一个谓词函数内部调用另一个谓词函数是可以的。例如：

```javascript
function isString(y): %checks {
  return typeof y === "string";
}

function isNumber(y): %checks {
  return typeof y === "number";
}

function isNumberOrString(y): %checks {
  return isString(y) || isNumber(y);
}

function foo(x): string | number {
  if (isNumberOrString(x)) {
    return x + x;
  } else {
    return x.length; // no error, because Flow infers that x can only be an array
  }
}

foo('a');
foo(5);
foo([]);
```

### 可调用对象

可调用对象可以用来类型化，例如:

```javascript
type CallableObj = {
  (number, number): number,
  bar: string
};

function add(x, y) {
  return x + y;
}

// $ExpectError
(add: CallableObj);

add.bar = "hello world";

(add: CallableObj);
```

### 函数类型

有时候写入一个授受任意函数的类型是很有用的，那样的话你应该写入`() => mixed`

```javascript
function(func:() => mixed){
	//.....
}
```

表示，参数`func`是一个任意的函数。

然而，如果你需要跳出类型检测并且又不想用`any`这种方法，你可以使用`Function`来代替。`Function`是不安全的，应该尽量避免使用。

例如下面的代码不会报任何错误：

```javascript
function method(func: Function) {
  func(1, 2);     // Works.
  func("1", "2"); // Works.
  func({}, []);   // Works.
}

method(function(a: number, b: number) {
  // ...
});
```

在使用`Function`时你应该遵循跟`any`一样的规则。