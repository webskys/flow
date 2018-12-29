# Linting

 > 了解如何配置`Flow`的`linter`来查找可能不良的代码。（服务`Flow`的小工具）

## Linting概述

 >对于`Flow`里`Linting`的介绍

`Flow`包含一个`Linting`框架，它可以告诉你不仅仅是类型错误的更多信息。这个框架是高度可配置的，以便向你显示所需的信息，并隐藏不需要的信息。

### 在 .flowconfig 里配置Lints

可以在`.flowconfig`文件的`[lints]`部分用`rule=severity`这样的对设置`Lint`，这些设置是全局的，对于整个项目都可以使用。

**Example:**

```javascript
[lints]
# all=off by default
all=warn
untyped-type-import=error
sketchy-null-bool=off
```

### 通过 CLI 配置Lints

可以通过指定`Flow`命令的`--lints`标签来设置`Lint`，把`rule=severity`这样的对用逗号隔开。这些设置是全局的，对于整个项目都可以使用。

**Example:**

```
flow start --lints "all=warn, untyped-type-import=error, sketchy-null-bool=off"
```

### 使用注释配置Lints

可以在一个文件中通过`flowlint`注释来设置`Lint`，这种设置适用于一个文件的一个区域，单行或一行的某部分。

**Example:**

```javascript
/* flowlint
  *   sketchy-null:error,
  *   untyped-type-import:error
  */
const x: ?number = 0;

if (x) {} // Error
import type {Foo} from './untyped.js'; // Error

// flowlint-next-line sketchy-null:off
if (x) {} // No Error

if (x) {} /* flowlint-line sketchy-null:off */ // No Error

// flowlint sketchy-null:off
if (x) {} // No Error
if (x) {} // No Error
import type {Bar} from './untyped.js'; // Error; unlike a $FlowFixMe, a flowlint comment only suppresses one particular type of error.
// flowlint sketchy-null:error
```

### Lint设置的优先级

使用`flowlint`注释设置的`Lint`有最高的优先级，其次是用`--lints`标记的，再就是通过`.flowconfig`文件的。这个顺序允许你使用`flowlint`注释来精细化`linting`控制，`--lints`标记可以用来使用新的`Lint`设置，`.flowconfig`用于相对稳定的设置。

在`--lints`标记和`.flowconfig`里，后面的规则覆盖前面的规则。

```javascript
[lints]
sketchy-null=warn
sketchy-null-bool=off
```

`lint`设置解析器相当智能，如果您编写了冗余规则、完全覆盖的规则或未使用的抑制，它将阻止您。这应该可以防止`lint`规则的大多数意外错误。

### 严重度及其意义

**off**：`Lint`就是忽略，调协一个`Lint`为`off`类似于用一个抑制注释来抑制一个类型错误，除了有更多的粒度

**warn**：警告是`linting`框架一个新引进的严重性级别,有两种不同于错误方式来对待他们：

 - 警告不影响`Flow`代码的出口，如果`Flow`找到警告而不是错误，它将返回`0`;
 - 避免出现喷涌，警告默认情况下不会展示在CLI。可以传递`–include-warnings`标记为`Flow`服务或`Flow`客户端来启动CLI显示警告，或者在`.flowconfig`文件里设置`include_warnings=true`，对于一些较小的程序想一次性查看项目中所有警告信息这是非常有用的。
 - 警告具有特殊的IDE集成

**error**:严重度是`error`的`lints`,被当作跟其它`Flow`错误一样来对待。


## flowlint 注释

 > 细粒度控制

设置`flowlint`注释来配置`lint`允许你在同一文件指定不同的设置或者在不同文件的不同区域指定不同的设置。这些注释有3种形式：

```
flowlint
flowlint-line
flowlint-next-line
```

在所有形式中，单词之间的空格和星号都被忽略，允许灵活的格式设置。

### flowlint

基本的`flowlint`注释采用逗号分隔的`rule:severity`对列表，并把这些设置应用到余下的源文件，直到覆盖他们。这有三个主要目的：在块上应用设置、在文件上应用设置以及在一行的一部分上应用设置。

**在块上应用设置**：在一个块代码上应用特定的设置使用一对`flowlint`注释，例如，为了禁止在类型块导入无类型的`Lint`，看起来如下：

```javascript
import type {
  // flowlint untyped-type-import:off
  Foo,
  Bar,
  Baz,
  // flowlint untyped-type-import:error
} from './untyped.js';
```

