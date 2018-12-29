# 工具类型

`Flow`提供了一组去操作其他类型的实用程序类型，对于不同的场景非常有用。

|$Keys<T>|$Values<T>|$ReadOnly<T>|
|:---|:---|:---|
|$Exact<T>|$Diff<A, B>|$Rest<A, B>|
|$PropertyType<T, k>|$ElementType<T, K>|$NonMaybeType<T>|
|$ObjMap<T, F>|$TupleMap<T, F>|$Call<F>|
|Class<T>|$Shape<T>|$Supertype<T>|
|$Subtype<T>|Existential Type (*)||

## $Keys<T>

在`Flow`里，你可以像枚举一样使用联合类型。

```javascript
// @flow
type Suit = "Diamonds" | "Clubs" | "Hearts" | "Spades";

const clubs: Suit = 'Clubs';
const wrong: Suit = 'wrong'; // 'wrong' is not a Suit
```

这非常方便，但有时你需要在运行时访问枚举定义（在值的层面上）。

例如，假设你想将一个值与前面示例`Suit`的每个成员都关联起来，你可以这样做：

```javascript
// @flow
type Suit = "Diamonds" | "Clubs" | "Hearts" | "Spades";

const suitNumbers = {
  Diamonds: 1,
  Clubs: 2,
  Hearts: 3,
  Spades: 4
};

function printSuitNumber(suit: Suit) {
  console.log(suitNumbers[suit]);
}

printSuitNumber('Diamonds'); // 2
printSuitNumber('foo'); // 'foo' is not a Suit
```

但是，这样并不觉得有多好，因为我们不得不明确定义`suit`列举两次。

在这种情况下，你可以使用`$Keys<T>`操作。让我们看看另一个例子，这次使用`$Keys`:

```javascript
// @flow
const countries = {
  US: "United States",
  IT: "Italy",
  FR: "France"
};

type Country = $Keys<typeof countries>;

const italy: Country = 'IT';
const nope: Country = 'nope'; // 'nope' is not a Country
```

在上面的例子中，`Country`的类型跟`type Country = 'US' | 'IT' | 'FR'`是相同的，但是`Flow`能够通过`countries`的`keys`提取出来。


## $Values<T>

`$Values<T>`代表对象类型（`Object Type T`）里所有枚举属性的类型值的联合类型

例如：

```javascript
// @flow
type Props = {
  name: string,
  age: number,
};

// The following two types are equivalent:
type PropValues = string | number;
type Prop$Values = $Values<Props>;

const name: Prop$Values = 'Jon';  // OK
const age: Prop$Values = 42;  // OK
const fn: Prop$Values = () => {};  // Error! function is not part of the union type
```

## $ReadOnly<T>

`$ReadOnly<T>`是一个类型，代表给定对象类型（`object type`）`T`的只读版本，一个只读对象类型是一个其`keys`都是只读的对象类型。

这意味着下面两个类型是相等的。

```javascript
type ReadOnlyObj = {
  +key: any,  // read-only field, marked by the `+` annotation
};
```

```javascript
type ReadOnlyObj = $ReadOnly<{
  key: any,
}>;
```

这是很有用的，当你需要使用一个己定义对象类型的只读版本，这样，不需要手动去重复定义和把它的每个`key`都注解为只读。例如：

```javascript
// @flow
type Props = {
  name: string,
  age: number,
  // ...
};

type ReadOnlyProps = $ReadOnly<Props>;

function render(props: ReadOnlyProps) {
  const {name, age} = props;  // OK to read
  props.age = 42;             // Error when writing
  // ...
}
```

除此之外，其它工具类型例如`$ObjMap<T>`，可以破坏任何读/写的注解，因此`$ReadOnly<T>`是一个便利的方法，它能在对象被操作之后快速地重新变为只读。

```javascript
type Obj = {
  +key: any,
};

type MappedObj = $ReadOnly<$ObjMap<Obj, TypeFn>> // Still read-only
```

 > 注意：`$ReadOnly`仅用来使对象类型变为只读。怎么使数组变为只读，可以查看文档`$ReadOnlyArray`。

## $Exact<T>

