# 中间件

中间件是一种函数，它在请求被任何特定的*路径操作*处理之前，以及在响应返回之前，对每个请求和响应进行处理。它提供了一种在应用程序全局层面运行代码的机制，从而影响所有传入的请求和传出的响应。

这对于实现横切关注点非常有用，例如日志记录、性能监控、身份验证、CORS 处理以及添加自定义标头。

## 创建自定义中间件

你可以使用 `@app.middleware("http")` 装饰器创建自己的中间件。中间件函数接收传入的 `request` 和一个 `call_next` 函数，`call_next` 函数会接收 `request` 作为参数。这个 `call_next` 函数将请求传递给处理链中的下一步，这可能是另一个中间件或实际的*路径操作*。

下面是一个计算请求处理时间并将其作为自定义标头 `X-Process-Time` 添加到响应中的示例。

```python 添加自定义标头 icon=logos:python
import time

from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()
    response = await call_next(request)
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

在此示例中：
1.  我们在处理请求前记录时间。
2.  我们调用 `await call_next(request)` 将请求传递给相应的*路径操作*，并获取其生成的响应。
3.  获得 `response` 后，我们可以对其进行修改，例如计算总处理时间并添加自定义标头。
4.  最后，我们返回修改后的 `response`。

## CORS (跨源资源共享)

中间件一个非常常见的用例是处理 CORS。出于安全原因，浏览器会限制网页向提供该网页的域之外的其他域发出请求。CORS 中间件允许你明确许可某些跨源请求。

FastAPI 提供了 `CORSMiddleware` 来轻松处理此问题。

```python CORSMiddleware 示例 icon=logos:python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost.tiangolo.com",
    "https://localhost.tiangolo.com",
    "http://localhost",
    "http://localhost:8080",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

你可以使用 `app.add_middleware()` 将其添加到你的应用程序中。主要配置选项如下：

*   `allow_origins`：允许发出跨源请求的源列表。你可以使用 `["*"]` 来允许所有源。
*   `allow_credentials`：指示跨源请求应支持 Cookie。
*   `allow_methods`：允许的 HTTP 方法列表。你可以使用 `["*"]` 来允许所有标准方法。
*   `allow_headers`：支持的 HTTP 请求头列表。你可以使用 `["*"]` 来允许所有请求头。

## 其他内置中间件

FastAPI 通过 Starlette 提供了其他几个有用的中间件组件，你可以将它们添加到你的应用程序中。

### HTTPSRedirectMiddleware

此中间件强制所有传入的请求必须是 `https` 或 `wss` (用于 WebSocket)。如果传入的请求是 `http` 或 `ws`，它将被重定向到安全协议。

```python HTTPSRedirectMiddleware 示例 icon=logos:python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

app.add_middleware(HTTPSRedirectMiddleware)


@app.get("/")
async def main():
    return {"message": "Hello World"}

```

### GZipMiddleware

对于 `Accept-Encoding` 头中包含 `"gzip"` 的任何请求，`GZipMiddleware` 都会压缩其响应。这可以显著减小发送到客户端的数据大小，从而提高性能。

```python GZipMiddleware 示例 icon=logos:python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

app.add_middleware(GZipMiddleware, minimum_size=1000)

# ... your path operations
```

## 总结

中间件是一个强大的工具，用于添加适用于应用程序中多个或所有端点的通用功能。你可以编写自己的自定义逻辑，或使用 FastAPI 和 Starlette 提供的预置中间件来执行 CORS、GZip 压缩和 HTTPS 重定向等任务。

既然你已经了解了如何使用中间件添加全局逻辑，接下来让我们探讨如何将你的应用程序连接到数据库。请继续阅读下一节，了解 [SQL 数据库](./tutorials-advanced-sql-databases.md)。