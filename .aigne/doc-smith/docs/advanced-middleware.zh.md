# 中间件

中间件是一个函数，它在每个请求到达特定路径操作前处理该请求，并在每个响应返回客户端前处理该响应。这使得你可以用一种集中的方式来实现诸如日志记录、性能监控、身份验证和响应头操作等横切关注点。

FastAPI 的中间件构建于 Starlette 的中间件系统之上。你可以通过两种主要方法向应用程序添加中间件：使用 `@app.middleware("http")` 装饰器创建自定义中间件函数，或使用 `app.add_middleware()` 添加中间件类。

## 创建自定义中间件

你可以使用 `@app.middleware("http")` 装饰器创建自己的中间件。这对于为每个请求实现需要运行的自定义逻辑非常有用。

例如，我们来创建一个中间件，用它计算每个请求的处理时间，并将其添加到一个自定义响应头 `X-Process-Time` 中。

```python
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
- 函数 `add_process_time_header` 由 `@app.middleware("http")` 装饰，从而将其注册为中间件。
- 它接收 `request` 对象和 `call_next` 函数。
- `call_next` 是一个接收 `request` 作为参数的函数，负责将请求传递给下一个中间件或实际的路径操作。
- 我们在调用 `call_next` 前记录时间。
- `await call_next(request)` 返回应用程序生成的 `response`。
- 生成响应后，我们计算总处理时间，并将其作为自定义响应头添加。

### 中间件流程

请求和响应以“洋葱式”结构通过中间件。请求会依次穿过每个中间件层，直到到达路径操作，然后响应再沿相同的路径返回。

```d2
direction: down

Client: "客户端"
Middleware1: "中间件 1 (例如 GZip)"
Middleware2: "中间件 2 (例如 Process Time)"
PathOperation: "路径操作"

subgraph "请求流程" {
  direction: down
  Client -> Middleware1: "请求"
  Middleware1 -> Middleware2: "call_next(request)"
  Middleware2 -> PathOperation: "call_next(request)"
}

subgraph "响应流程" {
  direction: up
  PathOperation -> Middleware2: "响应"
  Middleware2 -> Middleware1: "return response"
  Middleware1 -> Client: "return response"
}
```

## 内置中间件

FastAPI 包含几个有用的中间件类，你可以使用 `app.add_middleware()` 将它们添加到你的应用程序中。这些类直接从 Starlette 重新导出。

### HTTPSRedirectMiddleware

此中间件强制所有传入请求必须使用 `https` 或 `wss`。它会将任何 `http` 或 `ws` 请求重定向到其安全对应的协议。

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

此中间件通过确保传入请求的 `Host` 头在允许的主机列表中，来防范 HTTP Host 头攻击。它会根据允许的主机列表验证 `Host` 头，以防止此类攻击。

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

### GZipMiddleware

对于 `Accept-Encoding` 头中包含 "gzip" 的任何请求，此中间件都会压缩响应，从而减少带宽使用。你可以为其配置以字节为单位的 `minimum_size`（避免压缩过小的响应）和一个从 1 到 9 的 `compresslevel`。

```python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

app.add_middleware(GZipMiddleware, minimum_size=1000, compresslevel=5)


@app.get("/")
async def main():
    return "somebigcontent"
```

### CORSMiddleware

此中间件用于处理跨源资源共享 (CORS)，这对于构建从不同域与你的 API 交互的 Web 应用程序至关重要。你可以从 `fastapi.middleware.cors` 导入它，并对其进行配置，以指定允许的来源、方法和请求头。

通过利用中间件，你可以保持路径操作逻辑的整洁，专注于业务功能，同时以可重用且高效的方式处理通用任务。

既然你已经了解了如何使用中间件，你可能想进一步学习如何组织不断壮大的应用程序。请继续阅读，了解 [构建更大型应用程序](./advanced-bigger-applications.md) 的策略。