# 基于OAuth2.0协议实现GitHub账号登录留言

在OAuth2.0协议中的4中授权模式，功能最完整、流程最严密的授权模式是授权码模式（authorization code）。 这篇文章将通过一个GitHub账号登录来留言的例子说明如何应用这种授权模式。

## 授权码模式详细步骤

1. 用户访问客户端，后者将前者导向认证服务器。
2. 用户选择是否给予客户端授权。如果选`否`，整个授权流程结束。
3. 假设用户给予授权，认证服务器将用户导向客户端事先指定的`重定向URI`（redirection URI），同时附上一个授权码。
4. 客户端收到授权码，附上早先的`重定向URI`，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
5. 认证服务器核对了授权码和`重定向URI`，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

### 第1步详解

用户访问客户端，后者将前者导向认证服务器。

客户端申请认证的URI，包含以下参数：

* `response_type`：表示授权类型，必选项，此处的值固定为`code`
* `client_id`：表示客户端的ID，必选项
* `redirect_uri`：表示重定向URI，可选项
* `scope`：表示申请的权限范围，可选项
* `state`：表示客户端的当前状态，可以指定任意值（最好是随机字符串）， 认证服务器会原封不动地返回这个值，可防止CSRF攻击。

下面是一个例子：

```
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

### 第2步详解

用户选择是否给予客户端授权。如果选`否`，整个授权流程结束。

这一步其实就是引导用户登录认证服务器，查看获取的权限列表，点击同意授权按钮。 比如某个博客网站使用了畅言评论，它支持微信、QQ登录：

![](https://xnstatic-1253397658.file.myqcloud.com/oauth01.png)

我点击QQ登录后会跳出这个确认窗口（如果你没有登录QQ这里还需要输入账号和密码登录）：

![](https://xnstatic-1253397658.file.myqcloud.com/oauth02.png)

可以查看到畅言评论将获得的权限列表，你还可以选择哪些权限不让它获取，点击头像后授权成功。

### 第3步详解

假设用户给予授权，认证服务器将用户导向客户端事先指定的`重定向URI`（redirection URI），同时附上一个授权码。

服务器回应客户端的URI，包含以下参数：

* `code`：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次， 否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。
* `state`：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子：

```
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz
```

### 第4步详解

客户端收到授权码，附上早先的`重定向URI`，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。

客户端向认证服务器申请令牌的HTTP请求，包含以下参数：

* `grant_type`：表示使用的授权模式，必选项，此处的值固定为`authorization_code`。
* `code`：表示上一步获得的授权码，必选项。
* `redirect_uri`：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
* `client_id`：表示客户端ID，必选项。
* `client_secret`: 表示客户端密钥，必选项。

下面是一个例子：

```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/json

params = {
  grant_type: `authorization_code`,
  code: `SplxlOBeZQQYbYS6WxSbIA`,
  client_id: `s6BhdRkqt3`,
  client_secret: `xxx`,
  redirect_uri: `https://client.example.com/cb`
}
```

### 第5步详解

认证服务器核对了授权码和`重定向URI`，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

认证服务器发送的HTTP回复，包含以下内容：

* `access_token`：表示访问令牌，必选项。
* `token_type`：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
* `expires_in`：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
* `refresh_token`：表示更新令牌，用来获取下一次的访问令牌，可选项。
* `scope`：表示权限范围，如果与客户端申请的范围一致，此项可省略。

下面是一个例子：

```
 HTTP/1.1 200 OK
 Content-Type: application/json;charset=UTF-8
 Cache-Control: no-store
 Pragma: no-cache

 {
   `access_token`:`2YotnFZFEjr1zCsicMWpAA`,
   `token_type`:`bearer`,
   `expires_in`:3600,
   `refresh_token`:`tGzv3JOkF0XG5Qx2TlKWIA`,
   `example_parameter`:`example_value`
 }
