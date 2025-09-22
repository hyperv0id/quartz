## Cookie
因为http是无状态的，如果想要保存之前的访问消息，就需要 持久化消息。

cookie是web服务器保存的用户状态，用户再次访问时浏览器将cookie一并发送，服务器根据cookie辨认用户信息

## Session

基于cookie，服务端根据cookie维护一个session，保存会话信息。

## Token

用户登录反复查账密对数据库压力较大，服务端会生成Token，将Token发给用户，用户使用token就无需登录。

**功能**：认证和授权

Token的安全性要求较高，需要保证用户不能获取Token中的隐藏信息

## Cookie vs Session

**存储位置与安全性**：cookie数据存放在客户端上，安全性较差，session数据放在服务器上，安全性相对更高； 

**占用服务器资源**：session一定时间内保存在服务器上，当访问增多，占用服务器性能，考虑到服务器性能方面，应当使用cookie。

## Session vs Token

- **效率**：session机制存在服务器压力增大，CSRF跨站伪造请求攻击，扩展性不强等问题； 
- **存储方式**：session存储在服务器端，token存储在客户端
- 安全性：token提供认证和授权功能，作为身份认证，token安全性比session好； 
- **可用性**：session这种会话存储方式方式只适用于客户端代码和服务端代码运行在同一台服务器上，token适用于项目级的前后端分离（前后端代码运行在不同的服务器下）