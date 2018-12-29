# 类型变异

变异是一个在类型系统中经常出现的话题，并且当你第一次听到它可能会有点混乱。让遍历下知种形式的变异。

首先，我们将设置两个彼此扩展的类。

```javascript
class Noun {}
class City extends Noun {}
class SanFrancisco extends City {}
```

我们将使用这些类来编写一个拥有各种变异类型的方法

 > 注意: `*variantOf`类型不是`Flow`的一部分，它们用于解释变异。

## Invariance（不变式）

```javascript
function method(value: InvariantOf<City>) {...}

method(new Noun());         // error...
method(new City());         // okay
method(new SanFrancisco()); // error...
```
 - 不变式不接受父类型。
 - 不变式不接受子类型。

## Covariance（协变）

```javascript
function method(value: CovariantOf<City>) {...}

method(new Noun());         // error...
method(new City());         // okay
method(new SanFrancisco()); // okay
```

 - 协变不接受父类型。
 - 协变接受子类型。

## Contravariance（逆变）

```javascript
function method(value: ContravariantOf<City>) {...}

method(new Noun());         // okay
method(new City());         // okay
method(new SanFrancisco()); // error...
```

 - 逆变接受父类型。
 - 逆变不接受子类型。

## Bivariance（双变）

```javascript
function method(value: BivariantOf<City>) {...}
method(new Noun());         // okay
method(new City());         // okay
method(new SanFrancisco()); // okay
```

 - 双变接受父类型。
 - 双变接受子类型。

由于具有弱动态类型，JavaScript没有任何这些类型，因此您可以在任何时候使用任何类型。

## 对象中的变异

为了了解不同类型的变异什么时候和为什么重要，让我们讨论一下子类的方法以及如何类型检测。

我们能快速地设置`BaseClass`，它仅有一个方法，这个方法授受一个`City`类型的输入值并返回一个`City`类型的输出。

```javascript
class BaseClass {
  method(value: City): City { ... }
}
```

现在，让我们浏览几个不同的子类中的`method()`方法的不同定义。

### 同样具体的输入和输出— Good

开始，我们可以定义一个扩展`BaseClass`的子类，在这里，您可以看到，值和返回类型都是City，就像BaseClass中一样：

```javascript
class SubClass extends BaseClass {
  method(value: City): City { ... }
}
```

这样是可以的，因为如果你程序中的其它东西使用`SubClass`跟使用`BaseClass`一样，那么它仍然使用`City`，并且不会引起任何问题。

### 更具体的输出— Good

接下来，我们有一个不同的`SubClass`，它返回一个更具体的类型。

```javascript
class SubClass extends BaseClass {
  method(value: City): SanFrancisco { ... }
}
```

这样也是可以的，因为如果某些东西像使用`BaseClass`一样使用`SubClass`，那么它们仍然可以访问与以前相同的接口，因为`SanFrancisco`只是一个小且更多信息`City`。

### 不具体的输出— Bad

接下来，我们有一个返回不太具体类型的`SubClass`:

```javascript
class SubClass extends BaseClass {
  method(value: City): Noun { ... } // ERROR!!
}
```

在`Flow`里，这将导致一个错误，因为如果你期望得到一个`City`的返回值，你可能正在使用`Noun`上不存在的东西，这在运行时很容易导致错误。

### 不具体的输入 — Good

接下来，我们有另一个`SubClass`,其接受一个不太具体类型的值。

```javascript
class SubClass extends BaseClass {
  method(value: Noun): City { ... }
}
```

这非常好，因为如果我们传入一个更具体的类型，我们仍然拥有所有需要与`Noun`兼容的信息。

### 更具体的输入 — Bad

最后，我们还有一个另一个`SubClass`,它接受一个更具体类型的值。

```javascript
class SubClass extends BaseClass {
  method(value: SanFrancisco): City { ... } // ERROR!!
}
```

在`Flow`里，这是一种错误，因为你期望一个`SanFrancisco`并且你得到一个`City`，那么你可能正在使用仅存在于`SanFrancisco`上的一些东西，这会在运行时导致错误。

-----

所有这些表明为什么`Flow`有逆变输入（接受传入不具体的类型）、协变输出（允许返回更具体的类型）。
