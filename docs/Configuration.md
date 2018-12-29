# 配置

`Flow`尽可能地试图开箱即用的工作，但是，也可以配置它跟任何代码库存一起工作。


## .flowconfig

 > 如何配置`Flow`

每个`Flow`项目都包含一个`.flowconfig`文件。你可以通过修改`.flowconfig`文件来配置`Flow`,新项目或开始使用`Flow`的项目可以通过运行`flow init`命令来生成默认的`.flowconfig`。


### 格式

`.flowconfig`使用了一个与`INI`文件类似的自定义格式。我们并不为我们的自定义格式感到自豪，并将在未来使用更好的格式

`.flowconfig`由7个部分组成：

|[declarations]|[include]|[ignore]|
|:--:|:--:|:--:|
|[libs]|[lints]|[options]|
|[version]|||

### 注释

在v0.23.0中添加了注释支持。以零个或多个空格开始，并在后面紧跟一个`#`或`;`或者`💩` 的行将被忽略。例如

```
# This is a comment
  # This is a comment
; This is a comment
  ; This is a comment
💩 This is a comment
  💩 This is a comment
```

### 文件位置

`.flowconfig`的位置很重要，`Flow`把包含`.flowconfig`文件的目录作为其项目的根目录。默认情况下，`Flow`包含项目下面的所有源代码。在`[include]`这部分的路径是相对于项目的根目录。其他的配置通过`<PROJECT_ROOT>`来让你引用项目根目录。

大多数人将`.flowconfig`放入到他们项目的根下面(即：`package.json`那里)，有些人将他们的代码放在`src/`目录下面，因此`.flowconfig`的路径就是`src/.flowconfig`。

### 示例

假定你有如下包含`mydir/.flowconfig`的目录结构：

```
otherdir
└── src
    ├── othercode.js
mydir
├── .flowconfig
├── build
│   ├── first.js
│   └── shim.js
├── lib
│   └── flow
├── node_modules
│   └── es6-shim
└── src
    ├── first.js
    └── shim.js
```

下面是一个你如何使用`.flowconfig`指令的示例。

```
[include]
../otherdir/src

[ignore]
.*/build/.*

[libs]
./lib
```

现在`Flow`在他的检测里包含一个在`.flowconfig`文件路径之外的目录，忽略`build`目录并作用`lib`中的声明。

## .flowconfig [include]

> 如何配置`Flow`包含某些文件

`Flow`需要知道该读取哪些文件并监视更改。这些文件是通过获取所有包含的文件并排除忽略文件来决定的。

在`.flowconfig`文件里的`[include]`这部分告诉`Flow`包含的是那些文件或目录。包含的目录递归地包含了其目录下的所有文件或目录。连接符号一直遵循下去，直到指向被包含的一个文件或目录。每一行都是一个包含路径，路径可以是根目录的相对目录，也可以是绝对目录，并且支持单个或双个星号通配符。

项目根目录（`.flowconfig`所在的地方）自动被包含。

例如：如果`/path/to/root/.flowconfig`文件中包含如下`[include]`部分：

```
[include]
../externalFile.js
../externalDir/
../otherProject/*.js
../otherProject/**/coolStuff/
```

然后，当`Flow`在`/path/to/root`检测时，它将读取并监视：

 - `/path/to/root/`(自动包含)
 - `/path/to/externalFile.js`
 - `/path/to/externalDir/`
 - 在`/path/to/otherProject/`目录下的所有`.js`文件
 - 在`/path/to/otherProject`目录下的任何名叫`coolStuff/`的目录

## .flowconfig [ignore]

> 如何配置`Flow`忽略某些文件

`Flow`需要知道读取那些文件并监视他们的更改，这些文件是包含的文件并排除忽略文件来决定的。

`.flowconfig`文件中的`[ignore]`部分告诉`Flow`在类型检测时忽略与指定正则匹配的文件。默认情况下，不会忽略任何东西。

