# HTTP

## HTTP 压缩
HTTP 压缩是指在 HTTP 协议中对传输的数据进行压缩，以减少数据传输的大小和提高传输速度。常见的 HTTP 压缩算法包括 Gzip 和 Brotli。

客户端和服务器会先协商支持的压缩算法，然后在传输数据时使用这些算法进行压缩和解压缩。HTTP 压缩可以显著减少传输的数据量，尤其是在传输文本内容（如 HTML、CSS、JavaScript 等）时。

协商过程
1. 客户端发送请求时，在请求头中包含 `Accept-Encoding` 字段，告知服务器支持的压缩算法。例如：
```
Accept-Encoding: gzip, deflate, br
```
2. 服务器根据自身支持的算法和客户端提供的算法进行协商，选择一种算法进行响应。如果选择了压缩，服务器在响应头中添加 `Content-Encoding` 字段。例如：
```
Content-Encoding: gzip
```
3. 客户端接收到响应后，根据 `Content-Encoding` 字段使用相应的解压缩算法对响应内容进行解压缩。
