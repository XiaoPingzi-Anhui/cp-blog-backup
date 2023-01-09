<!-- category: "javaScript"
labels: "javaScript"
createdAt: 2021-06-11T09:02:03.015+00:00 -->
工作中经常要进行值的判等操作，下面介绍几种判等方法
## 1. 等于运算符（==）
工作中的代码里如果出现了 == 判等，可能会被吊锤吧。js是弱类型语言，各种变量的类型不好把握，就容易出bug，这也是为啥现在typeScript为啥这么火的主要原因，而 == 会尝试强制类型转换并且比较不同类型的操作数。这往往不是我们想要的结果，例如：
```typescript
'1' ==  1; // true,当数字与字符串进行比较时，会尝试将字符串转换为数字值
0 == false; // true,如果操作数之一是Boolean，则将布尔操作数转换为 1 或 0
null == undefined; // true,如果一个操作数是null，另一个操作数是undefined，则返回true
undefined == 0; // false,相等性检查时， null 和 undefined 不会转换成其他值。
NaN == NaN; // false,若某个运算数是 NaN，永远返回 false，
const number = new Number(3);
number == 3; // true, 如果操作数之一是对象，另一个是数字或字符串，会尝试使用对象的valueOf()和toString()方法将对象转换为原始值。
{} == {}; // false, 对象之间是浅比较，只看引用是否相等
```
可以看到，这些不同类型直接的判等结果往往不是我们想要的，所以大部分情况下我们并不推荐 == 判等方法
## 2. 全等运算符（===）
这应该工作中使用最多的一种方式了，可以满足绝大部分的需求。与==最明显的区别在于===不尝试类型转换。相反，===始终将不同类型的操作数视为不同。对象之间也是浅比较。
## 3. Object.is()
Object.is()与===差别是它对待有符号的零和NaN不同，例如，=== 运算符（也包括==运算符将数字-0和+0视为相等，而将Number.NaN与 NaN视为不相等。所以除了这两种情况以外，还是推荐使用===
```typescript
Object.is(0, -0); // false
Object.is(+0, -0); // false
Object.is(-0, -0); // true
Object.is(0n, -0n); // true
Object.is(NaN, 0/0); // true
Object.is(NaN, Number.NaN) // true
```
## 4. Same Value Zero
这是啥意思呢，其实就是一些检测值的api，例如数组的includes，map、set的has，这些api进行值比较时，也不会进行类型转换，不区分+-0，但是null，undefined，NaN与自身直接比较会认为相等！！！
```typescript
const array = [0,NaN,null,undefined];
array.includes(-0); // true
array.includes(NaN); // true
array.includes(null); // true
array.includes(undefined); // true

const set = new Set();
set.add(0).add(NaN).add(null).add(undefined);
set.has(-0); // true
set.has(NaN); // true
set.has(null); // true
set.has(undefined); // true

const map = new Map();
map.set(0, 1).set(NaN, 2).set(null, 3).set(undefined, 4);
console.log(map.has(-0)); // true
console.log(map.has(NaN)); // true
console.log(map.has(null)); // true
console.log(map.has(undefined)); // true
```