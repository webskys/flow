# 入门指南
（Getting Started）
 > `Flow`里类型检测介绍

`Flow` 是`JavaScript` 代码的静态类型检测器，它做了大量的工作使用你能更高效的工作。使你再快、更智能、更自信和更大规模地编写代码

`Flow`通过静态类型注解（`static type annotations`）检测你代码中的错误，这些类型（`types`）允许你告诉`Flow`你希望代码如何工作，`Flow`将确保它以这种方式工作。

```javascript
// @flow
function square(n: number): number {
  return n * n;
}

square("2"); // Error!
```
因为`Flow` 能很好的理解 `JavaScript`,所以这里这需要这么多的类型。你只需要为`Flow`做很少的工作，剩下的`Flow`将自行推断。很多时候，在根本没有任何类型时，`Flow`也能理解你代码。

```javascript
// @flow
function square(n) {
  return n * n; // Error!
}

square("2");
```
你可以递增地采用`Flow`，并且可以在任何时候轻松移除它 `Flow`类型检测，因此，你可以在任何代码里尝试`Flow`，看看你有多喜欢它。