<!-- category: "antd"
labels: "react,antd"
createdAt: 2022-08-08T17:25:22.832+00:00 -->
最近在antd的modal里，获取dom高度，发现总是在第一次打开modal时获取的高度并不正确，主要方法如下
```typescript
const handleLink = usePersistFn((type: string) => {
   const linkIdx = leftAnchorList.findIndex((o) => o === type);
   requestAnimationFrame(() => {
     const scrollTopDom: HTMLElement | null | undefined = domRef.current?.querySelector(`#anchor_content${linkIdx}`);
     scrollTopDom &&
       domRef.current?.scrollTo({
       top: scrollTopDom?.offsetTop,
     });
   });
 });
```
主要是想在modal打开时做一个锚点定位的效果，定位到scrollTopDom，scrollTopDom是modal内的内容，但是这个offsetTop偏移总是在第一次打开modal时，拿到的值并不正确，觉得很奇怪，考虑到是handleLink执行时scrollTopDom并没有正确渲染出来，有试过用延时函数执行，如下
```typescript
const handleLink = usePersistFn((type: string) => {
   const linkIdx = leftAnchorList.findIndex((o) => o === type);
   setTimeout(() => {
      const scrollTopDom: HTMLElement | null | undefined = domRef.current?.querySelector(`#anchor_content${linkIdx}`);
      scrollTopDom &&
        domRef.current?.scrollTo({
          top: scrollTopDom?.offsetTop,
        });
    }, 100);
 });
 ```
100ms延时，并没有用，offsetTop还是不对，有点懵，按理说正常的dom100ms足够加载了，而且我这个scrollTopDom是一段非常简单的静态结构，又试了把modal去掉，直接执行原先的方法，没有延时方法也正常定位了……，结合modal只在第一次打开才有offsetTop不准的情况，之后都正常，嗯……看来就是modal第一次加载时悄悄干了啥，导致内部的dom渲染变得异常缓慢！

那问题定位出来了，如何解决呢，正常情况下，只有用户点击过modal，modal才会在dom树上挂载，能不能让modal不管用户打没打开过，都直接挂载到dom树上呢？这样就不会在初次打开modal时，缓慢渲染内部节点，我们拿到的值就会是正确的了。翻了翻api文档，还真有！forceRender属性，强制渲染，给modal加了这个属性，那么即便用户没有打开过modal弹窗，modal也会一直挂载在dom上。加上了这个之后，问题完美解决！

其实antd的modal还有一个问题，每次关闭弹窗时，内部的状态state都还会保留，下次再打开时state还是上次的状态，没回到初始化状态，就挺烦的，需要在弹窗关闭时手动把state初始化。看了api文档，destroyOnClose属性，关闭时销毁 Modal 里的子元素，但是加上后state还是保留了，没啥用，不知道这算不算bug，有无大佬有解决办法。

总结：如果在antd的modal内做一些获取dom的操作，如果在第一次打开modal窗时dom不正常，那么不妨给modal加上forceRender属性试试！