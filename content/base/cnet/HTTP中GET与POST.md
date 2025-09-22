## 主要不同

从语义上来看，GET是从服务端读数据，只读就可以被缓存，是安全、幂等的，因为其不会引起服务端变化。

POST要对资源处理，因此



1. 目的和语义：
   - GET：用于获取资源。它应该只用于读取数据，不应该对服务器状态产生任何影响。
   - POST：用于向服务器提交数据，可能会导致服务器状态的改变或产生副作用。

2. 数据传输：
   - GET：参数通常附加在URL中（查询字符串）。
   - POST：数据通常在请求体中发送。

3. 数据量限制：
   - GET：由于数据在URL中，受到URL长度的限制（具体限制取决于浏览器和服务器）。
   - POST：可以发送大量数据，理论上没有限制。

4. 安全性：
   - GET：参数在URL中可见，不适合发送敏感信息。
   - POST：数据在请求体中，相对更安全，但仍然需要额外的加密措施来保护敏感信息。

5. 幂等性：
   - GET：是幂等的，多次相同请求应该返回相同结果。
   - POST：通常不是幂等的，多次请求可能会产生不同的结果。

6. 缓存：
   - GET：请求可以被缓存。
   - POST：通常不被缓存。

7. 浏览器历史：
   - GET：请求会被保存在浏览器历史中。
   - POST：不会被保存在浏览器历史中。

8. 书签：
   - GET：可以被收藏为书签。
   - POST：不能被收藏为书签。

9. 编码类型：
   - GET：application/x-www-form-urlencoded
   - POST：application/x-www-form-urlencoded 或 multipart/form-data

10. 后退/刷新：
    - GET：无害
    - POST：数据可能会被重新提交（浏览器通常会警告用户）

## 参考

[RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231)