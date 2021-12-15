# context + hooks = 真香

## 前沿

hooks 打开了新世界的大门, Context + hooks 让我们在状态分享上多了一个方案。

**代码警告!!!**

## 什么是 Context

引用官方文档中的一句话:

> Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

在 React 中当中, 数据流是从上而下的, 而 props 就是数据流中一个重要的载体, 通过 props 传递给子组件渲染出来的对应的视图, 但是如果嵌套层级过深, 需要一层层传递的 props 却显得力不从心, 就像上一篇文章说的[prop drilling](https://itsuki.cn/article/165)一样。

一个比较典型的场景就是: 应用程序进行主题、语言切换时, 一层层进行传递想象一下多么的痛苦, 而且你也不可能每一个元素都能够完全覆盖到,而 Context 就提供了一种可以在组件之间共享这些值的方法, 不需要再去显式的传递每一层 props。

## 基本使用

我们先来说说它的基本使用, 请注意我这里使用的是`tsx`, 我平时更加喜欢`typescript` + `react`, 体验感更好。

如果我们要创建一个 Context, 可以使用 `createContext` 方法, 它接收一个参数, 我们举一个简单的例子, 通过这个简单的例子来一点点掌握 context 的用法。

```tsx
const CounterContext = React.createContext<number>(0);

const App = ({ children }) => {
  return (
    <CounterContext.Provider value={0}>
      <Counter />
    </CounterContext.Provider>
  );
};

const Counter = () => {
  const count = useContext(CounterContext);

  return <h1> {count} </h1>;
};
```

这样子我们的组件无论嵌套多深, 都可以访问到`count`这个 props。 但是我们只是实现了访问, 那我们如果要进行更新呢?

### 如何实现更新

那么我们首先得先改造下参数类型, 从之前的一个联合类型变成一个对象类型, 有两个属性:

1. 一个是`count`表示数量。
2. 另一个是`setCount`实现更新数量的函数。

```tsx
export type CounterContextType = {
  count: number;
  setCount: Dispatch<SetStateAction<number>>;
};

const CounterContext = React.createContext<CounterContextType | undefined>(undefined);
```

初始化时, 使用默认参数`undefined`进行占位, 那我们的 App 组件也要进行相对应的修改。

最终的代码就是这样子:

```tsx
const App = ({ children }) => {
  const [count, setCount] = useState(0);
  return (
    <CounterContext.Provider value={{ count, setCount }}>
      <Counter />
    </CounterContext.Provider>
  );
};

const Counter = () => {
  const { count, setCount } = useContext(CounterContext);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
    </div>
  );
};
```

看起来很棒,我们实现了计数器。

那我们来看看加上 hooks 的 Context 可以怎么优化, 现在开始准备起飞了。

## ContextProvider

我们可以先封装一下`CounterContext.Provider`, 我希望`App`组件只是将所有组件做一个组合, 而不是做逻辑管理, 想象一下你的`App`组件有 n 个 Context(UIContext、AuthContext、ThemeContext 等等), 如果在这里进行 Context 管理那是多么痛苦的事情,每一个 Context 都需要一个 useState, 而`App`组件也失去了原始的意义。

```tsx
const App = ({ children }) => {
  const [ui, setUI] = useState();
  const [auth, setAuth] = useState();
  const [theme, setTheme] = useState();

  return (
    <UIContext.Provider value={{ ui, setUI }}>
      <AuthContext.Provider value={{ auth, setAuth }}>
        <ThemeProvider.Provider value={{ theme, setTheme }}>{children}</ThemeProvider.Provider>
      </AuthContext.Provider>
    </UIContext.Provider>
  );
};
```

太恐怖了, 现在我们来封装一下:

```tsx
export type Children = { children?: React.ReactNode };

export const CounterProvider = ({ children }: Children) => {
  const [count, setCount] = useState(0);

  return <CounterContext.Provider value={{ count, setCount }}>{children}</CounterContext.Provider>;
};
```

然后我就可以直接使用`CounterProvider`进行提供 counter,看起来很棒。

```tsx
const App = () => {
  return (
    <CounterProvider>
      <Counter />
    </CounterProvider>
  );
};
```

## read context hook

第二个可以优化的地方是: 提取出读取 context 的 hook,有两个理由:

1. 不想每次都导入一个`useContext`和一个`CounterProvider`读取 counter 的值, 代码变得更加精简.
2. **我想要我的代码看起来是我做了什么, 而不是我怎么做**。

所以这里我们可以使用自定义 hook 来实现这个功能。

```tsx
export const useCounter = () => {
  const context = useContext(CounterProvider);
  return context;
};
```

在`Counter`组件中可以直接使用这个 hook 来读取 CounterContext 的值。

```tsx
const { count, setCount } = useCounter();
// Property 'count' does not exist on type 'CounterContextType | undefined'
// Property 'setCount' does not exist on type 'CounterContextType | undefined'
```

但是我们直接使用的时候发现有类型错误, 再仔细一想, 在`createContext`中声明了有两个类型`CounterContextType`和`undefined`, 虽然我们在`ContextProvider`的时候已经注入了 `count` 和 `setCount` 。 但是 ts 并不能保证我们一定会有值, 所以我们怎么办呢?

我们可以在`useCounter`中做一层[类型保护](https://www.typescriptlang.org/docs/handbook/2/narrowing.html), 通过类型保护来缩小我们的类型, 从而知道是什么类型了。

```tsx
export const useCounter = () => {
  const context = useContext(CounterContext);
  if (context === undefined) {
    throw new Error('useCounter must in CounterContext.Provider');
  }
  return context;
};
```

我们使用`context === undefined`来实现缩小类型,其实它还有另外一个更加重要的作用: **检测当前使用`useCounter hook`的组件是否被正确的使用**。

正如上一篇[props drilling](https://itsuki.cn/article/165)文章所说的, 使用 Context 会受到约束, 也就是说, 如果使用了 `useCounter hook` 的组件在没有包裹在 `CounterProvider` 组件树中 , 那么读取到的值其实就是`createContext`时候的初始值(在这里的例子中也就是`undefined`)。 `undefined` 再去解构赋值是无法成功的, 所以这个时候就可以通过这一判断来防止这个问题。

## 读写分离

在讲解为什么进行读写分离时, 我们先来修改一下代码。

### 代码

```tsx
export type Children = { children?: React.ReactNode };
export type CounterContextType = {
  count: number;
  setCount: Dispatch<SetStateAction<number>>;
};

const CounterContext = React.createContext<CounterContextType['count'] | undefined>(undefined);

const CounterDispatchContext = React.createContext<CounterContextType['setCount'] | undefined>(
  undefined,
);

export const CountProvider = ({ children }: Children) => {
  const [count, setCount] = useState(0);

  return (
    <CounterDispatchContext.Provider value={setCount}>
      <CounterContext.Provider value={count}>{children}</CounterContext.Provider>;
    </CounterDispatchContext.Provider>
  );
};

export default CounterContext;
```

我们再提取出对应的读取 context 的 hooks。

```tsx
export const useCountDispatch = () => {
  const context = useContext(CounterDispatchContext);
  if (context === undefined) {
    throw new Error('useCountDispatch must be in CounterDispatchContext.Provider');
  }
  return context;
};

export const useCount = () => {
  const context = useContext(CounterContext);
  if (context === undefined) {
    throw new Error('useCount must be in CounterContext.Provider');
  }
  return context;
};
```

然后我们现在有两个组件:

- `CounterDisplay`: 负责展示
- `CounterAction`: 负责更新

### 为什么需要读写分离?

进行读写分离有两个好处:

1. 逻辑与状态分离。
2. Context 更新的时候所有订阅的子组件会 rerender, **减少不必要的 rerender**。

首先来说说逻辑与状态分离: 一个展示的组件只需要 `count` 就好了, 我不需要读取多余的 `setCount` 更新函数, 这对我来说是没有意义的, 而对于更新操作来说, 我可以通过`setCount(v => v + 1)`的方式来读取之前的值, 不需要再访问`count` 。

更重要的其实是后面一点: Context 的更新方法`setCount`往往不会改变, 而更新方法`setCount`控制的状态`count`却会改变。 如果你创建一个同时提供状态值和更新方法时, 那么所有订阅了该上下文的组件都会进行 rerender, 即使它们只是访问操作(可能还没有执行更新)。

所以我们可以避免这种情况, 将更新操作和状态值拆成两个上下文, 这样子依赖于更新操作的组件`CounterAction`不会因为状态值`count`的变化而进行没有**必要的渲染**。

我们来看看代码:

```tsx
// CounterDisplay.tsx
const CounterDisplay = () => {
  const count = useCount();
  return <h1>{count}</h1>;
};

// CounterAction.tsx
const CounterAction = () => {
  const setCount = useCountDispatch();

  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);

  // 只会执行一遍
  useEffect(() => {
    console.log('CounterAction render');
  });

  return (
    <div>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
    </div>
  );
};
```

Context rerender 的解决方案还有其他的,但是我自己个人是更加喜欢这种, 具体可以看看这个 [issue](https://github.com/facebook/react/issues/15156#issuecomment-474590693)。

## createCtx

现在看看上面的代码, 两个 Context 进行读写分离就这么多模版代码了, 显然这不符合我们的初衷, 那么我们可以使用一个函数`createCtx`创建 Context。

```tsx
function createCtx<T>(initialValue: T) {
  const storeContext = createContext<T | undefined>(undefined);
  const dispatchContext = createContext<Dispatch<SetStateAction<T>> | undefined>(undefined);

  const useStore = () => {
    const context = useContext(storeContext);
    if (context === undefined) {
      throw new Error('useStore');
    }
    return context;
  };

  const useDispatch = () => {
    const context = useContext(dispatchContext);
    if (context === undefined) {
      throw new Error('useDispatch');
    }
    return context;
  };

  const ContextProvider = ({ children }: PropsWithChildren<{}>) => {
    const [state, dispatch] = useState(initialValue);

    return (
      <dispatchContext.Provider value={dispatch}>
        <storeContext.Provider value={state}>{children}</storeContext.Provider>
      </dispatchContext.Provider>
    );
  };

  return [ContextProvider, useStore, useDispatch] as const;
}
export default createCtx;
```

然后如何使用呢?

```tsx
import createCtx from './createCtx';

const [CountProvider, useCount, useCountDispatch] = createCtx(0);

export { CountProvider, useCount, useCountDispatch };
```

其他地方一行也不要改就能实现!!!

### 多上下文

在真实项目中, 我们不止一个 Context, 那么怎么组合呢? 有了上面这个工具函数就很简单了。

```tsx
// theme.ts
const [ThemeProvider, useTheme, useThemeDispatch] = createCtx({ theme: 'dark' });
// ui.ts
const [UIProvider, useUI, useUIDispatch] = createCtx({ layout: '' });

// app.tsx
const App = ({ children }) => {
  return (
    <ThemeProvider>
      <UIProvider>{children}</UIProvider>
    </ThemeProvider>
  );
};
```

美滋滋有木有!!!

## action hooks

你以为到这里就结束了吗, nonono, 还有呢!

我们看到`CounterAction`组件。

```tsx
const increment = () => setCount(c => c + 1);
const decrement = () => setCount(c => c - 1);
```

这是不是很眼熟, 我们来把它封装成 hook。

```tsx
const useIncrement = () => {
  const setCount = useCountDispatch();
  return () => setCount(c => c + 1);
};

const useDecrement = () => {
  const setCount = useCountDispatch();
  return () => setCount(c => c - 1);
};
```

然后`CounterAction`组件就变成了:

```tsx
const CounterAction = () => {
  const increment = useIncrement();
  const decrement = useDecrement();

  return (
    <div>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
    </div>
  );
};
```

是不是很有意思, 当然假设我们还需要请求后端, 我们可以写一个 async hook。

```tsx
const useAsyncIncrement = () => {
  // 复用之前的hook
  const increment = useIncrement();

  return () =>
    new Promise(resolve =>
      setTimeout(() => {
        increment();
        resolve(true);
      }, 1000),
    );
};
```

```tsx
const CounterAction = () => {
  // ...
  const asyncIncrement = useAsyncIncrement();

  return (
    <div>
      {/* ... */}
      <button onClick={asyncIncrement}> async +1 </button>
    </div>
  );
};
```

## createReducerCtx

使用 Context + useReducer 我个人觉得更加适合我上面所说的: 我想知道我做了什么, 而不是我如何做。

```ts
dispatch({ type: 'increment' });

setCount(v => v + 1);
```

这两个我更喜欢第一种, 表达力更强。

当 Context value 太复杂时, 可以使用 useReducer 来进行管理, 我们可以改造一下`createCtx`来实现创建一个`createReducerCtx`。

```tsx
function createReducerCtx<StateType, ActionType>(
  reducer: Reducer<StateType, ActionType>,
  initialValue: StateType
) {
  const stateContext = createContext<StateType | undefined>(undefined);
  const dispatchContext = createContext<Dispatch<ActionType> | undefined>(undefined);

  const useStore = () => {
    // ...
  };
  const useDispatch = () => {
    // ...
  };

  const ContextProvider = ({ children }: PropsWithChildren<{}>) => {
    const [store, dispatch] = useReducer(reducer, initialValue);

    return (
      // ...
    );
  };

  return [ContextProvider, useStore, useDispatch] as const;
}
```

总体上和之前的差不多, 只是有一点点不同, 我们看看怎么使用。

```tsx
type CounterActionTypes = { type: 'increment' } | { type: 'decrement' };

function reducer(state = 0, action: CounterActionTypes) {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    default:
      return state;
  }
}

const [ReducerCounterProvider, useReducerCount, useReducerCountDispatch] = createReducerCtx(
  reducer,
  0,
);

export const useIncrement = () => {
  const setCount = useReducerCountDispatch();
  return () => setCount({ type: 'increment' });
};

export const useDecrement = () => {
  const setCount = useReducerCountDispatch();
  return () => setCount({ type: 'decrement' });
};

export const useAsyncIncrement = () => {
  const increment = useIncrement();

  return () =>
    new Promise(resolve =>
      setTimeout(() => {
        increment();
        resolve(true);
      }, 1000),
    );
};
```

完美!!!

## 性能

当 context value 进行更新的时候, 会进行 rerender , 所以有可能会出现性能的问题。 当我们的组件因为 context value 出现了性能问题的时候, 我们得看看有多少组件因为这个改变需要重新渲染, 然后判断这个 context value 改变的时候, 它们是否需要真的被重新渲染。

就像上面的`CounterAction`就没有 rerender, 因为它不需要`count`这个 context value,而更新函数其实一直都是不变的, 所以通过这种方式来达到避免 rerender(更加准确的来说是不必要的 rerender), 但是如果我们的组件渲染不会涉及到 Dom 更新以及副作用时, 那么这些组件可能在进行无意义的渲染。这其实在 React 当中是很常见的,它本身通常不是问题(这也是我们所说的不要过早地进行优化)。

如果真的出现了问题, 不妨试试这个:

1. 拆分, 将复合状态放到多个 Context 中, 这样子只是一个 context value 的改变只会影响到依赖的那部分组件, 而不是全部组件。
2. 优化 ContextProvider, 很多时候我们不必将 ContextProvider 放到全局, 它可能是一个局部的状态, 尽可能地把状态和需要它的地方放的近一些。
3. 不应该通过 Context 来解决所有的**状态共享问题**, 就像 React 官方文档所说的, 它只是用来共享数据, **它一直都不是状态管理!!!** 在大型项目中使用 Redux 或者其他库会让状态管理更加得心应手。

## 总结

使用 Context 之前思考一下, 是否真的需要呢? 解决[props drilling](https://itsuki.cn/article/165)问题? 是否可以使用组合解决? 有了`Context + useReducer`我们可能会想还需要`Redux`嘛?

当然需要, 就像 Redux 所推荐的: [全局状态放到 Redux, React 放本地状态](https://redux.js.org/tutorials/essentials/part-2-app-structure#component-state-and-forms), 如果并不是全局状态而需要多个组件共享状态时, 可以尝试使用 Context, 一个技术的出现从来都不是为了取代另一个技术, 而是多一个场景的解决方案。

虽然说的是 Context + hooks 如何使用, 但是本质上还是在说如何用 hooks 来复用逻辑, 我们可能还在使用 Class 的思维方式去写 hooks, 只是用 hooks 来减少一些代码. 希望这篇文章对你有所帮助!!!

这里[源代码]()

## 参考文章

- [react 新版官方文档](https://beta.reactjs.org/learn/passing-data-deeply-with-context)
- [魔术师卡颂-useContext 更佳实践](https://mp.weixin.qq.com/s/dFSpWCP1kLX4MtEeBb3eaA)
- [Context 读写分离的好处](https://stackoverflow.com/questions/66717457/split-up-context-into-state-and-update-to-improve-performance-reduce-renders)
- [redux](https://redux.js.org/tutorials/essentials/part-2-app-structure#component-state-and-forms)
