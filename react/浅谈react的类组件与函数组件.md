<!-- category: "typeScript"
labels: "typeScript"
createdAt: 2022-09-20T20:15:24.618+00:00 -->
20年刚接触react时，项目使用的是类组件，好吧我承认那时候都不知道这么叫，甚至不知道还有函数组件。21年后全面转入函数组件了，hooks真香，函数组件真香，再也不会被this搞得晕头转向了！下面就大改说一下两者的区别吧。
## 1.设计思想不同
函数式组件是函数式编程思想，就是通过函数编写的形式去实现一个React组件，是React中定义组件最简单的方式，而类组件是面向对象编程思想，也就是通过使用ES6类的编写形式去编写组件，该类必须继承React.Component。函数和类谁更简单？当然是函数，所以函数组件的写法简单方便，内存上的性能更好，但简单的东西功能往往不太强大，类组件在功能上确实也比函数组件更丰富，在某些功能里目前还必须用类组件实现。React现在的设计思路更推崇组合，而不是继承。在类组件中大量使用继承会造成组件过重，功能难以拆分。
## 2.状态管理不同
在hooks出来之前，函数组件就是无状态组件，不能保管组件的状态，所以那时候基本就只能使用类组件，现在有了hooks，函数组件App的状态是通过useState hooks实现的，而类组件ClassTest，我在其构造函数内通过this.state给其加了一个状态（类组件的状态管理当然没这么简单，我这里只是举个栗子）。
## 3.业务实现逻辑不同
类组件是用生命周期钩子函数来实现业务逻辑的，现在让我完整说出类组件的生命周期我已经说不出来了……而函数式组件使用react Hooks来实现业务逻辑。hooks不能在类组件使用哦！类组件的生命周期确实更强大，有的功能官方hooks还未实现，下面大概看几个主要的。
* 挂载阶段的getDerivedStateFromProps VS 无
该方法用于在props被传入后根据props更新state。函数组件中也可以写代码根据props更新state，但这样做会造成重复渲染。如果遇到需要根据props更新state的情况，应该考虑做状态提升。如果你发现在某个组件中必须要根据props更新state又无法做状态提升，那么该组件应该写成类式组件，而不是函数式组件
* 挂载阶段：componentDidMount VS useEffect
该方法用于在组件挂载以后执行副作用操作，如发起网络请求、设置计时器、创建订阅等。函数组件有对应的useEffect。
* render：
在类组件的render方法中返回要渲染的内容。render里不能有副作用和setState！函数组件的return和类组件render方法的return效果一致。
* 更新阶段：shouldComponentUpdate VS memo、useMemo、useCallback
该方法返回true表示需要更新、返回false表示无需更新。可在此添加判断条件做性能优化，另外PureComponent实现原理也相同。函数组件对应的hooks有很多，常用的有memo、useMemo、useCallback，同样可以做性能优化。
* 更新阶段：getSnapshotBeforeUpdate VS 无
该方法在最近一次渲染输出(提交到DOM节点)之前调用。它使得组件能在发生更改之前从DOM中捕获一些信息(如滚动位置等)。此生命周期方法的任何返回值将作为参数传递给componentDidUpdate()。此用法并不常见，但它可能出现在UI 处理中，如以特殊方式处理滚动位置的聊天线程等。函数组件无该方法对应的hooks。
* 生命周期，更新阶段：componentDidUpdate VS 无
组件更新后会立即调用该方法，首次渲染不会调用。当组件更新后，可以在此处对DOM进行操作。注意：在该方法中慎用setState，如果要用必须将其包裹在条件语句里。函数组件无该方法对应的hooks，因为React本身设计是减少直接操作DOM，在React中除了useRef外直接操作DOM的场景很少，函数组件没有该方法对应的hooks不算什么问题。
* 生命周期，卸载阶段：componentWillUnmount VS useEffect
该方法会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如：清除计时器、取消网络请求或清除订阅等。函数组件有useEffect。
* 其他，错误边界：componentDidCatch、static getDerivedStateFromError VS 无
在类组件中定义了static getDerivedStateFromError或componentDidCatch这两个生命周期方法中的任意一个或两个时，那么它就变成一个错误边界。当抛出错误后，请使用static getDerivedStateFromError渲染备用UI，使用componentDidCatch打印错误信息。函数组件无错误边界对应的hooks，所以要实现错误边界功能只能使用类组件

