# 任意类型
（Any Types）

 > 使用`any`跳出类型系统，注意，不要误解`mixed`和`any`。

使用`any`从类型检测中跳出来，不要把 `any` 和 `mixed` 混淆了。

如果你想一种方法来跳出使用类型检测器，`any`可以做到这一点。`any`是完全不安全的，尽量避免使用。

例如，下面的代码不会报告任何错误：

```javascript
//@flow
function add(one:any,two:any): number {
	return one + two;
}
add(1, 2);     // Works.
add("1", "2"); // Works.
add({}, []);   // Works.
```

甚至将导致运行时错误的代码，也不会被`Flow`捕捉到：

```javascript
// @flow
function getNestedProperty(obj: any) {
  return obj.foo.bar.baz;
}

getNestedProperty({});
```

这里只有几个情景你可能考虑使用`any`：

 - 当你转换现有代码使用`Flow`类型并且代码使用类型检测发生了阻塞时。
 - 当你确定代码能运行，但因为一些原因`Flow`不能正确检测类型时。`Flow`无法静态化类`Javascript`中一些惯用语。

## 避免泄漏`any`
（Avoid leaking any）

当你有一个类型为`any`的值时，你可能会导致`Flow`推断你所执行的所有操作结果也是`any`类型。

例如，如果你读取一个类型为`any`的对象的属性,结果值也是一个`any`类型。

```javascript
//@flow
function fn(obj: any){
	let foo /* (:any) */ = obj.foo;
}
```
你可以稍后在其他操作中使用这个结果，比如如果这是一个数值把它加起来，这样结果也将是`any`类型。

```javascript
// @flow
function fn(obj: any) {
  let foo /* (:any) */ = obj.foo;
  let bar /* (:any) */ = foo * 2;
}
```

你可以继续这个过程直到`any`的泄漏你所有代码。

```javascript
// @flow
function fn(obj: any) /* (:any) */ {
  let foo /* (:any) */ = obj.foo;
  let bar /* (:any) */ = foo * 2;
  return bar;
}

let bar /* (:any) */ = fn({ foo: 2 });
let baz /* (:any) */ = "baz:" + bar;
```

预防这种情况发生，可以尽快将其分配到另一个类型，来将`any`切断。

```javascript
// @flow
function fn(obj: any) {
  let foo: number = obj.foo;
}
```

现在你的代码不会泄漏 `any`。

```javascript
// @flow
function fn(obj: any) /* (:number) */ {
  let foo: number = obj.foo;
  let bar /* (:number) */ = foo * 2;
  return bar;
}

let bar /* (:number) */ = fn({ foo: 2 });
let baz /* (:string) */ = "baz:" + bar;
```
