# 模块类型

## 导入和导出类型

在模块(文件)之间分享类型通常是很有用的。在`Flow`里，你可以从一个文件里导出类型别名、接口和类并在另一个文件导入他们。

**exports.js**

```javascript
// @flow
export default class Foo {};
export type MyObject = { /* ... */ };
export interface MyInterface { /* ... */ };
```

**imports.js**

```javascript
// @flow
import type Foo, {MyObject, MyInterface} from './exports';
```

 > 在文件顶部，不要忘记`@flow`,否则`flow`不会报告错误


## 导入和导出值

`Flow`使用`typeof`也支持导入从另一个模块里导出的值的类型。

**exports.js**

```javascript
// @flow
const myNumber = 42;
export default myNumber;
export class MyClass {
  // ...
}
```

**imports.js**

```javascript
// @flow
import typeof myNumber from './exports';
import typeof {MyClass} from './exports';
```

正如导入其他类型一样，这段代码将被编译器破坏，并没有在加一个文件里添加依赖。
