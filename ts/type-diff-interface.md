# type、interface

## 扩展

interface 使用 extends 进行扩展

```ts
interface Person {
  name: string;
}

interface Student extends Person {
  age: number;
}
```

type 使用`&`交叉类型扩展

```ts
type Person = {
  name: string;
};
type Student = Person & {
  age: number;
};
```

## 添加新字段

interface 允许通过重复声明添加新字段

```ts
interface Person {
  name: string;
}
interface Person {
  age: number;
}

const p: Person = {
  name: 'lsl',
  age: 20,
};
```

type 创建后不允许更改

```ts
type Person = {
  name: string;
};
type Person = {
  age: number;
};
// 报错, 重复声明
```

## 区别

1. 在 Ts4.2 之前, type 可能会出现在错误信息中,有时会代替相应的匿名类型,
   而接口在错误信息中总是会被命名.

```ts
// 接口错误信息
interface Mammal {
    name: string
}
function echoMammal(m: Mammal) {
    console.log(m.name)
}
echoMammal({  name: 12343 })
 <!-- The expected type comes from property 'name' which is declared here on type 'Mammal' -->

// type 错误信息出现在Lizard
type Lizard = {
    name: string
}
function echoLizard(l: Lizard) {
    console.log(l.name)
}
echoLizard({ name: 12345})
<!-- The expected type comes from property 'name' which is declared here on type 'Lizard' -->

// 4.2以前版本 type不会出现在错误信息中
type Arachnid = Omit<{ name: string; legs: 8 }, 'legs'>;
function echoSpider(l: Arachnid) {
  console.log(l.name);
}
echoSpider({ name: 12345, legs: 8 });
<!-- The expected type comes from property 'name' which is declared here on type 'Pick<{ name: string; legs: 8; }, "name">' -->
```

2. type 不会做类型合并, 而 interface 可以
3. interface 只能用来声明对象类型, 不能重命名基本类型
4. 接口名称将始终以原始形式出现在错误信息中, 但仅当它们按名称使用时.