`$Exact<{name: string}>`是对象文件中`{| name: string |}`的代名词，返回精准的对象类型

```javascript
// @flow
type ExactUser = $Exact<{name: string}>;
type ExactUserShorthand = {| name: string |};

const user2 = {name: 'John Wilkes Booth'};
// These will both be satisfied because they are equivalent
(user2: ExactUser);
(user2: ExactUserShorthand);
```

## $Diff<A, B>

顾名思义，`$Diff<A, B>`是一个类型，其代表`A`和`B`两类型的差异集合，即`A \ B`,这里的`A`和`B`都是对象类型。这里有个例子：

```javascript
// @flow
type Props = { name: string, age: number };
type DefaultProps = { age: number };
type RequiredProps = $Diff<Props, DefaultProps>;

function setProps(props: RequiredProps) {
  // ...
}

setProps({ name: 'foo' });
setProps({ name: 'foo', age: 42, baz: false }); // you can pass extra props too
setProps({ age: 42 }); // error, name is required
```

你可能己经注意事，这不是一个随意的例子。在`React`定义文件里使用`$Diff`定义组件所接受的属性类型是精确的。

注意，从一个没有此属性的对象里删除该属性，`$Diff<A, B>`会报错，即，如果`B`有一个`A`中不存在的`key`。

 > 个人理解:把`A`中所有跟`B`相同`key`的属性删除，剩下的就是结果，如果`B`中有而`A`中没有就报错。

```javascript
// @flow
type Props = { name: string, age: number };
type DefaultProps = { age: number, other: string }; // Will error due to this `other` property not being in Props.
type RequiredProps = $Diff<Props, DefaultProps>;

function setProps(props: RequiredProps) {
  props.name;
  // ...
}
```

有一种解决方案，你可以在`B`中把现在`A`里不存在的属性指定为可选的。例如：

```javascript
type A = $Diff<{}, {nope: number}>; // Error
type B = $Diff<{}, {nope: number | void}>; // OK
```

## $Rest<A, B>

`$Rest<A, B>`代表运行时对象的剩余操作数（ES6里的对象的扩展运算符）  即：const {foo, ...rest} = obj。`A`和`B`都是对象类型，这个操作返回一个对象类型，包含`A`拥有但`B`中没有的属性。在`Flow`里，我们把所有精准对象类型上的属性作为是其自己拥有的。在不精准对象里，一个属性可能是也可能不是自己的属性。

```javascript
// @flow
type Props = { name: string, age: number };

const props: Props = {name: 'Jon', age: 42};
const {age, ...otherProps} = props;
(otherProps: $Rest<Props, {|age: number|}>);
otherProps.age;  // Error
```

跟`$Diff<A, B>`主要不同点是`$Rest<A, B>`目的在于代表真正运行时的操作，这意味着在`$Rest<A, B>`里精准的对象类型被区别对待，例如，`$Rest<{|n: number|}, {}>`导致结果为`{|n?: number|}`,因为为个精准的空对象里可以有`n`属性，而`$Diff<{|n: number|}, {}>`导致结果为`{|n: number|}`


## $PropertyType<T, k>

一个`$PropertyType<T, k>`是`T`中给定关键字`k`的类型，在`Flow`v0.36.0版本里，`k`必须是一个字面量字符串。

```javascript
// @flow
type Person = {
  name: string,
  age: number,
  parent: Person
};

const newName: $PropertyType<Person, 'name'> = 'Michael Jackson';
const newAge: $PropertyType<Person, 'age'> = 50;
const newParent: $PropertyType<Person, 'parent'> = 'Joe Jackson';
```

涉及`React`的属性（`props`）的类型甚至它的全部`props`的类型都是特别有用。

```javascript
// @flow
import React from 'react';
class Tooltip extends React.Component {
  props: {
    text: string,
    onMouseOver: ({x: number, y: number}) => void
  };
}

const someProps: $PropertyType<Tooltip, 'props'> = {
  text: 'foo',
  onMouseOver: (data: {x: number, y: number}) => undefined
};

const otherProps: $PropertyType<Tooltip, 'props'> = {
  text: 'foo'
  // Missing the `onMouseOver` definition
};
```