**在文件上应用设置**：A `flowlint` comment doesn’t have to have a matching comment to form a block. An unmatched comment simply applies its settings to the rest of the file. You could use this, for example, to suppress all sketchy-null-check lints in a particular file:

```javascript
// flowlint sketchy-null:off
...
```

**在一行的一部分上应用设置**：The settings applied by `flowlint` start and end right at the comment itself. This means that you can do things like

```javascript
function (a: ?boolean, b: ?boolean) {
  if (/* flowlint sketchy-null-bool:off */a/* flowlint sketchy-null-bool:warn */ && b) {
    ...
  } else {
    ...
  }
}
```

if you want control at an even finer level than you get from the line-based comments.

### flowlint-line

A `flowlint-line` comment works similarly to a `flowlint` comment, except it only applies its settings to the current line instead of applying them for the rest of the file. The primary use for `flowlint-line` comments is to suppress a lint on a particular line:

```javascript
function (x: ?boolean) {
  if (x) { // flowlint-line sketchy-null-bool:off
    ...
  } else {
    ...
  }
}
```

### flowlint-next-line

`flowlint-next-line` works the same as `flowlint-line`, except it applies its settings to the next line instead of the current line:

```javascript
function (x: ?boolean) {
  // flowlint-next-line sketchy-null-bool:off
  if (x) {
    ...
  } else {
    ...
  }
}
```

## IDE Integration

 > Warnings in your IDE

Flow has worked with Nuclide directly on adding support for the new warning severity level. Certain features are likely to be in other editors, but others might not yet be implemented.

### Nuclide

In Nuclide, Flow warnings are distinct from Flow errors and rendered in a different color.

Nuclide v0.243.0 onward has support for working with Flow to limit the reported warnings to the working fileset. This allows Nuclide and Flow to work efficiently on large codebases with tens of thousands of unsuppressed warnings.

### Other Editors

In most editors, Flow warnings are likely to be rendered the same way as other warnings are rendered by that editor.

Most editors will likely display all Flow warnings, which is fine for small- to medium-scale projects, or projects with fewer unsuppressed warnings.

## Lint Rule Reference

 > **Want a lint that isn’t here?** We’re looking to our community to add lints that leverage Flow’s type system. (Tutorial blog post coming soon!)

### Available Lint Rules


#### all
While all isn’t technically a lint rule, it’s worth mentioning here. all sets the default level for lint rules that don’t have a level set explicitly. all can only occur as the first entry in a .flowconfig or as the first rule in a –lints flag. It’s not allowed in comments at all because it would have different semantics than would be expected. (A different form of comment with the expected semantics is in the works.)

#### sketchy-null

Triggers when you do an existence check on a value that can be either null/undefined or falsey.

For example:

```javascript
const x: ?number = 5;
if (x) {} // sketchy because x could be either null or 0.

const y: number = 5;
if (y) {} // not sketchy because y can't be null, only 0.

const z: ?{foo: number} = {foo: 5};
if (z) {} // not sketchy, because z can't be falsey, only null/undefined.
```

Setting `sketchy-null` sets the level for all sketchy null checks, but there are more granular rules for particular types. These are:

 > sketchy-null-bool
 > sketchy-null-number
 > sketchy-null-string
 > sketchy-null-mixed


The type-specific variants are useful for specifying that some types of sketchy null checks are acceptable while others should be errors/warnings. For example, if you want to allow boolean sketchy null checks (for the pattern of treating undefined optional booleans as false) but forbid other types of sketchy null checks, you can do so with this `.flowconfig [lints]` section:


```javascript
[lints]
sketchy-null=warn
sketchy-null-bool=off
```

and now

```javascript
function foo (bar: ?bool): void {
  if (bar) {
    ...
  } else {
    ...
  }
}
```

doesn’t report a warning.

Suppressing one type of sketchy null check only suppresses that type, so, for example

```javascript
// flowlint sketchy-null:warn, sketchy-null-bool:off
const x: ?(number | bool) = 0;
if (x) {}
```

would still have a sketchy-null-number warning on line 3.

#### sketchy-number

Triggers when a `number` is used in a manner which may lead to unexpected results if the value is falsy. Currently, this lint triggers if a `number` appears in:

 - the left-hand side of an && expression.

As a motivating example, consider this common idiom in React:

```javascript
{showFoo && <Foo />}
```

Here, `showFoo` is a boolean which controls whether or not to display the `<Foo />` element. If `showFoo` is true, then this evaluates to `{<Foo />}`. If `showFoo` is false, then this evaluates to `{false}`, which doesn’t display anything.

