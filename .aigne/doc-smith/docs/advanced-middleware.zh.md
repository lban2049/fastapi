# 中间件

中间件是一个函数，它在每个请求到达特定路径操作之前以及每个响应返回给客户端之前处理它们。这允许你以集中的方式实现日志记录、性能监控、身份验证和响应头操作等横切关注点。

FastAPI 的中间件构建于 Starlette 的中间件系统之上。你可以使用两种主要方法向应用程序添加中间件：用于自定义中间件函数的 `@app.middleware("http")` 装饰器，或用于中间件类的 `app.add_middleware()`。

## 创建自定义中间件

你可以使用 `@app.middleware("http")` 装饰器创建自己的中间件。这对于实现需要在每个请求上运行的自定义逻辑非常有用。

例如，我们来创建一个中间件，用于计算每个请求的处理时间，并将其添加到一个自定义的响应头 `X-Process-Time` 中。

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
- 函数 `add_process_time_header` 使用 `@app.middleware("http")` 装饰，将其注册为中间件。
- 它接收 `request` 对象和一个 `call_next` 函数。
- `call_next` 是一个接收 `request` 作为参数的函数。它负责将请求传递给下一个中间件或实际的路径操作。
- 我们在调用 `call_next` 之前记录时间。
- `await call_next(request)` 返回应用程序生成的 `response`。
- 生成响应后，我们计算总处理时间并将其添加为自定义头。

### 中间件流程

请求和响应以“洋葱式”结构通过中间件。请求会穿过每个中间件层，直到到达路径操作，然后响应会沿着相同的路径反向传回。

```d2
direction: down

客户端
Middleware1: "中间件 1 (例如 GZip)"
Middleware2: "中间件 2 (例如处理时间)"
PathOperation: "路径操作"

subgraph "请求流程" {
  direction: down
  客户端 -> Middleware1: "请求"
  Middleware1 -> Middleware2: "call_next(request)"
  Middleware2 -> PathOperation: "call_next(request)"
}

subgraph "响应流程" {
  direction: up
  PathOperation -> Middleware2: "响应"
  Middleware2 -> Middleware1: "返回响应"
  Middleware1 -> 客户端: "返回响应"
}
```

## 内置中间件

FastAPI 包含几个有用的中间件类，你可以使用 `app.add_middleware()` 将它们添加到你的应用程序中。这些类直接从 Starlette 重新导出。

<x-cards data-columns="2">
  <x-card data-title="HTTPSRedirectMiddleware" data-icon="lucide:lock">
    强制所有传入请求必须使用 `https` 或 `wss`。它会将任何 `http` 或 `ws` 请求重定向到其安全对应的协议。
    ```python
    from fastapi import FastAPI
    from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

    app = FastAPI()

    app.add_middleware(HTTPSRedirectMiddleware)


    @app.get("/")
    async def main():
        return {"message": "Hello World"}
    ```
  </x-card>
  <x-card data-title="TrustedHostMiddleware" data-icon="lucide:shield-check">
    通过确保传入请求的 `Host` 头在允许的主机列表中，来防范 HTTP Host 头攻击。
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
  </x-card>
  <x-card data-title="GZipMiddleware" data-icon="lucide:file-archive">
    为 `Accept-Encoding` 头中包含 `"gzip"` 的任何请求压缩响应。这可以减少带宽使用。
    ```python
    from fastapi import FastAPI
    from fastapi.middleware.gzip import GZipMiddleware

    app = FastAPI()

    app.add_middleware(GZipMiddleware, minimum_size=1000, compresslevel=5)


    @app.get("/")
    async def main():
        return "somebigcontent"
    ```
    - `minimum_size`: 仅压缩大于此字节值的响应。
    - `compresslevel`: 设置 gzip 压缩级别 (1-9)。
  </x-card>
  <x-card data-title="CORSMiddleware" data-icon="lucide:globe">
    处理跨域资源共享 (CORS)。它允许你定义哪些源、方法和头对于跨域请求是允许的，这对于构建从不同域与你的 API 交互的 Web 应用程序至关重要。你可以从 `fastapi.middleware.cors` 导入它。
  </x-card>
</x-cards>

通过利用中间件，你可以保持路径操作逻辑的整洁，专注于业务功能，同时以可重用和高效的方式处理常见任务。

现在你已经了解了如何使用中间件，可能想学习如何随着应用程序的增长来组织它。请继续阅读，了解[构建更大型应用](./advanced-bigger-applications.md)的策略。