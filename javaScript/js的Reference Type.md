<!-- category: "javaScript"
labels: "javaScript"
createdAt: 2023-01-06T11:16:22.832+00:00 -->
让我们看一下下面的代码：
```ts
const people = {
  name: "Cai Gou",
  getName() {
    console.log(this.name);
  },
};

people.getName(); // Cai Gou

const getName = people.getName;
getName(); // undefined

setTimeout(people.getName, 1000); // undefined

setTimeout(() => people.getName(), 1000); // Cai Gou
```
嗯……为什么会出现这样的情况呢？每当 people.getName 和 () 分开后，this.name 都会变成 undefined，如果你在 getName 内直接打印 this,就会发现 this 竟然变成了 window 对象（非严格模式下，严格模式 this 是undefined, this.name 都会直接报错！），要搞清楚这个，就要深入 obj.method() 调用运行的本质。

obj.method() 语句中的其实有两个操作：
1. 首先，点 '.' 取了属性 obj.method 的值。
2. 接着 () 执行了它。

那么，this 的信息是怎么从第一部分传递到第二部分的呢？

其实，点 '.' 返回的不是一个函数，而是一个特殊的 Reference Type 的值。Reference Type 是 ECMA 中的一个“规范类型”。我们不能直接使用它，但它被用在 JavaScript 语言内部。

Reference Type 的值是一个三个值的组合 (base, name, strict)，其中：
* base 是对象。
* name 是属性名。
* strict 在 use strict 模式下为 true。
  
对属性 people.getName 访问的结果不是一个函数，而是一个 Reference Type 的值。对于 people.getName，其实是 (people, 'getName', false)。

当 () 被在 Reference Type 上调用时，它们会接收到关于对象和对象的方法的完整信息，然后可以设置正确的 this（在此处 = people）。Reference Type 是一个特殊的“中间人”内部类型，目的是从 . 传递信息给 () 调用。任何例如赋值 getName = people.getName 等其他的操作，都会将 Reference Type 作为一个整体丢弃掉，而会取 people.getName（一个函数）的值并继续传递。所以任何后续操作都“丢失”了 this。

因此，this 的值仅在函数直接被通过点符号 obj.method() 或方括号 obj['method'] 语法（此处它们作用相同）调用时才被正确传递。

原因找到了，那么怎么解决呢？像这样将一个对象方法传递到别的地方调用的需求其实还是挺常见的。最简单的方法就是用一个函数包装一下，就像上面代码的最后一行一样，是可以正常获取 this 的，但是有一个问题，如果在 setTimeout 触发之前（有一秒的延迟！）people 的值改变了怎么办？
```ts
const people = {
  name: "Cai Gou",
  getName() {
    console.log(this.name);
  },
};

setTimeout(() => people.getName(), 1000); // 我被修改了
people.getName = () => console.log('我被修改了');
```
取到的值也是更新后的值，但其实往往不是我们期望的。

还有一种方法，就是用 bind 方法手动绑定 this:
```ts
const people = {
  name: "Cai Gou",
  getName() {
    console.log(this.name);
  },
};

setTimeout(people.getName.bind(people), 1000); // Xiu Gou
people.getName = () => console.log('我被修改了');
```