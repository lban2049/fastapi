# WebSockets

FastAPI 支持 WebSockets，可以在客户端与服务器之间实现实时、双向的通信。

这对于需要即时更新的应用（例如聊天应用、实时通知或协作编辑工具）非常有用。

WebSocket 连接的生命周期如下图所示：

```d2
shape: sequence_diagram

Client; Server

Client -> Server: "HTTP GET /ws (Upgrade Request)"
Server -> Client: "101 Switching Protocols"
note over Client, Server: "WebSocket Connection Established"
Client -> Server: "Send Message"
Server -> Client: "Send Message"
Client <-> Server: "Bidirectional Communication..."
```

## 第一步

创建 WebSocket 端点与创建 HTTP 端点类似，都需要使用 `@app.websocket()` 装饰器。

下面是一个简单的 WebSocket 聊天服务器示例，它会将收到的任何消息回显。

### 服务器端代码

```python
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>聊天</title>
    </head>
    <body>
        <h1>WebSocket 聊天</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>发送</button>
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

在此示例中：

1.  `@app.websocket("/ws")` 声明了一个路径为 `/ws` 的 WebSocket 端点。
2.  该函数接收一个 `WebSocket` 对象作为参数。
3.  `await websocket.accept()` 用于建立并接受 WebSocket 连接。此操作必须在发送或接收消息前完成。
4.  使用 `while True` 循环持续监听传入的消息。
5.  `await websocket.receive_text()` 等待来自客户端的消息。
6.  `await websocket.send_text(...)` 向客户端发回消息。

### 客户端代码 (HTML 和 JavaScript)

该 Python 脚本还提供了一个简单的 HTML 页面，其中包含用于与 WebSocket 端点交互的 JavaScript。

-   `var ws = new WebSocket("ws://localhost:8000/ws");`：此行代码用于建立与服务器的连接。请注意 `ws://` 协议。
-   `ws.onmessage`：此函数是一个事件处理程序，每当从服务器收到消息时就会被调用。它会创建一个新的列表项并将其添加到页面中。
-   `ws.send(input.value)`：此行代码用于将文本输入框的内容发送到服务器。


## 使用 `Depends` 和其他依赖项

与常规的*路径操作*一样，你也可以在 WebSocket 端点中使用依赖项，包括 `Depends`、`Path`、`Query`、`Cookie` 等。

这使你可以为 WebSocket 连接添加身份验证、数据验证或其他共享逻辑。

```python
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

关键点：

-   `websocket_endpoint` 函数现在可以接受路径参数（`item_id`）、查询参数（`q`）和一个依赖项（`cookie_or_token`）。
-   `get_cookie_or_token` 依赖项函数需要 `session` Cookie 或 `token` 查询参数。如果两者均未提供，它将引发 `WebSocketException`，并使用特定的错误代码来正常关闭连接。


## 处理断开连接和广播

WebSocket 的一个常见用例是聊天应用，其中消息需要广播给多个客户端。要实现此功能，你需要管理一个活动连接列表。

下面是一个使用 `ConnectionManager` 类来处理连接和广播消息的示例。

```python
from typing import List

from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse

app = FastAPI()

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

在这个更高级的示例中：

-   `ConnectionManager` 类维护一个 `active_connections` 列表。
-   当客户端连接时，`connect` 方法会接受该连接并将其添加到列表中。
-   当收到消息时，该消息会作为个人消息发送回原始客户端，并同时广播给所有其他已连接的客户端。
-   主逻辑被包裹在 `try...except WebSocketDisconnect` 代码块中。当客户端断开连接时，会引发 `WebSocketDisconnect` 异常。`except` 代码块会捕获此异常，将该客户端从活动连接列表中移除，并广播一条消息，通知其他客户端该用户已离开。

现在你已经学会了如何创建 WebSocket 端点、使用依赖项以及管理多个连接以广播消息。

接下来，你可能想学习如何使用多个文件来组织大型应用。你可以在[更大型的应用](./advanced-bigger-applications.md)中阅读相关内容。