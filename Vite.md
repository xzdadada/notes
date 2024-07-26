# Vite

通过 ESM，在需要引用模块的时候，会发送 HTTP 请求获取需要的模块

对于修改中的代码，通过协商缓存 `304 Not Modified` 来进行更改以及缓存

对于依赖的模块，通过设置 HTTP 响应的 `Cache-Control: max-age=31536000,immutable`，进行强缓存