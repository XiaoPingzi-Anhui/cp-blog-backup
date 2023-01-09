<!-- category: "typeScript"
labels: "typeScript"
createdAt: 2022-09-24T14:56:49.019+00:00 -->
本文对tsconfig.json的配置信息做一个归纳整理，方便后续学习查阅，完整的可以在[官网](https://www.tslang.cn/docs/handbook/compiler-options.html)查看，这里只对一部分做介绍，后面也会继续学习补充。
## 1. files
该字段是用来指定哪些文件需要tsc编译，例如：
```json
{
  "files": ["index.ts", "global.d.ts"],
}
```
对于一般的前端工程来说，该字段不需要单独配置
## 2. include
该字段是用来指定哪些文件或者文件夹需要tsc编译，例如： 
```json
{
  "include": ["src", "global.d.ts"],
}
```
对于一般的前端工程来说，直接配置`["src"]`,编译整个src文件夹就可以了。同时也支持glob通配符：
- *匹配0或多个字符（不包括目录分隔符）
- ?匹配一个任意字符（不包括目录分隔符）
- **/递归匹配任意子目录
## 3. exclude
该字段是用来排除不需要tsc编译的文件或者文件夹，例如： 
```json
{
  "exclude": ["src/test.ts"],
}
```
注意： exclude字段中的声明只对include字段有排除效果，对files字段无影响（要排除就别在files加呗，不然岂不是脱裤子放屁？），即与 include 字段中的值互斥。如果 tsconfig.json文件中files和include字段都不存在，则默认包含tsconfig.json文件所在目录及子目录的所有文件，且排除在exclude字段中声明的文件或文件夹。
## 4. extends
该字段是用来继承已有的tsconfig配置规则文件，很有用，因为其实各个项目之间的配置大同小异，重复的很多，完全可以抽一个公共的配置文件并发包，然后在各个项目里继承。官方就有一个推荐的默认配置[@tsconfig/recommended](https://www.npmjs.com/package/@tsconfig/recommended),对于继承的配置，可以重写覆盖的。
## 5. compileOnSave
该字段是声明是否需要在保存时候自动触发tsc编译，一般来说，我们的代码编译过程会通过Rollup、Webpack等打包构建工具，并且使用热更新，因此无需配置该项，保持缺省或者配成false即可。 
## 6. references
该字段作用是指定工程引用依赖。在项目开发中，有时候我们为了方便将前端项目和后端node项目放在同一个目录下开发，两个项目依赖同一个配置文件和通用文件，但我们希望前后端项目进行灵活的分别打包，那么我们可以进行如下配置：
```json
{
  "references": [ // 指定依赖的工程
     {"path": "./common"}
  ]
}
```
## 7. compilerOptions
该字段是一个对象，有很多子元素配置,很多字段的作用是显而易见的，就不详细介绍了，下面只介绍一部分
### 7.1. lib
该字段用于为了在我们的代码中显示的指明需要支持的ECMAScript语法或环境对应的类型声明文件，什么意思？例如我们的代码会使用到浏览器中的一些对象window、document，这些全局对象API对于tsc来说是不能识别的,所以需要在lib字段中如下配置：
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
  }
}
```
### 7.2. module
该字段指明tsc编译后的代码应该符合何种“模块化方案”，可以指定的枚举值有：none, commonjs, amd, system, umd, es2015, es2020, 或 ESNext，默认值为 none。在如今的前端开发趋势来讲，主要是使用ESM、CommonJS、UMD、IIFE四种模块化方案，未来会趋向于ESM，当然我们会根据项目的应用场景来决定使用何种模块化方案，例如：NodeJS 使用CommonJS，浏览器里可以使用 ESM，不过现在的打包工具，会自动处理 CommonJS 和 ESM 的差异，并包装成符合指定模块化规范的代码。
### 7.3. moduleResolution
该字段指明编译器在查找导入模块内容时所遵循的流程，也就是[模块解析策略](https://www.chenping.space/blogDetail/6386fa49b69d9a11f5a382f9)。有两种取值：Node、Classic。我们一般用Node。若未指定，那么在使用了--module AMD | System | ES2015时的默认值为Classic，其它情况时则为Node。
### 7.4. allowSyntheticDefaultImports
当一个文件没有默认导出时，如果使用`import React from "react";`这样的导入语句，ts会报错，应该使用`import * as React from "react";`。虽然其实这样是可以正常工作的，因为为了使用方便，Babel 这样的转译器会在没有默认导出时自动为其创建，这更方便开发者。为了ts不报错。当allowSyntheticDefaultImports 设置为true 的时候，并且模块没有显式指定默认导出时，允许使用：`import React from "react";`用来替代:`import * as React from "react";`。
### 7.5. esModuleInterop
默认为false，开启了true后，allowSyntheticDefaultImports也会被设为true。他的作用和allowSyntheticDefaultImports类似，并多做了一些事。由上一点我们知道，allowSyntheticDefaultImports仅对类型检查起作用，不影响ts编译之后的结果。而esModuleInterop同时也会影响编译结果。

默认情况下，ts像ES6模块一样对待CommonJS/AMD/UMD。这样的行为有两个被证实的缺陷：
- 形如 import * as moment from "moment" 这样的命名空间导入等价于 const moment = require("moment")
- 形如 import moment from "moment" 这样的默认导入等价于 const moment = require("moment").default(没有默认导出不就报错了)

这种错误的行为导致了这两个问题：
- ES6 模块规范规定，命名空间导入（import * as x）只能是一个对象。TypeScript 把它处理成 = require("x") 的行为允许把导入当作一个可调用的函数，这样不符合规范。
- 虽然 TypeScript 准确实现了 ES6 模块规范，但是大多数使用 CommonJS/AMD/UMD 模块的库并没有像 TypeScript 那样严格遵守。
开启esModuleInterop选项将会修复 TypeScript 转译中的这两个问题。
### 7.6. preserveConstEnums
在默认情况下，对于使用了const修饰符的枚举，编译后，不会生成映射代码。如下：
```typescript
/* 源码 */
const enum RequestMethod {
  Get,
  Post,
  Put,
  Delete
}
let methods = [
  RequestMethod.Get,
  RequestMethod.Post
]

/* 编译后 */
"use strict";
let methods = [
    0 /* Get */,
    1 /* Post */
];
```
如果希望生成映射代码时，可以将preserveConstEnums设为true,上面的源码编译后就会变成这样：
```typescript
"use strict";
var RequestMethod;
(function (RequestMethod) {
    RequestMethod[RequestMethod["Get"] = 0] = "Get";
    RequestMethod[RequestMethod["Post"] = 1] = "Post";
    RequestMethod[RequestMethod["Put"] = 2] = "Put";
    RequestMethod[RequestMethod["Delete"] = 3] = "Delete";
})(RequestMethod || (RequestMethod = {}));
let methods = [
    0 /* Get */,
    1 /* Post */
];
```
一般情况下我们并不需要开启，不开启生成的结果还更简洁，节省开销。
### 7.7. isolateModules
推荐开启，直接看之前的文章[ts的isolateModules配置](https://www.chenping.space/blogDetail/63871e6cb69d9a11f5a382fa)