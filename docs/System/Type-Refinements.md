# 类型细化

细化在任何类型系统中都得到频繁地使用，它们在我们编程中根深蒂固，以至于我们基本没有注意到它。

在下面的代码中，值即可以是`A`也可以是`B`。

```javascript
// @flow
function method(value: "A" | "B") {
  if (value === "A") {
    // value is "A"
  }
}
```

在`if`块里，我们知道值必须是`A`，因为只有这个时候`if`语名为真。

静态类型检查器能够判断`if`语句中的值必须是“A”的能力称为细化。

下面我们添加另一个到`if`语句中。

```javascript
// @flow
function method(value: "A" | "B") {
  if (value === "A") {
    // value is "A"
  } else {
    // value is "B"
  }
}
```

在另一个块中，我们知道值肯定是`B`，因为函数只允许`A`或`B`并且己经移除了可能是`A`的情况。

您可以进一步扩展这个范围，并保持细化的可能性：

```javascript
// @flow
function method(value: "A" | "B" | "C" | "D") {
  if (value === "A") {
    // value is "A"
  } else if (value === "B") {
    // value is "B"
  } else if (value === "C") {
    // value is "C"
  } else {
    // value is "D"
  }
}
```

除了相等性测试之外，细化还可以以其他形式出现：

```javascript
// @flow
function method(value: boolean | Array<string> | Event) {
  if (typeof value === "boolean") {
    // value is a boolean
  } else if (Array.isArray(value)) {
    // value is an Array
  } else if (value instanceof Event) {
    // value is an Event
  }
}
```

或者，我们可以对对象的形态进行细化。

```javascript
// @flow
type A = { type: "A" };
type B = { type: "B" };

function method(value: A | B) {
  if (value.type === "A") {
    // value is A
  } else {
    // value is B
  }
}
```

也可以应用在对象的嵌套类型。

```javascript
// @flow
function method(value: { prop?: string }) {
  if (value.prop) {
    value.prop.charAt(0);
  }
}
```

## 无效细化

细化也有可能无效，例如：

```javascript
// @flow
function otherMethod() { /* ... */ }

function method(value: { prop?: string }) {
  if (value.prop) {
    otherMethod();
    // $ExpectError
    value.prop.charAt(0);
  }
}
```
原因在于,我们不知道`otherMethod()`是否对我们的值做过什么，假设下面的情景：

```javascript
// @flow
var obj = { prop: "test" };

function otherMethod() {
  if (Math.random() > 0.5) {
    delete obj.prop;
  }
}

function method(value: { prop?: string }) {
  if (value.prop) {
    otherMethod();
    // $ExpectError
    value.prop.charAt(0);
  }
}

method(obj);
```

在`otherMethod()`内部，我们有时移除了`prop`。`Flow`不知道`if (value.prop)`检测是否一直为`true`,因此它是无效细化。

有一个简单的方法可以解决这个问题。在调用另一方法之前存储这个值并用这个存储的值代操作。这种方法可以预防细化无效。

```javascript
// @flow
function otherMethod() { /* ... */ }

function method(value: { prop?: string }) {
  if (value.prop) {
    var prop = value.prop;
    otherMethod();
    prop.charAt(0);
  }
}
```