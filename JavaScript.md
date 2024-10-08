## 数据结构

### 数组

#### 方法

`Array()`：构造函数，可以使用 new 创建，或者直接调用进行创建

- 入口参数：传入多个元素或者传入单个整数作为数组长度
  - 传入单个整数时，会创建对应长度的**稀疏数组**

`.prototype.fill(value [, start [, end]])`：以固定值填充数组，可以指定范围

##### 数组迭代

`.map()`

`.filter()`

`.reduce()`

`.some()`

`.keys()`

`.values()`

`.forEach()`

`.entries()`

`.every()`

##### 数组操作

`.flat()`

`.concat()`

`.fill()`

`.join()`：返回一个将数组中的元素以传入的字符进行连接的字符串

`.push()`

`.pop()`

`.shift()`

`.slice()`

`.splice(start, deleteCount, item)`：就地操作数组

`.reverse()`：就地反转数组

##### 数组查找

`.find()`：返回第一个满足传入函数条件的值

`.findIndex()`：返回第一个满足传入函数条件的索引下标

`.includes()`

`.indexOf()`

稀疏数组：

空槽：`<n empty item>`

产生条件：

1. 通过修改 length 来增加数组长度时，多余的位置会被填充为空槽
2. 通过 `Array(n)` 来创建数组时，全部元素为空槽
   1. 通过 `Array.from()` 构造的数组，元素默认为 undefined
3. 通过 `delete` 删除数组中元素时，留下的位置为空槽

性质：

1. 索引和操作时，以 undefined 为值
2. 数组迭代方法中，会跳过空槽

## 网络请求

### Ajax

传统的浏览器是单线程的，所以渲染和网络请求之间只能同步执行，也就是发送请求后，需要等待返回结果，才能继续使用网页

而 Ajax 其中的 A，就代表异步的意思，其本质是一个在浏览器与服务器之间的媒介，用于收发 HTTP 请求，而能够异步的最底层原因，还是因为浏览器的多线程，提供了异步 xhr 线程用于进行网络请求

### Fetch

Ajax 和 fetch 的区别：

1. 语法：fetch 完全基于 Promise，Ajax 使用传统的回调方式
2. 进度检测：Fetch 无法检测进度，ajax 拥有 progress 事件来监听
3. 错误处理：Fetch 对于 400、500 等错误响应状态码，仍然以成功处理，而 Ajax 会触发 error 回调