你甚至可以嵌套查询：

```javascript
// @flow
type PositionHandler = $PropertyType<$PropertyType<Tooltip, 'props'>, 'onMouseOver'>;
const handler: PositionHandler = (data: {x: number, y: number}) => undefined;
const handler2: PositionHandler = (data: string) => undefined; // wrong parameter types
```

结合`Class<T>`你能获取静态`props`:

```javascript
// @flow
class BackboneModel {
  static idAttribute: string | false;
}

type ID = $PropertyType<Class<BackboneModel>, 'idAttribute'>;
const someID: ID = '1234';
const someBadID: ID = true;
```

## $ElementType<T, K>

`$ElementType<T, K>`代表一个数组、元组或对象类型`T`里每个元素的类型，这个类型匹配给定关键字`K`的类型。

例如：

```javascript
// @flow

// Using objects:
type Obj = {
  name: string,
  age: number,
}
('Jon': $ElementType<Obj, 'name'>);
(42: $ElementType<Obj, 'age'>);
(true: $ElementType<Obj, 'name'>); // Nope, `name` is not a boolean
(true: $ElementType<Obj, 'other'>); // Nope, property `other` is not in Obj

// Using tuples:
type Tuple = [boolean, string];
(true: $ElementType<Tuple, 0>);
('foo': $ElementType<Tuple, 1>);
('bar': $ElementType<Tuple, 2>); // Nope, can't access position 2
```

在上面的情况里，我们可以用字面量值作为`K`,跟`$PropertyType<T, k>`类似。然而，当使用`$ElementType<T, K>`时，`K`可以是任何类型，只要那个类型在`T`中关键字存在。例如：

```javascript
// @flow

// Using objects
type Obj = { [key: string]: number };
(42: $ElementType<Obj, string>);
(42: $ElementType<Obj, boolean>); // Nope, object keys aren't booleans
(true: $ElementType<Obj, string>); // Nope, elements are numbers


// Using arrays, we don't statically know the size of the array, so you can just use the `number` type as the key:
type Arr = Array<boolean>;
(true: $ElementType<Arr, number>);
(true: $ElementType<Arr, boolean>); // Nope, array indices aren't booleans
('foo': $ElementType<Arr, number>); // Nope, elements are booleans
```

你也可以嵌套调用`$ElementType<T, K>`,当你需要在嵌套结构里访问类型这是非常有用的。

```javascript
// @flow
type NumberObj = {
  nums: Array<number>,
};

(42: $ElementType<$ElementType<NumberObj, 'nums'>, number>);
```

此外，你可以使用泛型，这使得`$ElementType<T, K>`比`$PropertyType<T, k>`更强大

```javascript
// @flow
function getProp<O: {+[string]: mixed}, P: $Keys<O>>(o: O, p: P): $ElementType<O, P> {
  return o[p];
}

(getProp({a: 42}, 'a'): number); // OK
(getProp({a: 42}, 'a'): string); // Error: number is not a string
getProp({a: 42}, 'b'); // Error: `b` does not exist
```

## $NonMaybeType<T>

`NonMaybeType<T>`把一个类型`T`转换为非可能类型（`non-maybe`），换句话说，`$NonMaybeType<T>`的值就是`T`除了`null`和`undefined`的值。

```javascript
// @flow
type MaybeName = ?string;
type Name = $NonMaybeType<MaybeName>;

('Gabriel': MaybeName); // Ok
(null: MaybeName); // Ok
('Gabriel': Name); // Ok
(null: Name); // Error! null can't be annotated as Name because Name is not a maybe type
```

## $ObjMap<T, F>

`ObjMap<T, F>`以一个对象类型`T`和一个函数类型`F`来返回对象类型，返回`T`中每个值通过所提供的函数类型`F`进行映射而获得的对象类型。换名话说，`$ObjMap`将为每个`T`中的属性值的类型调用(在类型层面上)所给的函数类型`F`，并返回他们调用后的对象类型结果。

让我们看看一个例子，假如你有一个函数叫`run`，其接受一个形实转换对象（函数形式 `() => A`）作为输入：

