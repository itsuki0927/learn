# csrf

## 是什么

跨站请求伪造(CSRF)是一种常见的网络攻击方式, 它是在用户已经通过身份验证(比如说已经登陆的用户)的网站上执行一些不需要的操作。 一般来说, 它不会窃取用户数据, 而是利用用户在没有用户意愿的情况下进行操作。它可以引导更改其个人资料中的电子邮件地址或密码，甚至执行汇款等操作。

## 有什么影响

csrf 攻击的影响取决于攻击的用户以及在当前应用程序中的权限。

- 普通用户: 成功的 csrf 攻击通常会引入状态更改请求, 例如更改密码或用户信息、转账等操作。

- 管理员: 如果是更高权限的用户, 那么 csrf 攻击可能会导致系统受损, 后果更加严重。

## 与 xss 有什么不同

它们是不同的, 尽管它们可以一起结合使用。

要使 csrf 攻击成功, 用户必须在当前应用程序中进行身份验证, 然后攻击者可以通过该应用程序发送请求。 它被看作是一种"单向"攻击, 因为请求只能与 csrf 一起发送而不能被接收, 也就是无法查看其响应数据。这种攻击利用了应用程序或网站对用户的信任。

另一方面, xss 不需要身份验证, 只需要用户访问特定网站, 然后在用户浏览器执行 xss 脚步。 因为它允许发送和接收请求, 所以它被视为"双向"攻击。 这种攻击利用了用户对网站的信任。

## 攻击方式

### get 请求的 csrf

某些应用可以用**get 请求**的方式进行更改, 比如说: 更新密码, 在这种情况下, 攻击者发送恶意链接, 用户点击这个链接, 中招了, 或者使用`img`标签的 src 属性加载某一个 url 下的资源来执行攻击。

```html
<img src="http://xxxx.com/update" />
```

在任何一种情况下，浏览器的 GET 请求都将在后台执行以实现所需的结果, 而不会向用户表明这已经发生, 因为我们一般用 GET 请求来获取数据, 而不是其他操作, 但是 csrf get 获取不到响应内容, 所以 CSRF GET 请求漏洞利用相对微不足道, 但如果你用 GET 方式来更新或者删除操作时, 得注意了!!!

### post 请求的 csrf

对 CSRF 使用 GET 和 POST 请求的主要区别在于**提交请求的方式**。攻击者使用 HTTP POST 的原因是应用程序中的状态更改请求经常以这种方式发出, 将数据放到请求体 body 里面发送给服务端。

攻击者一般使用`<form>`表单来进行提交 post 请求, 这可能需要用户手动提交请求, 可以通过欺骗或者伪装的方式点击按钮来实现, 或者攻击者可能会在网站中嵌入 JavaScript, 自动 submit 执行表单请求。

## 流程

那么它的攻击流程是什么呢?

1. 攻击者引导用户执行操作, 比如说点击一个链接访问恶意网站等。
2. 在恶意网站通过点击按钮等操作代替用户向网站发送 Http 请求。
3. 如果用户在网上进行了身份验证, 则该请求将作为用户发送的合法请求进行处理。

正如上面流程所说, 网站收到 CSRF 攻击并不一定会攻击成功。 用户还必须在当前网站上进行了身份验证。 实际上, CSRF 漏洞依赖于经过身份验证的会话管理。

通常情况下, Web 应用程序是的会话管理是基于 cookie, 当用户登陆之后, 将验证的信息存放到 cookie, 后面每次请求服务端时, 都会携带上 cookie 进行验证, 如果验证通过, 则处理该请求。

## 举例子

这么说可能有点抽象, 来举一个例子, 假设有一个论坛网站, 可以进行发表评论等操作, 可以访问这个[在线 demo](), 来看看效果。

**只是为了讲清楚 csrf 攻击,所以尽可能的简单, 更加专注于 csrf 本身, 而不是其他东西。**

所以这个 demo 没有实现实际的登陆流程, 我们关注的重点是用户登录以后的事情, 通过点击登陆按钮就可以实现用户登陆, 用户登陆以后, 可以看到以下页面。

