# 类组件

在`typescript`中,`React.Component` 是一个范型(`React.Component<PropType, StateType>`),
你可以传递一个可选项的 props 和 state 类型参数.

## 基本使用

举个 🌰:

```tsx
import { Component } from 'react';

interface MyProps {
  message: string;
}
interface MyState {
  count: number;
}

class App extends Component<MyProps, MyState> {
  // 可选的第二个注释，用于更好的类型推断
  state: MyState = {
    count: 0,
  };

  constructor(props: MyProps) {
    super(props);
  }

  render() {
    return (
      <div>
        {this.props.message}
        {this.state.count}
      </div>
    );
  }
}
```

看到上面你可能会想为什么`React.Component<MyProps,MyState>`写了`state`的类型声明,
还要在`state:MyState`写第二次呢?

假设有这样子的情况.

```tsx
import { Component } from 'react';

interface MyProps {
  message: string;
}
// 这里只写了count
interface MyState {
  count: number;
}
interface ExpandState extends MyState {
  name: string;
}

class App extends Component<MyProps, MyState> {
  // 这里使用ExpandState
  state: ExpandState = {
    name: 'age',
    count: 2,
  };

  // 没问题
  handleChangeCount = () => {
    this.setState({ count: 2 });
  };

  // 出现类型错误, name 不存在 MyState
  handleChangeName = () => {
    this.setState({ name: 'new-name' });
  };

  render() {
    return (
      <div>
        {this.props.message}
        {this.state.count}
      </div>
    );
  }
}
```

看上面的例子就可以知道错误出现在`handleChangeName`方法, 为什么会出现这个问题呢?

`Component<Props,State>`提供第二个参数是为了`setState` 方法可以有类型提示以及定义 state 类型, 这是基类`Component`的实现,
但是组件初始化声明`state`的时候可以使用类型声明覆盖掉, 也就是上面的`state:ExpandState`,

当初始化的`state`类型(ExpandState)与`Component<Props,State>`组件(MyState)声明的类型不一致时就会出现问题(setState 的时候), 所以必须确保告诉编译器实际上没有做任何不同的事情,保持一致性, 或者你自己确保 state 只需要用到`State`类型的属性。

我们来看一下`@types/react`中`Component`的定义, `Component<MyProps,MyState>`

```ts
class Component<P, S> {
  state: Readonly<S>;
  setState<K extends keyof S>(
    state:
      | ((prevState: Readonly<S>, props: Readonly<P>) => Pick<S, K> | S | null)
      | (Pick<S, K> | S | null),
    callback?: () => void,
  ): void;
}
```

看到这里你就能明白为什么`Component<P,S>`第二个范型参数是用来干什么的了,
正如上面所说就是为了定义 state 类型, 以及 setState 的类型.

## 函数

就跟平时那样子去使用, 一样需要类型声明.

```tsx
import { Component } from 'react';

interface MyProps {
  message: string;
}
interface MyState {
  count: number;
}

class App extends Component<MyProps, MyState> {
  state = {
    count: 2,
  };

  handleChangeCount = (step: number) => {
    this.setState({ count: 2 });
  };

  render() {
    return (
      <div>
        <button onClick={() => this.handleChangeCount(1)}>增加</button>
        {this.props.message}
        {this.state.count}
      </div>
    );
  }
}
```

## 类属性

如果是声明类属性, 就跟上面例子 state 一样的声明, 但是可以不需要进行赋值.

```tsx
import { Component } from 'react';

class App extends Component<void, void> {
  count: number = 0; // 声明时赋初始值
  name: string;

  constructor(props: void) {
    super(props);
    this.name = 'init'; // 构造函数赋值
  }

  render() {
    return (
      <div>
        {this.count}
        {this.name}
      </div>
    );
  }
}
```

## getDerivedStateFromProps

### 使用 getDerivedStateFromProps 返回值必须符合

```tsx
class Comp extends React.Component<Props, State> {
  static getDerivedStateFromProps(props: Props, state: State): Partial<State> | null {
    // ...
  }
}
```

### 如果函数的返回时候再确认 state 类型

```tsx
class Comp extends React.Component<Props, ReturnType<typeof Comp['getDerivedStateFromProps']>> {
  static getDerivedStateFromProps(props: Props) {}
}
```

### 当您需要带有其他状态字段和记忆的派生状态时(不懂)

```tsx
type CustomValue = any;
interface Props {
  propA: CustomValue;
}
interface DefinedState {
  otherStateField: string;
}
type State = DefinedState & ReturnType<typeof transformPropsToState>;
function transformPropsToState(props: Props) {
  return {
    savedPropA: props.propA, // save for memoization
    derivedState: props.propA,
  };
}
class Comp extends React.PureComponent<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      otherStateField: '123',
      ...transformPropsToState(props),
    };
  }
  static getDerivedStateFromProps(props: Props, state: State) {
    if (isEqual(props.propA, state.savedPropA)) return null;
    return transformPropsToState(props);
  }
}
```
