# 类型别名

 > 便于重用的别名类型

当你想在多个地方重复使用一个复杂类型的时候，你可以在`Flow`里使用类型别名(`type alias`)给他们别名。

```javascript
// @flow
type MyObject = {
  foo: number,
  bar: boolean,
  baz: string,
};
```

这些类型别名可以在任何能使用类型的地方使用。

```javascript
// @flow
type MyObject = {
  // ...
};

var val: MyObject = { /* ... */ };
function method(val: MyObject) { /* ... */ }
class Foo { constructor(val: MyObject) { /* ... */ } }
```

## 语法

创建类型别名使用关键字`type`,然后在其后面跟上他的名字和一个等于符号`=`，接着一个类型字义。

```javascript
type Alias = Type;
```

任何类型都可以出现在类型别名中。

```javascript
type NumberAlias = number;
type ObjectAlias = {
  property: string,
  method(): number,
};
type UnionAlias = 1 | 2 | 3;
type AliasAlias = ObjectAlias;
```

### 类型别名泛型

类型别名也可以有他们自己的泛型

```javascript
type MyObject<A, B, C> = {
  property: A,
  method(val: B): C,
};
```

类型别名的泛型是参数化的，可你使用一个类型别名需要为他们每个泛型传递参数

```javascript
// @flow
type MyObject<A, B, C> = {
  foo: A,
  bar: B,
  baz: C,
};

var val: MyObject<number, boolean, string> = {
  foo: 1,
  bar: true,
  baz: 'three',
};
```
