通过网络暴露的接口，允许不同的应用程序通过网络进行通信。它们通常以 HTTP 或 HTTPS 为基础，并使用不同的协议和格式（如 JSON、XML）来传输数据。

## 工作方式

### [SOAP](SOAP.md) API 

这些 API 使用简单对象访问协议。客户端和服务器使用 XML 交换消息。这是一个不太灵活的 API，它在过去比较流行。

### [RPC](RPC.md) API

这些 API 称为远程过程调用。客户端在服务器上完成函数（或过程），而服务器将输出发回客户端。

### [Websocket](Websocket.md) API

Websocket API 是另外一种使用 JSON 对象传递数据的现代 Web API 开发方式。WebSocket API 支持在客户端应用程序和服务器之间进行双向通信。服务器可以向连接的客户端发送回调消息，使其比 REST API 更高效。

### [REST](RESTful.md) API

这些是如今最流行、最灵活的 Web API。客户端以数据形式向服务器发送请求。服务器使用该客户端输入来开始执行内部函数，并将输出数据返回到客户端。下面让我们进一步了解一下 REST API。