Now suppose that instead of a boolean, we have a numerical value representing, say, the number of comments on a post. We want to display a count of the comments, unless there are no comments. We might naively try to do something similar to the boolean case:

```javascript
{count && <>[{count} comments]</>}
```

If `count` is, say, `5`, then this displays “[5 comments]”. However, if `count` is `0`, then this displays “0” instead of displaying nothing. (This problem is unique to number because 0 and NaN are the only falsy values which React renders with a visible result.) This could be subtly dangerous: if this immediately follows another numerical value, it might appear to the user that we have multiplied that value by 10! Instead, we should do a proper conditional check:

```javascript
{count ? <>[{count} comments]</> : null}
```

#### untyped-type-import

Triggers when you import a type from an untyped file. Importing a type from an untyped file results in an any alias, which is typically not the intended behavior. Enabling this lint brings extra attention to this case and can help improve Flow coverage of typed files by limiting the spread of implicit `any` types.

#### untyped-import

Triggers when you import from an untyped file. Importing from an untyped file results in those imports being typed as `any`, which is unsafe.


#### unclear-type

Triggers when you use `any`, `Object`, or `Function` as type annotations. These types are unsafe.

#### unsafe-getters-setters

Triggers when you use getters or setters. Getters and setters can have side effects and are unsafe.

For example:

```javascript
const o = {
  get a() { return 4; }, // Error: unsafe-getters-setters
  set b(x: number) { this.c = x; }, // Error: unsafe-getters-setters
  c: 10,
};
```

#### nonstrict-import

Used in conjuction with `Flow Strict`. Triggers when importing a non `@flow strict` module. When enabled, dependencies of a `@flow strict` module must also be `@flow strict`.

#### deprecated-declare-exports

Note: This lint was removed in Flow version 0.68, along with the `declare var exports` syntax.

Triggers when the deprecated syntax is used to declare the default export of a `declared CommonJS module`.

Before Flow version 0.25, the way to declare the default exports looked like this:

```javascript
declare module "foo" {
  declare var exports: number; // old, deprecated syntax
}
```

In version 0.25, we introduced an alternative syntax:

```javascript
declare module "foo" {
  declare module.exports: number;
}
```

The new syntax is simpler and less magical. The old syntax will be removed in a future version of Flow.

This lint is enabled by default. If you see an error, you should try to rewrite the offending declaration. If you are unable to rewrite the declaration (for example, if it’s part of a node_module dependency), you can disable the lint in your `.flowconfig`.

To disable this lint, add a line to the `[lints]` section of your project’s `.flowconfig` file.

```javascript
[lints]
deprecated-declare-exports=off
```

However, note that this syntax will be removed soon, so you should file issues with any projects that still use it.

#### unnecessary-optional-chain

Triggers when you use `?.` where it isn’t needed. This comes in two main flavors. The first is when the left-hand-side cannot be nullish:

```javascript
type Foo = {
  bar: number
}

declare var foo: Foo;
foo?.bar; // Lint: unnecessary-optional-chain
```

The second is when the left-hand-side could be nullish, but the short-circuiting behavior of ?. is sufficient to handle it anyway:

```javascript
type Foo = {
  bar: {
    baz: number
  }
}

declare var foo: ?Foo;
foo?.bar?.baz; // Lint: unnecessary-optional-chain
```

In the second example, the first use of `?.` is valid, since `foo` is potentially nullish, but the second use of `?.` is unnecessary. The left-hand-side of the second `?. (foo?.bar)` can only be nullish as a result of `foo` being nullish, and when foo is nullish, short-circuiting lets us avoid the second `?.` altogether!

```javascript
foo?.bar.baz;
```

This makes it clear to the reader that `bar` is not a potentially nullish property.

#### unnecessary-invariant

Triggers when you use `invariant` to check a condition which we know must be truthy based on the available type information. This is quite conservative: for example, if all we know about the condition is that it is a `boolean`, then the lint will not fire even if the condition must be `true` at runtime.

Note that this lint does not trigger when we know a condition is always `false`. It is a common idiom to use `invariant()` or `invariant(false, ...)` to throw in code that should be unreachable.

#### deprecated-utility

Triggers when you use the `$Supertype` or `$Subtype` utility types, as these types are unsafe and usually just equivalent to `any`. If the utilities were being used in a sound manner, the desired behavior can usually be recovered through the `$Shape` utility or `bounded generics`.