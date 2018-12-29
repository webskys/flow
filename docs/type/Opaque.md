# 不透明类型别名

 > 在整个类型系统中强制抽象化

不透明的类型别名是类型别名，在其定义的文件外不允许访问他们的底层类型。

```javascript
opaque type ID = string;
```

不透明类型别名，像变通类型别名一样，可以一个类型能用的任何地方使用。

```javascript
// @flow
opaque type ID = string;

function identity(x: ID): ID {
  return x;
}
export type {ID};
```

## 语法

一个不透明类型别名使用`opaque type`加上其名称，然后在名称后面用等于符`=`和定义的类型来创建。

```javascript
opaque type Alias = Type;
```

你可以使用冒号`:`加一个类型使一个不透明类型别名成为受这个类型约束的子类。

```javascript
opaque type Alias: SuperType = Type;
```

任何类型都可以成为不透明类型别名的超类。

```javascript
opaque type StringAlias = string;
opaque type ObjectAlias = {
  property: string,
  method(): number,
};
opaque type UnionAlias = 1 | 2 | 3;
opaque type AliasAlias: ObjectAlias = ObjectAlias;
opaque type VeryOpaque: AliasAlias = ObjectAlias;
```

## 类型检测

不透明类型别名的类型检测

### 在定义文字内

在别名定义的文件内，不透明类型别名的行为跟普通类型别名的行为一样。

```javascript
//@flow
opaque type NumberAlias = number;

(0: NumberAlias);

function add(x: NumberAlias, y: NumberAlias): NumberAlias {
    return x + y;
}
function toNumberAlias(x: number): NumberAlias { return x; }
function toNumber(x: NumberAlias): number { return x; }
```

### 在定义文字外

当导入外部不透明类型别名时，它的行为类似于标称类型`nominal type`，隐藏了其底层类型。

#### exports.js

```javascript
export opaque type NumberAlias = number;
```

#### imports.js

```javascript
import type {NumberAlias} from './exports';

(0: NumberAlias) // Error: 0 is not a NumberAlias!

function convert(x: NumberAlias): number {
  return x; // Error: x is not a number!
}
```

### 子类型约束

当你为一个不透明类型别名添加了一个子类型约束，我们就允许了这不透明类型在其定义的文件外可以作为超类使用。

#### exports.js

```javascript
export opaque type ID: string = string;
```

#### imports.js

```javascript
import type {ID} from './exports';

function formatID(x: ID): string {
    return "ID: " + x; // Ok! IDs are strings.
}

function toID(x: string): ID {
    return x; // Error: strings are not IDs.
}
```

当你使用子类型约束创建了一个不透明类型别名，类型位置中的类型必须是超类型位置中类型的子类型


```javascript
//@flow
opaque type Bad: string = number; // Error: number is not a subtype of string
opaque type Good: {x: string} = {x: string, y: number};
```

### 泛型

不透明类型别名也有他们自己的泛型，而且它们的工作方式与普通类型别名中的泛型一样

```javascript
// @flow
opaque type MyObject<A, B, C>: { foo: A, bar: B } = {
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

### 库定义

大多数`JavaScript`程序依赖于第三方代码，`Flow`项目可能需要引用的第三方代码不一定有精准的类型信息，因此`Flow`支持一种`库定义`的概念，又叫做`libdef`。

你也可以在`libdefs`里声明不透明类型别名，在那里，你省略了底层类型，但仍可选择包含超类型。

```javascript
declare opaque type Foo;
declare opaque type PositiveNumber: number;
```