# 在 React 中使用 TypeScript 入门指南

> hooks 版本

### 引入 React

```tsx
import * as React from 'react'
import * as ReactDOM from 'react-dom'
```

如果直接使用 ESModule 的默认导入，会有报错信息

![p00](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/20220422215014p00.png)

原因就是为了满足 `common.js` 规范，在 React 声明文件里使用 `export = react` 来导出类型

![p01](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/20220422215106p01.png)

解决办法是在 `tsconfig.json` 的 `compilerOptions` 对象下配置 `allowSyntheticDefaultImports` 值为 `true`，允许对不包含默认导出的模块使用默认导入，就可以使用 ESModule 规范正常导入模块了，并且这个选项不会影响生成的代码，只是针对类型检查

![p03](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/20220422215249p03.png)

![p04](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/20220422215324p04.png)

***

### 声明函数式组件

> 有如下几种声明方式

##### 直接声明

```tsx
type AppProps = {
  message: string
  children?: React.ReactNode
}
const App = ({ message, children }: AppProps) => (
  <div>
    {message}
    {children}
  </div>
)
```

最大的缺点就是需要频繁的去定义 children 类型

##### 使用 **`PropsWithChildren`**

这种方式可以省去频繁定义 children 的类型，自动设置 children 类型为 ReactNode

```tsx
type AppProps = React.PropsWithChildren<{ message: string }>

const App = ({ message, children }: AppProps) => (
  <div>
    {message}
    {children}
  </div>
)
```



##### 使用 **`React.FunctionComponent`**，简写形式 **`React.FC`**

这也是**最推荐**的一种，这种声明函数组件的好处是：

首先，React.FC 显示定义了返回类型，其他方式都是隐式推导的

其次，React.FC 为 children 属性提供了隐式的类型，即 `ReactElement | null`

```tsx
type AppProps = {
  message: string
}

const App: React.FC<AppProps> = ({ message, children }) => (
  <div>
    {message}
    {children}
  </div>
)
```

当然，目前 React.FC 提供的类型还是存在一些问题

最开始 jsx 是不支持返回多个元素的，也就是说在 React 组件里如果想返回多个元素，必须在外层使用一个元素将它们包裹起来，这样就会产生很多没有意义的节点，导致不必要的嵌套；从 React 16 开始，支持直接返回一个字符串或者数组

```jsx
class App extends React.Component {
  render() {
    // 错误的写法
    return (
      <div>dom1</div>
      <div>dom2</div>
      <div>dom3</div>
    )
  }
}

class App extends React.Component {
  render() {
    // 正确的写法，当然也可以 React.Fragment
    return (
      <div>
        <div>dom1</div>
	      <div>dom2</div>
  	    <div>dom3</div>
      </div>
    )
  }
}

// React 16 以后
class App extends React.Component {
  render() {
    // ok
    return [
      <div>dom1</div>
      <div>dom2</div>
      <div>dom3</div>
		]
  }
}
```

现在函数组件里也可以这样使用，如下代码是可以正常渲染，但 `React.FC` 会报类型错误

![p05](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/20220422215343p05.png)

解决办法

```tsx
import React from "react";
import style from './style.less';

const Demo2:React.FC<{}> = () => {
  return [
    <div className={style.item} key={1}>dom1</div>,
    <div className={style.item} key={2}>dom2</div>,
    <div className={style.item} key={3}>dom3</div>,
  ] as any; // 方法1
}

const Demo2:React.FC<{}> = () => {
  return ([
    <div className={style.item} key={1}>dom1</div>,
    <div className={style.item} key={2}>dom2</div>,
    <div className={style.item} key={3}>dom3</div>,
  ] as unknown) as JSX.Element; // 方法2
}

export default Demo2;
```

---

### Hooks

#### useState<T>

TS 会根据初始值自动推导 state 的类型，比如下列代码，active 会被推导为 boolean 类型，对应的 setActive 方法只能接收 boolean 类型参数，否则将显示类型错误

```tsx
// ...
const [ active, setActive ] = useState(false);
const handleChangeActive = () => {
  setActive(prev => !prev) // ok
  // setActive(3) // err
}
```

但是当一些状态初始值为空时，就需要通过泛型显示的声明期望的类型，这样在渲染时直接使用 Optional Chaining 就可以避免渲染错误

