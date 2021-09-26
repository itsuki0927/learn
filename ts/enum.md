# 枚举

enum 是 Ts 当为数不多的特性之一, 它允许我们去定义一组命名常量, 相对于一个变量来说, 使用 enum 能够更加清晰明了, 也能让自己和别人明白其意图.

## 基本使用

### 数字枚举

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

这样子就定义了一个 number enum, 如果赋初始值的话, 默认就是从 0 开始的, 所以这里`Up = 0`, `Down = 1`,
`Left = 2`, `Right = 3`

如果给它赋初始值的话, 就是枚举属性值以递增的方式计算

```ts
enum Direction {
  Up = 5,
  Down,
  Left,
  Right,
}
```

所以这里与之对应的就是 `Up = 5`, `Down = 6`, `Left = 7`, `Right = 8`

还有一种情况

```ts
enum Direction {
  Up,
  Down = 5,
  Left,
  Right,
}
```

这种就是前面两种的结合版, 前面以 0 进行初始化, 然后后面有值再以该值递增初始化, 所以`Up = 0`, `Down = 5`, `Left = 6`, `Right = 7` 这样子的

这就是 number enum 的声明方式, 我们使用 number enum 的时候可能并不关心其值是多少, 但是枚举的值都不一样的时候, 使用其递增的方式更加合适

### 字符串枚举

string enum 是 枚举的另外一种方式, 它和 number enum 差不多, 不同的是使用 string enum 的时候, 必须为其赋初始值

```ts
enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT',
}
```

string enum 是没有其递增的行为的, 但是 string enum 的好处在于可以很好的 "序列化", 换句话来说就是: 我们使用 number enum 进行定义枚举类型的时候, 读取 number enum 的值其实是不透明的也可以理解成它本身不会传递任何有用的信息, string enum 允许你在读取的时候提供有意义且可读的值, 与枚举本身的名称无关

## 指定枚举值

## 数字枚举的缺点

### 缺点一: 打印

### 缺点二: 宽松的类型检查

### 用字符串枚举代替数字枚举

## 枚举使用案例

## 运行时的枚举类型

### 反向映射

### 字符串枚举在运行时的表现

## 常量枚举

## 编译时的枚举类型

## 枚举类型的替换方式
