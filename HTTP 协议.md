# HTTP 协议

## 基础知识

### 请求/响应头

#### 头部

- 缓存控制
  - Cache-Control：max-age / no-cache / no-store…
  - Expires：响应应被视为过期的时间
  - If-None-Match
  - If-Modified-Since
- 连接管理
  - Connection
  - Keep-Alive
  - Upgrade：升级协议
- Cookie
- 请求信息
  - User-Agent：客户端使用的浏览器和操作系统
  - Host：服务器的域名或 IP
  - Method：请求方式
  - Range：请求内容范围
- 内容管理
  - Content-Type：请求体中的数据格式
    - `multipart/form-data; boundary=something`：用于发送文件等不能被编码的元素，boundary 表示每个 part 之间的分隔符，在每个 part 中，有 `Content-Disposition` 和 `Content-Type` 标识这个 part 的格式
      - `application/octet-stream`：表示二进制文件，浏览器通过 `Content-Disposition` 来决定如何处理
  - Accept：可接受的响应内容类型
  - Content-Length：请求体长度
  - Content-Disposition：指示响应的内容以何种形式展示
    - `inline`：默认值，以网页的一部分或整个页面展示
    - `attachment; filename="..."`：指明应该下载响应体至本地，并设置保存的文件名称

- 状态码：

  - 100 Continue：
  - 101 Switching Protocols：服务端通知切换协议
  - 200 OK：请求成功
  - 204 No Content：请求成功，但是没有返回内容
  - 301 Moved Permanently：被请求的资源已经永久移动位置
  - 302 Found：请求的资源临时从不同的 URL 进行响应
  - 304 Not Modified：资源未被修改，可以直接从缓存中加载

  - 400 Bad Request：客户端请求格式错误

  - 401 Unauthorized：请求身份验证不通过

  - 403 Forbidden：服务器拒绝请求

  - 404 Not Found：服务器找不到请求的资源

  - 500 Internal Server Error：服务器发生错误，无法处理请求

  - 502 Bad Gateway：服务器作为网关或代理，从上游服务器收到了无效的响应
  
  - 504 Gateway Timeout：服务器充当网关无法及时获得回应

### HTTP 缓存

#### 强缓存

命中响应状态：`200 from …`

命中强缓存后，根据 expires 字段判断资源是否过期，若未过期，则直接从缓存中获取，不再对服务器发送请求

- `Expires`：过期时间，一个固定的时间点（由服务端发送）
- `Cache-Control`：使用 max-age 指定资源可以被缓存多久（请求体和响应体均可包含）
  - 作用：用于解决 Expires 中客户端和服务端时间之间的误差问题
  - `no-cache`：禁止使用强缓存（在命中时，强制服务器验证请求）
  - `no-store`：禁止任何的缓存

缓存的区域：

1. Service Worker：拦截网络请求并管理资源的缓存
2. memory cache
3. disk cache
4. push cache

#### 协商缓存

命中响应状态：`304 Not Modified`

命中缓存后，向服务器发送请求，携带资源的响应标识（`If-None-Match: ETag` 和 `If-Modified-Since: Last-Modified`）验证资源是否发生改变，若未改变，则从缓存中获取使用

- `ETag` / `If-None-Match`：通过唯一标识来验证缓存，以弥补 `Last-Modified` 的不足
  - `ETag`：由服务器生成，通常通过文件的大小、修改时间、哈希值等生成，**注意**：`ETag` 是否变化和资源是否发生改变之间，即不充分也不必要
- `Last-Modified` / `If-Modified-Since`：通过资源的最后修改时间来验证缓存，有如下缺点
  - 只能精确到秒，即在 1s 内多次修改资源，可能更新不及时
  - 文件可能周期性改变，但内容不需要更新

---

## HTTP 发展

### HTTP/1.1

#### 长连接

通过 Connection: keep-alive 开启（默认开启）
建立 TCP 连接之后，浏览器会保持 TCP 连接，即可以在单次连接中执行多个请求，从而减少了每次请求都需要重新进行 TCP 连接的建立以及断开
单一域名，最多同时保持 6 个 TCP 连接

> 可以通过域名分片的方式来增加 TCP 连接的个数

#### 队头阻塞

HTTP 的请求是队列的形式，即每个请求都需要等待前一个请求完毕才可以进行，所以，当队列头部的请求没有成功响应的时候，后续的请求也就无法开始，从而阻塞了整个队列

### HTTP/2

#### 单 TCP 连接

HTTP2 中，单一域名仅建立一个 TCP 连接

#### 多路复用

采用二进制分帧的方式，将 HTTP 数据包进行拆分，通过 id 进行标识以及连接，帧会在流中进行传输，一个 TCP 连接中可以有多个流，从而在单一 TCP 连接中可以并行发送多个请求

#### HPACK 算法

用于对 HTTP 的头部进行压缩
客户端以及服务端共同维护一份静态字典以及动态字典，如下表所示

```txt
          +-------+-----------------------------+---------------+
          | Index | Header Name                 | Header Value  |
          +-------+-----------------------------+---------------+
          | 1     | :authority                  |               |
          | 2     | :method                     | GET           |
          | 3     | :method                     | POST          |
          | 4     | :path                       | /             |
          | 5     | :path                       | /index.html   |
          | 6     | :scheme                     | http          |
          | 7     | :scheme                     | https         |
          | 8     | :status                     | 200           |
          | 9     | :status                     | 204           |

```

即用更短的字符来存储常用的 HTTP 头部字段，从而减少 HTTP 头部的大小

#### 服务端推送

服务端在客户端请求之后，主动发送请求之外的其余资源，以减少客户端需要的请求次数

1. 客户端请求 HTML 文件
2. 服务端发送 PUSH_PROMISE 帧，告知客户端需要发送多个资源
3. 客户端收到特殊帧，为资源预留空间
4. 服务端发送 HTML 文件，之后推送其他资源
5. 客户端直接使用服务端推送的资源，而无需再发送额外的请求

### HTTP/3

使用 QUIC 协议 ，解决了 TCP 队头阻塞的问题

---

## HTTPS

HTTP 与 HTTPS 的区别：

1. HTTP 默认端口为 80，HTTPS 默认端口为 443
2. HTTP 为明文传输，HTTPS 为加密传输
3. HTTP 不需要证书，HTTPS 需要 SSL 证书

对称加密：双方使用相同的加解密规则

非对称加密：私钥+公钥

### TLS 握手

使用**非对称加密的方式传输预主密钥**，之后使用**对称加密的方式传输信息**

TCP 连接之后

1. 客户端发送 Client Hello，告知服务端支持的 TLS 版本以及加密套件，生成 **随机数1** 发送给服务端
2. 服务端发送 Server Hello，告知客户端支持的 TLS 版本以及选择的加密套件，生成 **随机数2** 发送给客户端
3. 服务端发送 Certificate，将自己的证书发送给客户端
4. 服务端发送 Server Key Exchange，将公钥发送给客户端
5. 服务端发送 Server Hello Done
6. 客户端验证整数，并获取公钥，之后发送
   1. Client Key Change，生成**预主密钥**（随机数3），用**公钥进行加密后的发送给服务端**
   2. Change Cipher Spec，告知服务端，后续内容使用设定好的算法进行加密
   3. Encrypted Handshake Message，告知加密开始
7. 服务端**使用私钥解密**（公钥和私钥均只存在于服务器中），获取预主密钥，发送 Encrypted Handshake Message，回应加密，TLS 握手 完成

**会话密钥** = 随机数1 + 随机数2 + 预主密钥（随机数3）