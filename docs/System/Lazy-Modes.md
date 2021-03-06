# 惰性模式

 > `Flow`的惰性模式解释

默认情况下，`Flow`服务会对你所有代码进行类型检测。这就可以回答“能在我代码里任何地方都检测`Flow`错误吗？”类似的问题。这对于一个连续集成钩子工具非常有有，可以预防代码更改而引入`Flow`错误。

然而，有时`Flow`用户并不会关心所有代码。如果他们编辑一个`foo.js`文件，他们可能只希望`Flow`对`foo.js`所需库的子集进行检测。因此`Flow`只检查少量的文件，这样会更快，这就是`Flow`惰性模式动机。

 ## 文件分类

惰性模式尝试将代码分为四个类别

 - **焦点文件**：这些都是用户所关心的文件;
 - **依赖文件**：这些文件依赖于焦点文件，对焦点文件修改可能会造成依赖文件类型错误;
 - **从属文件**：这些文件是用于对聚焦或依赖文件进行类型检查的文件;
 - **不受控制文件**. 所有其他文件。

惰性模式仍然会找到所有`JavaScript`文件并解析它们。但它不会对不受控制文件进行类型检查。

## 选择焦点文件

`Flow`可以使用下面两种方法来告诉用户关心哪些文件。

 - **集成开发环境惰性模式**：通过`flow ide`（以及未来的`flow lsp`）IDE告诉`Flow`哪些文件己经打开或关闭。`Flow`把从`Flow`服务启动以来一直打开的文件作为焦点对待。
 - **文件系统惰性模式**：`Flow`把在文件系统里被修改的文件作为焦点对待。通过命令行，这种模式更容易使用，但是`rebase`能使每个文件都成为焦点

## 使用IDE惰性模式

在IDE惰性模式下启动`Flow`服务，你运行：

```javascript
flow server --lazy-mode ide
```

IDE需要结合`flow ide`(或未来的`flow lsp`)来告诉`Flow`哪些文件是打开的。

## 使用文件系统惰性模式

在文件系统惰性模式下启动`Flow`服务，你运行：

```javascript
flow server --lazy-mode fs
```

## 强制文件作为焦点

你可以对过命令行，强制`Flow`把一个或多个文件作为焦点对待。

```javascript
flow force-recheck --focus ./path/to/A.js /path/to/B.js
```