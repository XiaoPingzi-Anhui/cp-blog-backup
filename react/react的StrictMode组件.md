<!-- category: "react"
labels: "react"
createdAt: 2022-08-11T12:44:08.623+00:00 -->
今天在写自己的博客网站时，发生了一件很奇怪的情况，我在文章列表组件中请求文章的接口，被意外地请求了两次!
```typescript
// 请求hook
export default function useArticleLists() 
  const { error, loading, data } = useRequest(getArticleLists);
  return { articleLists, loading, error };
}

const Inner = () => {
  const { data, loading } = useArticleLists();
  return (
    <div>blog</div>
  );
};
```
我在组件Inner中使用了useArticleLists hook，在useArticleLists中请求了一次接口getArticleLists，按理说在Inner组件的生命周期内，接口getArticleLists只会在组件Inner一开始挂载时请求一次，但是我的浏览器告诉我，getArticleLists接口请求了两次，两次的时间间隔极短。一开始我怀疑是我代码哪里写错了，可是我盯着这几行代码来来回回检查了几遍，没啥问题啊，就这几行代码能错到哪，我明明只触发了一次请求。

我开始意识到也许是Inner组件被创建之后又立马被卸载了，然后又被创建了，那这样接口请求两次就能解释通了。我使用了ahooks的useUnmount进行检查
```typescript
const Inner = () => {
  const { data, loading } = useArticleLists();
  useUnmount(() => {
    console.log("unmount");
  });
  return (
    <div>blog</div>
  );
};
```
如果Inner组件被卸载过，那么控制台就会打印出‘unmount’。果然，控制台打印了……Inner组件确实被卸载过一次，我在页面看到的其实是第二次创建的Inner，喵的，要不是发现接口被掉了两次，这问题还发现不了呢。那到底是哪位小可爱把我Inner组件卸载了呢。我开始一层层检查Inner组件的祖先组件挂载情况。越检查头越大，一直到最顶层的App组件，依然是被莫名其妙地卸载了一遍。真是离谱，我都开始怀疑是不是react18版本什么奇奇怪怪的bug了……

最后又通过一系列的排查操作，发现原来是因为我使用了StrictMode
```typescript
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```
react的严格模式，具体可以看[react官网](https://zh-hans.reactjs.org/docs/strict-mode.html)，嗯……这是我用脚手架创建项目时自动生成的。我知道它的作用是可以检测一些过时、废弃的api，或者一些不安全的代码。但具体的没研究过，不知道在开发环境它会把子组件调用两次啊！而且这是react18版本新加的特性,说实话有点坑，网上也没找到啥解决办法，唯一的办法似乎就是不使用StrictMode。反正我用的函数组件，那些过时的api根本不会使用，果断去掉StrictMode。再刷新看看，我的接口终于正常请求一次了。