```tsx
type UserInfo = {
  name: string
  age: number
}

const Demo3: React.FC = () => {
  const [user, setUser] = useState<UserInfo | null>(null);
  return (
    <div className={style.item}>
      {user?.name}
    </div>
  )
}
```

#### useRef<T>

```tsx
const valRef = useRef(0);

// 用作 DOM 引用，初始值为 null
const Demo3: React.FC = () => {
  const inputRef2 = useRef<HTMLInputElement>(null);
  const handleFocus = () => {
    inputRef.current?.focus();
  }
  return (
    <div className={style.item}>
      <Button onClick={handleFocus}>聚焦</Button>
      <input type="text" ref={inputRef2} />
    </div>
  )
}
```

#### useEffect

需要注意的只有一点 useEffect 传入的回调函数，返回值只能是 undefined 或者一个方法（清理函数）

所以 useEffect 里直接传入 async 函数一直不被推荐，因为 async 函数默认会返回一个 Promise

```tsx
// err
useEffect(async () => {
  const user = await getUser()
  setUser(user)
}, [])
```



---

#### 组件常用属性声明

```tsx
type AppProps = {
  msg: string
  checked: boolean
  names: string[] // 字符串数组
  status: "waiting" | "success" // 联合类型
  // 拥有具体属性的对象类型
  item: {
    value: string
    key: number
    optional?: OptionalType // 可选 
  },
  list: {
    value: string,
    key: number,
  }[] // 对象数组
  onClick: () => void // 无参数不需要返回值的函数
  onChange: (id: number) => void // 带参数的函数
  /**
  *推荐，可包含所有 children 的情况，像其他 JSX.Element 或 React.ReactChild[] 等数组或者 null 的情况没		*有考虑到
  */ 
  children: React.ReactNode
  functionChildren: (name: string) => React.ReactNode // 返回 react 节点的函数
  // 表单事件，泛型参数是 e.target 的类型
  onInputChange: React.FormEventHandler<HTMLInputElement>
}
```

---

#### 组件需要设置默认属性值

```tsx
type Demo1Props = {
  value?: number
}
const Demo1 = ({ value=18 }: Demo1Props) => {
  return (
    <div>
      {value}
    </div>
  )
}
export default Demo1;

```

---

#### 事件处理

处理事件以及 Event 对象在日常开发中非常普遍，比如 onClick 时获取 e.target，表单的 onChange 事件获取 value 值。当然可以把 event 设为 any，但这样就失去了对代码进行静态检查的意义，对于合成事件，我们可以直接使用 React 提供的事件对象类型声明

```tsx
const Demo4: React.FC = () => {

  /*
  * click 使用 React.MouseEvent 加 DOM 的泛型进行约束
  * 这里用到的是 HTMLButtonElement，常用的还有比如 HTMLDivElement
  */ 
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log(e.target);
  }

  // onChange使用 React.ChangeEvent 加 dom 的泛型进行约束
  // 这里用到的是 HTMLInputElement, 还有比如 HTMLSelectElement
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  }
  return (
    <div>
      <input type="text" onChange={handleChange} />
      <br />
      <br />
      <button onClick={handleClick}>click</button>
    </div>
  )
}
```

*React 提供的事件对象声明有*

*ClipboardEvent<T = Element> 剪切板事件对象*

*DragEvent<T =Element> 拖拽事件对象*

*KeyboardEvent<T = Element> 键盘事件对象*

*TouchEvent<T = Element> 触摸事件对象*

*WheelEvent<T = Element> 滚轮时间对象*

*AnimationEvent<T = Element> 动画事件对象*

*TransitionEvent<T = Element> 过渡事件对象*

*ChangeEvent<T = Element> Change事件对象*

*MouseEvent<T = Element> 鼠标事件对象*

对于原生事件，**不使用** React 的合成事件类型，比如监听鼠标指针位置

```tsx
const Demo4: React.FC = () => {
  function handler(e: MouseEvent) {
    console.log(e.clientY, e.clientY)
  }

  useEffect(() => {
    window.addEventListener('mousemove', handler, false);
    return () => {
      window.removeEventListener('mousemove', handler, false);
    }
  }, [])
  return (
    <div>
    </div>
  )
}
```

#### 事件传递

