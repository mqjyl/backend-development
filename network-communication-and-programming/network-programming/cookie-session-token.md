# cookie、session、token

&#x20;`HTTP` 协议是一种**无状态协议**，即每次服务端接收到客户端的请求时，都是一个全新的请求，服务器并不知道客户端的历史请求记录；`Session` 和 `Cookie` 的主要目的就是为了弥补 `HTTP` 的无状态特性。

## :pencil2: 1、Cookie

HTTP 协议中的 Cookie 包括 Web Cookie 和浏览器 Cookie，它是服务器发送到 Web 浏览器的一小块数据。服务器发送到浏览器的 Cookie，浏览器会进行存储，并与下一个请求一起发送到服务器。通常，它用于判断两个请求是否来自于同一个浏览器，例如用户保持登录状态。 HTTP Cookie 机制是 HTTP 协议无状态的一种补充和改良。

### :pen\_fountain: 1.1、Cookie 主要用于下面三个目的

1. 会话管理：登陆、购物车、游戏得分或者服务器应该记住的其他内容
2. 个性化：用户偏好、主题或者其他设置
3. 追踪：记录和分析用户行为

Cookie 曾经用于一般的客户端存储。虽然这是合法的，因为**它们是在客户端上存储数据的唯一方法**，但如今建议使用现代存储 API。Cookie 随每个请求一起发送，因此它们可能会降低性能（尤其是对于移动数据连接而言）。&#x20;

### :pen\_fountain: 1.2、创建Cookie

当接收到客户端发出的 HTTP 请求时，服务器可以发送带有响应的 `Set-Cookie` 标头，Cookie 通常由浏览器存储，然后将 Cookie 与 HTTP 标头一同向服务器发出请求。

#### **Set-Cookie 和 Cookie 标头**

`Set-Cookie` HTTP 响应标头将 cookie 从服务器发送到用户代理，告诉客户端存储 Cookie。

如：

```
Set-Cookie: lu=Rg3vHJZnehYLjVg7qi3bZjzg; Expires=Tue, 15 Jan 2013 21:47:38 GMT; Path=/; Domain=.169it.com; HttpOnly
```

### :pen\_fountain: 1.3、Cookie 类型

有两种类型的 Cookies，一种是 Session Cookies，一种是 Persistent Cookies，如果 Cookie 不包含到期日期，则将其视为会话 Cookie。会话 Cookie 存储在内存中，永远不会写入磁盘，当浏览器关闭时，此后 Cookie 将永久丢失。如果 Cookie 包含有效期 ，则将其视为持久性 Cookie。在到期指定的日期，Cookie 将从磁盘中删除。

会话 Cookies：在 Session Cookies 中，用户的登录状态会保存在服务器的内存中。当用户登录时，Session 就被服务端安全的创建。在每次请求时，服务器都会从会话 Cookie 中读取 `SessionId`，如果服务端的数据和读取的 `SessionId` 相同，那么服务器就会发送响应给浏览器，允许用户登录。会话 Cookie 有个特征，客户端关闭时 Cookie 会删除，因为它没有指定`Expires`或 `Max-Age` 指令。但是，Web 浏览器可能会使用会话还原，这会使大多数会话 Cookie 保持永久状态，就像从未关闭过浏览器一样。

永久性 Cookies：永久性 Cookie 不会在客户端关闭时过期，而是在特定日期（Expires）或特定时间长度（Max-Age）外过期。例如`Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;`&#x20;

### ****:pen\_fountain: 1.4、**Cookie 的 Secure 和 `HttpOnly` 标记**

**`secure`**：表示该cookie只能用`https`传输。一般用于包含认证信息的cookie，要求传输此cookie的时候，必须用`https`传输。即使这样是安全的，也不应该将敏感信息存储在cookie 中，因为它们本质上是不安全的，并且此标志不能提供真正的保护。

**`HttpOnly` 的作用**

1. 会话 Cookie 中缺少 `HttpOnly` 属性会导致攻击者可以通过程序(`JS`脚本、Applet等)获取到用户的 Cookie 信息，造成用户 Cookie 信息泄露，增加攻击者的跨站脚本攻击威胁。
2. `HttpOnly` 是微软对 Cookie 做的扩展，该值指定 Cookie 是否可通过客户端脚本访问。
3. 如果在 Cookie 中没有设置 `HttpOnly` 属性为 true，可能导致 Cookie 被窃取。窃取的 Cookie 可以包含标识站点用户的敏感信息，如 ASP.NET 会话 ID 或 Forms 身份验证票证，攻击者可以重播窃取的 Cookie，以便伪装成用户或获取敏感信息，进行跨站脚本攻击等。&#x20;