## 4.调用方式不同
```typescript
import { FC, Component, useState } from 'react';

const FunctionTest: FC<{ goods: string }> = ({ goods }) => {
  console.log('FunctionTest渲染了');
  const showMessage = () => alert(`你下单的是“${goods}”！`);
  const handleClick = () => setTimeout(showMessage, 3000);
  return <button onClick={handleClick}>购买</button>;
};

class ClassTest extends Component<{ goods: string }> {
  constructor(props: { goods: string }) {
    super(props);
	this.state = this.props.goods;
    console.log('类组件的构造函数被调用了');
  }
  showMessage = () => alert(`你下单的是“${this.props.goods}”！`);
  handleClick = () => setTimeout(this.showMessage, 3000);

  render() {
    console.log('ClassTest渲染了');
    return <button onClick={this.handleClick}>购买</button>;
  }
}

const App = () => {
  const [goods, setGoods] = useState('苹果');
  return (
    <>
      <label>
        请选择：
        <select value={goods} onChange={(e) => setGoods(e.target.value)}>
          <option value="苹果">苹果</option>
          <option value="香蕉">香蕉</option>
        </select>
      </label>
      <h1>{goods}</h1>
      <p>
        <FunctionTest goods={goods} />
        <b> (function)</b>
      </p>
      <p>
        <ClassTest goods={goods} />
        <b> (class)</b>
      </p>
    </>
  );
};

export default App;
```
看上面的一个简单的例子，App组件有一个状态goods，保存用户购买的商品信息，还有两个‘购买’按钮，功能都一样，用户点击可以在3s后提示用户购买了什么商品。

现在请你把App组件挂载到dom上，App创建了，看看控制台打印信息：FunctionTest渲染了、类组件的构造函数被调用了、ClassTest渲染了。再把select的值改为香蕉，此时goods改变，会导致FunctionTest和ClassTest更新，在看控制台打印信息：FunctionTest渲染了、ClassTest渲染了。

嗯……虽然在jsx中使用FunctionTest和ClassTest的方式都一样，但显然react内部调用方式不同。在创建组件时，对于FunctionTest就是FunctionTest({goods})，执行了一次FunctionTest函数，但对于ClassTest来说却是 let instance = new ClassTest({name});（当然没这么简单，只是最简化的例子），创建了一个ClassTest的实例，并保存下来了。在创建组件时，对于FunctionTest就是FunctionTest({goods})，还是执行了一次FunctionTest函数，但对于ClassTest来说是调用instance.render()。这也是为什么说函数组件在内存性能上表现更好，因为它不需要创建实例，执行完中间变量就可以被回收，类组件的实例则在卸载之前都一直在内存中。

继续往下看，现在goods值是香蕉，请点击函数组件的‘购买’按钮，并立即在3s内将select值改为苹果，同样立即在3s内，3s后弹窗提示：你下单的是香蕉！没啥问题。请再点击类组件的购买按钮购买苹果，同样立即在3s内将select值改为香蕉，3s后弹窗提示：你下单的是香蕉！……什么鬼，买的明明不是苹果吗？原因其实很简单：
* 类组件：
 当类组件重新渲染时，只调用了render函数。组件的this不变。等定时器到了时，读取属性的的值时，先通过this找到props。再通过props找到name。而此时，name的值，已经发生了变化。所以，自然读到的是新值“张四疯” ，这个应该好理解。
* 函数式组件：
其实就是函数的闭包，handleClick里有定时函数，使用了外部词法环境的showMessage，showMessage又使用了外部词法环境的goods，所以即便FunctionTest更新，重新调用了，旧的goods值在setTimeout执行前都不会被回收，还存在！如果你不清楚这段话，可以看一下我之前写的这篇文章。

那类组件怎么解决这个问题呢？很简单，用个临时变量存着呗，也可以用闭包。
## 5.为什么更推崇函数组件了
