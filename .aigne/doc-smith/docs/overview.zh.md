# 概述

FastAPI 是一个现代、高性能的 Web 框架，用于基于标准 Python 类型提示构建 API。它被设计为易于使用、编码快速，并为生产环境做好准备。

## 主要特性

FastAPI 提供了高效且愉悦的开发体验，专注于速度、简洁性和标准。

<x-cards data-columns="3">
  <x-card data-title="性能卓越" data-icon="lucide:rocket">
    得益于其 Starlette 和 Pydantic 基础，性能可与 NodeJS 和 Go 相媲美。它是现有最快的 Python 框架之一。
  </x-card>
  <x-card data-title="编码快速" data-icon="lucide:zap">
    将功能开发速度提高 200% 到 300%。最大限度地减少代码重复，用更少的代码完成更多工作。
  </x-card>
  <x-card data-title="更少 Bug" data-icon="lucide:shield-check">
    将开发人员导致的错误减少约 40%。类型提示和结构化数据验证可在 Bug 到达生产环境之前将其捕获。
  </x-card>
  <x-card data-title="直观易用" data-icon="lucide:lightbulb">
    受益于出色的编辑器支持和无处不在的自动补全功能。花更少的时间调试，更多的时间进行构建。
  </x-card>
  <x-card data-title="易于学习" data-icon="lucide:book-open">
    设计旨在使其易于学习和使用。文档清晰明了，让您可以专注于应用程序的逻辑。
  </x-card>
  <x-card data-title="基于标准" data-icon="lucide:file-json-2">
    完全兼容 API 的开放标准，包括 OpenAPI（前身为 Swagger）和 JSON Schema。
  </x-card>
</x-cards>

## 核心架构

FastAPI 的性能和特性之所以成为可能，是因为它站在巨人的肩膀上：

-   **[Starlette](https://www.starlette.io/)**：用于所有 Web 部分，提供了一个轻量级、高性能的 ASGI 框架。
-   **[Pydantic](https://docs.pydantic.dev/)**：处理所有数据部分，基于 Python 类型提示提供强大的数据验证、序列化和文档功能。

这种关注点分离使得 FastAPI 在 Web 处理和数据管理方面都表现出色。

```d2
direction: down

"Your-API-Code": {
  label: "你的 API 代码"
  shape: rectangle
  style.fill: "#DDF0FF"
}

FastAPI: {
  shape: rectangle
  "Your-API-Code"
}

Starlette: {
  shape: rectangle
  label: "Starlette (Web 层)"
  style.fill: "#D5E8D4"
}

Pydantic: {
  shape: rectangle
  label: "Pydantic (数据层)"
  style.fill: "#FAD7AC"
}

FastAPI -> Starlette: "用于所有 Web 部分"
FastAPI -> Pydantic: "用于所有数据部分"

```

## 代码一瞥

看看创建一个功能齐全并带有自动文档的 API 是多么简单。

### 1. 创建文件

创建一个名为 `main.py` 的文件，内容如下：

```python title="main.py"
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

在终端中执行此命令：

```console
$ fastapi dev main.py

INFO:     Uvicorn 正在 http://127.0.0.1:8000 上运行 (按 CTRL+C 退出)
```

### 3. 查看交互式文档

FastAPI 会根据你的代码自动生成交互式 API 文档。只需在浏览器中打开 [`http://127.0.0.1:8000/docs`](http://127.0.0.1:8000/docs) 即可查看。

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)

## 深受行业领导者信赖

全球领先的公司都在生产环境中使用 FastAPI。

> “[...] 我最近在大量使用 **FastAPI**。[...] 我实际上计划将其用于我团队在 **微软** 的所有 **ML 服务**。其中一些服务正在被集成到核心 **Windows** 产品和一些 **Office** 产品中。”
> 
> **Kabir Khan - 微软**

> “我们采用 **FastAPI** 库来生成一个 **REST** 服务器，可以通过查询该服务器来获取**预测结果**。[针对 Ludwig]”
> 
> **Piero Molino、Yaroslav Dudin 和 Sai Sumanth Miryala - 优步**

> “**Netflix** 宣布开源其**危机管理**编排框架：**Dispatch**！[使用 **FastAPI** 构建]”
> 
> **Kevin Glisson、Marc Vilanova、Forest Monsen - Netflix**

## 后续步骤

本概述介绍了 FastAPI 的高级优势。要开始构建您的第一个 API，请转到我们的分步教程。

<x-card data-title="快速入门" data-icon="lucide:play-circle" data-href="/getting-started" data-cta="开始教程">
  安装 FastAPI 并创建您的第一个应用程序的分步指南。
</x-card>