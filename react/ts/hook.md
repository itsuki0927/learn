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

有效的 `React` 节点与 `React.CreateElement` 的返回结果不同。无论组件最终渲染，`React.createElement` 始终返回一个对象，它是 `JSX.Element` 接口，但 `react.createElement` 是组件的所有可能返回值的集合。

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

### useRef

在`typescript`中, `useRef` 返回的引用要么是只读`RefObject`的, 要么是可变`MutableRefObject`的,
这取决于参数类型是否完成覆盖了初始值.

#### DOM element ref

访问一个 DOM 元素: 只提供元素类型作为参数, 使用 null 作为初始值. 在这种情况下,
返回的引用将是一个只读的`.current`

```tsx
import { useRef, useEffect } from 'react';

function Foo() {
  // 参数类型尽可能的具体, 使用`HTMLDivElement`比`HTMLElement`要更加具体
  // `HTMLDivElement`没有包含`null`, 所以是返回的只读的`RefObject<HTMLDivElement>`
  const divRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    // 缩小类型, 使用`if`语句排除`null`的可能性
    if (!divRef.current) throw Error('divRef is not assigned');

    console.log(divRef.current);
  });

  return <div ref={divRef}>etc</div>;
}
```

如果你可以确保`divRef.current`不为`null`, 可以在初始化时使用非空断言`!`

```tsx
const divRef = useRef<HTMLDivElement>(null!);
// 不需要做非null检查
console.log(divRef.current);
```

请注意!!! 这里还有两点补充.

1. 在这里使用非空断言选择了类型安全, 如果没有将`divRef` 绑定到一个元素,
   或者如果引用的元素被条件渲染时, 会产生运行时错误.
2. 并且这里的`divRef`是一个`MutableRefObject<HTMLDivElement>`.

第一点, 直接看下面的 🌰.

```tsx
import React, { useRef, useEffect } from 'react';
interface Example2Props {
  shouldRender: boolean;
}

// 忘记绑定到元素
const Example1 = () => {
  const divRef = useRef<HTMLDivElement>(null); // RefObject<HTMLDivElement>
  useEffect(() => {
    console.log(divRef.current); // null
  });
  return <div> etc </div>; // 忘记把divRef给绑定到div上了
};

// 条件渲染
const Example2 = ({ shouldRender }: Example2Props) => {
  const divRef = useRef<HTMLDivElement>(null); // RefObject<HTMLDivElement>
  useEffect(() => {
    console.log(divRef.current); // 只有当shouldRender为true的时候才会绑定上
  });
  return shouldRender ? <div ref={divRef} /> : <div> etc </div>;
};
```

第二点, 我们来看一下`useRef`的 ts 定义.

```tsx
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

看到上面的定义, 结合下面的例子

```tsx
useRef<HTMLDivElement>(null!); // MutableRefObject
useRef<HTMLDivElement>(null); // RefObject
useRef<HTMLDivElement | null>(null); // MutableRefObject
```

想想为什么是这样子?

1. ref1 初始值是非空(使用了非空断言), 所以匹配第一个类型声明, 返回`MutableRefObject`
2. ref2 初始值是`null`, 所以匹配第二个类型声明, 返回`RefObject`
3. ref3 初始值是`null`, 传递的类型的是`HTMLDivElement|null`,
   所以匹配第一个类型返回`MutableRefObject`

`Ref` 需要更加具体的类型, 如果只使用`HTMLElement`其实是不够的,
像上面的例子来说使用`HTMLDivElement`才不会出现类型错误
![](https://user-images.githubusercontent.com/6764957/116914284-1c436380-ac7d-11eb-9150-f52c571c5f07.png)

#### 可变的 ref

具有可变的 ref: 提供所需的类型, 并且确保初始值是包含在提供的类型中.

```tsx
import { useRef, useEffect } from 'react';

function Foo() {
  // 将返回 MutableRefObject<number|undefined>
  const intervalRef = useRef<number>();

  useEffect(() => {
    // 可以给它赋值
    intervalRef.current = setInterval(() => {}, 1000);
    return () => clearInterval(intervalRef.current);
  }, []);

  return <button onClick={() => {}}>Cancel timer</button>;
}
```

## useImperativeHandle

```tsx
import React, { useImperativeHandle, useRef, forwardRef, useEffect } from 'react';

type ListRef<ItemType> = { scrollToItem(item: ItemType): void };

type ListProps<ItemType> = {
  items: ItemType[];
  innerRef?: React.Ref<ListRef<ItemType>>;
};

function List<ItemType>(props: ListProps<ItemType>) {
  useImperativeHandle(props.innerRef, () => ({
    scrollToItem() {},
  }));
  return null;
}

type FancyInputRef = {
  focus: () => void;
};
const FancyInput = forwardRef<FancyInputRef, any>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current?.focus();
    },
  }));
  return <input ref={inputRef} />;
});

const ListDemo = () => {
  const ref = useRef<ListRef<number>>(null);
  const inputRef = useRef<FancyInputRef>(null);

  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  });

  return (
    <div>
      <List items={[1, 2, 3, 4, 5]} innerRef={ref} />
      <FancyInput ref={inputRef} />
    </div>
  );
};
```

### 自定义 hooks

如果在指定 hooks 中返回一个数组, 默认情况下会自动推断出一个联合类型,
但是实际上我们可能希望推断出一个元祖, 这个时候可以使用`const`断言.

```tsx
// type '10'
let x = 10 as const;

// type 'readonly [10,20]'
let y = [10, 20] as const;

// type '{readonly text: "hello"}'
let z = { text: 'hello' } as const;
```

```tsx
import { useState } from 'react';

export function useLoading() {
  const [isLoading, setState] = useState(false);
  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };
  return [isLoading, load] as const;
}
```

当然我们也可以使用类型断言`as`的方式.

```tsx
import { useState } from 'react';

export function useLoading() {
  const [isLoading, setState] = useState(false);
  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };
  return [isLoading, load] as [boolean, (aPromise: Promise<any>) => Promise<any>];
}
```

如果大量的自定义 hook 都是返回数组的话, 可以实现一个辅助函数`tuplify`.

```tsx
import { useState } from 'react';
const tuplify = <T extends any[]>(...elements: T) => elements;

const useToggle = () => {
  const [value, setValue] = useState(false);
  const toggle = () => setValue(!value);

  return [value, toggle]; // (boolean | (() => void))[]
};

const useToggleTuple = () => {
  const [value, setValue] = useState(false);
  const toggle = () => setValue(!value);

  return tuplify(value, toggle); // [boolean, ()=>void]
};
```

如果返回值大于两个以后的 hook 返回对象更加合适, 而不是返回元祖

```tsx
import { useState } from 'react';

const useCounter = (initialValue?: number) => {
  const [count, setCount] = useState(initialValue || 0);

  const increment = () => setCount(x => x + 1);
  const decrement = () => setCount(x => x - 1);
  const reset = () => setCount(initialValue || 0);

  return {
    count,
    increment,
    decrement,
    reset,
    setCount,
  } as const;
};
```