### :pen\_fountain: 1.5、Cookie 的作用域&#x20;

**Cookie具有不可跨域名性。**Domain 和 Path 标识定义了 Cookie 的作用域：即 Cookie 应该发送给哪些 URL。Domain 标识指定了哪些主机可以接受 Cookie。如果不指定，默认为当前主机(不包含子域名）。如果指定了Domain，则一般包含子域名。

例如，如果设置 `Domain=mozilla.org`，则 Cookie 也包含在子域名中（如`developer.mozilla.org`）。

例如，设置 `Path=/docs`，则以下地址都会匹配：

* /docs&#x20;
* /docs/Web/&#x20;
* /docs/Web/HTTP

## :pencil2: 2、Session

Session 代表着服务器和客户端一次会话的过程。Session 对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的 Web 页之间跳转时，存储在 Session 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。当客户端关闭会话，或者 Session 超时失效时会话结束。至于客户端怎么保存Session 对象，可以有很多种方式，**对于浏览器客户端，默认采用 cookie 的方式。**

服务器第一次接收到请求时，开辟了一块 Session 空间（创建了Session对象），同时生成一个 `sessionId` （这个对象便是 Session 对象，存储结构为 `ConcurrentHashMap`），并通过响应头的 **`Set-Cookie：JSESSIONID=XXXXXXX` ** 命令，向客户端发送要求设置 Cookie 的响应； 客户端收到响应后，在本机客户端设置了一个 **`JSESSIONID=XXXXXXX` ** 的 Cookie 信息，该 Cookie 的过期时间为浏览器会话结束；接下来客户端每次向同一个网站发送请求时，请求头都会带上该 `Cookie` 信息（包含 `sessionId` ）， 然后，服务器通过读取请求头中的 `Cookie` 信息，获取名称为 `JSESSIONID` 的值，得到此次请求的 `sessionId`。

Cookie 和 Session 有什么不同？

* 作用范围不同，Cookie 保存在客户端（浏览器），Session 保存在服务器端，客户端只存储`SessionId`。`SessionID` 是连接 Cookie 和 Session 的一道桥梁，大部分系统也是根据此原理来验证用户登录状态。
* 存取方式的不同，Cookie 只能保存 ASCII，Session 可以存任意数据类型，一般情况下我们可以在 Session 中保持一些常用变量信息，比如说 `UserId` 等。
* 有效期不同，Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭或者 Session 超时都会失效。
* 隐私策略不同，Cookie 存储在客户端，比较容易遭到不法获取，早期有人将用户的登录名和密码存储在 Cookie 中导致信息被窃取；Session 存储在服务端，安全性相对 Cookie 要好一些。
* 存储大小不同， 单个 Cookie 保存的数据不能超过 `4K`，Session 可存储数据远高于 Cookie。

### :pen\_fountain: 2.1、session 的实现方式（也就是传递方式）

第一种通过cookies实现：就是把session的 id 放在cookie里面（为什么是使用cookies存放呢，因为cookie有临时的，也有定时的，临时的就是当前浏览器什么时候关掉即消失，也就是说session本来就是当浏览器关闭即消失的，所以可以用临时的cookie存放。保存在cookie里的`sessionID`一定不会重复，因为是独一无二的），当允许浏览器使用cookie的时候，session就会依赖于cookies，当浏览器不支持cookie后，就可以通过第二种方式获取session内存中的数据资源。

第二种通过URL重写来实现：在客户端不支持cookie的情况下使用。必须自己编程使用URL重写的方式实现。

### :pen\_fountain: 2.2、Session 的期限

session有 **过期时间**，当最近一次访问的时候开始计时，每刷新一次重写开始计时。在到达过期时间前，没有访问这个session，session就要关闭。session的过期时间可以配置。

### :pen\_fountain: 2.3、Session 的缺点

Session 机制有个缺点，比如 A 服务器存储了 Session，做了负载均衡后，假如一段时间内 A 的访问量激增，会转发到 B 进行访问，但是 B 服务器并没有存储 A 的 Session，会导致 Session 的失效。

### :pen\_fountain: 2.4、分布式 Session 问题

分布式 Session 一般会有以下几种解决方案：

* `Nginx ip_hash` 策略，服务端使用 `Nginx` 代理，每个请求按访问 IP 的 hash 分配，这样来自同一 IP 固定访问一个后台服务器，避免了在服务器 A 创建 Session，第二次分发到服务器 B 的现象。
* Session 复制，任何一个服务器上的 Session 发生改变（增删改），该节点会把这个 Session 的所有内容序列化，然后广播给所有其它节点。
* 共享 Session，服务端无状态话，将用户的 Session 等信息使用缓存中间件来统一管理，保障分发到每一个服务器的响应结果都一致。

建议采用第三种方案。

### :pen\_fountain: 2.5、如何解决跨域请求？`Jsonp` 跨域的原理是什么？

说起跨域请求，必须要了解浏览器的同源策略，同源策略/SOP（Same origin policy）是一种约定，由 Netscape 公司 1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到 `XSS`、`CSFR` 等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个 `ip` 地址，也非同源。

解决跨域请求的常用方法是：

* 通过代理来避免，比如使用 `Nginx` 在后端转发请求，避免了前端出现跨域的问题。
* 通过`Jsonp` 跨域
* 其它跨域解决方案

重点谈一下 `Jsonp` 跨域原理。浏览器的同源策略把跨域请求都禁止了，但是页面中的 `<script><img><iframe>`标签是例外，不受同源策略限制。`Jsonp` 就是利用 `<script>` 标签跨域特性进行跨域数据访问。

`JSONP` 的理念就是，与服务端约定好一个回调函数名，服务端接收到请求后，将返回一段 `Javascript`，在这段 `Javascript` 代码中调用了约定好的回调函数，并且将数据作为参数进行传递。当网页接收到这段 `Javascript` 代码后，就会执行这个回调函数，这时数据已经成功传输到客户端了。

`JSONP` 的缺点是：它只支持 GET 请求，而不支持 POST 请求等其他类型的 HTTP 请求。

## :pencil2: 3、`JWT`

`Json Web Token` 的简称就是 `JWT`，通常可以称为 `Json` 令牌。它是RFC 7519 中定义的用于安全的将信息作为 `Json` 对象进行传输的一种形式。`JWT` 中存储的信息是经过数字签名的，因此可以被信任和理解。可以使用 `HMAC` 算法或使用 `RSA/ECDSA` 的公用/专用密钥对 `JWT` 进行签名。

使用 `JWT` 主要用来下面两点：

1. 认证(`Authorization`)：这是使用 `JWT` 最常见的一种情况，一旦用户登录，后面每个请求都会包含 `JWT`，从而允许用户访问该令牌所允许的路由、服务和资源。单点登录是当今广泛使用 `JWT` 的一项功能，因为它的开销很小。&#x20;
2. 信息交换(`Information Exchange`)：`JWT` 是能够安全传输信息的一种方式。通过使用公钥/私钥对 `JWT` 进行签名认证。此外，由于签名是使用 head 和 payload 计算的，因此你还可以验证内容是否遭到篡改。

### :pen\_fountain: 3.1、Token的起源

基于服务器的验证：我们都是知道HTTP协议是无状态的，这种无状态意味着程序需要验证每一次请求，从而辨别客户端的身份。在这之前，程序都是通过在服务端存储的登录信息来辨别请求的。这种方式一般都是通过存储Session来完成。随着Web应用程序，以及移动端的兴起，这种验证的方式逐渐暴露出了问题：

1. `Seesion`：每次认证用户发起请求时，服务器需要去创建一个记录来存储信息。当越来越多的用户发请求时，内存的开销也会不断增加。
2. 可扩展性：在服务端的内存中使用`Seesion`存储登录信息，伴随而来的是可扩展性问题。
3. `CORS`(跨域资源共享)：当我们需要让数据跨多台移动设备上使用时，跨域资源的共享会是一个让人头疼的问题。在使用Ajax抓取另一个域的资源，就可以会出现禁止请求的情况。
4. `CSRF`(跨站请求伪造)：用户在访问银行网站时，他们很容易受到跨站请求伪造的攻击，并且能够被利用其访问其他的网站。

在这些问题中，可扩展行是最突出的。

### :pen\_fountain: 3.2、`JWT` **的格式**

`JWT` 主要由三部分组成，每个部分用 `.` 进行分割，各个部分分别是**：**

* `Header`
* `Payload`
* `Signature`

#### Header

Header 是 `JWT` 的标头，它通常由两部分组成：令牌的类型(即 `JWT`)和使用的 签名算法，例如 `HMAC SHA256` 或 `RSA`。

例如

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

`Header Json` 块被 `Base64Url` 编码形成 `JWT` 的第一部分。

#### Payload

Token 的第二部分是 Payload，Payload 中包含一个声明。声明是有关实体（通常是用户）和其他数据的声明。共有三种类型的声明：`registered`，`public` 和 `private` 声明。

* `registered` 声明： 包含一组建议使用的预定义声明，主要包括

| ISS                   | 签发人  |
| --------------------- | ---- |
| iss (issuer)          | 签发人  |
| exp (expiration time) | 过期时间 |
| sub (subject)         | 主题   |
| aud (audience)        | 受众   |
| nbf (Not Before)      | 生效时间 |
| iat (Issued At)       | 签发时间 |
| jti (JWT ID)          | 编号   |

* `public`声明：公共的声明，可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密。
* &#x20;`private`声明：自定义声明，旨在在同意使用它们的各方之间共享信息，既不是注册声明也不是公共声明。

例如：

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

&#x20;然后 `payload Json` 块会被`Base64Url` 编码形成 `JWT` 的第二部分。

#### signature

`JWT` 的第三部分是一个签证信息，这个签证信息由三部分组成

* header (`base64`后的)&#x20;
* payload (`base64`后的)&#x20;
* secret&#x20;

比如我们需要 `HMAC SHA256` 算法进行签名

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用于验证消息在此过程中没有更改，并且对于使用私钥进行签名的令牌，它还可以验证 `JWT` 的发送者的真实身份。

测试编写`JWT`，可以访问 `JWT` 官网 [https://jwt.io/#debugger-io](https://jwt.io/#debugger-io)。

### :pen\_fountain: 3.3、基于Token的验证原理

基于Token的身份验证是无状态的，我们不将用户信息存在服务器或Session中。这种概念解决了在服务端存储信息时的许多问题。`NoSession`意味着你的程序可以根据需要去增减机器，而不用去担心用户是否登录。

基于Token的身份验证的过程如下:

1. 用户通过用户名和密码发送请求。
2. 程序验证。
3. 程序返回一个签名的 token 给客户端。
4. 客户端储存token，并且每次用于每次发送请求。
5. 服务端验证token并返回数据。

每一次请求都需要token。token应该在HTTP的头部发送从而保证了`Http`请求无状态。我们同样通过设置服务器属性`Access-Control-Allow-Origin:*` ，让服务器能接受到来自所有域的请求。需要注意的是，在`ACAO`头部标明`(designating)*`时，不得带有像HTTP认证，客户端`SSL`证书和cookies的证书。

### :pen\_fountain: 3.4、`JWT`和 Session Cookies 的区别

#### :hamster: 3.2.1、密码签名&#x20;

`JWT` 具有加密签名，而 Session Cookies 则没有。

#### :hamster: 3.2.2、`JSON` 是无状态的&#x20;

`JWT` 是无状态的，因为声明被存储在客户端，而不是服务端内存中。

身份验证可以在本地进行，而不是在请求必须通过服务器数据库或类似位置中进行。 这意味着可以对用户进行多次身份验证，而无需与站点或应用程序的数据库进行通信，也无需在此过程中消耗大量资源。

#### :hamster: 3.2.3、可扩展性&#x20;

Session Cookies 是存储在服务器内存中，这就意味着如果网站或者应用很大的情况下会耗费大量的资源。由于 `JWT` 是无状态的，在许多情况下，它们可以节省服务器资源。因此 `JWT` 要比 Session Cookies 具有更强的可扩展性。

#### :hamster: 3.2.4、`JWT` 支持跨域认证

Session Cookies 只能用在单个节点的域或者它的子域中有效。如果它们尝试通过第三个节点访问，就会被禁止。如果你希望自己的网站和其他站点建立安全连接时，这是一个问题。

使用 `JWT` 可以解决这个问题，使用 `JWT` 能够通过多个节点进行用户认证，也就是我们常说的跨域认证。

### :pen\_fountain: 3.5、`JWT` 和 Session Cookies 的选型

对于只需要登录用户并访问存储在站点数据库中的一些信息的中小型网站来说，Session Cookies 通常就能满足。如果你有企业级站点，应用程序或附近的站点，并且需要处理大量的请求，尤其是第三方或很多第三方（包括位于不同域的API），则 `JWT` 显然更适合。