```tsx
//demo 1
// parent.tsx
import Child from './components/Child1';
const Demo = ({ value=18 }: DemoProps) => {
  const fromChild = (payload: string) => {
    console.log(payload);
  }
  const childInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
  }
  return (
    <div>
      {value}
      <div style={btnStyle}>内容很长</div>
      <Child valueProps={value} fnProps={fromChild} childInputChange={childInputChange} />
    </div>
  )
}

// Child.tsx

const Child: React.FC<{
  valueProps: number;
  fnProps: (params: string) => void;
  childInputChange: React.FormEventHandler<HTMLInputElement>
}> = ({ valueProps, fnProps, childInputChange }) => {
  const callFnProps = () => {
    fnProps('from child');
  };
  return (
    <div>
      <div>父组件属性：{valueProps}</div>
      <br />
      <input type="text" onChange={childInputChange} placeholder="子组件输入框" />
      <div><button onClick={callFnProps}>调用fnProps</button></div>
    </div>
  );
};

export default Child;
```

---

#### Promise 类型

在进行异步编程时，经常使用 async/await 函数，return 一个 Promise 对象

`Promise<T>` 是一个泛型类型，T 泛型变量代表 then 方法接收的第一个回调函数的参数类型

比如获取 jsConfig 的接口，接口返回的数据如下

```tsx
const = {
  code: 200,
  message: 'success',
  obj: {
    agent_id: 'xxxx';
    corp_id: '1';
    time_stamp: 'xxxx';
    nonce_str: 'xxxx';
    signature: 'xxxx';
  }
}
```

请求函数的类型声明如下

```tsx
type JsSign = {
  agent_id: string;
  corp_id: string;
  time_stamp: string;
  nonce_str: string;
  signature: string;
};

type JsConfigRes<T> = {
  code: number;
  obj: T;
  message: string;
};

export async function postJSConfig(params: Yach.JSConfigParams): Promise<JsConfigRes<JsSign>> {
  return request('url', {
    method: 'POST',
    data: params,
    requestType: 'form',
  });
}
```

首先声明 JsConfigRes 的泛型接口用于定义 response 的类型，其中通过 T 泛型变量确定 obj 的类型

异步请求函数 postJSConfig 的返回值类型定义为 Promise<JsConfigRes<JsSign>>

此时调用得到的结果就与后端接口定义的格式保持一致了

---

#### 业务场景

==时间选择器==

moment 是一个 TS 库，自带声明文件，因此直接引入使用即可

```tsx
import { DatePicker } from 'antd';
import { Moment } from 'moment';
const disabledStart = (date: Moment) => {
  return date < moment('2019-01-01') || date > moment();
};
<DatePicker
  placeholder="开始时间"
  onChange={handleChangeStart}
  disabledDate={disabledStart}
/>
```

==动态增加属性==

将 key 设置为 any，person 就变成了可以是任意数量属性的字典，方便我们添加 key

```tsx
type Person = {
  name: string;
  age?: number;
  [propName: string]: any;
};
```

==行间样式==

可以声明为 React.CSSProperties 类型

```tsx
// demo1
const btnStyle: React.CSSProperties = {
  width: '30px',
  overflow: 'hidden',
  textOverflow: 'ellipsis',
  whiteSpace: 'nowrap',
  fontSize: '12px',
}
const Demo1 = ({ value=18 }: Demo1Props) => {
  return (

    <div>
      {value}
      <div style={btnStyle}>button</div>
    </div>

  )
}
```

==清晰的注释==

要使用 `/**/`，vscode 识别不了 `//`

---

#### Type 与 Interface

虽然在 TS 中 Type 和 Interface 是两个不同的概念，但在 react 开发中，Interface 和 type 都可以达到相同的功能效果

二者最大的区别在于：type 类型不能二次编辑，而 Interface 可以随时扩展

因此在使用场景上，如果是定义公共 API 或者开发一个库，使用 Interface，这样可以方便使用者继承接口

而在定义组件的 state 和 props 时，使用 type，因为约束性更强

---

#### 小结

npm 包的引入

函数式组件的声明

常用 hooks

组件内的事件

表单事件

原生事件

组件的默认属性

组件之间属性的传递、事件的传递

异步请求 Promise 的类型定义

一些业务中的使用场景，特殊组件时间选择器，需要动态更新对象属性，定义行间样式等

> 扩展阅读

[Typescript-面向编辑器编程](https://mp.weixin.qq.com/s/h4xmxQGFHiAiaqMrJ3JQiA)

![p06](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/20220422215415p06.png)