```javascript
// @flow
function run<O: {[key: string]: Function}>(o: O) {
  return Object.keys(o).reduce((acc, k) => Object.assign(acc, { [k]: o[k]() }), {});
}
```

这个函数的目的是运行所有形实转换对象交返回一个对象值，这个函数返回的类型是什么？

键是一样的,但值有不同的类型,即每个函数的返回类型。在值的层面上（函数的实现）我们本质上是映射一个对象的键生成新的值。如何在类型层次上表达这一点？

这就是`ObjMap<T, F>`的方便之处。

```javascript
// @flow

// let's write a function type that takes a `() => V` and returns a `V` (its return type)
type ExtractReturnType = <V>(() => V) => V

function run<O: {[key: string]: Function}>(o: O): $ObjMap<O, ExtractReturnType> {
  return Object.keys(o).reduce((acc, k) => Object.assign(acc, { [k]: o[k]() }), {});
}

const o = {
  a: () => true,
  b: () => 'foo'
};

(run(o).a: boolean); // Ok
(run(o).b: string);  // Ok
// $ExpectError
(run(o).b: boolean); // Nope, b is a string
// $ExpectError
run(o).c;            // Nope, c was not in the original object
```

表达函数操作对象时的返回类型这是极其有用的，你可以使用类似的方法提供`Promise.props`函数的返回类型，其像`Promise.all`,但是有一个对象作为输入。

这里是这个函数可能的声明，它跟我们第一个例子非常类似：

```javascript
// @flow
declare function props<A, O: { [key: string]: A }>(promises: O): Promise<$ObjMap<O, typeof $await>>;
```

也可以使用：

```javascript
// @flow
const promises = { a: Promise.resolve(42) };
props(promises).then(o => {
  (o.a: 42); // Ok
  // $ExpectError
  (o.a: 43); // Error, flow knows it's 42
});
```

## $TupleMap<T, F>

`$TupleMap<T, F>`有接受一个可迭代的类型`T`(如：元组或数组)和一个函数类型`F`，并返回`T`每个值的类型通过所提供函数`F`映射出的可迭代类型。它跟`Javascript`中函数`map`相似。

Following our example from `$ObjMap<T>`, let’s assume that run takes an array of functions, instead of an object, and maps over them returning an array of the function call results. We could annotate its return type like this:

```javascript
// @flow

// Function type that takes a `() => V` and returns a `V` (its return type)
type ExtractReturnType = <V>(() => V) => V

function run<A, I: Array<() => A>>(iter: I): $TupleMap<I, ExtractReturnType> {
  return iter.map(fn => fn());
}

const arr = [() => 'foo', () => 'bar'];
(run(arr)[0]: string); // OK
(run(arr)[1]: string); // OK
(run(arr)[1]: boolean); // Error
```

##　$Call<F>

`$Call<F>`代表调用给定函数类型`F`的结果类型。这跟在运行时调用一个函数相似（它类似于调用`Function.prototype.call`）。但是，在类型层面，意思是函数类型调用是静态发生的，即不是在运行时。

让我们看看一组例子：

```javascript
// @flow

// Takes an object type, returns the type of its `prop` key
type ExtractPropType = <T>({prop: T}) => T;
type Obj = {prop: number};
type PropType = $Call<ExtractPropType, Obj>;  // Call `ExtractPropType` with `Obj` as an argument
type Nope = $Call<ExtractPropType, {nope: number}>;  // Error: argument doesn't match `Obj`.

(5: PropType); // OK
(true: PropType);  // Error: PropType is a number
(5: Nope);  // Error
```

```javascript
// @flow

// Takes a function type, and returns its return type
// This is useful if you want to get the return type of some function without actually calling it at runtime.
type ExtractReturnType = <R>(() => R) => R;
type Fn = () => number;
type ReturnType = $Call<ExtractReturnType, Fn> // Call `ExtractReturnType` with `Fn` as an argument

(5: ReturnType);  // OK
(true: ReturnType);  // Error: ReturnType is a number
```

`$Call`非常强大，因为它允许你在类型着陆时调用，否则你只能是运行时调用。类型着陆调用是静态发生的，并在运行时被清除。


让我们来看看更高级一点的例子:

