### 介绍

1. **OAuth** 是一个开放标准，该标准允许用户让第三方应用访问该用户在某一网站上存储的私密资源（如头像、照片、视频等），而在这个过程中无需将用户名和密码提供给第三方应用。实现这一功能是通过提供一个令牌（token），而不是用户名和密码来访问他们存放在特定服务提供者的数据。采用令牌（token）的方式可以让用户灵活的对第三方应用授权或者收回权限。

2. **OAuth2** 是 OAuth 协议的下一版本，但不向下兼容 OAuth 1.0。传统的 Web 开发登录认证一般都是基于 session 的，但是在前后端分离的架构中继续使用 session 就会有许多不便，**因为移动端（Android、iOS、微信小程序等）要么不支持 cookie（微信小程序），要么使用非常不便**，对于这些问题，使用 OAuth2 认证都能解决。

3. 对于大家而言，我们在互联网应用中最常见的 OAuth2 应该就是各种第三方登录了，例如 QQ 授权登录、微信授权登录、微博授权登录、GitHub 授权登录等等。

### 令牌与密码

令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异。

1. 令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。
2. 令牌可以被数据所有者撤销，会立即失效。以上例而言，屋主可以随时取消快递员的令牌。密码一般不允许被他人撤销。
3. 令牌有权限范围（scope），比如只能进小区的二号门。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。

### 四种授权模式


* 授权码（authorization-code）
* 隐藏式（implicit）
* 密码式（password）
* 客户端凭证（client credentials）


### 授权码

**授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

> A网站要获取B网站的授权

第一步，A 网站(云打印)提供一个链接，用户点击后就会跳转到 B (Google)网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。

```
https://b.com/oauth/authorize?
	response_type=code&
	client_id=CLIENT_ID&
	redirect_uri=CALLBACK_URL&
	scope=read
```

上面 URL 中，`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这里是只读）。

第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，就像下面这样。

```
https://a.com/callback?code=AUTHORIZATION_CODE
```

第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。

```
https://b.com/oauth/token?   
        client_id=CLIENT_ID&
        client_secret=CLIENT_SECRET&
		grant_type=authorization_code&
		code=AUTHORIZATION_CODE&
		redirect_uri=CALLBACK_URL
```

上面 URL 中，`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。

第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。

```
{
  "access_token": "ACCESS_TOKEN",
  "expires_in": 2592000,
  "refresh_token": "REFRESH_TOKEN",
  "scope": "read",
  "uid": 100101,
  "info": {...}
}
```

上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。

```
【前端直接发起】✅

1. 用户浏览器访问前端页面
   https://your-app.com
   
2. 用户点击"授权登陆"按钮
   
3. 前端JavaScript构造授权URL并重定向
   window.location.href = 'https://oauth-server.com/authorize?...'
   
4. 浏览器跳转到授权服务器
   用户在授权服务器登录并授权
   
5. 授权服务器重定向回前端
   https://your-app.com/callback?code=abc123
   
6. 前端获取code，发送给后端
   POST /api/auth/callback { code: 'abc123' }
   
7. 后端用code换取token
   后端 → 授权服务器
   返回 access_token
   
8. 后端返回token给前端
   前端保存token用于后续API调用
```

### 隐藏式(纯前端应用)

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**

第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。

```
https://b.com/oauth/authorize?
	response_type=token&
	client_id=CLIENT_ID&
	redirect_uri=CALLBACK_URL&
	scope=read
```

上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。

第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

```
https://a.com/callback#token=ACCESS_TOKEN
```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

###  密码式

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**

第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。
```
https://oauth.b.com/token? 
	grant_type=password& 
	username=USERNAME& 
	password=PASSWORD& 
	client_id=CLIENT_ID
```

上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。

第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。

**这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。**

### 凭证式(命令行应用)

**最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。**

这种更多的服务之间调用，不是要用户信息，比如一个服务在天气查询平台注册了，有 `client_id` 与  `client_secret` 就可以直接获取天气信息了，不需要用户的信息授权

第一步，A 应用在命令行向 B 发出请求。

```
https://oauth.b.com/token? 
	grant_type=client_credentials&
	client_id=CLIENT_ID& 
	client_secret=CLIENT_SECRET
```

上面 URL 中，`grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`用来让 B 确认 A 的身份。

第二步，B 网站验证通过以后，直接返回令牌。

**这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。**

### 令牌的使用

A 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。

此时，每个发到 API 的请求，都必须带有令牌。具体做法是在请求的头信息，加上一个`Authorization`字段，令牌就放在这个字段里面。

```
curl -H "Authorization: Bearer ACCESS_TOKEN" \ 
	"https://api.b.com"
```

上面命令中，`ACCESS_TOKEN`就是拿到的令牌。

### 更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

具体方法是，B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。

```
https://b.com/oauth/token? 
	grant_type=refresh_token& 
	client_id=CLIENT_ID& 
	client_secret=CLIENT_SECRET& 
	refresh_token=REFRESH_TOKEN
```

上面 URL 中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。

B 网站验证通过以后，就会颁发新的令牌。