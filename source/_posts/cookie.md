title: cookie 详解
categories: 学习
meta:
  date: true
tags:
  - http js  
---

HTTP Cookie（也叫Web Cookie或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。

<!-- more -->


## 简介

HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

### Cookie 主要用于以下三个方面：

+ 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
+ 个性化设置（如用户自定义设置、主题等）
+ 浏览器行为跟踪（如跟踪分析用户行为等）

Cookie曾一度用于客户端数据的存储，因当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie渐渐被淘汰。由于服务器指定Cookie后，浏览器的每次请求都会携带Cookie数据，会带来额外的性能开销（尤其是在移动环境下）。新的浏览器API已经允许开发者直接将数据存储到本地，如使用 Web storage API （本地存储和会话存储）或 IndexedDB 。


## 创建Cookie

当服务器收到HTTP请求时，服务器可以在响应头里面添加一个Set-Cookie选项。浏览器收到响应后通常会保存下Cookie，之后对该服务器每一次请求中都通过Cookie请求头部将Cookie信息发送给服务器。另外，Cookie的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

### `Set-Cookie`响应头部和`Cookie`请求头部
服务器使用`Set-Cookie`响应头部向用户代理（一般是浏览器）发送Cookie信息。一个简单的Cookie可能像这样：
```
Set-Cookie: <cookie名>=<cookie值>
```


通过 Node.JS (koa2 框架)服务端程序的设置`Set-Cookie` 响应头信息 :


```js
/* 服务器端 */

/**
  * ctx.cookies.set(name, value, [options])
  * 通过 options 设置 cookie name 的 value
  * maxAge 一个数字表示从 Date.now() 得到的毫秒数
  * signed cookie 签名值
  * path cookie 路径, 默认是'/'
  * domain cookie 域名
  * secure 安全 cookie
  * httpOnly 服务器可访问 cookie, 默认是 true
  * overwrite 一个布尔值，表示是否覆盖以前设置的同名的 cookie (默认是 false)
  * 如果是 true, 在同一个请求中设置相同名称的所有 Cookie（不管路径或域）是否在设置此Cookie 时从 Set-Cookie 标头中过滤掉。
  */
router.post("/login", function(ctx, next) {

  const { userName, password } = ctx.request.body;
  ctx.cookies.set("userName", userName, { httpOnly: false });

  ctx.cookies.set("sessionId", 9527, {
    httpOnly: true, // 是否 document.cookies可获取
    maxAge: 10 * 1000 // 设置过期时间，单位是毫秒数
  });

  ctx.body = {
    message:"登录成功",
    result: true
  };
});

```

**http Response Headers**

```
Connection: keep-alive
Content-Length: 49
Content-Type: application/json; charset=utf-8
Date: Thu, 02 Jan 2020 08:48:08 GMT
Set-Cookie: userName=admin; path=/
Set-Cookie: password=123456; path=/
Set-Cookie: sessionId=9527; path=/; expires=Thu, 02 Jan 2020 08:48:18 GMT; httponly

```

现在，对该服务器发起的每一次新请求，浏览器都会将之前保存的Cookie信息通过Cookie请求头部再发送给服务器。

```
Accept: application/json, text/plain, */*
Content-Length: 40
Content-Type: application/json;charset=UTF-8
Cookie: userName=admin; password=123456
```


### Cookie的`Secure` 和`HttpOnly` 标记

标记为 `Secure` 的 Cookie 只应通过被HTTPS协议加密过的请求发送给服务端。但即便设置了 `Secure` 标记，敏感信息也不应该通过 Cookie 传输，因为 Cookie 有其固有的不安全性，`Secure` 标记也无法提供确实的安全保障。从 Chrome 52 和 Firefox 52 开始，不安全的站点（http:）无法使用 Cookie 的 `Secure` 标记。

为避免跨域脚本 (XSS) 攻击，通过 JavaScript 的 Document.cookie API无法访问带有 `HttpOnly` 标记的 Cookie，它们只应该发送给服务端。如果包含服务端 Session 信息的 Cookie 不想被客户端 JavaScript 脚本调用，那么就应该为其设置 `HttpOnly` 标记。


