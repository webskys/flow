# 深度子类型

假设我们有两个类，它们具有子类型关系：

```javascript
class Person { name: string }
class Employee extends Person { department: string }
```

在期望使用`Person`实例的地方使用`Employee`实例是有效的。

```javascript
// @flow
class Person { name: string }
class Employee extends Person { department: string }

var employee: Employee = new Employee;
var person: Person = employee; // OK
```

然而，在期望使用包含一个`Person`实例的对象的地方使用一个包含`Employee`实例的对象是无效的。

```javascript
// @flow
class Person { name: string }
class Employee extends Person { department: string }

var employee: { who: Employee } = { who: new Employee };
// $ExpectError
var person: { who: Person } = employee; // Error
```

这是错误的，因为对象是可变的。通过`employee`变量引用的值跟通过`person`变量引用的值是相同的

```javascript
person.who = new Person;
```

如果我们编写`person`对象的`who`属性，我们也改变了`employee.who`的值，该值己被明确地注解为`Employee`实例。

如果我们阻止任何代码通过`person`变量来向对象写入新值，那么使用`employee`变量就是安全的了。`Flow`为此提供了一种语法：

```javascript
// @flow
class Person { name: string }
class Employee extends Person { department: string }

var employee: { who: Employee } = { who: new Employee };
var person: { +who: Person } = employee; // OK
// $ExpectError
person.who = new Person; // Error!
```

这个`+`表示`who`属性是`协变的`,协变属性允许我们使用兼容该属性子类型值的对象，默认情况下，对象属性是不变的，它允许读和写，但是对它们接受的值有很大的限制。