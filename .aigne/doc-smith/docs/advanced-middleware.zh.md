# 中间件

中间件是一个函数，它在任何特定*路径操作*处理请求之前以及在返回响应之前，都会对每个请求和响应进行处理。它提供了一种挂接到请求和响应处理管道的机制，以执行横切操作。

中间件的常见用例包括：

*   向请求或响应添加自定义标头。
*   记录每个请求。
*   处理身份验证或授权。
*   实现 GZip 压缩。
*   管理 CORS（跨源资源共享）标头。

中间件按其添加顺序处理请求，并按相反顺序处理响应。这可以形象地比作洋葱的层次：请求必须穿过这些层次才能到达应用程序代码，然后在返回时再反向穿过。

```d2
direction: down

Client: {
  shape: person
}

Middleware-Stack: {
  label: "中间件堆栈"
  shape: package
  grid-columns: 1

  Middleware-1: {
    label: "中间件 1（例如 GZip）"
    shape: rectangle
  }
  Middleware-2: {
    label: "中间件 2（例如 CORS）"
    shape: rectangle
  }
  Custom-Middleware: {
    label: "自定义中间件"
    shape: rectangle
  }
}

FastAPI-Application: {
  label: "FastAPI 应用程序"
  shape: rectangle
  Path-Operation-Code: {
    label: "路径操作代码"
  }
}

Client -> Middleware-Stack.Middleware-1: "1. 请求"

Middleware-Stack.Middleware-1 -> Middleware-Stack.Middleware-2: "2. 请求"
Middleware-Stack.Middleware-2 -> Middleware-Stack.Custom-Middleware: "3. 请求"
Middleware-Stack.Custom-Middleware -> FastAPI-Application: "4. 请求端点"

FastAPI-Application -> Middleware-Stack.Custom-Middleware: "5. 来自端点的响应"
Middleware-Stack.Custom-Middleware -> Middleware-Stack.Middleware-2: "6. 响应"
Middleware-Stack.Middleware-2 -> Middleware-Stack.Middleware-1: "7. 响应"

Middleware-Stack.Middleware-1 -> Client: "8. 最终响应"
```

## 创建自定义中间件

你可以使用 `@app.middleware("http")` 装饰器创建自己的中间件。该函数接收 `request` 对象和一个 `call_next` 函数，`call_next` 函数会将 `request` 作为参数接收。`call_next` 会将请求传递给下一个中间件或路径操作。

以下示例计算请求的处理时间，并将其作为自定义标头 `X-Process-Time` 添加到响应中。

```python
import time

from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
asnyc def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()
    response = await call_next(request)
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

在此示例中：
1.  在处理请求之前记录开始时间。
2.  `await call_next(request)` 将控制权传递给应用程序的下一层（另一个中间件或实际的路径操作）。
3.  生成并返回响应后，代码会计算总处理时间。
4.  计算出的时间被添加到响应标头中。
5.  返回最终修改后的响应。

## 内置中间件

FastAPI 包含了几个来自 Starlette 的有用中间件实现，你可以使用 `app.add_middleware()` 将它们添加到你的应用程序中。

### HTTPSRedirectMiddleware

此中间件强制所有传入请求必须是 `https` 或 `wss`。如果请求以 `http` 或 `ws` 协议到达，它将被重定向到安全协议。

```python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

app.add_middleware(HTTPSRedirectMiddleware)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

### TrustedHostMiddleware

此中间件强制所有传入请求都必须正确设置 `Host` 标头，以防止 HTTP Host 标头攻击。你必须指定一个允许的主机名列表。

```python
from fastapi import FastAPI
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app = FastAPI()

app.add_middleware(
    TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"]
)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

如果请求的 `Host` 标头与 `allowed_hosts` 中的任何模式都不匹配，它将收到一个 400 Bad Request 响应。

### GZipMiddleware

此中间件处理响应的 GZip 压缩。如果客户端支持 GZip（`Accept-Encoding` 标头），响应将被压缩，这可以减少带宽使用。

```python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

app.add_middleware(GZipMiddleware, minimum_size=1000, compresslevel=5)


@app.get("/")
async def main():
    # 如果响应大小超过 1000 字节
    # 并且客户端支持 gzip，则该响应将被压缩。
    return "somebigcontent" * 200
```

关键参数：

| 参数 | 类型 | 描述 |
|---|---|---|
| `minimum_size` | `int` | 仅压缩大于此字节数的响应。默认为 500。 |
| `compresslevel` | `int` | 一个从 0 到 9 的整数，指定压缩级别。9 是最慢但压缩率最高，1 是最快但压缩率最低。默认为 6。 |

### CORSMiddleware

此中间件处理跨源资源共享 (CORS)，当运行在不同域上的前端应用程序需要与你的 API 通信时，这是必需的。它允许你指定允许哪些源、方法和标头。

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost",
    "http://localhost:3000",
    "https://your-frontend-domain.com",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins, # 允许特定的源
    allow_credentials=True,
    allow_methods=["*"],      # 允许所有方法
    allow_headers=["*"],      # 允许所有标头
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

此配置使来自指定 `origins` 的客户端能够向你的 API 发出请求。

---

通过利用中间件，你可以为 FastAPI 应用程序添加强大的、可重用的功能。设置好中间件后，你可能希望探索如何在应用程序中启用实时通信。请参阅 [WebSockets](./advanced-websockets.md) 部分了解更多信息。