![图片]()

可以点击个人信息来跳转到更新个人信息页面, 点击保存可以保存用户信息。

![图片]()

具体的代码可以看这个[仓库]()

### 为什么 csrf 会攻击成功呢?

我们可以更新用户信息, 点击个人信息按钮进入到页面, 输入姓名和邮箱点击保存即可更新成功, 这看起来没什么问题, 但是假如说这个更新的请求不是由用户发起的呢? 而是由黑客通过某种操作来发送这个网络请求, 而 csrf 攻击就是要用户在不知情的情况下发送网络请求。

在攻击之前, 我们先用`node server.js` 运行我们的黑客代码, 访问`http://localhost:4000`端口,即可看到这样子的界面, 在现实世界中可能就是一封邮件等其他方式的手段, 点击这个链接, 它会进入到一个黑客页面, 然后你点击了提交按钮然后它就会跳转到`/user`页面, ok, 攻击结束。

什么? 我点击一个链接就攻击成功了? 什么也没做?

在解释之前, 我们不妨来看看代码。

#### 前端代码

我们来看看代码, 更新用户信息的代码比较简单。 就是输入用户名、邮箱点击保存即可更新成功。

```html
<form method="post" action="user">
  <div>
    <label for="username">姓名:</label>
    <input name="username" type="text" value="<%= username %>" class="thin" />
  </div>

  <div>
    <label for="email">邮箱:</label>
    <input name="email" type="email" value="<%= email %>" class="thin" />
  </div>
  <button type="submit" class="thin">保存</button>
</form>
```

#### 后端代码

再来看到对应的后端代码, 判断用户是否登陆, 只有登陆了才会执行后面的操作, 把新的用户名和邮箱更新到 session 中。 我这里就没有做真正的验证, 只是通过 session 的字段以及通过简单的判断来验证用户是否登陆。

```js
app.post('/user', function (req, res) {
  if (req.session.isValid && req.cookies.auth === req.session.username) {
    req.session.email = req.body.email;
    res.redirect('/user');
  } else {
    res.redirect('/');
  }
});
```

#### 攻击者代码

我们再来看看这个看起来无害的链接是如何实现的。 可以看到其实就是一个隐藏的表单, 这个表单的`action`指向的就是个人信息页面, 然后点击"查看详情"就会提交表单跳转到个人信息页面。

```html
<form method="post" action="http://localhost:3000/user">
  <input type="hidden" name="username" value="攻击者" />
  <input type="hidden" name="email" value="攻击者邮箱" />
</form>
<a href="#" onclick="document.forms[0].submit()">点击查看详情</a>
```

#### 为什么这样子就攻击成功了呢?

看到服务端代码, 可以看到如果当前网站用户没有登陆的话, 因为这个时候`session`是没有信息的, 这种方式是不会有影响的, 后端会拒绝更改用户的个人资料, 也就是它会进入到`else`语句当中。

```js
if (req.session.isValid) {
  // false
} else {
  res.redirect('/');
}
```

但是, 如果用户登陆了, 这个更改会被看起来是合法请求, 并且会正确响应。

我们再来看看用户登陆时的代码。

```js
app.get('/login', function (req, res) {
  res.cookie('auth', 'itsuki');
  req.session.isValid = true;
  req.session.username = 'itsuki';
  req.session.email = 'itsuki@qq.com';
  res.redirect('/');
});
```

点击登陆之后, 给前端设置一个 cookie, 保存验证信息, 然后后端将验证信息保存起来, 下次登陆时携带认证信息进行验证。

由于用户浏览器上的 cookie 保存了当前用户的验证信息, 每次请求时会自动携带上这个 cookie 来做判断是否是当前登陆的用户。 当有漏洞的网站收到更新请求时, 它因为有正确的 cookie, 理所当然的代表了是当前这个用户, 对应的这个请求也是合法的。

即使攻击者无法直接访问受害者的网站, 它们也可以利用用户和 csrf 漏洞来执行未经授权的操作。

所以现在就明白了吧, 为什么我点击这个链接就可以攻击成功了。

