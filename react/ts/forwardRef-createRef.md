# forwardRef/createRef

## 基本使用

### createRef

```tsx
class ThemeProvider extends React.PureComponent<Props> {
  private rootRef = React.createRef<HTMLDivElement>();

  render() {
    return <div ref={this.rootRef}>{this.props.children}</div>;
  }
}
```

### forwardRef

```tsx
type Props = {
  children: React.ReactNode;
  type: 'submit' | 'button';
};

export type Ref = HTMLButtonElement;

export const FancyButton = React.forwardRef<Ref, Props>((props, ref) => (
  <button ref={ref} type={props.type}>
    {props.children}
  </button>
));
```

`forwardRef` 返回的是`ref`是一个可变的, 所以我们可以给它赋值.

如果想要不可变, 可以使用`React.Ref`, 确保没有人给它赋值.

```tsx
type Props = { children: React.ReactNode; type: 'submit' | 'button' };
export type Ref = HTMLButtonElement;
export const FancyButton = React.forwardRef(
  (
    props: Props,
    ref: React.Ref<Ref>, // <-- here!
  ) => (
    <button ref={ref} type={props.type}>
      {props.children}
    </button>
  ),
);
```

## forwardRef 通用型组件

上面讲述了`forwardRef`如何使用, 假设我们有一个可以点击列表组件, 它的 props 是一个范型,
这可能就会有问题.

```tsx
interface ListProps<T> {
  data: T[];
  onSelect: (item: T) => void;
}

function List<T>({ data, onSelect }: ListProps<T>) {
  return (
    <ul>
      {data.map(item => (
        <li key={item}>
          <button onClick={() => onSelect(item)}>选择</button>
        </li>
      ))}
    </ul>
  );
}
```

基本使用

```tsx
const items = [1, 2, 3, 4];
<List
  data={items}
  onSelect={item => {
    console.log(item);
  }}
/>;
```

没有问题的, `data props`被推断出`number[]`, `onSelect` 推断出`(item:number)=>void`.

假如这个时候需要传入一个`ref`到内部的`ul`元素呢? 我们第一时间会想到使用`React.forwardRef`方法,
我们来看看.

```tsx
function ListInner<T>({ data, onSelect }: ListProps<T>, ref: React.ForwardedRef<HTMLUListElement>) {
  return (
    <ul ref={ref}>
      {data.map(item => (
        <li key={item}>
          <button onClick={() => onSelect(item)}>选择</button>
        </li>
      ))}
    </ul>
  );
}

const List = React.forwardRef(ListInner);
```

这看起来是没有问题的, 但是我们知道`ListProps`是一个范型, `forwardRef<ListInner>`, 没有传入类型,
我们也知道我们是想在运行确定这个类型, 所以这个时候变成了`unknown`, 那么怎么去解决呢?

## 解决方式

### 类型断言

第一种就是使用`as`进行类型断言

```tsx
const List = React.forwardRef(ListInner) as <T>(
  props: ListProps<T> & { ref?: React.ForwardedRef<HTMLUListElement> },
) => ReturnType<typeof ListInner>;
```

在 Ts 中可以使用`as`来做类型断言, 其中语言中也有, 但是在写 Ts 的时候应该尽量避免使用这个,
它实质上就是告诉 Ts, "这里不需要做类型检查, 请相信我, 我知道我在干什么",在编译阶段就不会出现, 但是这样子,
我们就弱化了类型检查, 也是使用 Ts 的核心目的.

### 创建自定义 prop 来包装组件

我们知道`ref`是 React 的保留词, 我们可以使用自定义 props 来模拟类似的行为.

```tsx
interface ListProps<T> {
  data: T[];
  onSelect: (item: T) => void;
  listRef?: React.Ref<HTMLUListElement> | null;
}

function List<T>({ data, onSelect, listRef }: ListProps<T>) {
  return (
    <ul ref={listRef}>
      {data.map(item => (
        <li key={item}>
          <button onClick={() => onSelect(item)}>选择</button>
        </li>
      ))}
    </ul>
  );
}
```

这样子的方式添加了一个`listRef`的 api. 我们也可以使用一个包装器的组件来进行包装.

```tsx
function ListInner<T>({ data, onSelect }: ListProps<T>, ref: React.ForwardedRef<HTMLUListElement>) {
  return (
    <ul ref={ref}>
      {data.map(item => (
        <li key={item}>
          <button onClick={() => onSelect(item)}>选择</button>
        </li>
      ))}
    </ul>
  );
}

const ListWithRef = React.forwardRef(ListInner);

type ListWithRefProps<T> = ListProps<T> & {
  listRef?: React.Ref<HTMLUListElement>;
};

function List<T>({ listRef, ...props }: ListWithRefProps<T>) {
  return <ListWithRef ref={listRef} {...props} />;
}
```

### 增强 forwardRef 类型声明

Ts 中有一个高阶函数类型推断的特性, 它允许类型参数传播到外部函数.

我们可以自己重新声明和重新定义全局模块、命令空间以及接口声明, 它会做一个声明合并, 并且这个是只在模块生效, 不会影响到其他模块的`forwardRef`.

```tsx
declare module 'react' {
  function forwardRef<T, P = {}>(
    render: (props: P, ref: React.Ref<T>) => React.ReactElement | null,
  ): (props: P & React.RefAttributes<T>) => React.ReactElement | null;
}

export const List = React.forwardRef(ListInner);
```
