<!-- category: "奇技淫巧"
labels: "javaScript"
createdAt: 2022-08-30T15:36:51.107+00:00 -->
在对字符串或者数组做元素存在性检查时，很多的代码都使用了indexOf()方法，但是再if里做判断时确比较麻烦往往需要这么写
```typescript
if(str.indexOf(targetStr) !== -1)
```
有一个小技巧可以这样写
```typescript
if(~str.indexOf(targetStr))
```
效果是一样的。因为对于 32-bit 整数，~n 等于 -(n+1)。
但是！使用indexOf()判断是比较老派的做法了，现在推荐直接使用includes()方法，直接返回布尔值，字符串、数组都支持，不香吗？