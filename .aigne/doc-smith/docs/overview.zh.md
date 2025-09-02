# 概述

FastAPI 是一个用于通过 Python 构建 API 的现代化、高性能 Web 框架，它利用了标准的 Python 类型提示。其设计宗旨是易于使用、编码快速且可用于生产环境。


### 主要特性

FastAPI 旨在优化开发体验和最终应用程序的性能。

<x-cards data-columns="2">
  <x-card data-title="性能卓越" data-icon="lucide:rocket">
    得益于其基于 Starlette（用于 Web 部分）和 Pydantic（用于数据部分）的基础，实现了可与 **NodeJS** 和 **Go** 相媲美的高性能。它是目前可用的最快的 Python 框架之一。
  </x-card>
  <x-card data-title="编码快速" data-icon="lucide:zap">
    将开发速度提升 200% 到 300%。该框架旨在帮助您以最少、最直观的代码快速构建功能。
  </x-card>
  <x-card data-title="更少 Bug" data-icon="lucide:bug-off">
    将人为导致的错误减少约 40%。通过类型提示，您可以获得出色的编辑器支持和数据验证，从而在开发过程中捕获许多错误。
  </x-card>
  <x-card data-title="稳健且生产就绪" data-icon="lucide:shield-check">
    通过基于开放标准的自动交互式文档、数据验证和序列化，获得生产就绪的代码。
  </x-card>
</x-cards>

### 核心架构

FastAPI 站在两大巨人的肩膀上：Starlette 负责所有 Web 部分，Pydantic 负责所有数据部分。这种分层方法使其能够同时提供高性能和稳健的数据处理能力。

```d2
direction: right

"用户请求" -> "FastAPI 引擎"

"FastAPI 引擎": {
  shape: cloud
  "你的 API 代码 (带类型提示)": {
    shape: document
  }
  "Starlette (Web 工具包)": {
    shape: hexagon
  }
  "Pydantic (数据验证)": {
    shape: hexagon
  }
}

"FastAPI 引擎" -> "API 响应 (JSON)"

"你的 API 代码 (带类型提示)" -> "Starlette (Web 工具包)": 用于路由
"你的 API 代码 (带类型提示)" -> "Pydantic (数据验证)": 用于验证和序列化
```

### 一个快速示例

创建一个 FastAPI 应用程序非常简单。下面是一个完整的示例：

**1. 创建一个 `main.py` 文件：**

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

**2. 运行服务器：**

```console
$ fastapi dev main.py

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

**3. 检查结果：**

在浏览器中打开 [http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。你将看到以下 JSON 响应：

```json
{"item_id":5,"q":"somequery"}
```

### 自动交互式文档

FastAPI 最受重视的功能之一是自动生成交互式 API 文档。无需任何额外工作，您就可以获得两个文档用户界面：

- **Swagger UI**，可在 `/docs` 访问：

  ![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

- **ReDoc**，可在 `/redoc` 访问：

  ![ReDoc](https://fastapi.tiangolo.com/img/index/index-02-redoc-simple.png)

### 深受行业领导者信赖

FastAPI 已被领先的科技公司用于生产环境中的关键服务。

> “如今，我正在大量使用 **FastAPI**。[...] 我实际上正计划将其用于我团队在 **Microsoft** 的所有 **ML 服务**。其中一些服务正在被集成到核心的 **Windows** 产品和一些 **Office** 产品中。”
> <div style="text-align: right; margin-right: 10%;">Kabir Khan - <strong>Microsoft</strong></div>

> “我们采用了 **FastAPI** 库来生成一个 **REST** 服务器，该服务器可以被查询以获取**预测结果**。[用于 Ludwig]”
> <div style="text-align: right; margin-right: 10%;">Piero Molino, Yaroslav Dudin, and Sai Sumanth Miryala - <strong>Uber</strong></div>

> “**Netflix** 很高兴地宣布，我们开源了我们的**危机管理**编排框架：**Dispatch**！[使用 **FastAPI** 构建]”
> <div style="text-align: right; margin-right: 10%;">Kevin Glisson, Marc Vilanova, Forest Monsen - <strong>Netflix</strong></div>

### 后续步骤

本概述让您得以一窥 FastAPI 成为 API 开发的有力选择的原因。您已经看到了它的主要特性、一个简单的代码示例以及其自动文档的强大功能。

准备好构建您的第一个应用程序了吗？请前往我们的[入门指南](./getting-started.md)查看分步教程。
