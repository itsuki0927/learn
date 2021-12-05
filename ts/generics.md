# 泛型

## hello, 泛型

假设要我们实现一个打印函数, 可以传入任意类型, 它的返回值就是输入, 类似于`echo`命令一样.

如果没有泛型, 我们必须给函数指定一个类型, 比如说: 字符串.

```ts
function identity(args: string): string {
  return args;
}
```

或者, 我们使用`any`来表现的更加通用, 因为上面的函数只适合`string`类型,
如果是`number`就会出现类型错误.

```ts
function identity(args: any): any {
  return args;
}
```

但是参数使用了`any`, 返回值也是`any`, 就失去了类型检查, 显然得不出来, 我们使用 Ts 的初衷就是有类型检查, 而不是"anyscript".

所以, 我们需要一种捕获参数类型的方法, 以便我们可以使用它来表示返回的内容.在这里, 我们使用类型变量,
一种特殊的变量, **它作用于类型而不是值**.

```ts
function identity<Type>(args: Type): Type {
  return args;
}
```

我们现在已经函数添加来一个类型变量`Type`, 这个`Type`能够捕获用户提供的类型, 以便我们可以使用这个信息. 在这里我们使用`Type`作为返回值. 所以 Ts 在类型检查时, 会判断函数 identity 的输入输出类型是不是一样的.

通过添加类型变量, `identity`函数变成通用的了, 因为它可以在一系列类型上工作.

正如上面所说, 输入输出都会做类型检查, 输入是`any`, 输出也是`any`, 就会出现问题.
而使用类型变量就保证了输入输出的类型是准确的.

如果写了泛型函数, 我们使用两种方式调用它.

第一种: 将所有参数(包括类型参数)传递给函数:

```ts
const output = identity<string>('myString');
```

这里我们显示的设置了`Type`为`string`作为函数调用的参数之一.

第二种: 可能是最常见的, 就是使用类型参数推断, 也就是编译器根据我们传入的参数类型自动为我们设置类型:

```ts
const output = identity('myString');
```

这里我们不需要在`<>`显式传递类型, 编译器通过参数 value:`myString`, 设置了`Type`的类型.

虽然类型参数推断可以是保持代码更短和更具可读性的效果，但当编译器无法推断类型时，我们可能还是需要想第一种一样显示的传递类型参数, 这可能发生在更加复杂的示例中.

## 使用泛型类型变量

当我们一开始使用泛型的时候, 比如说创建了像`identity`这样子的范型函数,
编译器会强制要求你在函数主体中正确使用任何泛型参数, 也就是说, 你实际上是把这些参数当作任何类型.

假设还有一个函数我们想将参数的长度打印出来, 我们可能会这么写:

```ts
function loggingIdentity<Type>(args: Type): Type {
  console.log(args.length); // length does not exist on type 'Type'
  return args;
}
```

当我们这么做的时候, 编译器会报错, 显然易见类型变量`Type`并没有`length`属性, 就像前面所说,
类型变量代表任何和所有类型, 因此假设我传入一个`number`, 但是`number`就没有`.length`成员.

所以我们改一下代码,接收一个数组的参数.

```ts
function loggingIdentity<Type>(args: Type[]): Type[] {
  console.log(args.length);
  return args;
}
```

这样子就没问题了, 因为数组有`.length`属性, 如果我们传入的是一个数字数组, 函数也会返回一个数字数组,
类型变量`Type`会绑定到`number`, **这允许我们将泛型变量绑定到我们正在使用类型的一部分,
而不是整个类型**, 从俄日为我们提供更大的灵活性.

我们也可以使用内置的泛型`Array`来实现.

```ts
function loggingIdentity<Type>(args: Array<Type>): Array<Type> {
  console.log(args.length);
  return args;
}
```

## 通用约束

有时候编写一个通用函数的时候, 想要获取类型参数上的某些属性, 很显然是不成功的, 比如上面`loggingIdentity`函数中想要访问`length`属性, 就会出现错误, 因为编译器无法保证每个类型都有`length`属性, 至少`boolean`和`number`没有.

那如果我们想要访问呢? 应该怎么去做呢? 可以使用类型约束, 这个时候就不是使用任何和所有类型,
而是希望函数限制使用具有`.length`属性的类型, 只要类型有这个属性, 就会通过检查, 至少需要有这个成员.
所以在上面例子中, 我们需要给`Type`做一层限制.

我们可是使用接口来描述这种约束, 然后使用`extends`关键来表示约束关系即可.

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(args: Type): Type {
  console.log(args.length);
  return args;
}
```

所以这个函数有了类型约束, 它将不适合与所有类型.

```ts
loggingIdentity(3); // 'error'
loggingIdentity(boolean); // 'error'
```

如果要正确使用则必须有`length`属性值.

```ts
loggingIdentity([]);
loggingIdentity('123');
loggingIdentity({ length: 10 });
loggingIdentity({ length: 10, value: 3 });
```

## 通用约束中使用类型参数

我们还可以声明受另一个类型参数约束的类型参数。例如，在这里我们想从一个给定名称的对象中获取一个属性。我们想确保我们不会意外地获取在 上不存在的属性 obj，因此我们将在两种类型之间放置一个约束：

```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, 'a');
getProperty(x, 'm');
```