```

从上面代码可以看到，相关参数使用JSON格式发送（Content-Type: application/json）。 此外，HTTP头信息中明确指定不得缓存。

## 场景介绍

假如我有一个网站，你是我网站上的访客，看了文章想留言表示「朕已阅」，留言时发现有这个网站的帐号才能够留言， 此时给了你两个选择：

* 一个是在我的网站上注册拥有一个新账户，然后用注册的用户名来留言；
* 一个是使用 github 帐号登录，使用你的 github 用户名来留言。

前者你觉得过于繁琐，于是惯性地点击了 github 登录按钮，此时 OAuth 认证流程就开始了。

需要明确的是，即使用户刚登录过 github，我的网站也不可能向 github 发一个什么请求便能够拿到访客信息，这显然是不安全的。 就算用户允许你获取他在 github 上的信息，github 为了保障用户信息安全，也不会让你随意获取。
所以操作之前，我的网站与 github 之间需要要有一个协商。

## 网站和 Github 之间的协商

Github 会对用户的权限做分类，比如读取仓库信息的权限、写入仓库的权限、读取用户信息的权限、修改用户信息的权限等等。 如果我想获取用户的信息，Github
会要求我，先在它的平台上注册一个应用，在申请的时候标明需要获取用户信息的哪些权限， 用多少就申请多少，并且在申请的时候填写你的网站域名，Github 只允许在这个域名中获取用户信息。

此时我的网站已经和 Github 之间达成了共识，Github 也给我发了两张门票， 一张门票叫做 Client Id，另一张门票叫做 Client Secret。

## 用户和 Github 之间的协商

用户进入我的网站，点击 github 登录按钮的时候，我的网站会把上面拿到的 Client Id 交给用户，让他进入到 Github 的授权页面， Github
看到了用户手中的门票，就知道这是我的网站让他过来的，于是它就把我的网站想要获取的权限摆出来，并询问用户是否允许我获取这些权限。

```
// 用户登录 github，协商
GET //github.com/login/oauth/authorize
// 协商凭证
params = {
  client_id: `xxxx`,
  redirect_uri: `http://my-website.com`
}
```

如果用户觉得我的网站要的权限太多，或者压根就不想我知道他这些信息，选择了拒绝的话，整个 OAuth 2.0 的认证就结束了，认证也以失败告终。 如果用户觉得
OK，在授权页面点击了确认授权后，页面会跳转到我预先设定的 `redirect_uri` 并附带一个盖了章的门票 `code`：

```
// 协商成功后带着盖了章的 code
Location: http://my-website.com?code=xxx
```

这个时候，用户和 Github 之间的协商就已经完成，Github 也会在自己的系统中记录这次协商， 表示该用户已经允许在我的网站访问上直接操作和使用他的部分资源。

## 告诉 Github 我的网站要来拜访了

第二步中，我们已经拿到了盖过章的门票 code，但这个 code 只能表明，用户允许我的网站从 github 上获取该用户的数据， 如果我直接拿这个 code 去 github 访问数据一定会被拒绝，因为任何人都可以持有
code，github 并不知道 code 持有方就是我本人。

还记得之前申请应用的时候 github 给我的两张门票么，Client Id 在上一步中已经用过了，接下来轮到另一张门票 Client Secret：

```
// 网站和 github 之间的协商
POST //github.com/login/oauth/access_token
// 协商凭证包括 github 给用户盖的章和 github 发给我的门票
params = {
  code: `xxx`,
  client_id: `xxx`,
  client_secret: `xxx`,
  redirect_uri: `http://my-website.com`
}
```

拿着用户盖过章的 `code` 和能够标识个人身份的 `client_id`、`client_secret` 去拜访 github，拿到最后的绿卡 `access_token`：

```
// 拿到最后的绿卡
response = {
  access_token: `e72e16c7e42f292c6912e7710c838347ae178b4a`
  scope: `user,gist`
  token_type: `bearer`,
  refresh_token: `xxxx`
}
```

## 用户开始使用 github 帐号在我的页面上留言

```
// 访问用户数据
GET //api.github.com/user?access_token=e72e16c7e42f292c6912e7710c838347ae178b4a
```

上一步 github 已经把最后的绿卡 `access_token` 给我了，通过 github 提供的 API 加绿卡就能够访问用户的信息了， 能获取用户的哪些权限在 response 中也给了明确的说明，`scope` 为 user 和
gist，也就是只能获取 user 组和 gist 组两个小组的权限， user 组中就包含了用户的名字和邮箱等信息了：

```
// 告诉我用户的名字和邮箱
response = {
  username: `barretlee`,
  email: `barret.china@gmail.com`
}
```

整个 OAuth2 流程在这里也基本完成了，文章中的表述很粗糙，比如 `access_token` 这个绿卡是有过期时间的， 如果过期了需要使用 refresh_token 重新签证。重点是让读者理解整个流程，
细节部分可以阅读 [RFC6749 文档](http://www.rfcreader.com/#rfc6749)。

