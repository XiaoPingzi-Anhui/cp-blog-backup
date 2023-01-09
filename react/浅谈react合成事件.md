<!-- category: "typeScript"
labels: "typeScript"
createdAt: 2022-09-17T17:14:08.623+00:00 -->
浏览器的原生事件，前面已经大概讲过了，而React 合成事件（SyntheticEvent）是 React 模拟原生 DOM 事件所有能力的一个事件对象，即浏览器原生事件的跨浏览器包装器。它根据 W3C 规范 来定义合成事件，兼容所有浏览器，拥有与浏览器原生事件相同的接口。下面着重看一下react合成事件和原生事件的差异点。（17以后的版本）
##1.绑定元素的不同
如果你在某个元素上注册了一个处理程序，对于原生事件来说，处理程序就绑定在该元素上，但是react不一样，React的合成事件不会直接绑定到目标DOM节点上，用事件委托机制，以队列的方式，从触发事件的组件向父组件回溯，直到Root节点。因此，React组件上声明的事件最终绑定到了Root节点（React17之前是Document）上。在Root节点，用一个统一的监听器去监听，这个监听器上保存着目标节点与事件对象的映射。当组件挂载或卸载时，只需在这个统一的事件监听器上插入或删除对应对象；当事件发生时（即Root上的事件处理函数被执行），在映射里按照冒泡或捕获的路径去组件中收集真正的事件处理函数，然后，由这个统一的事件监听器对所收集的事件逐一执行。
这样做的好处：
+ 对事件进行归类，可以在事件产生的任务上包含不同的优先级
+ 提供合成事件对象，抹平浏览器的兼容性差异
+  减少内存消耗，提升性能，不需要注册那么多的事件了，一种事件类型只在 Root上注册一次

##2.注册事件处理程序方法不同
前面讲的三种注册方法，注册的都是原生事件，react合成事件注册是要用驼峰命名，且传入一个函数，例如onClick、onClickCapture
##3.阻止浏览器默认方式不同
原生事件可以通返回false来阻止浏览器默认方式，但是react合成事件中，必须显式调用e.preventDefault()。
##4.执行顺序
如果同时注册原生和合成事件，那么执行顺序有什么区别呢？
```typescript
import { useRef } from 'react';
import { useEventListener } from 'ahooks';

export default function Test() {
  const parentRef = useRef<HTMLDivElement>(null);
  const childRef = useRef<HTMLDivElement>(null);

  useEventListener('click', () => console.log('parent原生捕获'), { target: parentRef, capture: true });
  useEventListener('click', () => console.log('parent原生冒泡'), { target: parentRef });
  useEventListener('click', () => console.log('child原生捕获'), { target: parentRef, capture: true });
  useEventListener('click', () => console.log('child原生冒泡'), { target: childRef });

  return (
    <div
      ref={parentRef}
      onClick={() => console.log('parent合成冒泡')}
      onClickCapture={() => console.log('parent合成捕获')}
    >
      <div
        ref={childRef}
        onClick={() => console.log('children合成冒泡')}
        onClickCapture={() => console.log('children合成捕获')}
      >
        child
      </div>
    </div>
  );
}
```
这里在Test组件中，点击 child ,控制台会怎么打印？（这里使用ahooks提供的useEventListener来注册原生事件，挺好用的）答案是：parent合成捕获 children合成捕获 parent原生捕获 child原生捕获 child原生冒泡 parent原生冒泡 children合成冒泡 parent合成冒泡。

可以发现，捕获过程中，合成事件先执行，但在冒泡过程 中，原生事件先执行。其原因就是react通过事件委托，将处理程序全绑定在root节点，所以在捕获阶段，在root节点时，parent和child的合成捕获事件就已经执行了，而原生捕获事件要等到捕获阶段真正下探到parent和child节点才执行，所以捕获阶段合成事件先于原生事件执行，相反，冒泡阶段，要等事件冒泡到root，合成事件再执行，在冒泡过程中，原生事件已经执行了，所以冒泡过程中，原生先于合成。

所以如果你原生事件。合成事件混着用时就要注意了，一定要慎用e.stopPropagation()，这里只拿冒泡阶段举个例子，捕获阶段不研究。
```typescript
import { useRef, SyntheticEvent } from 'react';
import { useEventListener } from 'ahooks';

export default function Test() {
  const parentRef = useRef<HTMLDivElement>(null);
  const childRef = useRef<HTMLDivElement>(null);

  useEventListener('click', () => console.log('parent原生捕获'), { target: parentRef, capture: true });
  useEventListener('click', () => console.log('parent原生冒泡'), { target: parentRef });
  useEventListener('click', () => console.log('child原生捕获'), { target: parentRef, capture: true });
  useEventListener(
    'click',
    (e: SyntheticEvent) => {
      e.stopPropagation();
      console.log('child原生冒泡');
    },
    { target: childRef },
  );

  return (
    <div
      ref={parentRef}
      onClick={() => console.log('parent合成冒泡')}
      onClickCapture={() => console.log('parent合成捕获')}
    >
      <div
        ref={childRef}
        onClick={(e: SyntheticEvent) => {
          // e.stopPropagation();
          console.log('children合成冒泡');
        }}
        onClickCapture={() => console.log('children合成捕获')}
      >
        child
      </div>
    </div>
  );
}
```
如果阻止了原生事件的冒泡，那么是会影响到合成事件的，因为合成事件是基于事件委托的，没了冒泡当然不会执行，所以上面的代码，控制台的打印结果为：parent合成捕获 children合成捕获 parent原生捕获 child原生捕获 child原生冒泡。那么如果在合成事件中阻止冒泡会影响到原生事件吗？其实是不会的，因为
e.stopPropagation()其实只是阻止了react的合成事件冒泡，原生事件还会继续冒泡。