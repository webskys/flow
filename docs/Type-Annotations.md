# 类型注解
（Type Annotations）

 > 学习如何在你的代码里添加类型注解

添加类型注解是你跟 `Flow` 交互的一个重要组成部分。

`Flow` 有强大的能力去推断你程序的类型，你的大多数代码都可以信赖它。不过，有些地方你需要添加类型。

假设下面连接两个字符串的函数 `concat`：

```javascript
function concat(a, b) {
  return a + b;
}
```

当你使用这个函数，`Flow` 准确知道要发生什么。

```javascript
concat("A", "B"); // Works!
```

然而，你可以在字符串或者数值上使用加号（`+`）操作符，所以下面这也是有效的。

```javascript
concat(1, 2); // Works!
```

但是假如你仅允许在函数使用字符串，为此，你可以添加类型。

```javascript
function concat(a: string, b: string) {
  return a + b;
}
```

现在如果你试图使用数值，你将从`Flow`里得到一个警告。

```javascript
// @flow
function concat(a: string, b: string) {
  return a + b;
}

concat("A", "B"); // Works!
concat(1, 2); // Error!
```

用类型设置"限制"的意思是，你可以告诉 `Flow`在其己经推断之上你的期望。