<!-- category: "react"
labels: "react"
createdAt: 2022-10-18T19:34:22.832+00:00 -->
```typescript
const App = () => {
  return (
    <Screen/>
  );
};
```
今天碰到一个需求，例如在上方的App组件中，我需要给组件Screen加一些额外的props,props又有自己的一套逻辑。问题就在于props相关的逻辑比较复杂，我又需要在很多个类似App的组件给Screen加，而且一个App中可能有多个Screen。一个个加肯定不现实，写出来会被同事拖出去打。

一开始想着把props相关的逻辑抽成一个hooks，在每个app中都调用这个hooks，但是问题来了，一个App中可能有多个Screen，那我就需要在一个App中多次调用hooks,也不太优雅。

最后我就找到了React.cloneElement()这个api，React.cloneElement()函数创建给定元素的克隆，通过这个api我就可以写一个组件，里面做一些newProps的逻辑处理，然后把Screen组件作为子元素传进去，通过cloneElement把newProps给Screen，最后返回克隆元素就行了，这样我只要给每个Screen包一层ControlledScreenWrapper，就达到了我的目的，大概代码如下：
```typescript
import { cloneElement, ReactElement, FC, memo,  useMemo } from 'react';

interface Props {
  children?: ReactElement;
}

const ControlledScreenWrapper: FC<Props> = ({ children, }) => {
 /* newProps的巴拉巴拉一堆逻辑 */

  const rebuildChildren = useMemo(() => {
    if (children) {
      return cloneElement(children, {
        ...(children?.props ?? {}),
        newProps, // 我要加的props
      });
    } else return null;
  }, [children]);

  return <>{rebuildChildren}</>;
};

const App = () => {
  return (
  <ControlledScreenWrapper>
    <Screen />
  </ControlledScreenWrapper>
  );
};
```
下面来具体看一下React.cloneElement()的用法：
```typescript
React.cloneElement(
  element,
  [props],
  [...children]
)
```
其api很简单，有三个参数，第一个element就是我们要克隆的元素，可以是 真实的dom结构 也可以是 自定义的。第二个参数返回旧元素的props、key、ref。你可以在这里自定义你需要的props，但似乎不能修改key、ref，这个我还没试过。第三个是props.children，不指定默认展示我们调用时添加的子元素。如果指定会**覆盖**我们调用克隆组件时里面包含的元素。

但其实实际使用时，还经常需要搭配React.Children使用。就像上面的ControlledScreenWrapper组件，在我的需求里其子组件只有一个Screen，但往往我们的子组件是多个，多个的时候就要配合React.Children方法使用，例如下方的代码，给Container的每一个子元素字体都改成红色。
```typescript
import { Children, cloneElement, ReactNode, FC, useMemo } from 'react';

interface Props {
  children?: ReactNode;
}

const Container: FC<Props> = ({ children }) => {
  const rebuildChildren = useMemo(() => {
    if (children) {
      return Children.map(children, (child: any) => {
        return cloneElement(child, { ...child.props, style: { color: 'red' } });
      });
    } else return null;
  }, [children]);

  return <>{rebuildChildren}</>;
};

const App = () => {
  return (
    <Container>
      <div>xixi</div>
      <div>xixi</div>
    </Container>
  );
};

export default App;
```
上面用到的是Children.map()方法，React.children有5个方法：React.Children.map()，React.Children.forEach()、React.Children.count()、React.Children.only()、React.Children.toArray()。
React.Children.map()有些类似Array.prototype.map()。如果children是数组则此方法返回一个数组，如果是null或undefined则返回null或undefined。第一参数是children，即示例中的Container组件里的<div>xixi</div><div>xixi</div>。第二个参数是function，function的参数第一个是遍历的每一项，第二个是对应的索引。

React.Children.forEach()跟React.Children.map()一样，区别在于无返回。

React.Children.count()用来计数，返回child个数。不要用children.length来计数，如果子组件有一项是字符串'hello world!'会返回12，显然是错误的结果。
```typescript
import { Children, ReactNode, FC } from 'react';

interface Props {
  children?: ReactNode;
}

const Container: FC<Props> = ({ children }) => {
  return (
    <>
      <div>{Children.count(children)}</div> {/* 是1哦对的 */}
      <div>{children?.length}</div>  {/* 是12哦不对的 */}
    </>
  );
};

const App = () => {
  return <Container>hello world!</Container>;
};

export default App;
```
React.Children.only()是用来验证children里只有唯一的孩子并返回他。否则这个方法抛出一个错误。

 React.Children.toArray()将children转换成Array，对children排序时需要使用。
 ```typescript
 import { Children, ReactNode, FC } from 'react';

interface Props {
  children?: ReactNode;
}

const Container: FC<Props> = ({ children }) => {
  let children1 = Children.toArray(children);
  return <div>{children1.sort().join(' ')}</div>;
};

// 最后渲染结果会是 aaa bbb ccc
const App = () => {
  return (
    <Container>
      {'ccc'}
      {'aaa'}
      {'bbb'}
    </Container>
  );
};

export default App;
```