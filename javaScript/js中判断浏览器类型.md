<!-- category: "javaScript"
labels: "javaScript,浏览器"
createdAt: 2022-08-18T15:54:26.014+00:00 -->
今天在看ahooks源码是看到了这样一个方法
```typescript
const isBrowser = !!(
  typeof window !== 'undefined' &&
  window.document &&
  window.document.createElement
);
```
这其实是用来检查当前代码是否运行在浏览器环境中的。但是我之前在别的地方看到的只是简单地对window对象做undefined检查，而不会同时检查document属性。一般来说也确实够用了。难道是为了避免有人错误地创建了window全局对象而影响了判断？有大佬知道的话请告诉我，不胜感激。

JavaScript是一种通用语言，不仅可以在浏览器中运行，还可以在Node.js中编写服务器端代码，在不同的上下文中，脚本的全局范围内有不同的对象可用。
1. 在浏览器中
  - Window（接口）暴露为window全局对象，这也是是全局范围的对象（因此在全局范围内声明var foo实际上会创建一个属性window.foo!)
  - document，document 全局对象实际上是在访问window.document 属性。
2. 在nodeJs编写的服务器中
  - process
  - exports
下面记录下不同浏览器的判别方法，虽然现在很少用到了
```typescript
const inBrowser = typeof window !== 'undefined'; // 检测是否是浏览器环境
const UA = inBrowser && window.navigator.userAgent.toLowerCase(); // 获取UserAgent
const isIE = UA && /msie|trident/.test(UA); // 是否是IE（谢天谢地，ie终于死了）
const isEdge = UA && UA.indexOf('edge/') > 0; // 是否是edge
const isAndroid = (UA && UA.indexOf('android') > 0) ; // 是否在安卓手机浏览器
const isIOS = (UA && /iphone|ipad|ipod|ios/.test(UA)); // 是否是ios浏览器
const isChrome = UA && /chrome\/\d+/.test(UA) && !isEdge; // 是否是chrome
const isFireFox = UA && UA.match(/firefox\/(\d+)/); // 是否是火狐浏览器
```