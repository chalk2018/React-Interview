# React 面试题

** By Chalk Yu **

## 技术分类

### React 考核 -- 2 题

1. 和 vue 的 v-bind 一样的方式，在 React 中怎么写？

> {...props} 解构方式可实现

2. React 中的一个组件，我想传入 key='first'的属性，子组件能不能收到？

> 不能
> 一般在什么情况下会用到？
> 数组通过 map 渲染的时候会用到
> 为什么收不到？
> 加分项：key 是给 react 自己用的，做为每个虚拟 Dom 的唯一标识

### Hook 考核 -- 3 题

1. React“函数组件”重新渲染页面有哪几种方式？

> useState 更改状态、useReducer 更改状态、Redux 状态变化
> 加分项：能说出 React 渲染页面的原理 diff 算法？
> 三个策略：
> tree diff : 两个 Dom 树 一层一层的去比较，比较慢但考虑跨层操作比较少，所以不会太消耗效率。（优化：所以，尽量不要有太多移动 Dom 的操作，也就是控制 Dom 显隐会比 if 判断更有效率）
> component diff : 引入相同的组件，会形成相类似的 Dom 树，所以组件会单独比较。（优化：公共的功能尽量封装成组件，调用组件引用，也就是说不要把子组件定义父组件的函数内部，这会使父组件渲染的时候生成不同的子组件，增加消耗。）
> element diff : 对于同一层的节点，由于没有办法区分增加、修改还是删除，当前层的每个节点会去和变化前的这一层的每个节点进行比较，不同类别的节点还好比较，如果是相同类别，比如说 list，只能通过 key 值进行比较。（由于 list 会做全面的遍历，时间复杂度比较高，尽量不要跨度大的移动节点）

2. useEffect 都有哪些用法？怎么写？

```
//-1- 初始化
useEffect(()=>{
  //初始化内容，第一次渲染时被调用
  return ()=>{
    //页面销毁时被调用
  }
},[])
//-2- 监听状态
const [open] = useState(false)
useEffect(()=>{
  //状态值发生变化时被调用
  return ()=>{

  }
},[open])
//-3- 页面更新时被触发
useEffect(()=>{
  //更新时被触发
  return ()=>{

  }
})
```

3. 怎么获取某个标签的 html dom 对象，比如说 <div>hello</div>，怎么获取它的 html dom 对象？

   > 方式 1 useRef() 钩子
   > 方式 2 ref 属性回调 然后通过 useState 钩子绑定
   > 如果是一个自定义组件<Mine ref={myRef} />想要获取到 html dom，子组件要怎么做？
   > 要用 React.forwardRef(myComponent)，装载子组件，然后子组件就能从 props 中接收到 ref 参数了，forwardRef 方法名说不出不要紧，意思到了就行。

4. useContext 和 useReducer 都能做些什么？要怎么用？
   > 能模式实现 redux 状态管理，能模拟 useState 状态钩子
   > 能分别说说 useContext 和 useReducer 的作用吗？
   > useContext 用于共享状态，可以把 value 属性中的值下发到下面所有的组件，类似于 Redux 中的 Store。如果 value 是个变量，子组件就可以实现跨组件传递状态，模拟 Redux。
   > useReducer 可以通过自定义的 reducer 转化成 state 和 action，绑定到 AppContext.Provider 的 value 上，实现派发。
   > 下面是例子。
   > store

```
import * as React from "react";
// 创建context要指定初始值，这个初始值是<Provider>里value的初始值，
// 本工程里value的值是addReducer钩子返回项目，所以初始值的类型必须是 Array<any,Function>，
// Array<any,Function>类型的初始值实例就是 [{} as any,(()=>{}) as Function]
export const AppContext = React.createContext([
  {} as any,
  (() => {}) as Function
]);

export const Provider = AppContext.Provider;
```

> reducer

```
export const AppReducer = (state: any, action: any) => {
  let init_state = {};
  console.log(state, action);
  return action || init_state;
};
```

> app

