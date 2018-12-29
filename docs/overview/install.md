# 安装
（Installation）
 > 为一个项目安装和设置`Flow`

依赖于你使用的工具和环境，这里有几种方式安装`Flow`。

## 安装编译器

首先，你需要一个编译器来剔除`Flow`的类型注解。你可以选择`Babel`或`flow-remove-types`.

### Babel

`Babel`是一个支持`Flow`的`Javascript`编译器。`Babel`我们通俗的理解就是把现在浏览器不支持的新特性（比如：ES6）转换为可以支持的ES5语法，从而使我们能在项目中体测新标准带来的福利。`Babel`使用我们`Flow`代码，并剔除所有的类型注解。

首先得安装 `babel-cli` 或 `babel-preset-flow`

#### `npm` 环境

```
npm install --save-dev babel-cli babel-preset-flow
```

#### `Yarn` 环境
```
yarn add --dev babel-cli babel-preset-flow
```

然后，你需要在项目根目录里新建一个 `.babelrc`文件，并在 `presets`属性里设置 `flow`

```javascript
{
  "presets": ["flow"]
}
```
如果你的源代码在 `src` 目录里，你可以将这些代码编译到另一个目录中

#### `npm` 环境
```
./node_modules/.bin/babel src/ -d lib/
```

#### `Yarn` 环境
```
yarn run babel src/ -- -d lib/
```

当然，在 `package.json`配置运行

```javascript
{
  "name": "my-project",
  "main": "lib/index.js",
  "scripts": {
    "build": "babel src/ -d lib/"
  }
}
```
这样运行 `npm run build` 或 `yarn run build` 即可 。

### flow-remove-types

`flow-remove-types`是一个简单去掉类型注释的命令行工具，他是一个不需要 `Babel` 支持的轻量级代替 `Babel`的工具。

首行安装 `flow-remove-types`

#### `npm` 环境

```
npm install --save-dev flow-remove-types
```

#### `Yarn` 环境

```
yarn add --dev flow-remove-types
```


如果你的源代码在 `src` 目录里，你可以将这些代码编译到另一个目录中

#### `npm` 环境
```
./node_modules/.bin/flow-remove-types src/ -d lib/
```

#### `Yarn` 环境
```
yarn run flow-remove-types src/ -- -d lib/
```

在 `package.json`配置运行

```javascript
{
  "name": "my-project",
  "main": "lib/index.js",
  "scripts": {
    "build": "flow-remove-types src/ -d lib/"
  }
}
```

使用`npm run build` 或 `yarn run build` 运行。

## 安装 Flow

把一个明确版本的`Flow` 单独安装到项目比全局安装更好。如果你熟悉 `npm` 或者 `yarn`安装过程更加容易。

### `npm` 环境

把`npm`包安装到`devDependency`上

```
npm install --save-dev flow-bin
```

在 `package.json` 的 `scripts` 里添加 `"flow"`

```javascript
{
  "name": "my-flow-project",
  "version": "1.0.0",
  "devDependencies": {
    "flow-bin": "^0.86.0"
  },
  "scripts": {
    "flow": "flow"
  }
}
```
首先运行
```
npm run flow init
```

在第一次运行了 `init` 后，运行

```
npm run flow
```

### `Yarn` 环境

把`npm`包安装到`devDependency`上

```
yarn add --dev flow-bin
```

首先运行

```
yarn run flow init
```

然后运行`Flow`

```
yarn run flow
```