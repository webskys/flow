# 库定义

> 第三方库

## 什么是“库定义”

大多数`Javascript`程序都依赖第二方代码，而不仅仅是项目控制下的代码。这意味着使用`Flow`的项目可能需要引入外部代码，而这些外部代码可能没有类型信息或没有精准的类型类型，为了解决这个问题，`Flow`支持一个`库定义`的概念（也就是`libdef`）。

一个`libdef`是一个特殊的文件，他告诉`Flow`关于应用程序使用的特定第三方模块或模块包的类型签名，如果你熟悉那些有文件头的语言（像 C++），你可以将`libdefs`视为类似的概念。

这些特定的文件使用跟普通`JS`一样的`.js`扩展名，但是他们放在你项目根目录下一个叫`flow-typed`的文件夹下。放置于这个目录中告诉`Flow`作为`libdefs`来解释，而不是普通JS文件。

 > 注意，为`libdefs`使用`/flow-typed`目录是一种约定，使`Flow`能够开箱即用，同时也鼓励在使用`Flow`的项目中保持一致。但是，使用`.flowconfig`文件的`[libs]`也可以把`Flow`查找`libdefs`显式地配置到别处。

## 最佳做法

试图为你项目中所有第三方库都提供`libdef`。

如果你的项目使用了一个没有类型信息的第三方库，`Flow`对待它，就像其它没有类型依赖的一样，并将所有的导出视为`any`。有趣的是，这是`Flow`隐式地将`any`注入到程序中的唯一地方。

你可以尽可能多地使用第三方库，因为这个行为，它是查找或编写`libdefs`的最佳实践。我们推荐查看`flow-typed` [工具和存](https://github.com/flow-typed/flow-typed/blob/master/README.md)，它能帮助我们为第三方库快速查找到并安装所依赖的预先存在的`libdefs`。

## 创建库定义

花时间编写你自己的`libdef`之前，我们建议你查看下是否已经存在正在处理的第三方代码的`libdef`,`flow-typed`是`Flow`社区一个共享公用的`libdefs`工具和库。它有很多你项目可能需要的公共`libdefs`。

然而，有时并没有预先存在的`libdefs`，或者你的第三方库并不是公开，你不得不自己编写`libdefs`。为此，你得为每个打算编写的`libdef`分别创建一个`.js`的文件，并把他们放在根目录下`/flow-typed`文件夹里。在这些`libdef`文件里，您将使用一组特殊的`Flow`语法来描述相关的第三方代码的接口。

### 声明一个全局函数

在`libdef`文件里声明在整个项目中可访问的全局函数，使用`declare function`语法。

**flow-typed/myLibDef.js**

```javascript
declare function foo(a: number): string;
```

这样就告诉`Flow`项目里的任何代码都能引用`foo`全局函数，并且这个函数有一个参数(数值)和它将返回一个字符串。

### 声明一个全局类

在`libdef`文件里声明在整个项目中可访问的全局类，请使用`declare class`语法：

**flow-typed/myLibDef.js**

```javascript
declare class URL {
  constructor(urlStr: string): URL;
  toString(): string;

  static compare(url1: URL, url2: URL): boolean;
};
```

这样就告诉`Flow`项目里的任何代码都能引用`URL`全局类，注意，这个类定义没有任何实现细节，它仅仅定义了类的接口。

### 声明一个全局变量

在`libdef`文件里声明整个项目都能访问的全局变量，使用`declare var`语法。

**flow-typed/myLibDef.js**

```javascript
declare var PI: number;
```

这样就告诉`Flow`项目里的任何代码都能引用`PI`全局变量，在本例中，它是一个`number`。

### 声明一个全局类型

在`libdef`文件里声明整个项目都能访问的全局类型，使用`declare type`语法。

**flow-typed/myLibDef.js**

```javascript
declare type UserID = number;
```

这样就告诉`Flow`项目里的任何代码都能引用`UserID`全局类型，在本例中，它是一个`number`的别名。

### 声明一个模块

通常，是根据模块而不是全局来把第三方代码组织起来，要编写声明模块存在`libdef`,需要使用声明模块语法：

```javascript
declare module "some-third-party-library" {
  // This is where we'll list the module's exported interface(s)
}
```

在`declare module`后面用引号指定的名称可以是任意字符串，但它应该跟你用来`require`或`import`第三方模块到项目中的字符串相同。

在`declare module`模块主体中，你可以指定那个模块的导出集表。然而，在开始讨论导出之前，我们必须讨论`Flow`支持的两种模块：`CommonJS`模块和`ES`模块。

`Flow`可以同时处理`CommonJS`模块和`ES`模块，但是，在使用`declare module`时，他们有一些相关的差异需要考虑。

#### 声明一个 ES 模块

`ES`模块有两种类型的导出：一种具名导出和一种默认导出。在一个`declare module`主体中`Flow`支持声明导出之一或两者的能力。

##### 具名导出

**flow-typed/some-es-module.js**

```javascript
declare module "some-es-module" {
  // Declares a named "concatPath" export
  declare export function concatPath(dirA: string, dirB: string): string;
}
```

注意，在`declare module`主体中，你也可以声明其他东西，并且这些东西的作用域只限于`declare module`主体内（他们不会从这个模块被导出）。

**flow-typed/some-es-module.js**

```javascript
declare module "some-es-module" {
  // Defines the type of a Path class within this `declare module` body, but
  // does not export it. It can only be referenced by other things inside the
  // body of this `declare module`
  declare class Path {
    toString(): string;
  };

  // Declares a named "concatPath" export which returns an instance of the
  // `Path` class (defined above)
  declare export function concatPath(dirA: string, dirB: string): Path;
}
```

##### 默认导出

**flow-typed/some-es-module.js**

```javascript
declare module "some-es-module" {
  declare class URL {
    constructor(urlStr: string): URL;
    toString(): string;

    static compare(url1: URL, url2: URL): boolean;
  };

  // Declares a default export whose type is `typeof URL`
  declare export default typeof URL;
}
```

在`declare module`主体，可能会同时声明具名和默认导出。

#### 声明一个 CommonJS 模块

`CommonJS` 模块导出单个值（`module.exports`），在`declare module`主体中声明这个值的类型，你应使用`declare module.exports`语法：

**flow-typed/some-commonjs-module.js**

```javascript
declare module "some-commonjs-module" {
  // The export of this module is an object with a "concatPath" method
  declare module.exports: {
    concatPath(dirA: string, dirB: string): string;
  };
}
```

注意，你也可以在`declare module`主体内部声明其他东西，并且这些东西的作用域只限于`declare module`主体内（他们不会从这个模块被导出）。

**flow-typed/some-commonjs-module.js**

```javascript
declare module "some-commonjs-module" {
  // Defines the type of a Path class within this `declare module` body, but
  // does not export it. It can only be referenced by other things inside the
  // body of this `declare module`
  declare class Path {
    toString(): string;
  };

  // The "concatPath" function now returns an instance of the `Path` class
  // (defined above).
  declare module.exports: {
    concatPath(dirA: string, dirB: string): Path,
  };
}
```

注意，因为一个给定的模块不可能既是`ES`模块又是`CommonJS`模块,所以在同一个`declare module`主体中把`declare export [...]`和`declare module.exports: ...`混合起来是一个错误。