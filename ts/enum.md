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

string enum 是 枚举的另外一种方式, 它和 number enum 差不多, 不同的是使用 string enum 的时候, **必须为其赋初始值**

```ts
enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT',
}
```

string enum 是没有其递增的行为的, 但是 string enum 的好处在于可以很好的 "序列化", 换句话来说就是: 我们使用 number enum 进行定义枚举类型的时候, 读取 number enum 的值其实是不透明的也可以理解成它本身不会传递任何有用的信息, string enum 允许你在读取的时候提供有意义且可读的值, 与枚举本身的名称无关

### 混合枚举

最后一种是混合枚举, 也就是数字和字符串混合在一起使用

```ts
enum MixinEnum {
  Name = 'itsuki',
  age = 21,
}
```

混合枚举不怎么常用, 所以这里也是点到为止

### 总结

枚举的基本使用有三种: 数字枚举、字符串枚举、混合枚举, 所以可以看出枚举只支持数字、字符串两种类型, 不支持其它类型

## 枚举值的初始化

Ts 规定了三种对枚举值初始化的方式:

- 指定/忽略值进行初始化
  - 忽略值, 也就是默认进行初始化(仅限于数字枚举)
  - 指定数字/字符串进行初始化
- 常量枚举表达式进行初始化, 它可以在编译时进行求值
- 计算枚举表达式进行初始化, 通过任意表达式初始化。

### 指定/忽略值进行初始化

<!-- TODO:  -->

如果指定了值

### 常量枚举表达式进行初始化

Ts 中枚举值允许在编译的时候计算出来结果, 该成员值为常量, 通过这种方式进行初始化的, 可以理解成常量枚举表达式进行初始化, 我们可以隐式指定它的值(这种就是 Ts 为我们默认指定的值), 或者通过下面这种方式显式的指定它的值

- 显示指定数字/字符串
- 对先前定义的常量枚举成员的引用（可以源自不同的枚举）
- 带括号的常量枚举表达式
- 一元操作符: `+`, `-`, `~`
- 二元操作符: `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^`

```ts
enum Permission {
  UserRead = +'10', // 10
  UserWrite = -'2', // 2
  UserExecute = ~'3', // -4
  GroupRead = 2 + 2, // 4
  GroupWrite = 8 - 3, // 5
  GroupExecute = 4 * 4, // 16
  AllRead = 100 / 2, // 50
  AllWrite = 100 % 2, // 0
  AllExecute = 1 << 2, // 4
  CompanyRead = 125 >> 2, // 31
  CompanyWrite = 125 >>> 3, // 15
  CompanyExecute = 125 & 3, // 1
  StudentRead = 125 | 3, // 127
  StudentWrite = 125 ^ 3, // 126
}
```

### 计算枚举表达式进行初始化

计算枚举的值可以通过任意表达式来指定, 比如说

```ts
enum ComputeEnum {
  ONE = 1,
  NUM = Math.random(),
  LEN = '123'.length,
}
```

可以看到这样子可以正确的编译, 并且可以打印出来结果.

## 数字枚举的缺点

### 缺点一: 打印

打印数字枚举的成员时，我们只能看到数字：

```ts
enum NumEnum {
  NO,
  YES,
}

console.log(NumEnum.NO); // 0
console.log(NumEnum.YES); // 1
```

### 缺点二: 宽松的类型检查

如果使用数字枚举作为类型的时候, 任何数字也可以通过静态编译阶段

```ts
enum NumEnum {
  NO,
  YES,
}

const isYes = (num: NumEnum) => num === NumEnum.YES;

console.log(isYes(666)); // 编译通过
```

### 用字符串枚举代替数字枚举

```ts
enum StrEnum {
  YES = 'YES',
  NO = 'NO',
}

console.log(NumEnum.NO); // "NO"
console.log(NumEnum.YES); // "YES"
```

推荐使用字符串枚举

一方面是打印出来的信息更加具体、 有效
另一方面是类型检查更加严格

```ts
enum StrEnum {
  NO = 'NO',
  YES = 'YES',
}

const isYes = (str: StrEnum) => str === StrEnum.YES;

console.log(isYes('abc'));
// "abc" is not assignable to parameter of type 'StrEnum'.
```

## 枚举使用案例

### 位运算

### 多个常量

假如说有一组常量, 我们这么来表示:

```ts
const info = Symbol('info');
const debug = Symbol('debug');
const warn = Symbol('warn');
const error = Symbol('error');
```

更好的方式是使用枚举

```ts
enum Logger {
  info = 'info',
  debug = 'debug',
  warn = 'warn',
  error = 'error',
}
```

使用枚举的好处在于:

1. 常量名称被分组并嵌套在命名空间 Logger, 更好的组织代码
2. 如果需要其中一个常量的话, 可以使用枚举 Logger, 并且 Ts 还会做静态检查有没有使用其他值

### 比 boolean 更具有描述性

当布尔用于表示备选方案时，枚举通常是一种更具自描述性的选择。

假如说有一个属性值`success:true`来表示接口是否请求成功, 我们可能会用下面的代码

```ts
type Result = {
  success: boolean;
};
```

然而，枚举更有描述性，并且还有一个额外的好处，即如果需要，我们可以在以后添加更多的替代方案。

```ts
enum Success {
  YES,
  NO,
}

type Result = {
  success: Success;
};
```

## 运行时的枚举类型

### 反向映射

### 字符串枚举在运行时的表现

## 常量枚举

## 编译时的枚举类型

## 枚举类型的替换方式
