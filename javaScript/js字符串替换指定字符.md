<!-- category: "javaScript"
labels: "javaScript"
createdAt: 2022-05-31T15:49:49.019+00:00 -->
替换字符串指定字符的需求经常能遇到，最常见的应该是全部替换，随后是替换第一个，或者指定第几个的。下面介绍几种方法。

## 1.replaceAll
最常见的需求，替换所有的指定字符，这种情况用replaceAll方法应该是最爽的，但是呢，这个方法兼容性很差，你永远也不知道测试、ui、产品小姐姐们到底用的啥浏览器看的，我就被提过这个兼容性的bug，之后就没再用过这方法。这里我们也不做过多介绍。
## 2.split+join
这个方法思路还是比较讨巧的，主要分为两步：
1. 根据指定字符，用split将整个字符串分为一个个小段。
2. 再用join将一个个小段用替换的字符连接在一起。
下面举个小栗子：
```typescript
let str = '我是一个原始的字符串，一个帅气的字符串，嘻嘻'; // 需求：把该字符串的所有，替换成！
const replaceTarget = ',';
const replaceWith = '!';
const result = str.split(replaceTarget).join(replaceWith);
```
如果是要替换指定位置的，顺着这个思路，也不是很麻烦
```typescript
/**
* 把str字符串的指定位置的指定字符串替换掉
* @param str 原始字符串
* @param replaceTarget 需要替换掉的字符串
* @param replaceWith 用来代替的字符串
* @param replaceIndex 替换第几个位置，不传为全部替换,从0开始计数
* @returns 替换后的字符串
*/
const replaceStr = (str: string, replaceTarget: string, replaceWith: string, replaceIndex?: number) => {
  const splitAry = str.split(replaceTarget);
  return splitAry
    .reduce((pre, cur, i) => {
      pre.push(cur);
      if (i === replaceIndex) {
        pre.push(replaceWith);
      } else if (i !== splitAry.length - 1) {
        pre.push(replaceIndex === undefined ? replaceWith : replaceTarget);
      }
      return pre;
    }, [] as string[])
    .join('');
};
```
## 3.replace
好了，接下来就要介绍我们的重头戏了,replace方法是真的强大，用法str.replace(regexp|subStr, newSubStr|function)。下面就介绍我碰到的几种需求
```typescript
// 将第一次匹配的'a'换成'b'
'abcdaA'.replace('a', 'b'); // bbcdaA

// 将全部的'a'换成'b'
'abcdaA'.replace(/a/g, 'b'); // bbcdbA

// 将所有的'a' 或者'A'换成'b'
'abcdaA'.replace(/a/gi, 'b'); // bbcdbb

// 将'1234567890'加上千分位
'1234567890'.replace(/\B(?=(\d{3})+(?!\d))/g, ',') // '1,234,567,890'

// 将所有的'高'，'矮'，'胖'，'瘦'都换成'好'
'我高高的，矮矮的，胖胖的，瘦瘦的'.replace(/(高|矮|胖|瘦)/g, '好'); // 我好好的，好好的，好好的，好好的


// 当第二个参数replaceValue是一个字符串、第一个参数searchValue是一个正则，且含有分组，那么如果replaceValue里面带有‘$’,则会有特殊含义，其中用的最多就是$number，如$1(分组1捕获的文本)、$2(分组2捕获的文本)
// 将'2022-05-31'这串字符串替换成'05/31 2022'
'2022-05-31'.replace(/(\d{4})-(\d{2})-(\d{2})/g, '$2/$3 $1');

// 第二个参数还可以是一个函数，操作更加灵活方便
// 将'2022-05-31 2022-04-30 2022-03-31'换成'05/31 20xx 04/30 20xx 02/31 20xx',
'2022-05-31 2022-04-30 2022-03-31'.replace(/(\d{4})-(\d{2})-(\d{2})/g, (all, p1, p2, p3) => `${p2}/${p3} ${p1}`)
```
replace的强大主要还是源于正则的灵活强大，能干的事太多了，后面还需要补一下正则的知识呀