<!-- category: "typeScript"
labels: "typeScript,node"
createdAt: 2022-06-11T16:23:49.019+00:00 -->
模块解析是指编译器在查找导入模块内容时所遵循的流程。假设有一个导入语句 `import { a } from "moduleA";` 编译器需要准确的知道a表示什么，并且需要检查它的定义moduleA到底在哪里。这听上去很简单，但moduleA可能在你写的某个.ts/.tsx文件里或者在你的代码所依赖的.d.ts里。

目前共有两种策略告诉编译器到哪里去查找moduleA。Classic和Node。现在的前端项目里大多用的是Node.
## 1. 相对和非相对模块导入
相对导入是以/，./或../开头的。 例如：
- `import Entry from "./components/Entry";`
- `import { DefaultHeaders } from "../constants/http";`
- `import "/mod";`

所有其它形式的导入被当作非相对的。 例如：：

- `import * as $ from "jQuery";`
- `import { Component } from "@angular/core";`

根据模块引用是相对的还是非相对的，模块导入会以不同的方式解析。
## 2. Classic
这种策略在以前是TypeScript默认的解析策略。 现在很少用了，它存在的理由主要是为了向后兼容。
## 2.1. Classic的相对导入
相对导入的模块是相对于导入它的文件进行解析的。 因此对于`/root/src/folder/A.ts`文件里的`import { b } from "./moduleB"`会使用下面的查找流程：
1. /root/src/folder/moduleB.ts
2. /root/src/folder/moduleB.d.ts
## 2.2. Classic的非相对导入
对于非相对模块的导入，编译器则会从包含导入文件的目录开始依次向上级目录遍历，尝试定位匹配的声明文件，例如在`/root/src/folder/A.ts`文件里的`import { b } from "moduleB"`,其查找流程是这样的：
1. /root/src/folder/moduleB.ts
2. /root/src/folder/moduleB.d.ts
3. /root/src/moduleB.ts
4. /root/src/moduleB.d.ts
5. /root/moduleB.ts
6. /root/moduleB.d.ts
7. /moduleB.ts
8. /moduleB.d.ts
   
## 3. Node
TypeScript的Node解析策略是模仿Node.js模块解析机制,所以要理解TypeScript编译依照的解析步骤，先弄明白Node.js是如何解析模块的。
### 3.1. NodeJs的模块解析
完整的Node.js解析算法可查看[Node.js module documentation](https://nodejs.org/api/modules.html#modules_all_together)。在Node.js里导入是通过require函数调用进行的。 Node.js会根据require的是相对路径还是非相对路径做出不同的行为。
#### 3.1.1. NodeJs的相对路径模块解析
如果在`/root/src/moduleA.js`，包含了一个导入`var x = require("./moduleB")`; Node.js以下面的顺序解析这个导入，直到有一个匹配上：
1. 检查`/root/src/moduleB.js`文件是否存在。
2. 检查`/root/src/moduleB`目录是否包含一个package.json文件，且package.json文件指定了一个"main"模块。 在我们的例子里，如果Node.js发现文件`/root/src/moduleB/package.json`包含了`{ "main": "lib/mainModule.js" }`，那么Node.js会引用`/root/src/moduleB/lib/mainModule.js`。
3. 检查`/root/src/moduleB`目录是否包含一个index.js文件。 这个文件会被隐式地当作那个文件夹下的"main"模块。
#### 3.1.2. NodeJs的绝对路径模块解析
非相对模块名的解析是个完全不同的过程。 Node会直接到node_modules里查找你的模块。node_modules可能与当前文件在同一级目录下，或者在上层目录里。Node会向上级目录遍历，查找每个 node_modules直到它找到要加载的模块。如果在`/root/src/moduleA.js`，包含了一个导入`var x = require("moduleB")`; Node.js以下面的顺序解析这个导入，直到有一个匹配上：
1. /root/src/node_modules/moduleB.js
2. /root/src/node_modules/moduleB/package.json (如果指定了"main"属性)
3. /root/src/node_modules/moduleB/index.js
4. /root/node_modules/moduleB.js (跳到上级目录了)
5. /root/node_modules/moduleB/package.json (如果指定了"main"属性)
6. /root/node_modules/moduleB/index.js
7. /node_modules/moduleB.js (跳到上级目录了)
8. /node_modules/moduleB/package.json (如果指定了"main"属性)
9. /node_modules/moduleB/index.js
### 3.2. TypeScript的Node模块解析
TypeScript的Node解析模式是模仿Node的，在Node解析逻辑基础上增加了TypeScript源文件的扩展名（ .ts，.tsx和.d.ts）。 同时，TypeScript在package.json里使用字段"types"来表示类似"main"的意义 - 编译器会使用它来找到要使用的"main"定义文件。
#### 3.2.1. TypeScript的Node相对路径模块解析
如果在`/root/src/moduleA.ts`，包含了一个导入`import { b } from "./moduleB"`;会以下面的顺序解析这个导入，直到有一个匹配上：
1. /root/src/moduleB.ts
2. /root/src/moduleB.tsx
3. /root/src/moduleB.d.ts
4. /root/src/moduleB/package.json (如果指定了"types"属性)
5. /root/src/moduleB/index.ts
6. /root/src/moduleB/index.tsx
7. /root/src/moduleB/index.d.ts
#### 3.2.2. TypeScript的Node相对路径模块解析
如果在`/root/src/moduleA.ts`，包含了一个导入`import { b } from "moduleB"`;会以下面的顺序解析这个导入，直到有一个匹配上：
1. /root/src/node_modules/moduleB.tsx
2. /root/src/node_modules/moduleB.ts
3. /root/src/node_modules/moduleB.d.ts
4. /root/src/node_modules/moduleB/package.json (如果指定了"types"属性)
5. /root/src/node_modules/moduleB/index.ts
6. /root/src/node_modules/moduleB/index.tsx
7. /root/src/node_modules/moduleB/index.d.ts
8. /root/node_modules/moduleB.ts (跳到上级目录了)
9. /root/node_modules/moduleB.tsx
10. /root/node_modules/moduleB.d.ts
11. /root/node_modules/moduleB/package.json (如果指定了"types"属性)
12. /root/node_modules/moduleB/index.ts
13. /root/node_modules/moduleB/index.tsx
14. /root/node_modules/moduleB/index.d.ts
15. /node_modules/moduleB.ts (跳到上级目录了)
16. /node_modules/moduleB.tsx
17. /node_modules/moduleB.d.ts
18. /node_modules/moduleB/package.json (如果指定了"types"属性)
19. /node_modules/moduleB/index.ts
20. /node_modules/moduleB/index.tsx
21. /node_modules/moduleB/index.d.ts

看着多，但和Node.js类似，只是多了两个文件类型+不断往上级目录查找而已