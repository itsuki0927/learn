# 数组高阶函数

## map

map api: 返回一个新数组, 这个新数组的每个元素是调用一次提供的函数后的返回值。

举个例子, 如果有一个`{ name:'xx', age:0 }`的对象数组, 我们只需要 name 组成的数组, 那么这个时候可以使用 map api

```js
// 生成 3 个长度的 [{},{},{}]
const list = Array.from({ length: 3 }, (_, i) => ({ name: `name-${i + 1}`, age: 14 + i }));

const nameList = list.map(item => item.name);

console.log(nameList);
// 输出
// ['name-1', 'name-2', 'name-3']
```

## filter

filter api: 将满足函数的列表项筛选出来, 返回一个新数组

举个例子, 我想把`age>20`的 item 筛选出来

```js
const list = [1, 2, 3, 4, 5];

const thanFiveList = list.filter(v => v > 3);

console.log(thanFiveList);

// 输出
// [4, 5]
```

## some

some api: 如果数组中**有一个元素满足提供的函数**就返回 true, 否则就返回 false

```js
const list = [1, 2, 3, 4, 5];

const existLargest5 = list.some(v => v > 3);
const existLargest7 = list.some(v => v > 7);

console.log(existLargest5); // true
console.log(existLargest7); // false
```

## every

every api: 如果数组中**每一个元素都满足提供的函数**就返回 true, 否则就返回 false

every 和 some 可以理解成互斥的

```js
const list = [1, 2, 3, 4, 5];

const existLargest5 = list.every(v => v > 0);
const existLargest7 = list.every(v => v > 4);

console.log(existLargest5); // true
console.log(existLargest7); // false
```

## find

find api: 如果数组中**有一个元素满足提供的函数**就返回该元素, 否则就返回 undefined

```js
const list = [1, 2, 3, 4, 5];

const existLargest5 = list.find(v => {
  console.log(v); // 1 , 2
  return v > 1;
});
const existLargest7 = list.find(v => v > 4);

console.log(existLargest5); // 2
console.log(existLargest7); // undefined
```

## findIndex

findIndex api: 如果数组中**有一个元素满足提供的函数**就返回该元素的下标, 否则就返回 -1

```js
const list = [1, 2, 3, 4, 5];

const existLargest5 = list.findIndex(v => {
  console.log(v); // 1 , 2
  return v > 1;
});
const existLargest7 = list.findIndex(v => v > 4);

console.log(existLargest5); // 1
console.log(existLargest7); // -1
```