要记住的事情：

 - 他们是`OCaml`正则表达式。[详情](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Str.html#TYPEregexp)
 - 这些正则表达式与绝对路径匹配，他们应该以`.*`开始。
 - 要先包含忽略的东西之后才能使其忽略，如果你既包含又忽略一个文件，它将被忽略

`[ignore]`部分的例子看起来像这样：

```
[ignore]
.*/__tests__/.*
.*/src/\(foo\|bar\)/.*
.*\.ignore\.js
```

上面这个`[ignore]`部分将忽略：
 - 忽略`__tests__`下面的任何文件或目录。
 - 忽略`.*/src/foo`或`.*/src/bar`下面的任何文件或目录。
 - 忽略扩展名是`.ignore.js`的任何文件。

从`Flow`的`v0.23.0`版本开始，你可以在你的正则表达式里使用`<PROJECT_ROOT>`点位符，在运行时，`Flow`将这个点位符当作项目根目录的绝对路径，这对于编写相对而不是绝对的正则表达式很有用。

例如，你可以这样写：

```
[ignore]
<PROJECT_ROOT>/__tests__/.*
```

上面将忽略项目要目录下一个名为`__tests__/`目录下的任何文件或目录，但是，与前面的例子`.*/__tests__/.*`不同，这里的不会忽略其他目录下名为`__tests__/`的目录，比如`src/__tests__/`。

## .flowconfig [libs]

 > 如何配置`Flow`来包括`lib`文件

`.flowconfig`文件里的`[libs]`部分，告诉`Flow`在对你的代码类型检测时包含指定的[库定义](#docs/libdefs)。可以指定多个库。默认情况下，你项目根目录下的`flow-typed`文件夹作为库定义被包含了。这个默认值在不用额外配置的情况下允许你使用`flow-typed`来安装库定义。

`[libs]`部分的每一行都是指向你希望包含的库的文件或目录。这些路径可以是相对于项目根目录的，也可以是绝对的，递归包含库文件下所有文件和目录。

## .flowconfig [lints]

 > 在`Flow`中如何配置`linter`

 > 注：linter 是一类小程序的总称，它不像编译器程序那么大，它可以用来检查程序的 文体、语句的语法、句法错误，并即时标注和指出来（例如，把声明了但没使用的多余变量指出来，把错误语句 变成黑体）， 是程序开发的辅助工具。

`.flowconfig`文件中的`[lints]`可以包含数个键-值对的形式：

```
[lints]
ruleA=severityA
ruleB=severityB
```

查看[linting文档](#docs/linting)能够获取更多信息。

## .flowconfig [options]

 > 如何配置`Flow`的各种选项

`.flowconfig`文件中的`[options]`部分可以包含数个键-值对的形式：

```
[options]
keyA=valueA
keyB=valueB
```

任何`[options]`被省略的选项都将使用它们的默认值。可以使用命令行标志覆盖某些选项。

#### all (boolean)

设置为`true`将检测所有文件，不仅仅是那些有`@flow`的文件。

默认值是`false`。

#### emoji (boolean)

设置为`true`将为`Flow`检测项目时为输出的状态信息添加表情符号，

默认值是`false`。

#### esproposal.class_instance_fields (enable|ignore|warn)

设置为`warn`以指示`Flow`将为每个使用尚未规范的 [class fields](https://github.com/tc39/proposal-class-public-fields)实例给出一个警告。

你也可以设置为`ignore`，以指示`Flow`简单地忽略这种语法（即：`Flow`不会在这种类实例存在的属性上使用这样的语法）。

默认值是`enable`,即允许使用这个还在提案阶段的语法

#### esproposal.class_static_fields (enable|ignore|warn)

设置为`warn`以指示`Flow`将为每个使用尚未规范的静态 [class fields](https://github.com/tc39/proposal-class-public-fields) 给出一个警告。

你也可以设置为`ignore`，以指示`Flow`简单地忽略这种语法（即：`Flow`不会在这种类存在的静态属性上使用这样的语法）。

默认值是`enable`,即允许使用这个还在提案阶段的语法。

#### esproposal.decorators (ignore|warn)

你也可以设置为`ignore`，以指示`Flow`忽略提案阶段的修饰符

默认值是`warn`，即使用的话抛出警告，因为这种提案还处于早期阶段。

#### esproposal.export_star_as (enable|ignore|warn)

设置为`enable`，指示`Flow`支持`export * as`语法形式。你也可以设置为`ignore`,指示`Flow`简单地忽略这种语法。

默认值是`warn`,因为这种提案还是早期阶段，所以将抛出警告。

#### esproposal.optional_chaining (enable|ignore|warn)

设置为`enable`，指示`Flow`支持使用尚未规范的 [optional chaining](https://github.com/tc39/proposal-optional-chaining)。

你也可以设置为`ignore`，指示`Flow`简单地忽略这种语法。

默认值是`warn`,将抛出警告,因为这种提案还是早期阶段。


#### esproposal.nullish_coalescing (enable|ignore|warn)

设置为`enable`，指示`Flow`支持使用尚未规范的 [nullish coalescing](https://github.com/tc39/proposal-nullish-coalescing)。

你也可以设置为`ignore`，指示`Flow`简单地忽略这种语法。

默认值是`warn`,将抛出警告,因为这种提案还是早期阶段。

#### experimental.const_params

设置为`true`使得`Flow`将所有函数参数视为常量绑定，重新分配参数是一个错误，它让`Flow`在细化时不那么保守。

默认值是`false`。

#### include_warnings (boolean)

设置为`true`使得`Flow`命令输出错误时包含警告信息。为了避免控制台喷涌，在CLI中默认情况下，警告是隐藏的（展示警告，IDE是一个更好的界面）

默认值是`false`。

#### log.file (string)

日志文件的路径（默认是`/tmp/flow/<escaped root path>.log`）

#### max_header_tokens (integer)

`Flow`尝试避免解析非`Flow`文件。意思是，`Flow`需要分析一个文件看他里面是不是有`@flow`或者`@noflow`。在`Flow`判决没有相应文本块之前，这个选项让你配置`Flow`分析文件的多少。

 - 既没有`@flow`也没有`@noflow`,使用`Flow`语法不允许解析该文件,并且不能对它进行类型检测。
 - `@flow`，允许使用`Flow`语法析该文件，并具对其进行类型检测。
 - `@noflow`，允许使用`Flow`语法析该文件，但不对其进行类型检测。意思是一个文件不用删除所有`Flow`特定语法，使用``@noflow`来作为废除`Flow`的办法。

默认值是`10`。

#### module.file_ext (string)

默认情况下，`Flow`将查寻以`.js`、`.jsx`、`.mjs`和`.json`为扩展名的文件。你可以使用这个选项来覆盖这种行为。

例如，你可以这样做：

```
[options]
module.file_ext=.foo
module.file_ext=.bar
```

然后，`Flow`将查寻以`.foo`和`.bar`为扩展名的文件。

 > 你可以多次指定`module.file_ext`。

#### module.ignore_non_literal_requires (boolean)

设置为`true`，当你使用`require()`,除了字符串字面量，`Flow`将不再抱怨。

默认为`false`。

#### module.name_mapper (regex -> string)

指定一个去匹配模块名的正则表达式和替换模式，他们以`->`分割开。

```
module.name_mapper='^image![a-zA-Z0-9$_]+$' -> 'ImageStub'
```

这样就使得`Flow`对待`require('image!foo.jpg')`像`require('ImageStub')`一样

这些是[OCaml](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Str.html#TYPEregexp)正则表达式，使用 `\(` 和 `\)` （需要斜杠）来创建一个捕获组，你也可以在替换模式中作为\1` (直到 `\9`)引用 。

 > 你可以多次指定`module.name_mapper`。

#### module.name_mapper.extension (string -> string)

指定一个去匹配的文件扩展名和换模式夫，他们用`->`分割开。

 > 注意：`module.name_mapper='^\(.*\)\.EXTENSION$' -> 'TEMPLATE')`这仅仅是为了速记。

例如：

```
module.name_mapper.extension='css' -> '<PROJECT_ROOT>/CSSFlowStub.js.flow'
```

这就让`Flow`对待`require('foo.css')`像`require(PROJECT_ROOT + '/CSSFlowStub')`一样。

 > 你可以为不同的扩展多次指定`module.name_mapper.extension`。

#### module.system (node|haste)

模块系统用来解决`import` 和 `require`， [Haste](https://github.com/facebookarchive/node-haste)用于`React`内部。

模块系统默认的是`node`

#### module.system.node.resolve_dirname (string)

默认情况下，`Flow`将查找一个名为`node_modules`作为`node`模块，你可以用这个选项来配置这个模块。

例如，你可以这样做：

```
module.system.node.resolve_dirname=node_modules
module.system.node.resolve_dirname=custom_node_modules
```

然后`Flow`将查找名为`node_modules`或`custom_node_modules`的目录了

 > 你可以多次指定`module.system.node.resolve_dirname`

#### module.use_strict (boolean)

如果你使用为每个模块顶部添加`"use strict";`的编译器，则设置为`true`。

默认值`false`。

#### munge_underscores (boolean)

设置为`true`,`Flow`把类属性和方法的前辍下划线作为私有属性和方法看待。这个应该结合[jstransform](https://github.com/facebookarchive/jstransform/blob/master/visitors/es6-class-visitors.js)的`ES6`类转换一起使用,后者在运行时强制执行相同的隐私。

默认值`false`。

#### no_flowlib (boolean)

`Flow`有内置库定义，设置这个选项为`true`将告诉`Flow`忽略内置库定义。

默认值`false`。

#### server.max_workers (integer)

`Flow`服务能启动的最大工作线程，默认情况下，服务将使用所有可用的内核。

#### sharedmemory.dirs (string)

这只对`Linux`有效。

`Flow`的共享内存存在于内存映射文件中，在`Linux`现代版本(3.17+)中，有一个系统调用`memfd_create`,它允许`Flow`仅在内存中匿名地创建文件。然而，在旧内核中，`Flow`需要在文件系统上创建一个文件。理想情况下，该文件存在于内存支持的`tmpfs`上。这个选项让你知道在哪里创建该文件。

默认情况，这个选项设置为`/dev/shm`和`/tmp`

 > 你可以多次指定`sharedmemory.dirs`

#### sharedmemory.minimum_available (unsigned integer)

这只对`Linux`有效。

作为解释`sharedmemory.dirs`这个选项的描述，`Flow`需要为为旧内核在文件系统创建一个文件。`sharedmemory.dirs`指定了可以创建共享内存文件的地址列表。对于每个地址，`Flow`将检查以确保文件系统有足够的空间用于共享内存文件，`Flow`很可能用完空间，它将跳过该地址并尝试下一个地址。这个选项允许您为共享内存配置文件系统上需要的最小空间量。

默认是`536870912` (2^29个字母,也就是一半Gb).

#### sharedmemory.dep_table_pow (unsigned integer)

共享内存的最大3部分是依赖表、哈希表和堆，当堆增长和收缩时，另两个表被完全分配。此选项允许更改依赖表的大小。

将此选项设置为`X`意味着表将支持最多`2^X`元素，即`16*2^X`字节

默认情况下，设置为`17`（表大小为`2^17`，即`2MB`）

#### sharedmemory.hash_table_pow (unsigned integer)

共享内存的最大3部分是依赖表、哈希表和堆，当堆增长和收缩时，另两个表被完全分配。此选项允许更改哈希表的大小。

将此选项设置为`X`意味着表将支持最多`2^X`元素，即`16*2^X`字节

默认情况下，设置为`19`（表大小为`2^19`，即`8MB`）

#### sharedmemory.heap_size (unsigned integer)

此选项配置共享堆的最大可能大小。您很可能不需要配置它，因为它不会真正影响`Flow`使用的`RSS`。但是，如果您正在处理大型代码库，那么在初始化之后您可能会看到以下错误："Heap init size is too close to max heap size; GC will never get triggered!"，这种情况下，您可能需要增加堆的大小。

默认情况下，设置为`268435600`（`25*2^30`字节，即`25GB`）

#### sharedmemory.log_level (unsigned integer)

设置为`1`,将使`Flow`输出关于共享内存序列化和共享内存反序列化的数据信息，

默认是`0`。

#### strip_root (boolean)

过时了，不要使用这个选项。在命令行使用`--strip-root`代替，在错误信息中把文件路径中的根目录删除。


#### suppress_comment (regex)

定义一个神奇的注释，来封锁接下来行中的任何`Flow`错误。例如

```
suppress_comment= \\(.\\|\n\\)*\\$FlowFixMe
```

将匹配像这样的注释并封锁了错误：

```
// $FlowFixMe: suppressing this error until we can refactor
var x : string = 123;
```

如果接下来的行没有错误（封锁不必要），将显示一个`"Unused suppression"`警告。

如果你在你配置中没有指定封锁注释，`Flow`将应用一个默认的`// $FlowFixMe`。

 > 你可以多次指定`suppress_comment`。如果指定了`suppress_comment`,内建的`$FlowFixMe`将会被移除，从而来支持你指定的正则。如果你希望使用带附加自定义封锁注释的`$FlowFixMe`，你必须在你自定义封锁列表里手动指定`\\(.\\|\n\\)*\\$FlowFixMe`。

#### suppress_type (string)

这个选项允许你使用给定字符串为`any`指定别名，这有助于解释为什么要使用`any`。例如，你有时使用`any`来封锁一个错误，有时又用它来标记一个`TODO`。

 - TODO：表示需要实现，但目前还未实现的功能
 - FIXME：代码是错误的，不能工作，需要修复

你的代码可以像这样：

```
var myString: any = 1 + 1;
var myBoolean: any = 1 + 1;
```

如果在你的配置里添加如下信息

```
[options]
suppress_type=$FlowFixMe
suppress_type=$FlowTODO
```

你可以把代码修改为更易读：

```
var myString: $FlowFixMe = 1 + 1;
var myBoolean: $FlowTODO = 1 + 1;
```

 > 你可以多次使用`suppress_type`

#### temp_dir (string)

告诉`Flow`使用哪个目录作为临时目录。可以用命令行标志`--temp-dir`覆盖。

默认值是`/tmp/flow`。

#### traces (integer)

启用所有错误输出（通过系统类型，显示关于`flow`的附加信息）追溯到指定的深度，这可能消耗非常大，因此默认情况下禁用。

## .flowconfig [version]

> 如何将项目绑定到`Flow`的特定版本

你可以在`.flowconfig`文件里指定你期望使用的是那个`Flow`版本，为此可以使用`[version]`部分。如果把这个部分省略或空白，那么任何版本都是允许的。如果指定版本不匹配，`Flow`将立即报名并退出。

因此，如果在`.flowconfig`有如下信息：

```
[version]
0.22.0
```

而你试图使用`v0.21.0`版本的`Flow`,`Flow`将立即给出错误的信息

```
"Wrong version of Flow. The config specifies version 0.22.0 but this is version 0.21.0"
```

到目前为止,我们支持以下方式指定支持的版本

 - 明确的版本（即：`0.22.0`，仅匹配`0.22.0`）;
 - 交叉范围（即：`>=0.13.0 <0.14.0`,匹配`0.13.0`、`0.13.5`，但不匹配`0.14.0`）;
 - 脱字符范围,中（即：`^0.13.0`扩展到`>=0.13.0<0.14.0`，而`^0.13.1`扩展到`>=0.13.1<0.14.0`，而`^1.2.3`扩展到`>=1.2.3<2.0.0`）


 > 使用脱字符`^`，版本号三个位置的数值，从左边起直到第一个非零位置上的值是不能变的，变的是这个非零位置后面的值。