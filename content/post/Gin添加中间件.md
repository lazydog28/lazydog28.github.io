---
title: "Gin添加中间件"
date: 2023-07-30T20:49:05+08:00
tags:
  - golang
categories:
  - golang web
---

## 1. 什么是中间件
为了更好理解 HTTP 中间件是什么，先要解释一些基本概念。假如一个开发者想要建立两台计算机之间的通信（其中一台计算机为另一台提供资源或服务），他将会构建一个 client/server 系统来实现。服务器等待客户端请求资源或服务，并将请求的资源转发给客户端作为响应。请求的资源或服务可能为：

* 客户端身份校验
* 确认客户端对服务器提供的特定服务是否有访问权限
* 提供服务
* 保障数据安全，确保客户端无法访问未授权数据，防止数据被窃取
* 服务器分为无状态和有状态两类，无状态服务器不关心客户端通信状态，而有状态服务器则关心。

中间件是一种将软件或企业应用连接到另一个软件应用，并构成分布式系统的软件实体。HTTP 请求被发送到 API 服务器，而服务器向客户端返回 HTTP 响应。

中间件具备接收请求功能，可以在请求到达处理方法之前对其进行预处理。然后，它将处理具体方法，并将其响应结果发送给客户端。

## 2. Gin 中间件
Gin 中间件是一种用于 Golang Web 应用程序的功能强大的工具。它允许您在请求到达处理程序之前或之后执行代码，这使您可以添加各种功能，例如日志记录、身份验证、缓存等。

Gin 中间件是函数，具有 `func(c *gin.Context) { ... }` 签名。在中间件中，您可以访问 `c` 上下文对象，该对象包含有关请求的信息，例如请求方法、路径、查询参数等。

要注册中间件，您可以使用 gin.Use 函数。该函数接受中间件函数作为参数。例如，要注册一个记录请求日志的中间件，您可以使用以下代码：
```go
gin.Use(gin.Logger())
```
Gin 中间件可以用于添加各种功能。以下是一些常见的用法：

* 日志记录：记录请求信息，例如请求方法、路径、查询参数等。
* 身份验证：验证用户是否有权访问请求的资源。
* 缓存：缓存请求结果，以提高性能。
* 错误处理：处理请求错误，例如 404 错误或 500 错误。
* 自定义请求处理：自定义请求处理，例如添加请求头或请求体。
Gin 中间件是一种强大而灵活的工具，可用于添加各种功能到您的 Golang Web 应用程序。

## 3. Gin 中间件的执行顺序
Gin 中间件按注册的顺序执行。也就是说，第一个注册的中间件将是第一个执行的中间件，最后一个注册的中间件将是最后一个执行的中间件。

以下是使用 Gin 中间件的示例：
```go
router.Use(gin.Logger())
router.Use(gin.Recovery())

router.GET("/", func(c *gin.Context) {
    c.String(200, "Hello, World!")
})
```
在上述示例中，`Logger()` 中间件将在 `Recovery()` 中间件之前执行。`Logger()` 中间件将记录请求信息，而 `Recovery()` 中间件将捕获并处理错误。

您可以在任何路由注册中间件，包括根路由。您也可以在路由组中注册中间件。

Gin 中间件是一种强大而灵活的工具，可用于添加各种功能到您的 Golang Web 应用程序。

## 4. Gin 自定义中间件
正如上文所述，Gin 中间件是函数，具有 `func(c *gin.Context) { ... }` 签名。在中间件中，您可以访问 `c` 上下文对象，该对象包含有关请求的信息，例如请求方法、路径、查询参数等。
我们接下来定义一个自定义中间件，用于记录请求信息：
```go
func requestLog(c *gin.Context) {
	// 记录请求日志
	rawByte, err := c.GetRawData()
	if err != nil {
		log.Println("获取请求数据失败:", err.Error())
		c.Abort()
		return
	}
	// 将数据重新写入
	c.Request.Body = io.NopCloser(bytes.NewBuffer(rawByte))
	logContent := time.Now().String() + " 请求方法:" + c.Request.Method + "请求路径:" + c.Request.URL.Path + "请求参数:" + string(rawByte)
	f, err := os.OpenFile("request.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0777)
	if err != nil {
		log.Println(logContent)
		c.Next()
		return
	}
	defer func(f *os.File) {
		err := f.Close()
		if err != nil {
		}
	}(f)
	// 将内容写入文件
	_, err = f.WriteString(logContent + "\n")
	// 验证通过，继续后续处理
	c.Next()
}
```
然后，我们使用 `gin.Use()` 函数注册中间件：
```go
router.Use(requestLog)
```
这样，我们就可以在每个请求到达处理程序之前记录请求信息。
## 5. Gin 中间件传递参数
我们在日常使用中有时需要将请求中的数据进行预处理后再传递给处理函数，这时我们就需要在中间件中对请求进行处理，然后将处理后的数据传递给处理函数。这里我们以一个简单的例子来说明：
```go
func main() {
    router := gin.Default()
    router.Use(func(c *gin.Context) {
        c.Set("example", "12345")
    })
    router.GET("/test", func(c *gin.Context) {
        example := c.MustGet("example").(string)
        log.Println(example)
    })
    router.Run(":8080")
}
```
在上文中，我们使用 `c.Set()` 函数将数据设置到上下文中，然后使用 `c.MustGet()` 函数从上下文中获取数据。这样，我们就可以在中间件中对请求进行处理，然后将处理后的数据传递给处理函数。