```
import * as React from "react";
import "./App.css";
import Layout from "./framework/Layout";
import { BrowserRouter } from "react-router-dom";

import { Provider } from "./store/context";
import { AppReducer } from "./store/reducer";

const App: React.FC = (props: any) => {
  //contextValue是一个useReducer，就可以用 [state,action] = useContext() 的方式使用useContext了
  // 否则useContext无法解构出[state,action]，需要自己调用
  const contextValue = React.useReducer(AppReducer, {});
  return (
    <div className="App">
      <BrowserRouter>
        <Provider value={contextValue}>
          <Layout />
        </Provider>
      </BrowserRouter>
    </div>
  );
};

export default App;
```

> component reduce

> component

```
import { FrameworkReducer } from "./reducer";
import { AppContext } from "../store/context";
const Layout: React.FC = (props: any) => {
  // useContext 全局状态
  const [appState, appDispatch] = React.useContext(AppContext);
  // useReducer 相当于自定义的 useState
  const [state, dispatch] = React.useReducer(FrameworkReducer, { open: false });
  const handleDrawerOpen = () => {
    // dispatch参数中只能传值(包括对象)，不能传递回调函数
    dispatch({ open: true });
    // useContext 设置全局状态
    appDispatch({ open: true });
  };
  return (...)
}
```

### Redux 考核 -- 3 题

1. <Provider>是干啥用的？为什么最外层要包一个<Provider>
   > 为了把全局的 store 传递给每个组件中。
2. Redux 调试工具用的哪个？ 是 redux-devtools-extension 还是 redux-devtools？ 能简单介绍一下用法吗？
   > redux-devtools 是侵入式，需要在项目文件里引入。
   > redux-devtools-extension 是非侵入式，嵌在 window 全局变量里，可以添加判断是不是 product 环境，决定用不用这个。
   > 在控制台就能看到数据流向，进行调试。
3. redux 的 Saga 中间件用过吗？（es6）怎么创建一个迭代器？

```
//创建
function* countAppleSales () {
  var saleList = [3, 7, 5];
  for (var i = 0; i < saleList.length; i++) {
    yield saleList[i];
  }
}
//调用
var appleStore = countAppleSales(); // Generator { }
console.log(appleStore.next()); // { value: 3, done: false }
console.log(appleStore.next()); // { value: 7, done: false }
console.log(appleStore.next()); // { value: 5, done: false }
console.log(appleStore.next()); // { value: undefined, done: true }
```

### Router 考核 -- 1 题

1. 函数组件怎么跳转路由？

> 路由指定 history={history.createBrowserHistory()}参数，如：<Router history={history}，history 是对 window.history 封装的常用库，然后组件内部就能调用 props.history.push('/api/xxx')

### Ts 考核 -- 2 题

1. 下面数据定义一下类型

```
const props = {
  greetings: {
    en: 'hello',
    jp: 'こんにちは',
    cn: '你好'
  },
  farewell: {
    cn: '再见'
  }
}


//类型
type t = {[key:string]:{en?:string,jp?:string,cn?:string}}
interface t {
  [key:string]:{en?:string,jp?:string,cn?:string}
}
```

2. 对于不清楚的类型，应该用什么类型？

unknown 和 any 的区别？

> unknown 自动做类型检查，缩小范围，any 不做类型检查

### 代理请求考核 -- 2 题

1. react 跨域访问有哪些解决方式？
   > jsonp, iframe （不安全）
   > 反向代理 (需要前端服务器伪装，相对比较麻烦)
   > CORS 跨域资源共享， 例如：'Content-Type'='application/x-www-form-urlencoded'，后台接收从 header 中取数据，不从 body 中取。（常用）
   > 同源，压缩静态包发到后端服务器上（hybrid 就不适用了）
   > 服务端指定源。（常用）

### ES6 关联语法点 -- 1 题

let x = 'a'

1. { [x] : 1 } 什么意思？

### 设计模式

1. 观察者模式

   > 介绍一下观察者模式，react 中有没有？

2. react 哪些地方应用单向数据流了？
   > 组件中只能父传子，不能子传父。
   > Redux 中的 state 属于数据快照（叫数据切片也行），不能修改其引用（但可以通过 reducer 拦截）。

### Jtest 考核 -- 1 题

1. 说几个常用的测试方法