```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

### Cookie的作用域 `Domain` 和 `Path`

`Domain` 和 `Path` 标识定义了 Cookie 的作用域：即Cookie应该发送给哪些URL。

`Domain` 标识指定了哪些主机可以接受 Cookie。如果不指定，默认为当前文档的主机（不包含子域名）。如果指定了 `Domain` ，则一般包含子域名。

例如，如果设置 Domain=mozilla.org，则Cookie也包含在子域名中（如developer.mozilla.org）。

`Path` 标识指定了主机下的哪些路径可以接受Cookie（该URL路径必须存在于请求URL中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。

例如，设置 Path=/docs，则以下地址都会匹配：

```
/docs
/docs/Web/
/docs/Web/HTTP
```


### `SameSite` Cookies 

`SameSite` Cookie 允许服务器要求某个cookie在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）。

SameSite cookies是相对较新的一个字段，[所有主流浏览器都已经得到支持](https://developer.mozilla.org/en-US/docs/Web/HTTP/headers/Set-Cookie#Browser_compatibility)。

下面是例子：

```
Set-Cookie: key=value; SameSite=Strict
```

SameSite可以有下面三种值：

**None**
浏览器会在同站请求、跨站请求下继续发送 cookies ，不区分大小写。

**Strict**
浏览器将只发送相同站点请求的 cookie (即当前网页URL与请求目标URL完全一致)。如果请求来自与当前 location 的 URL 不同的 URL，则不包括标记为 `Strict` 属性的 cookie。

**Lax**
在新版本浏览器中，为默认选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者frames的调用，但只有当用户从外部站点导航到URL时才会发送。如link链接

**注意：**

```yml
以前，如果SameSite属性没有设置，或者没有得到运行浏览器的支持，那么它的行为等同于None，Cookies会被包含在任何请求中——包括跨站请求。

但是，在新版本的浏览器中，SameSite的默认属性是SameSite=Lax。换句话说，当Cookie没有设置SameSite属性时，将会视作SameSite属性被设置为Lax——这意味着Cookies将会在当前用户使用时被自动发送。如果想要指定Cookies在同站、跨站请求都被发送，那么需要明确指定SameSite为None
```

### JavaScript 通过 `Document.cookie` 访问 Cookie

通过 `Document.cookie` 属性可创建新的 Cookie ，也可通过该属性访问非 HttpOnly 标记的 Cookie 。

```js
document.cookie = "yummy_cookie=choco"; 
document.cookie = "tasty_cookie=strawberry"; 
console.log(document.cookie); 
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
```

### http 请求跨域时，如何能够使请求传递 cookie ？



在 Web 页面中可以随意地载入跨域的图片、视频、样式等资源， 但 AJAX 请求通常会被浏览器应用同源安全策略，禁止获取跨域数据，以及限制发送跨域请求。默认情况下浏览器对跨域请求不会携带 Cookie，但鉴于 Cookie 在身份验证等方面的重要性， CORS 推荐使用额外的响应头字段来允许跨域发送 Cookie。


#### 客户端代码

在`open XMLHttpRequest`后，设置`withCredentials`为`true`即可让该跨域请求携带 Cookie。 注意携带的是目标页面所在域的 Cookie。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.withCredentials = true;
xhr.send();
```

**jQuery 的使用方式**
```js
$.ajax({
   url: a_cross_domain_url,
   xhrFields: {
      withCredentials: true
   }
});
```

**axios 的使用方式**
```js
// create an axios instance
const service = axios.create({
  withCredentials: true, // 是否允许跨域携带cookie
  baseURL: "/", // api 的 base_url
  headers: { token },
  timeout: 1000 * 35 // 超时时间-35秒
});
```

#### Access-Control-Allow-Credentials

只设置客户端当然是没用的，还需要目标服务器接受你跨域发送的 Cookie。 否则会被浏览器的同源策略挡住。服务器同时设置`Access-Control-Allow-Credentials`响应头为"true"， 即可允许跨域请求携带 Cookie。

#### Access-Control-Allow-Origin

除了`Access-Control-Allow-Credentials`之外，跨域发送 Cookie 还要求 `Access-Control-Allow-Origin`不允许使用通配符。 事实上不仅不允许通配符，而且只能指定单一域名，否则，浏览器还是会挡住跨域请求。

#### 计算 Access-Control-Allow-Origin

既然`Access-Control-Allow-Origin`只允许单一域名， 服务器可能需要维护一个接受 Cookie 的 Origin 列表， 验证 Origin 请求头字段后直接将其设置为`Access-Control-Allow-Origin`的值。 （这一实践来自 Stackoverflow） 值得注意的是在 CORS 请求被重定向后 Origin 头字段会被置为 null。 此时可以选择从Referer 头字段计算得到 Origin。

在正确配置的情况下，在 Chrome Network 就可以看到 Cookie 请求头被跨域发送了 （注意 Host 和Referer不同域，但仍然带了Cookie）：
```
Accept:*/*
Accept-Encoding:gzip, deflate, sdch, br
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6,nl;q=0.4,zh-TW;q=0.2,fr;q=0.2,de;q=0.2,ja;q=0.2
Connection:keep-alive
Cookie:auhtor:harttle; _gat=1; _ga=GA1.1.221305049.1482947002
Host:dest.com:4001
Origin:http://index.com:4001
Referer:http://index.com:4001/
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36
```

__参考资料：__

[HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)

[CORS 跨域发送 Cookie](https://harttle.land/2016/12/28/cors-with-cookie.html)