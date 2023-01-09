<!-- category: "javaScript"
labels: "javaScript"
createdAt: 2022-07-18T14:28:51.107+00:00 -->
闭包的定义，是指一个函数可以记住其外部变量并可以访问这些变量。对于我这种java转前端的人来说，一开始确实很难理解闭包，直到我看了这篇[文章](https://zh.javascript.info/closure)，才发现原来js内部做了很多我们看不见的处理。要理解闭包，得先来看看词法环境。
## 1.词法环境
在js中，无论是全局的脚本、代码块{...}还是函数，都有一个对应的特殊隐藏对象，叫做词法环境。全局脚本的词法环境又叫全局词法环境。可以理解为，每一个代码块，js都帮我们维护了一个与之对应的看不见的对象，既然是对象，那么肯定有值。词法环境主要存了下面两部分内容
1.  环境记录（Environment Record） —— 一个存储所有局部变量作为其属性（包括一些其他信息，例如 this 的值）的对象。
2. 对外部词法环境的引用，与外部代码相关联。
一个“变量”只是 环境记录 这个特殊的内部对象的一个属性。“获取或修改变量”意味着“获取或修改词法环境的一个属性”。而对外部词法环境的引用我们可以理解为类似于单向链表那样的东西，可以通过引用，一层一层访问外部的词法环境。当代码要访问一个变量时 —— 首先会搜索内部词法环境，然后通过引用搜索外部环境，然后搜索更外部的环境，以此类推，直到全局词法环境。如果在任何地方都找不到这个变量，那么在严格模式下就会报错（在非严格模式下，为了向下兼容，给未定义的变量赋值会创建一个全局变量）。
## 2.闭包
理解了词法环境，再来看一个闭包的例子:
```typescript
function makeCounter() {
  let count = 0;
  return function() {
    return count++;
  };
}
let counter1 = makeCounter();
let counter2 = makeCounter();
console.log( counter1() ); // 0
console.log( counter1() ); // 1
console.log( counter1() ); // 2
console.log( counter2() ); // 0
console.log( counter2() ); // 1
```
函数makeCounter有个内部变量count，并返回了一个函数，对count进行+1操作。为什么我们每次执行counter时，makeCounter里的count都会被记住呢？因为当执行let counter1 = makeCounter()时，js创建了一个对应的词法环境makeCounter1，count变量就被保存在里面了,而当执行counter1时，js也会创建一个词法环境，并且引用是指向makeCounter1的，counter1里读取的count就来自与makeCounter1里。每次执行counter1，都会创建新的对应词法环境，但是！其对外部词法环境的引用都是同一个，也就是makeCounter1，每次count++都是对makeCounter1里的count进行操作，所以每次counter执行都能‘记住’上一次的结果。

那counter2为啥又记不住counter1的操作结果呢？因为每次函数执行都会创建新的词法环境。let counter1 = makeCounter()和let counter2 = makeCounter()这两条语句，其实是创建了两个词法环境，执行counter1和counter2时，其外部引用根本不是同一个词法环境，那count值当然不会同步！