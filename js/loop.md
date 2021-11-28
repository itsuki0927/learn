# for vs for-in vs forEach vs for-of

如果要遍历数组可以使用`for`、`for-in` 、`forEach`、`for-of`, 那么这几种到底有什么区别呢?

## for

JavaScript 中的 for 循环很古老。它已存在于 ECMAScript 1 中。此 for 循环遍历数组的下标, 并且通过数组下标去读取元素

```js
const list = ['a', 'b', 'c'];

for (let i = 0; i < list.length; i++) {
  console.log(i, list[i]);
}

// 输出
// 0 'a'
// 1 'b'
// 2 'c'
```

优缺点:

1. 通用, 但是如果我们想要去循环数组的话, 比其他方式要冗余一点
2. 可以从指定下标开始循环遍历, 这比其他方式要好

## for in

for-in 循环与 for 循环一样古老，它也已经存在于 ECMAScript 1 中。与 for 循环不同的是它遍历的是 key

```js
const list = ['a', 'b', 'c'];
list.prop = 'd';

for (let k in list) {
  console.log(k);
}

// 输出
// '0'
// '1'
// '2'
// 'd'
```

如果用 for in 来遍历数组并不是一个好的选择,

1. for in 拿到的是 key, 对应的就是数组的下标
2. 它访问所有可枚举属性键（包括自己的属性/原型链上继承的属性），而不仅仅是数组元素的属性键

## forEach

考虑到 for 和 for-in 都不太适合在数组上循环，在 ECMAScript 5 中加入了一个 api forEach()

```js
const list = ['a', 'b', 'c'];
list.prop = 'd';

list.forEach(item => {
  console.log(item);
});

// 输出
// a
// b
// c
```

可以看到这个方法非常的方便, 它能使我们访问数组元素、数组下标以及整个数组, 加上 Es6 出现的箭头函数, 简化了 function 写法, 语法上更加的优雅

缺点:

1. 不能在 forEach 当中使用 await 关键字
2. 不能进行 break 退出循环

### 使用 some 来实现退出循环

如果要实现 forEach 退出循环的效果, 可以使用 some 这个 api, 它也会循环整个数组, 碰到第一个返回值为 true 的元素就停止循环

所以如果返回值都是 false 值的话, 我们可以理解成为是 forEach
如果返回值有一个是 true 值的话, 我们可以理解成为可中断循环的 forEach

```js
const list = ['a', 'b', 'c'];
list.prop = 'd';

list.some(el => {
  console.log(el);
});

// 输出
// 'a'
// 'b'
// 'c'

list.some(el => {
  if (el === 'b') return true;
  console.log(el);
});

// 输出
// 'a'
```

其实这种方式是一种 hack 方式, 引用 mdn 文档上 some 函数的定义

> 判断是不是至少有 1 个元素通过了被提供的函数测试。它返回的是一个 Boolean 类型的值。

所以 some api 其实是用来判断数组中是否有一个元素满足我们提供的验证函数, 用来做一个中断更新其实是对 some 的滥用, 但是也算是一个思维上的进步

## for of

for of 是 Es6 中新增的 api

```js
const list = ['a', 'b', 'c'];
for (let el of list) {
  console.log(el);
}

// 输出
// 'a'
// 'b'
// 'c'
```

可以看到 for of 在数组循环上很好用

1. 遍历的是数组的值
2. 可以使用 break 和 continue
3. 可以使用 await 关键字

### for of 与 迭代器属性的对象

for of 的另一个好处是，我们不仅可以在数组上循环，还可以在任何 iterable 对象上循环–例如，在 Map 上：

```js
const map = new Map([
  ['name', 'itsuki'],
  ['sex', '男'],
]);

for (const [k, v] of map) {
  console.log([k, v]);
}

// 输出
// 'name', 'itsuki'
// 'sex', '男'
```

或者自己定义迭代器属性来自定义自己的逻辑

```js
const obj = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  },
};

for (const k of obj) {
  console.log(k);
}
```

## 总结

- 如果想要从指定下标开始遍历, 使用 for 循环
- 如果想快速遍历一个数组, 使用 forEach+箭头函数
- 如果想遍历可迭代对象的话, 使用 for of
- 如果想要遍历对象, 使用 for in
