---
aliases:
  - REST
---

## 特点

当然，REST（Representational State Transfer）是一种软件架构风格，它主要用于构建分布式系统和网络应用。RESTful 架构的设计原则包括：

1. **基于资源（Resources）**: REST 是基于资源的概念，每个资源都有一个唯一的标识符（URI）。客户端通过 URI 访问和操作这些资源。

2. **CS-架构**: RESTful API 使用 HTTP 方法来执行操作，例如，GET（获取资源）、POST（创建资源）、PUT（更新资源）、DELETE（删除资源）等。

3. **无状态（Stateless）**: 每个请求都包含足够的信息，使得服务器能够理解请求而不依赖于之前的请求。服务器不保存客户端的状态信息。

4. **统一接口（Uniform Interface）**: RESTful 架构提供了一种统一的接口，使得客户端和服务器之间的通信能够简化。这包括使用标准的 HTTP 方法，以及使用标准的媒体类型（如 JSON、XML）来交换数据。

5. **分层系统（Layered System）**: REST 允许系统以层次化的方式组织，每一层都有特定的功能。客户端只需了解与它直接交互的层，而无需了解整个系统的结构。

6. **缓存（Cacheable）**: RESTful 架构支持缓存，使得客户端可以缓存服务器的响应，提高性能和效率。

RESTful API 遵循这些原则，提供了简单、可扩展和灵活的方式来构建和使用网络服务。在现代的 Web 应用程序和服务中，RESTful API 是非常常见的一种设计方式。

### 缺点

- 很依赖API文档
- 获取不足、获取过多：只有洗剪吹，不能只剪头发，也不能按摩。

## 例子

假设我们要设计一个博客系统的 RESTful API。

1. **非 RESTful 风格的 API 设计**：

    ```
    GET /getBlogPosts      // 获取所有博客文章
    POST /createBlogPost   // 创建一篇博客文章
    GET /getBlogPostById/:postId   // 根据ID获取博客文章
    PUT /updateBlogPost/:postId    // 更新博客文章
    DELETE /deleteBlogPost/:postId // 删除博客文章
    ```

    这种设计将操作和资源一起表示在 URL 上，使用不够规范的方法命名，并且未使用 HTTP 方法来区分不同的操作。

2. **RESTful 风格的 API 设计**：

    ```
    GET /blogPosts          // 获取所有博客文章
    POST /blogPosts         // 创建一篇博客文章
    GET /blogPosts/:postId  // 根据ID获取博客文章
    PUT /blogPosts/:postId  // 更新博客文章
    DELETE /blogPosts/:postId  // 删除博客文章
    ```

    这种设计更符合 RESTful 风格，使用了合适的资源名词（`blogPosts`），并通过 HTTP 方法来表达对资源的不同操作。URI 通过资源的名称和标识符来表示，并且操作使用了标准的 HTTP 方法。

在比较这两种设计时，RESTful 风格的 API 更加规范和符合标准，使得 API 更易于理解和使用。它利用了 HTTP 方法和资源的概念，使得接口更具表达性和清晰性。同时，RESTful 设计也更符合 HTTP 标准，使得客户端和服务器之间的通信更加简单和直观。

## 参考

- 除了RESTful还有：[GraphQL](GraphQL.md)