### 隐藏 csrf 攻击

上面说了, 点击链接之后会看到黑客网站, 然后跳转到访问的网站, 当用户点击之后看到不是目标网站还是黑客网站时, 可能已经意外到: 坏了, 被攻击了。 那如何隐藏 csrf 攻击, 也就是没有这一段白屏的时间呢?

```js
<script>document.forms[0].submit();</script>
```

## 防范

csrf 重要的是 cors site, 也就是所谓的跨站, 你怎么样确定服务端允许接收某些特定的网站的请求, 拒绝掉其他请求, 这是第一个要做的,第二个就是 request, csrf 攻击能够成功的另外一部分是因为攻击者模拟了一个请求, 怎么要做到无法模拟, 这也可以抵御一部分。

### referer

第一个就是 referer, 在 http 请求中有一个请求头 referer, 它标记了当前请求的 origin, 我们可以通过检查这个请求头来验证是不是合法的 domain, 不合法的就直接拒绝。

但是使用这个需要注意, 第一个就是有些浏览器可能不会携带 referer; 第二个就是浏览器提供了关闭 referer 的接口(比如说使用`<meta name=”referrer” content=”never”>`来绕过 referer 验证), 你可能刚好关闭这个, 这个时候即使你是正确的 domain 也会被拒绝; 第三个就是 referer 检查时必须要做到没有 bug。

```js
const referer = request.headers.referfer;

if (referer.includes('xxx.com')) {
}
```

上面这段代码看起来没问题, 但是如果是`xxx.com.xxx`子域名呢? 这个检查就失效了, 所以检查 referer 并不是一个很完善的解法, 至少你需要考虑到所有的场景, 在没有考虑到所有场景之前这个都是不安全的, 问题是真的能考虑到所有场景吗?。

### 验证码

第二个就是验证码, 我们在使用一些应用的时候, 有碰到手机验证码的情况, 你只有填写了正确的验证码才会发送请求, 多了一道这样子的检查确保不会 csrf 攻击。 这是一个完善的防御方法, 但是还是有一个缺点:对于每个请求用户如果都得来一遍接收验证码然后输入, 用户估计会烦死 😡。

### csrf token

csrf token 其实就是一个随机值, 它就是证明我正在从服务端生成的页面中发送网络请求, 而不是其他地方, 换句话来说: 服务端生成某一个页面的时候, 会在页面上注入一个标记(csrf token), 然后客户端提交表单时带上这个标记(csrf token), 如果相同的话就验证通过处理该请求, 如果不同或者出现丢失说明是 csrf 攻击, 拒绝掉该请求。

```js
<form>
  // ...
  <input type='hidden' value='randomValue' />
</form>
```

服务端注入了一个标记, 攻击者在构建表单的时候,是无法知道这个随机值, 所以这个时候就无法模拟出提交的数据, 即使提交表单数据是对的, 因为没有这个标记, 所以就无法造成 csrf 攻击。

### double submit token

csrf token 看起来挺不错的, 但是服务器还有保存一份, 才能保证正确性, 但是如果应用程序过大的话, 后端需要记住几十万甚至上百万的 token , 难以想象会有多大压力, 那能不能不需要服务端保存一份呢? 不妨可以试试 double submit token 。

它的前半部分和 csrf token 挺像的, 一开始也是服务端会生成一个标记, 注入到客户端页面上, 但是这个它还会将这个标记, 添加到客户端 cookie 中, 所以客户端就保留了两份相同的标记, 但是位置不一样, 一个是在页面中的某一个地方, 一个是存在 cookie 中。

```js
// Set-Cookie: csrfToken=randomValue

<form>
  ...
  <input type='hidden' value='randomValue' />
</form>
```

想想 csrf 攻击的请求和本人发出的请求有什么不一样? 前者是来自不同的 domain , 而后者来自相同的 domain , 我们只要判断是否来自相同的 domain 即可, double submit token 就是利用的这个思路。

当用户按下提交按钮之后 ,服务端比较 cookie 中 token 和表单 token 就可以了, 检查是否有值且相等, 就知道是不是用户发出的请求了。

