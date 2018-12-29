# 字面量类型
（Literal Types）

 > 把字面量值作为类型

`Flow`有许多用于字面量值的原始类型，但是，也可以使用字面量值作为类型。

例如，我们可以只接受字面量值`2`，来代替接受`number`类型。

```javascript
//@flow

function acceptsTwo(value:2){
	//...
}
acceptsTwo(2);   // Works!
// $ExpectError
acceptsTwo(3);   // Error!
// $ExpectError
acceptsTwo("2"); // Error!
```

你可以为这些类型使用原始值：

 - Booleans: 例如： `true` 或者`false`
 - Numbers: 例如： `42` 或者 `3.14`
 - Strings: 例如： `"foo"` 或者 `"bar"`

与联合类型（`union types`）一起使用功能更加强大：

```javascript
// @flow
function getColor(name: "success" | "warning" | "danger") {
  switch (name) {
    case "success" : return "green";
    case "warning" : return "yellow";
    case "danger"  : return "red";
  }
}

getColor("success"); // Works!
getColor("danger");  // Works!
// $ExpectError
getColor("error");   // Error!
```