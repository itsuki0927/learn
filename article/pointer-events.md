# pointer-events

## 场景

在公司项目中遇到了这样子的一个场景: 有一个商品(有图标、价格等信息), 当悬浮到图标时, 要有一个悬浮的 tooltips 显示它的详细信息, 商品禁用时, 会有一个黑色的遮罩盖在上面, 此时也不能选中, 然后问题就来了, 我又想悬浮到图标上显示 tooltips, 但是上面又有一个黑色的遮罩, 怎么办呢?

代码如下(为了防止代码过长, 删除了一些标签):

```html
<style>
  .container {
    width: 500px;
    height: 300px;
    background: red;
    position: relative;
  }
  .mask {
    width: 100%;
    height: 100%;
    background: green;
    position: absolute;
    z-index: 2;
  }
  .content {
    width: 500px;
    height: 300px;
    background: blue;
  }
</style>
<body>
  <div class="container">
    <div class="mask"></div>
    <p class="content">我是内容</p>
  </div>

  <script>
    const mask = document.querySelector('.mask');
    const container = document.querySelector('.container');
    const content = document.querySelector('.content');

    content.addEventListener('mouseenter', () => {
      console.log('content mouseenter');
    });
    mask.addEventListener('mouseenter', () => {
      console.log('mask mouseenter');
    });
    container.addEventListener('mouseenter', () => {
      console.log('container mouseenter');
    });
  </script>
</body>
```

然而, 它现在的效果就是我悬浮上去时, 打印顺序如下:

```js
// container mouseenter
// mask mouseenter
```

当时, 我思前想后, 一直在想这个怎么解决, 上面又有一个遮罩, 但是`mouseenter`要在`content`上触发, 实在搞不明白了, 我就问了一下导师, 她告诉我试一下`pointer-events`, 我当时自己写了 demo, 然后来了句: 妙啊.

## pointer-events 是什么呢?

我们来看看 Mdn 上它的定义

> pointer-events CSS 属性指定在什么情况下 (如果有) 某个特定的图形元素可以成为鼠标事件的触发者 .

它的值有很多:

```css
/* Global values */
pointer-events: inherit;
pointer-events: initial;
pointer-events: unset;

/* Keyword values */
pointer-events: auto;
pointer-events: none;
/* 以下的属性仅适用于SVG */
pointer-events: visiblePainted;
pointer-events: visibleFill;
pointer-events: visibleStroke;
pointer-events: visible;
pointer-events: painted;
pointer-events: fill;
pointer-events: stroke;
pointer-events: all;
```

而要解决上面这个问题, 只需要在`<div class='mask'>`上添加一个`pointer-events: none`即可, 我们先来看看打印顺序:

```js
// container mouseenter
// content mouseenter
```

为什么呢? 我们再来看看 Mdn 文档上关于`pointer-events:none`的解释:

> 元素永远不会成为鼠标事件的触发者。但是，当其后代元素的 pointer-events 属性指定其他值时，鼠标事件可以指向后代元素，在这种情况下，鼠标事件将在捕获或冒泡阶段触发父元素的事件侦听器。

所以我们在`.mask`上添加了这个属性, 它就不会成为鼠标事件的触发者了, 这样子就鼠标事件的触发者就顺利的成为了`.content`, 就成功的解决了这个问题.

但是我们看到这个定义, **但是, 当其后代元素也指定了这个属性, 并且不是 none 的话, 鼠标事件可以指向后代元素, 剩下的省略...**, 我们再来写个 demo 试试:

```html
<style>
  .mask-child {
    width: 200px;
    height: 300px;
    background: yellow;
    pointer-events: auto;
  }
</style>
<body>
  <div class="container">
    <div class="mask">
      <div class="mask-child"></div>
    </div>
    <p class="content">我是内容</p>
  </div>

  <script>
    const maskChild = document.querySelector('.mask-child');

    maskChild.addEventListener('mouseenter', () => {
      console.log('maskChild mouseenter');
    });
  </script>
</body>
```

在刚刚的代码加上这一部分, 我们再来看看, 需要注意的是我们给`.mask`加了一个孩子`.mask-child`, 然后也监听它的`mouseenter`事件, 我们再来看看效果:

```js
// container mouseenter
// mask mouseenter
// maskChild mouseenter
```

不出意外的, 还是一样会触发`mask`的`mouseenter`事件, 所以我们就知道了, 如果你想跳过某一个元素的指定事件, 我们可以在它上面设置一个`pointer-events:none`, 但是需要注意的是, 其后代不要再指定`pointer-events`属性, 不然还是会失效的.

## 示例

1. 不会触发鼠标事件(拖动、悬浮、点击等事件).

```html
<style>
  button {
    pointer-events: none;
  }
</style>
<button onclick="handleClick()">测试</button>
<script>
  function handleClick() {
    console.log('handleClick');
  }
</script>
```

并不会打印出 handleClick.

2. 还有如果设置了`:hover`、`:active`等伪类也不会触发.

像之前我们如果想给一个禁用的元素进行取消`hover`效果, 我们可能会这样子写:

```html
<p class="content disable"></p>

<style>
  .content:not(disable):hover {
  }
</style>
```

而现在我们可以直接这么写了, 是不是简单一点呢?

```html
<p class="content disable"></p>

<style>
  .content.disable {
    pointer-events: none;
  }
  .content:hover {
  }
</style>
```

其实还有更简单一点的方法, 就是把 `.disable`放在下面, 因为 css 的优先级啦.

```html
<p class="content disable"></p>

<style>
  .content:hover {
    background: black;
  }
  .content.disable {
    background: yellow;
  }
</style>
```

## 总结

以上就是该文章的全部内容啦, 算是给自己做一个小小的记录, 希望能够帮助到你.
