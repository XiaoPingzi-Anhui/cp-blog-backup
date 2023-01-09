<!-- category: "javaScript"
labels: "javaScript"
createdAt: 2021-07-04T15:59:12.322+00:00 -->
js基础类型现在有7种，String，Number，BigInt,Boolean,null,undefined,Symbol，基础类型之间也可以相互转换，你可以手动调用String()、Number()、Boolean()进行转换，这是显式的，如果你把两个不同类型的变量进行运算，js也会帮你进行类型转换，这是隐式的。
## 1.转成String
最常用的就是String(),其他的方法还有例如.toString(),+''的方法，
对于基本类型来说，String()和+''都可以正常工作，但undefined、null中没有toString()方法。字符串转换也最明显。false 变成 "false"，null 变成 "null" 等。
## 2.转成Number
可以使用Number()、parseInt()或者parseFloat()，或者+
```typescript
let a = true, b = '1', c = '22,a';
+a; // 1
+b; // 1
+c; // NaN
```
number 类型转换规则：
1. undefined ->	NaN
2. null -> 0
3. true / false -> 0 / 1
4. string —> 去掉首尾空白字符（空格、换行符 \n、制表符 \t 等）后的纯数字字符串中含有的数字。如果剩余字符串为空，则转换结果为 0。否则，将会从剩余字符串中“读取”数字。当类型转换出现 error 时返回 NaN。
## 3.转成Boolean
比较简单，可以用Boolean(),或者!!()连续取反，规则如下：
1. 直观上为“空”的值（如 0、空字符串、null、undefined 和 NaN）将变为 false。
2. 其他值变成 true。
## 4.表达式中的隐式转化
如果+运算里有字符串，在运算到字符串后会转成字符串,其他情况如果有数字会转成数字，上面的+ '' 转字符串和 + 值转数字就是这个原理。
```typescript
2 * 3 + 4 + '2' + 1; // '1021' 前面正常运算，碰到+字符串时转成字符串
true + '3' * 2; // 7 true转成1，'3'转成3
```
总结：
1. 对 undefined 进行数字型转换时，输出结果为 NaN，而非 0。
2. 对 "0" 和只有空格的字符串（比如：" "）进行布尔型转换时，输出结果为 true。