# WebSockets

FastAPI provides support for WebSockets to enable real-time, bidirectional communication between clients and your server.

This is useful for applications that require instant updates, such as chat applications, live notifications, or collaborative editing tools.

Here's a visual representation of the WebSocket connection lifecycle:

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

## First Steps

Creating a WebSocket endpoint is similar to creating an HTTP endpoint. You use the `@app.websocket()` decorator.

Here's a simple example of a WebSocket chat server that echoes back any message it receives.

### Server-side Code

```python
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

In this example:

1.  `@app.websocket("/ws")` declares a WebSocket endpoint at the path `/ws`.
2.  The function receives a `WebSocket` object as a parameter.
3.  `await websocket.accept()` establishes and accepts the WebSocket connection. This must be done before you can send or receive messages.
4.  A `while True` loop is used to continuously listen for incoming messages.
5.  `await websocket.receive_text()` waits for a message from the client.
6.  `await websocket.send_text(...)` sends a message back to the client.

### Client-side Code (HTML & JavaScript)

The Python script also serves a simple HTML page with JavaScript to interact with the WebSocket endpoint.

-   `var ws = new WebSocket("ws://localhost:8000/ws");`: This line establishes the connection to the server. Note the `ws://` protocol.
-   `ws.onmessage`: This function is an event handler that gets called whenever a message is received from the server. It creates a new list item and adds it to the page.
-   `ws.send(input.value)`: This sends the content of the text input to the server.


## Using `Depends` and Other Dependencies

Just like with regular *path operations*, you can use dependencies with your WebSocket endpoints. This includes `Depends`, `Path`, `Query`, `Cookie`, and more.

This allows you to add authentication, data validation, or other shared logic to your WebSocket connections.

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

Key points:

-   The `websocket_endpoint` function now accepts path parameters (`item_id`), query parameters (`q`), and a dependency (`cookie_or_token`).
-   The `get_cookie_or_token` dependency function requires either a `session` cookie or a `token` query parameter. If neither is present, it raises a `WebSocketException` to cleanly close the connection with a specific error code.


## Handling Disconnections and Broadcasting

A common use case for WebSockets is a chat application where messages are broadcast to multiple clients. To do this, you need to manage the list of active connections.

Here is an example that uses a `ConnectionManager` class to handle connections and broadcast messages.

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

In this more advanced example:

-   The `ConnectionManager` class maintains a list of `active_connections`.
-   When a client connects, the `connect` method accepts the connection and adds it to the list.
-   When a message is received, it is sent back to the original client as a personal message and also broadcast to all other connected clients.
-   The main logic is wrapped in a `try...except WebSocketDisconnect` block. When a client disconnects, a `WebSocketDisconnect` exception is raised. The `except` block catches this, removes the client from the list of active connections, and broadcasts a message informing other clients that they have left.

You have now learned how to create WebSocket endpoints, use dependencies, and manage multiple connections for broadcasting messages.

Next, you might want to learn how to structure larger applications with multiple files. You can read about that in [Bigger Applications](./advanced-bigger-applications.md).