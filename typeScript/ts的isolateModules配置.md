<!-- category: "typeScript"
labels: "typeScript"
createdAt: 2022-06-15T11:03:49.019+00:00 -->
isolatedModules是一个将每个文件作为单独的模块。参考如下情况:
```typescript
/* 在一个`type.ts`的文件定义了一个type并导出 */
export type People = {
  name: string;
  age: number;
};

/* 情况1：在a文件引入并使用 */
import { People } from "./type";

export const people: People = {
  name: "114514",
  age: 22,
};

/* 情况2：在b文件引入再直接导出 */
import { People } from "./type";
export { People };

/* 情况3：在c文件引入再直接导出 */
export { People } from './type';
```
我们知道tsc在编译时，是会把类型定义擦除的，tsc是可以跨文件分析类型的，但是对于仅做类型擦除的编译器，比如被广泛使用的Babel，它对.ts文件的类型分析，仅限于当前文件，并没有跨文件分析的能力。那么对于上面三种情况，它就会做这样的处理。

对于a文件，People已经作为类型使用了，Babel明确知道People是类型定义，所以Babel会把People的定义引用都删除，Babel编译a文件的代码就是这样的：
```typescript
const people= {
  name: "114514",
  age: 22,
}
```
而type文件里的定义导出都会删除，目前来看没什么问题是吧。但对于b文件和c文件，Babel并没有办法从单个文件中分析出，People是一个类型还是其他什么东西。那么经过Babel处理之后它就会被保留下来。在b、c文件里，对People的导入导出都还将存在。那问题就来了，type里People导出已经没了，运行出错。

对于单个模块（文件）如果它的导入导出是确定的， 我们称呼他们是模块隔离的，显然，b、c两个文件并不是模块隔离的。因为People不确定是Typescript中类型还是JavaScript运行时的值。除了引入类型不使用，重导出类型之外，还有两种情形也会导致模块隔离被破坏
- 引入并使用了const enum的值；
- Namespaces，同名的namespace可以跨越多个文件，最后编译时他们会被tsc合并，这种跨文件的文件处理babel是做不到的。

为了避免这种错误，我们一般都会在tsconfig.json中，把isolatedModules设为true，它会强制开发者的每个模块都能作为单个模块独立编译，不符合的会报错提示开发者修改，如果你新建一个.ts文件，然后什么也不做，就可以看到报错了，会让你起码导出一个空对象。

当你开启了isolatedModules，b、c文件的那种写法就会报错了，为了修复报错，可以像下面这样写：
```typescript
/* b文件修复方法一： */
import type { People } from "./type";
export { People };

/* b文件修复方法二： */
import { People } from "./type";
export type { People };

/* c文件 */
export type { People } from './type';
```
是的，既然编译器不知道，那么就用type显式告诉编译器，我们引入或导出的内容，就是一个类型，这样编译器就可以正常删除类型了。