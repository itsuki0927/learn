# react ts cheat

## Typing Component Props

```ts
type AppProps = {
  message: string;
  count: number;
  disabled: boolean;
  /** array of a type! */
  names: string[];
  /** string literals to specify exact string values, with a union type to join them together */
  status: 'waiting' | 'success';
  /** any object as long as you dont use its properties (NOT COMMON but useful as placeholder) */
  obj: object;
  obj2: {}; // almost the same as `object`, exactly the same as `Object`
  /** an object with any number of properties (PREFERRED) */
  obj3: {
    id: string;
    title: string;
  };
  /** array of objects! (common) */
  objArr: {
    id: string;
    title: string;
  }[];
  /** a dict object with any number of properties of the same type */
  dict1: {
    [key: string]: MyTypeHere;
  };
  dict2: Record<string, MyTypeHere>; // equivalent to dict1
  /** any function as long as you don't invoke it (not recommended) */
  onSomething: Function;
  /** function that doesn't take or return anything (VERY COMMON) */
  onClick: () => void;
  /** function with named prop (VERY COMMON) */
  onChange: (id: number) => void;
  /** alternative function type syntax that takes an event (VERY COMMON) */
  onClick(event: React.MouseEvent<HTMLButtonElement>): void;
  /** an optional prop (VERY COMMON!) */
  optional?: OptionalType;
};
```

### JSX.Element vs React.ReactNode

一个更技术解释的是，有效的 React 节点与 React.Createelement 的返回结果不同。无论组件最终渲染，React.createElement 始终返回一个对象，它是 JSX.Element 接口，但 react.createElement 是组件的所有可能返回值的集合。

## 函数式组件

函数式组件返回`JSX.Element`

```tsx
const App: React.FC<{ message: string }> = ({ message }) => <div>{message}</div>;
```

我们经常看到这样子的代码, 但是不鼓励使用`FC`,它和我们正常写的函数不一样:

- `FC` 对于返回类型是显式的, 而普通函数是隐式的(需要额外的注释)
- `FC` 自动添加了`displayName`、`propType`和`defaultProps` 属性检查
  - `defaultProps` 和 `FC`一起使用, 可能不会正常工作
- `FC`自动添加了`children`属性, -然而隐含的 children 类型有一些问题（[DefinitelyTyped#33006](https://github.com/DefinitelyTyped/DefinitelyTyped/issues/33006)），无论如何，对消耗 children 的组件进行显式处理可能更好。

```tsx
const Title: React.FunctionComponent<{ title: string }> = ({ children, title }) => (
  <div title={title}>{children}</div>
);
```

### 条件渲染

```tsx
const MyConditionalComponent = ({ shouldRender = false }) => (shouldRender ? <div /> : false); // don't do this in JS either
const el = <MyConditionalComponent />; // throws an error
```

这是因为由于编译器的限制，函数组件不能返回 `JSX` 表达式或 `null` 以外的任何东西，否则它就会以一个神秘的错误信息，说其他类型不能分配给 `Element`.

## hooks

### useState

默认情况下我们直接使用`useState`会直接推断其类型

```tsx
const [val, toggle] = useState(false);

// val 是boolean
// toggle 只接受boolean类型的参数
```

如果不指定初始值(默认就是`undefined`)但是显示指定了类型的话,也会进行类型推断, 最终的类型会于显示指定的类型进行联合

```tsx
const [user, setUser] = useState<User>();

// user: User | undefined
```

但是, 很多情况下我们使用`null`、`undefined`作为初始值, 如果我们还想知道类型的话, 就得显示声明类型(使用联合类型)

```tsx
const [user1, setUser1] = useState<User | null>(null);

const [user2, setUser2] = useState<User | undefined>(undefined);
```

我们也可以使用`as`关键字进行类型断言

```tsx
interface User {
  name: string;
  age: number;
}
const [user, setUser] = useState<User>({} as User);
```

上面的例子就是这样子, 如果不使用`as User`的话, `{}`是不能作为初始值的, 因为缺少了`name`、`age`属性, ide 就会提示你无法正确赋值, 使用了`as`运算符做了一个类型断言, 就相当于将`{}`强制转化为`User`类型, 就不会出现类型错误

### useReducer

```tsx
const initialState = { count: 0 };

type ActionTypes = { type: 'increment'; payload: number } | { type: 'decrement'; payload: string };

const countReducer = (state = initialState, action: ActionTypes) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.payload };
    case 'decrement':
      return { count: state.count - Number(action.payload) };
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(countReducer, initialState);

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement', payload: '5' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment', payload: 5 })}>+</button>
    </>
  );
};
```

如果使用`Redux`的话,可以使用它提供的`Reducer<State,Action>`做类型推断与检查

```tsx
import {Reducer} from 'redux'

export function reducer: Reducer<AppState,Action>(){}
```
