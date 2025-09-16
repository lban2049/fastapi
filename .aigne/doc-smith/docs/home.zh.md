# 主页

FastAPI 是一个现代、快速（高性能）的 Web 框架，用于基于标准 Python 类型提示使用 Python 构建 API。

它建立在巨人的肩膀上：
- **[Starlette](https://www.starlette.io/)** 用于 Web 部分。
- **[Pydantic](https://docs.pydantic.dev/)** 用于数据部分。

## 主要特性

<x-cards data-columns="2">
  <x-card data-title="快速" data-icon="lucide:rocket">
    性能极高，可与 NodeJS 和 Go 相媲美。它是可用的最快的 Python 框架之一。
  </x-card>
  <x-card data-title="快速编码" data-icon="lucide:file-code-2">
    将开发速度提高 200% 到 300%。
  </x-card>
  <x-card data-title="更少错误" data-icon="lucide:bug-off">
    减少约 40% 的人为错误。
  </x-card>
  <x-card data-title="直观易用" data-icon="lucide:lightbulb">
    强大的编辑器支持，随处可自动补全，最大限度地减少调试时间。
  </x-card>
  <x-card data-title="简单易学" data-icon="lucide:graduation-cap">
    设计上易于使用和学习，因此您花费更少的时间阅读文档。
  </x-card>
  <x-card data-title="健壮" data-icon="lucide:shield-check">
    通过自动生成的交互式文档，获得可用于生产的代码。
  </x-card>
  <x-card data-title="简短" data-icon="lucide:file-minus-2">
    最大限度地减少代码重复。通过每个参数声明获得多种功能。
  </x-card>
  <x-card data-title="基于标准" data-icon="lucide:book-check">
    基于并完全兼容 API 的开放标准：OpenAPI 和 JSON Schema。
  </x-card>
</x-cards>

## 一个简单的示例

让我们通过一个基本示例来了解 FastAPI 的实际应用。

### 1. 创建代码

创建一个名为 `main.py` 的文件：

```python main.py icon=logos:python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

### 2. 运行服务器

在终端中执行以下命令：

```console Terminal icon=mdi:console
$ fastapi dev main.py

 ╭────────── FastAPI CLI - Development mode ───────────╮
 │                                                     │
 │  Serving at: http://127.0.0.1:8000                  │
 │                                                     │
 │  API docs: http://127.0.0.1:8000/docs               │
 │                                                     │
 │  Running in development mode, for production use:   │
 │                                                     │
 │  fastapi run                                        │
 │                                                     │
 ╰─────────────────────────────────────────────────────╯

INFO:     Will watch for changes in these directories: ['/home/user/code/awesomeapp']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [2248755] using WatchFiles
INFO:     Started server process [2248757]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

### 3. 检查 API

打开浏览器并访问 [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。您将看到以下 JSON 响应：

```json Response icon=logos:json
{
  "item_id": 5,
  "q": "somequery"
}
```

仅用这几行代码，您就创建了一个 API，它能够：
*   在 `/` 和 `/items/{item_id}` 接收 HTTP 请求。
*   验证 `item_id` 是一个整数。
*   处理一个可选的字符串查询参数 `q`。
*   自动在 JSON 和数据之间进行转换。

### 4. 交互式 API 文档

FastAPI 会根据 OpenAPI 标准自动生成交互式 API 文档。

#### Swagger UI

访问 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看 Swagger UI 文档。您可以直接在浏览器中查看您的端点并与 API 进行交互。

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

#### ReDoc

如需其他风格的文档，请访问 [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)。

![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

## 性能

独立的 TechEmpower 基准测试表明，在 Uvicorn 下运行的 FastAPI 应用程序是[可用的最快的 Python 框架之一](https://www.techempower.com/benchmarks/#section=test&runid=7464e520-0dc2-473d-bd34-dbdfd7e85911&hw=ph&test=query&l=zijzen-7)，这得益于它基于 Starlette 进行 Web 处理和基于 Pydantic 进行数据验证与序列化。

## 后续步骤

这只是对 FastAPI 功能的快速一瞥。要真正了解其强大功能和特性，请深入阅读我们的分步指南。

<x-card data-title="教程：第一步" data-href="/tutorials/first-steps" data-icon="lucide:play-circle" data-cta="开始教程">
  开始本教程，从头开始安装 FastAPI 并构建您的第一个应用程序。
</x-card>