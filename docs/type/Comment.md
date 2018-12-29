# 注释类型

 > `Flow`靠一种特殊的注释语法来使用纯`Javascript`代码。

`Flow`支持一种基本注释语法，它能在不编译代码的情况下就可以使用`Flow`。

```javascript
// @flow

/*::
type MyAlias = {
  foo: number,
  bar: boolean,
  baz: string,
};
*/

function method(value /*: MyAlias */) /*: boolean */ {
  return value.bar;
}

method({ foo: 1, bar: true, baz: ["oops"] });
```

在没有任何额外处理的情况下，这些注释允许`Flow`在纯`Javascript`文件中工作。

## 语法

这里有两个注释语法主要的部分：类型包含和类型注解

### 注释包含

如果你希望`Flow`将注释视为正常的语法，你可以在注释前面添加两个冒号

```javascript
/*::
type Foo = {
  foo: number,
  bar: boolean,
  baz: string
};
*/

class MyClass {
  /*:: prop: string; */
}
```

这个语法中包含的代码`Flow`当作是：

```javascript
type Foo = {
  foo: number,
  bar: boolean,
  baz: string
};

class MyClass {
  prop: string;
}
```

但是`JavaScript`忽视这些注释的内容，所以它拥有的只是有效的语法。

```javascript
class MyClass {

}
```

在`flow-include`形式里，这种语法也是有效的。

```javascript
/*flow-include
type Foo = {
  foo: number,
  bar: boolean,
  baz: string
};
*/

class MyClass {
  /*flow-include prop: string; */
}
```

## 注释注解

而不是每次都输入一个完整的`包含`,你可以在注释的前面使用单个冒号`:`来标注类型注解

```javascript
function method(param /*: string */) /*: number */ {
  // ...
}
```

在注释包含内部使用注释注解是一样的。

```javascript
function method(param /*:: : string */) /*:: : number */ {
  // ...
}
```

 > 注意：如果你想使用可选的函数参数，你得使用注释包含的形式。
