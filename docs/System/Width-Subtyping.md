# 宽度子类型

在一个注解为具体属性集对象的地方使用具有“额外”属性的对象，是安全的。

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

在`method`里面，我们知道`obj`至少有一个`foo`属性，并且属性存取表达式`obj.foo`将得到`string`。

这是一种通常称为“宽度子类型”的子类型，因为“更宽”（即，具有更多属性）的类型是窄类型的子类型。

因此，下面例子中，`obj2`是`obj1`的子类型。

```javascript
let obj1 = { foo: 'test' };
let obj2 = { foo: 'test', bar: 42 };
```

然而，知道某个属性确实不存在通常很有用。

```javascript
// @flow
function method(obj: { foo: string } | { bar: number }) {
  if (obj.foo) {
    (obj.foo: string); // Error!
  }
}
```

上面的代码会出现错误，`Flow`也允许调用表达式`method({ foo: 1, bar: 2 })`,因为`{ foo: number, bar: number }`是`{ bar: number }`的子类型，是参数的联合类型之一。

像这种情况，断言不存在属性非常有用。Flow为[精确对象类型](#docs/type/Object#精确对象类型)提供了特殊的语法。

```javascript
// @flow
function method(obj: {| foo: string |} | {| bar: number |}) {
  if (obj.foo) {
    (obj.foo: string); // Works!
  }
}
```

[精确对象类型](#docs/type/Object#精确对象类型)禁用宽度子类型，不允许额外的属性存在。

使用[精确对象类型](#docs/type/Object#精确对象类型)让`Flow`知道在运行时没有额外的属性存在，这使[细化](#docs/System/Type-Refinements)更具体。

```javascript
// @flow
function method(obj: {| foo: string |} | {| bar: number |}) {
  if (obj.foo) {
    (obj.foo: string); // Works!
  }
}
```