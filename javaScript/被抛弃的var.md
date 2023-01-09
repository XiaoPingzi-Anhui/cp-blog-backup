<!-- category: "javaScript"
labels: "javaScript"
createdAt: 2021-04-16T20:07:03.889+00:00 -->
js定义变量主要有var、let、const三种方式，我刚开始进行前端工作时是20年左右，那个时候var就已经不被待见了，我也很少使用，但我总是能在网上各种地方碰到它，既然绕不过去就还是好好学学吧，常用的let与const就不介绍了，着重说一下var的特点。注意，以下的代码都是在非严格模式下运行的
## 1.var的变量提升
在一般的语言中，要使用一个变量，应该先去声明定义它，但是js可不一定，使用var定义的变量，可以在定义之前使用。包括函数
```typescript
console.log(a); // undefined
var a = 1;
console.log(a); // 1

func(); // 调用了
function func() {
  console.log("调用了");
}
```
这样的代码真是不符合习惯，那为什么js能这样做呢？因为js引擎的工作方式是，先预解析代码，获取所有被声明的变量和函数声明，然后再一行一行地运行，这就使所有变量声明的语句，都会被提升到代码的头部，这就是变量提升。
对于上面的代码来说，真正的执行其实是这样的
```typescript
var a ;
console.log(a); // undefined
a = 1;
console.log(a); // 1
```
a变量被提升到顶部，但一开始虽然声明了却没有赋值，此时的值是undefined。
## 2.var的作用域
var也没有自己的块作用域，对于var来说只有全局作用域和函数作用域。这是因为在早期的js中，块没有词法环境，后面我会专门写一篇文章介绍词法环境。
```typescript
if (true) {
  var a = 1; // 虽然在if代码块中定义，但其实是一个全局变量
}
console.log(a); // 1，if代码块定义的也能被访问到

function func(params) {
  var b = 2; // 函数内的在外部无法访问
  console.log(b); // 2
}
func();
console.log(b); // 'b' is not defined.
```
## 3.var允许重新声明
习惯来说，一个变量被定义之后可以被修改，但不能重复定义，不过var可以。不会报错
```typescript
var a = 1;
var a = 2;
console.log(a); // 2
```
## 4.var与window
在浏览器环境中，如果你在用var声明了一个变量a，那么这个变量其实是被放到了window对象中，可以通过window.a访问
```typescript
var a = 1;
console.log(window.a); // 1
```
以上都是var的一些特点，相比之下，let和const就让人舒服很多了，主要还是var的作用域问题，太不符合一般的语言习惯了！