为什么呢? 攻击者如果想要产生 csrf 攻击, 可以在表单中自己写一个标记 , 这看起来没问题, 还需要在 cookie 中设置相同值的标记, 但是因为同源策略, 他在自己 domain(`a.com`) 下不能设置用户网站(`b.com`)上的 cookie, 这个时候攻击者模拟发出的请求 cookie 中是没有 token 中, 所以攻击就会失效。

这种方法看起来有效, 实际上还是会有问题, 如果攻击掌握了你底下任何一个子域名, 就可以帮你写 cookie , 举个例子。

- 前端域名是`a.com`, 后端是`api.a.com`, 那么前端`a.com`如果拿不到后端`api.a.com`的时候, 就无法完成验证。
- 所以 cookie 必须写在`a.com`根域名下, 这样子每个子域名都可以访问。
- **任何一个子域名都可以修改`a.com`中的 cookie**
- 如果某一个子域名`static.a.com`有漏洞遭到了 xss 攻击, 没有造成危害, 但写入了 cookie。
- 攻击者就可以使用自己写的 token, 对 xss 中招的用户在进行 csrf 攻击。

### same-site

Cookie 的 SameSite 属性用来限制第三方 Cookie，从而减少安全风险。

它有三个取值:

- Strict
- Lax
- None

#### Strict

严格模式, 表明这个 cookie 在任何情况下都不能作为第三方 cookie, 比如说`b.com`设置了如下 cookie:

```txt
Set-Cookie: foo=1;SameSite=Strict
Set-Cookie: bar=2
```

我们在`a.com`网站下发起对`b.com`的网络请求, foo 这个 cookie 是不会被包含到请求中, 但是 bar 会。

#### Lax

宽松模式, 比 Strict 放宽了限制, 假如这个请求是下面表格(阮一峰老师 🐮🍺)的请求类型的话, 允许 cookie 作为第三方 cookie

| 请求类型  | 示例                                 | 正常情况    | Lax         |
| --------- | ------------------------------------ | ----------- | ----------- |
| 链接      | `<a href="..."></a>`                 | 发送 Cookie | 发送 Cookie |
| 预加载    | `<link rel="prerender" href="..."/>` | 发送 Cookie | 发送 Cookie |
| GET 表单  | `<form method="GET" action="...">`   | 发送 Cookie | 发送 Cookie |
| POST 表单 | `<form method="POST" action="...">`  | 发送 Cookie | 不发送      |
| iframe    | `<iframe src="..."></iframe>`        | 发送 Cookie | 不发送      |
| AJAX      | `$.get("...")`                       | 发送 Cookie | 不发送      |
| Image     | `<img src="...">`                    | 发送 Cookie | 不发送      |

#### None

之前 Chrome 浏览器默认采用的就是 None, 如果设置为 None 的话, 必须使用`https`协议才会携带, 否则无效。

```txt
Set-Cookie: foo=1;SameSite=None;Secure
```

所以我们可以使用 Lax 属性来防范 csrf 攻击。

## 参考资料

- [阮一峰老师的 samesite](https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)
- [前端安全系列（二）：如何防止 CSRF 攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)
- [要我们谈一谈 csrf](https://blog.techbridge.cc/2017/02/25/csrf-introduction/)
- [XSS vs CSRF](https://portswigger.net/web-security/csrf/xss-vs-csrf)
- [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [double submit token](https://security.stackexchange.com/questions/59470/double-submit-cookies-vulnerabilities)
- [Prevent Cross-Site Request Forgery (CSRF) Attacks](https://auth0.com/blog/cross-site-request-forgery-csrf/#CSRF-Defenses-Strategies)
- [How to protect your website using anti-CSRF tokens](https://www.netsparker.com/blog/web-security/protecting-website-using-anti-csrf-token/)
- [WHAT IS CROSS-SITE REQUEST FORGERY (CSRF) & HOW TO PREVENT IT](https://crashtest-security.com/cross-site-request-forgery-csrf/)
