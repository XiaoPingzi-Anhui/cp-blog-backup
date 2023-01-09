<!-- category: "react"
labels: "react,ahooks"
createdAt: 2022-12-01T16:37:08.623+00:00 -->
useEffect是一个用了执行副作用的hook，还能清除副作用，以前刚开始学的时候，网上一大堆博客都说，清除副作用方法会在组件卸载的时候执行。一直也这样认为了很久，但是最近发现这种说法其实并不准确，有点以偏概全了，然后自己研究了一下。示例代码如下：
```typescript
import { useEffect, useState } from "react";

export default function Test() {
  const [clickDiv1Times, setClickDiv1Times] = useState(0);
  const [clickDiv2Times, setClickDiv2Times1] = useState(0);
  /* 第二个参数不传，副作用会在每次Test函数执行完，或者说Test组件更新完之后执行 */
  useEffect(() => {
    console.log("run1");
    return () => console.log("clear1");
  });

  /* 第二个参数传个空数组，副作用只会在Test创建后执行 */
  useEffect(() => {
    console.log("run2");
    return () => console.log("clear2");
  }, []);

  /* 第二个参数传了依赖状态，副作用只会在依赖状态变化，引起Test重新执行后执行 */
  useEffect(() => {
    console.log("run3");
    return () => console.log("clear3");
  }, [clickDiv1Times]);

  return (
    <>
      <div
        onClick={() => setClickDiv1Times(clickDiv1Times + 1)}
      >
        {clickDiv1Times}
      </div>
      <div
        onClick={() => setClickDiv2Times1(clickDiv2Times + 1)}
      >
        {clickDiv2Times}
      </div>
    </>
  );
}
```
让我们观察这三种情况，副作用和清除副作用什么时候执行。

首先，Test组件初次创造时，控制台打印：run1 run2 run3。可以看到，初次创建时，三个副作用都执行了，清除副作用都没有执行。

然后点击第一个div，更改clickDiv1Times的值，控制台打印：clear1 clear3 run1 run3。结果是，1和3的清除副作用先执行了，然后再执行1和3的副作用。所以可以看到以前的说法：清除副作用方法会在组件卸载的时候执行，是错误的，Test并没有卸载，只是更新了。而且我们能得到一个结论：每次执行副作用之前，会按顺序把之前执行过的副作用清除掉。

再点击第二个div，更改clickDiv2Times的值，控制台打印：clear1 run1。

再卸载掉Test组件，控制台打印：clear1 clear2 clear3，可以看到组件卸载，副作用不会执行，但会清除所有的副作用。

所以那种说法只适用于第二种空数组的情况，这也是ahooks里的useUnmount的原理。真正的结论是：每次执行副作用之前，会按顺序把之前执行过的副作用清除掉；组件卸载时会清除所有的副作用，副作用不会执行