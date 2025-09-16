# WebSockets

FastAPI 为 <a href="https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API" target="_blank">WebSockets</a> 提供了出色的支持，能够实现客户端和服务器之间的实时双向通信。这非常适合聊天室、实时通知或协作编辑工具等应用。

在底层，FastAPI 使用 Starlette 强大的 WebSocket 处理功能，因此你可以开箱即用地获得一个健壮且高性能的实现。

## 快速入门

让我们从一个基本的“回声” WebSocket 开始。服务器将从客户端接收一条消息，然后将相同的消息发送回去。

首先，你需要使用 `@app.websocket()` 装饰器来定义一个 WebSocket 端点。

```python title="main.py" icon=logos:python
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""


@app.get("/")
async def get():
    return HTMLResponse(html)


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")

```

让我们来分解一下这个 WebSocket 端点：

1.  `@app.websocket("/ws")`: 这个装饰器将它下面的函数声明为路径 `/ws` 上的 WebSocket 处理器。
2.  `async def websocket_endpoint(websocket: WebSocket):`: 该函数是 `async` 异步的，并接收一个 `WebSocket` 对象作为参数。
3.  `await websocket.accept()`: 这一点至关重要。你必须先 `accept` 连接，然后才能与客户端通信。这一步完成了 WebSocket 握手。
4.  `while True:`: 我们进入一个无限循环来监听传入的消息。
连接将保持打开状态，直到客户端或服务器断开连接。
5.  `data = await websocket.receive_text()`: 这行代码等待从客户端接收文本消息。
你也可以使用 `receive_bytes()` 或 `receive_json()`。
6.  `await websocket.send_text(...)`: 这行代码将文本消息发送回客户端。你也可以使用 `send_bytes()` 或 `send_json()`。

HTML 和 JavaScript 部分建立了一个简单的前端，用于连接 WebSocket、发送消息以及显示从服务器接收返回的消息。

## 使用 Depends 和其他依赖项

FastAPI 的一个很棒之处在于，你可以为 WebSocket 端点使用你所熟悉的同一套依赖注入系统。

这意味着你可以使用路径参数、查询参数、Cookie、标头以及带 `Depends` 的依赖项。

下面是一个使用路径参数、查询参数以及一个检查 Cookie 或查询令牌的依赖项的示例。

```python title="main.py" icon=logos:python
from typing import Union

from fastapi import (
    Cookie,
    Depends,
    FastAPI,
    Query,
    WebSocket,
    WebSocketException,
    status,
)
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <label>Item ID: <input type="text" id="itemId" autocomplete="off" value="foo"/></label>
            <label>Token: <input type="text" id="token" autocomplete="off" value="some-key-token"/></label>
            <button onclick="connect(event)">Connect</button>
            <hr>
            <label>Message: <input type="text" id="messageText" autocomplete="off"/></label>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
        var ws = null;
            function connect(event) {
                var itemId = document.getElementById("itemId")
                var token = document.getElementById("token")
                ws = new WebSocket("ws://localhost:8000/items/" + itemId.value + "/ws?token=" + token.value);
                ws.onmessage = function(event) {
                    var messages = document.getElementById('messages')
                    var message = document.createElement('li')
                    var content = document.createTextNode(event.data)
                    message.appendChild(content)
                    messages.appendChild(message)
                };
                event.preventDefault()
            }
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""


@app.get("/")
async def get():
    return HTMLResponse(html)


async def get_cookie_or_token(
    websocket: WebSocket,
    session: Union[str, None] = Cookie(default=None),
    token: Union[str, None] = Query(default=None),
):
    if session is None and token is None:
        raise WebSocketException(code=status.WS_1008_POLICY_VIOLATION)
    return session or token


@app.websocket("/items/{item_id}/ws")
async def websocket_endpoint(
    websocket: WebSocket,
    item_id: str,
    q: Union[int, None] = None,
    cookie_or_token: str = Depends(get_cookie_or_token),
):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(
            f"Session cookie or query token value is: {cookie_or_token}"
        )
        if q is not None:
            await websocket.send_text(f"Query parameter q is: {q}")
        await websocket.send_text(f"Message text was: {data}, for item ID: {item_id}")

```

在此示例中：
- WebSocket 路径 `/items/{item_id}/ws` 包含一个路径参数 `item_id`。
- 函数签名包含路径参数 `item_id`、一个可选的查询参数 `q` 以及我们的依赖项 `cookie_or_token`。
- `get_cookie_or_token` 依赖项会尝试获取 `session` Cookie 或 `token` 查询参数。如果两者都不存在，它会引发一个 `WebSocketException`。这将使用一个特定的 WebSocket 错误代码拒绝连接。

## 处理断开连接和广播

一个常见的用例是聊天应用，其中来自一个客户端的消息会广播给所有其他已连接的客户端。为此，你需要维护一个活动连接列表。

正确处理断开连接以在客户端离开时将其从列表中移除也至关重要。

```python title="main.py" icon=logos:python
from typing import List

from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <h2>Your ID: <span id="ws-id"></span></h2>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var client_id = Date.now()
            document.querySelector("#ws-id").textContent = client_id;
            var ws = new WebSocket(`ws://localhost:8000/ws/${client_id}`);
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""


class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)


manager = ConnectionManager()


@app.get("/")
async def get():
    return HTMLResponse(html)


@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.send_personal_message(f"You wrote: {data}", websocket)
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")

```

此示例中的关键模式：
- 创建了一个 `ConnectionManager` 类来管理活动 `WebSocket` 连接的列表。
- 当客户端连接时，`manager.connect(websocket)` 会将其添加到列表中。
- 核心逻辑被包装在一个 `try...except WebSocketDisconnect:` 块中。当客户端断开连接时，`websocket.receive_text()` 会引发一个 `WebSocketDisconnect` 异常。
- 在 `except` 块中，我们使用 `manager.disconnect(websocket)` 将客户端从列表中移除，并通知其他客户端他们已经离开。
- `manager.broadcast()` 方法遍历所有活动连接并向每个连接发送消息。

## 总结

你已经学会了如何使用 FastAPI 的 WebSockets 构建强大的实时应用：
- 使用 `@app.websocket()` 装饰器创建一个 WebSocket 端点。
- 使用 `websocket.accept()` 和处理 `WebSocketDisconnect` 异常来管理连接生命周期。
- 使用 `websocket.receive_text()` 和 `websocket.send_text()` 进行通信。
- 利用 FastAPI 依赖注入系统的全部功能，包括路径参数、查询参数和 `Depends`。
- 使用连接管理器来管理多个客户端并广播消息。

现在你已经掌握了为 FastAPI 应用添加实时功能的工具。下一步，一个很好的进阶是学习如何组织大型项目。请查阅关于[大型应用](./tutorials-advanced-bigger-applications.md)的指南。