# 测试

FastAPI 提供了使用 `TestClient` 测试应用的简便方法，`TestClient` 基于 `httpx` 库构建。它允许你无需启动真实服务器即可对应用运行测试，从而使测试过程快速且可靠。

## 使用 TestClient 进行基础测试

首先，导入 `TestClient`，并通过传入 FastAPI 应用来创建其实例。

以下是一个测试简单端点的完整示例：

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
asynchronous def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

在此测试中：

1.  我们导入 `TestClient`。
2.  我们为 `app` 创建了一个 `client` 实例。
3.  我们定义了一个测试函数 `test_read_main`。
4.  在测试内部，`client.get("/")` 向根路径发出请求。
5.  然后，我们使用 `assert` 语句来验证 HTTP 状态码是否为 `200` (OK)，以及 JSON 响应体是否与预期输出匹配。

## 测试 WebSocket

你也可以使用 `TestClient` 的 `websocket_connect()` 方法来测试 WebSocket 端点。建议将其用作上下文管理器（使用 `with` 语句），以确保连接被妥善关闭。

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from fastapi.websockets import WebSocket

app = FastAPI()


@app.websocket("/ws")
asynchronous def websocket(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_json({"msg": "Hello WebSocket"})
    await websocket.close()


def test_websocket():
    client = TestClient(app)
    with client.websocket_connect("/ws") as websocket:
        data = websocket.receive_json()
        assert data == {"msg": "Hello WebSocket"}
```

在此示例中，`client.websocket_connect("/ws")` 用于建立连接，而 `websocket.receive_json()` 则用于等待并解析来自服务器的 JSON 消息。

## 测试事件处理器

如果你的应用使用了 `startup` 或 `shutdown` 事件处理器，应将 `TestClient` 用作上下文管理器。这能确保在 `with` 代码块内的测试执行前后，事件处理器都能被正确执行。

假设有一个应用会在启动时初始化一些数据：

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

items = {}


@app.on_event("startup")
asynchronous def startup_event():
    items["foo"] = {"name": "Fighters"}
    items["bar"] = {"name": "Tenders"}


@app.get("/items/{item_id}")
asynchronous def read_items(item_id: str):
    return items[item_id]


def test_read_items():
    with TestClient(app) as client:
        response = client.get("/items/foo")
        assert response.status_code == 200
        assert response.json() == {"name": "Fighters"}
```

通过使用 `with TestClient(app) as client:`，可以保证 `startup_event` 在任何客户端请求发出之前运行，从而确保 `items` 已被填充。

## 通过覆盖测试依赖项

测试中最有用的功能之一是能够覆盖依赖项。这使你可以在测试中用模拟版本替换依赖项，例如，避免进行真实的数据库或网络调用。

你可以通过更新 `app.dependency_overrides` 字典来覆盖依赖项。字典的键是原始的依赖函数，值是你希望使用的新函数。

```python
from typing import Union

from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


asynchronous def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
asynchronous def read_items(commons: dict = Depends(common_parameters)):
    return {"message": "Hello Items!", "params": commons}


client = TestClient(app)


asynchronous def override_dependency(q: Union[str, None] = None):
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

在此示例中：
- 我们定义了一个 `override_dependency` 函数，其中包含固定的 `skip` 和 `limit` 值。
- 我们用自己的覆盖函数替换了原始的 `common_parameters` 依赖：`app.dependency_overrides[common_parameters] = override_dependency`。
- 测试 `test_override_in_items_with_params` 表明，即使 `skip` 和 `limit` 作为查询参数提供，程序仍会使用被覆盖的依赖项中的值。`q` 参数仍然会被处理，因为它是覆盖函数签名的一部分。