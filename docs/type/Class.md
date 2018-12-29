# Class 类型

> 类类型和使用类作为类型

`JavaScript`中的`classes`在`Flow`中既可作为一个值也可作为一个类型来操作.

像没有用`flow`的方法一样来编写类，并且后面你可以使用这个类的名字作为一个类型。

```javascript
class MyClass {
  // ...
}

let myInstance: MyClass = new MyClass();
```

这是因为类在`Flow`里是名义类型`nominally typed`

## 语法

`Flow`中的类跟普通`Javascript`类完全相同，只是可以用来作为类型

### 类方法

跟函数一样，类方法中参数（输入）和返回值（输出）可以有类型注解。

```javascript
class MyClass {
  method(value: string): number { /* ... */ }
}
```

### 类字段（属性）

当你在`Flow`里想使用类的字段（属性）的时候，你必须首先给他一个注解

```javascript
// @flow
class MyClass {
  method() {
    // $ExpectError
    this.prop = 42; // Error!
  }
}
```

在类的内部，字段注解是在字段名的后面添加一个冒号`:`和其类型。

```javascript
// @flow
class MyClass {
  prop: number;
  method() {
    this.prop = 42;
  }
}
```

在类定义外面添加的字段应该在类的内部类型注解。

```javascript
// @flow
function func_we_use_everywhere (x: number): number {
  return x + 1;
}
class MyClass {
  static constant: number;
  static helper: (number) => number;
  method: number => number;
}
MyClass.helper = func_we_use_everywhere
MyClass.constant = 42
MyClass.prototype.method = func_we_use_everywhere
```

`method: number => number`表示参数要一个数值，返回值也是一个数值

`Flow`也支持使用类属性语法

```javascript
class MyClass {
  prop = 42;
}
```

当使用这种语法，你不必给他一类型注解，但是如果你要给也是可以的，例如：

```javascript
class MyClass {
  prop: number = 42;
}
```

### 类泛型

`classes` 也可以有自己的泛型

```javascript
class MyClass<A, B, C> {
  property: A;
  method(val: B): C {
    // ...
  }
}
```

类泛型是参数化的。当您使用类作为类型时，您需要为每个泛型传递参数。

```javascript
// @flow
class MyClass<A, B, C> {
  constructor(arg1: A, arg2: B, arg3: C) {
    // ...
  }
}

var val: MyClass<number, boolean, string> = new MyClass(1, true, 'three');
```