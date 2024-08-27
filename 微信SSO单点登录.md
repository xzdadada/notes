# 微信 SSO

## OAuth 2.0

### 主要流程

![OAuth2.0时序图](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240827113224097.png)

#### 为什么需要两个 token？

1. token 一经颁发，无法撤销，所以当 token 泄露后，攻击者可以在 token 有效期内随意使用 token 获取资源，所以，access_token 的存活时间设置较短
2. 为了避免用户每次 access_token 过期后都需要重新授权登录，使用一个保存时间较长的 refresh_token 作为换取新的 access_token 的凭证

### 安全问题

### CSRF 攻击

#### 攻击流程

以上面的时序图为例，用户的 access_token 仅由 code 和 client_id 决定

假设有要使用微信账号绑定网站 W，W 获取 token 的回调为 callback

1. 攻击者 A 执行登录流程至获取 code
2. 受害者 B 已经登录了 W 的账号，准备进行微信绑定
3. A 伪造钓鱼链接，点击后会携带 code 执行 callback
4. B 点击 A 创造的钓鱼链接，使用 A 的 code 获取了 token 进行绑定，也就是通过 A 的微信账户与自己在 W 网站的账号进行了绑定
5. A 可以通过微信登录 B 在 W 网站的账号来获取信息

#### 解决方法

以微信的 SSO 为例，在获取 code 的 url 中，可以传入 state 参数，该参数是一个随机生成的字符串，保存在 session 当中，当获取完 code 后，微信会重定向到 redirect_url 同时携带相同的 state，由于攻击者无法获取到该 state，故其钓鱼链接会被第三方网站所识别并拦截