```javascript
// @flow

// Extracting deeply nested types:
type NestedObj = {|
  +status: ?number,
  +data: ?$ReadOnlyArray<{|
    +foo: ?{|
       +bar: number,
    |},
  |}>,
|};

// If you wanted to extract the type for `bar`, you could use $Call:
type BarType = $Call<
  <T>({
    +data: ?$ReadOnlyArray<{
      +foo: ?{
        +bar: ?T
      },
    }>,
  }) => T,
  NestedObj,
>;

(5: BarType);
(true: BarType);  // Error: `bar` is not a boolean
```

```javascript
// @flow

// Getting return types:
function getFirstValue<V>(map: Map<string, V>): ?V {
  for (const [key, value] of map.entries()) {
    return value;
  }
  return null;
}

// Using $Call, we can get the actual return type of the function above, without calling it at runtime:
type Value = $Call<typeof getFirstValue, Map<string, number>>;

(5: Value);
(true: Value);  // Error: Value is a `number`


// We could generalize it further:
type GetMapValue<M> =
  $Call<typeof getFirstValue, M>;

(5: GetMapValue<Map<string, number>>);
(true: GetMapValue<Map<string, boolean>>);
(true: GetMapValue<Map<string, number>>);  // Error: value is a `number`
```

## Class<T>

表示类`C`实例的给定类型`T`，类型`Class<T>`是类`C`的类型。例如：

Given a type `T` representing instances of a class `C`, the type `Class<T>` is the type of the class `C`. For example:

```javascript
// @flow
class Store {}
class ExtendedStore extends Store {}
class Model {}

function makeStore(storeClass: Class<Store>) {
  return new storeClass();
}

(makeStore(Store): Store);
(makeStore(ExtendedStore): Store);
(makeStore(Model): Model); // error
(makeStore(ExtendedStore): Model); // Flow infers the return type
```

对于采用类型参数的类，你还必须提供参数。例如:

```javascript
// @flow
class ParamStore<T> {
  constructor(data: T) {}
}

function makeParamStore<T>(storeClass: Class<ParamStore<T>>, data: T): ParamStore<T> {
  return new storeClass(data);
}
(makeParamStore(ParamStore, 1): ParamStore<number>);
(makeParamStore(ParamStore, 1): ParamStore<boolean>); // failed because of the second parameter
```

## $Shape<T>

复制所提供类型的形状，但是会把每个字段都变为可选的。

```javascript
// @flow
type Person = {
  age: number,
  name: string,
}
type PersonDetails = $Shape<Person>;

const person1: Person = {age: 28};  // Error: missing `name`
const person2: Person = {name: 'a'};  // Error: missing `age`
const person3: PersonDetails = {age: 28};  // OK
const person4: PersonDetails = {name: 'a'};  // OK
const person5: PersonDetails = {age: 28, name: 'a'};  // OK
```

## $Supertype<T>

已弃用，避免使用

## $Subtype<T>

已弃用，避免使用

## Existential Type (*)

`*` 作为己知存在的类型.

一个存在的类型作为一个点位符来告诉`Flow`类型推断.

例如，在`Class<ParamStore<T>>`例子中，我们可以为返回值使用一个存在类型：

```javascript
// @flow
function makeParamStore<T>(storeClass: Class<ParamStore<T>>, data: T): * {
  return new storeClass(data);
}
(makeParamStore(ParamStore, 1): ParamStore<number>);
(makeParamStore(ParamStore, 1): ParamStore<boolean>); // failed because of the second parameter
```

可以把`*`看作是`Flow`的`自动`指令，告诉它从上下文填充类型。

跟`any`相比较，`*`可以使您避免丢失类型的安全性。

无需冗长，`存在运算符`对于自动填充类型也是有用的:


```javascript
// @flow
class DataStore {
  data: *; // If this property weren't defined, you'd get an error just trying to assign `data`
  constructor() {
    this.data = {
      name: 'DataStore',
      isOffline: true
    };
  }
  goOnline() {
    this.data.isOffline = false;
  }
  changeName() {
    this.data.isOffline = 'SomeStore'; // oops, wrong key!
  }
}
```
