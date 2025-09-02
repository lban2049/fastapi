# 测试

FastAPI 提供了一种简单直接的方式来测试你的 API。它基于 Starlette 的 `TestClient` 并使用 `httpx`，使其能够与 `async` 应用兼容。你可以直接将其与 `pytest` 等测试框架结合使用。

本指南将介绍如何为你的应用编写有效的测试，内容涵盖 WebSocket、事件处理器和依赖项覆盖。

## 一个基本示例

让我们从一个简单的 FastAPI 应用开始，该应用位于一个文件中，例如 `main.py`：

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

要测试该应用，你需要导入 `TestClient` 并将 FastAPI `app` 对象传递给它。然后，你就可以在测试函数中向应用发出请求，就像使用 `httpx` 或 `requests` 一样。

测试函数 `test_read_main` 的作用如下：
1.  使用 `client.get("/")` 向 `/` 发出 `GET` 请求。
2.  断言响应状态码为 `200` (OK)。
3.  断言响应的 JSON 主体为 `{"msg": "Hello World"}`。

## 测试 WebSocket

测试 WebSocket 端点遵循类似的模式，即使用上下文管理器来建立连接。

假设有一个包含 WebSocket 端点的应用：

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from fastapi.websockets import WebSocket

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


@app.websocket("/ws")
async def websocket(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_json({"msg": "Hello WebSocket"})
    await websocket.close()


def test_websocket():
    client = TestClient(app)
    with client.websocket_connect("/ws") as websocket:
        data = websocket.receive_json()
        assert data == {"msg": "Hello WebSocket"}

```

`client.websocket_connect("/ws")` 方法提供了一个上下文管理器。在 `with` 代码块内部，你可以与 WebSocket 进行交互，例如，通过 `websocket.receive_json()` 接收 JSON 数据，然后对这些数据进行断言。

## 测试事件处理器

如果你的应用使用了启动或关闭事件，你应该将 `TestClient` 作为上下文管理器使用（`with TestClient(app) as client:`），以确保这些事件在测试过程中被正确触发。

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

items = {}


@app.on_event("startup")
async def startup_event():
    items["foo"] = {"name": "Fighters"}
    items["bar"] = {"name": "Tenders"}


@app.get("/items/{item_id}")
async def read_items(item_id: str):
    return items[item_id]


def test_read_items():
    with TestClient(app) as client:
        response = client.get("/items/foo")
        assert response.status_code == 200
        assert response.json() == {"name": "Fighters"}
```

在本例中，`startup_event` 用一些数据填充了一个字典。通过使用 `with` 语句，我们确保该事件在 `client` 发出任何请求之前运行。这使得测试能够正确访问在应用启动阶段创建的 `/items/foo` 处的数据。

## 测试依赖项覆盖

测试中最有用的功能之一是能够覆盖依赖项。这使你可以用模拟或简化版本来替换复杂的依赖项（如数据库连接或外部 API 客户端），以便进行测试。你可以通过修改 `app.dependency_overrides` 字典来实现此目的。

```python
from typing import Union

from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return {"message": "Hello Items!", "params": commons}


client = TestClient(app)


async def override_dependency(q: Union[str, None] = None):
    return {"q": q, "skip": 5, "limit": 10}


app.dependency_overrides[common_parameters] = override_dependency


def test_override_in_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }


def test_override_in_items_with_params():
    response = client.get("/items/?q=foo&skip=100&limit=200")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }
```

下面是该过程的分解说明：
1.  我们有一个由端点使用的 `common_parameters` 依赖项。
2.  为了测试，我们创建了一个 `override_dependency` 函数，它为 `skip` 和 `limit` 返回固定的值。
3.  `app.dependency_overrides[common_parameters] = override_dependency` 这一行告诉 FastAPI，每当需要 `common_parameters` 作为依赖项时，都应改用 `override_dependency`。
4.  测试表明，即使我们在 URL 中为 `skip` 和 `limit` 提供了不同的查询参数，使用的仍然是被覆盖的依赖项返回的值，这证实了覆盖操作是成功的。

---

现在你已经了解了如何为应用编写测试、处理 WebSocket、确保事件处理器得以执行，以及如何通过覆盖依赖项进行隔离测试。要了解更多关于如何随着项目规模的增长来组织项目的信息，请查阅关于[更大型应用](./advanced-bigger-applications.md)的指南。