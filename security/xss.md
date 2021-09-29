# xss

跨站脚本攻击, 指的是用户在网站交互时利用恶意脚本进行攻击, 攻击者绕过了浏览器的同源策略, 一般攻击者伪装成受害者, 执行用户的任何操作, 访问用户的所有数据, 如果受害者是管理员等重要权限人员的话, 那么攻击者就是获得该网站的所有功能和数据

## 如何工作?

xss 攻击原理就是: 对一个易受攻击的网站, 将恶意的 JavaScript 返回给用户, 在用户在浏览网页时, 攻击者可以完全破坏他们与网站之间的交互

## 反射型

反射型 xss 是最简单的一种 xss 攻击方式, 当应用程序接收 HTTP 请求中的数据并以不安全的方式将该数据包含在即时响应中时，就会出现这种情况。

假设有一个搜索功能, 然后它接收一个参数来表示搜索的关键字
`https://xxx.com?keywords=name`

网页需要对这个关键字进行显示:

`<p>keywords: name</p>`

如果网页不进行任何处理的话, 如果关键字内容是脚步的话就会造成 xss 攻击, 如下

搜索路径:
`https://xxx.com?keywords=<script>alert(1)</script>`
html 内容:

```html
<p>
  keywords:
  <script>
    alert(1);
  </script>
</p>
```

如果应用程序的另一个用户请求攻击者的 URL，那么攻击者提供的脚本将在受害者用户的浏览器中，在他们与应用程序的会话上下文中执行。

此时攻击者注入的脚本内容会在网站中执行, 想像一下此时执行的不是简单的`alert`, 还是其他操作, 会有什么后果呢?

## 存储型

存储型 xss: 当应用程序从不受信任的来源接收数据并以不安全的方式在其后续 HTTP 响应中包含该数据时, 就会出现存储型 xss 攻击

有问题的数据可能 http 请求提交给应用程序的, 比如说: 博客文章评论, 聊天室的用户昵称以及客户的联系方式

举一个例子: 用户在博客下评论, 评论的信息会展示给其他用户看

`<p>some comment</p>`

如果不对评论信息做任何处理的话, 就会出现这样子的情况

```html
<p>
  <script>
    // xss
    alert(1);
  </script>
</p>
```

## 基于 Dom

基于 Dom 的 xss 是在应用程序包含一些客户端 JavaScript 时产生的，这些 JavaScript 通常通过将数据写回 DOM，以不安全的方式处理来自不受信任源的数据。

在以下示例中，应用程序使用一些 JavaScript 从输入字段读取值并将该值写入 HTML 中的元素：

```js
const input = document.querySelector('input');
const button = document.querySelector('button');
const result = document.querySelector('#result');
button.addEventListener('click', () => {
  result.innerHTML = input.value;
});
```

如果攻击者可以控制输入字段的值，他们就可以很容易地构造一个恶意值来执行自己的脚本：

```js
result.innerHTML = `<img src='x' onerror='alert(1)'/>`;
```

在典型情况下，输入字段将从 HTTP 请求的一部分填充，例如 URL 查询字符串参数，允许攻击者以与反射 XSS 相同的方式使用恶意 URL 进行攻击。
