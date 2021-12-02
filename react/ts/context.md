# context

## 基本示例

```tsx
import { createContext, FC, useContext } from 'react';

interface AppContextInterface {
  name: string;
  author: string;
  url: string;
}

const AppContext = createContext<AppContextInterface | null>(null);

const initialValue: AppContextInterface = {
  name: 'use context in ts app',
  author: 'itsuki',
  url: 'xxx',
};

export const App: FC = ({ children }) => (
  <AppContext.Provider value={initialValue}>{children}</AppContext.Provider>
);

export const PostInfo = () => {
  const appContext = useContext(AppContext);

  return (
    <div>
      <p>Name: {appContext?.name}</p>
      <p>Author: {appContext?.author}</p>
      <p>Url: {appContext?.url}</p>
    </div>
  );
};
```

## 初始值为 undefined

如果初始值是`undefined`时, 还需要进行检查或者使用非空断言.

```tsx
const appContext = React.createContext<string | undefined>(undefined);

return <div>{app!.toUpperCase()}</div>;
```

有两个解决方案:

1. 初始值使用非空断言
2. 使用一个辅助函数

### 非空断言

我们可以初始值使用非空断言来防止进行判断

```tsx
const appContext = React.createContext<string>(undefined!);
```

这是一个简单快速的方法, 但是会有问题, 假如我忘记赋值, 就会出现错误.

### 使用一个辅助函数

我们使用一个辅助函数`createCtx`来做类型保护, 防止访问未提供值的`Context`, 通过这样做，可以使我们不必提供一个默认值，也不必检查未定义。

```tsx
import * as React from 'react';

function createCtx<A extends {} | null>() {
  const context = React.createContext<A | undefined>(undefined);

  function useCtx() {
    const c = React.useContext(context);
    if (c === undefined) {
      throw new Error('useCtx must be inside a Provider with a value');
    }
    return c;
  }

  return [useCtx, context.Provider] as const;
}

const [useUserName, UserProvider] = createCtx<string>();

function Avatar() {
  const userName = useUserName();
  return <div>hello {userName.toUpperCase()}</div>;
}

function App() {
  return (
    <UserProvider value='itsuki'>
      <Avatar />
    </UserProvider>
  );
}
```

### useReducer 与 useContext 搭配使用更佳

```tsx
import * as React from 'react';

export function createCtx<StateType, ActionType>(
  reducer: React.Reducer<StateType, ActionType>,
  initialState: StateType,
) {
  const defaultDispatch: React.Dispatch<ActionType> = () => initialState;
  const ctx = React.createContext({
    state: initialState,
    dispatch: defaultDispatch,
  });
  function Provider(props: React.PropsWithChildren<{}>) {
    const [state, dispatch] = React.useReducer<React.Reducer<StateType, ActionType>>(
      reducer,
      initialState,
    );
    return <ctx.Provider value={{ state, dispatch }} {...props} />;
  }
  return [ctx, Provider] as const;
}

const initialState = { count: 0 };
type AppState = typeof initialState;
type Action =
  | { type: 'increment' }
  | { type: 'add'; payload: number }
  | { type: 'minus'; payload: number }
  | { type: 'decrement' };

function reducer(state: AppState, action: Action): AppState {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'add':
      return { count: state.count + action.payload };
    case 'minus':
      return { count: state.count - action.payload };
    default:
      return { ...state };
  }
}

const [ctx, CountProvider] = createCtx(reducer, initialState);
export const CountContext = ctx;

function Counter() {
  const { state, dispatch } = React.useContext(CountContext);

  return (
    <div>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'add', payload: 5 })}>+5</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'minus', payload: 5 })}>+5</button>
    </div>
  );
}

export function App() {
  return (
    <CountProvider>
      <Counter />
    </CountProvider>
  );
}
```
