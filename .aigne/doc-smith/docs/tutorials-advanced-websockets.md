# WebSockets

FastAPI provides excellent support for <a href="https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API" target="_blank">WebSockets</a>, enabling real-time, bidirectional communication between the client and server. This is perfect for applications like chat rooms, live notifications, or collaborative editing tools.

Under the hood, FastAPI uses Starlette's powerful WebSocket handling capabilities, so you get a robust and high-performance implementation out of the box.

## First Steps

Let's start with a basic "echo" WebSocket. The server will receive a message from a client and send the same message back.

First, you need to define a WebSocket endpoint using the `@app.websocket()` decorator.

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

Let's break down the WebSocket endpoint:

1.  `@app.websocket("/ws")`: This decorator declares the function below it as the handler for the WebSocket at the path `/ws`.
2.  `async def websocket_endpoint(websocket: WebSocket):`: The function is `async` and receives a `WebSocket` object as a parameter.
3.  `await websocket.accept()`: This is crucial. You must `accept` the connection first before you can communicate with the client. This completes the WebSocket handshake.
4.  `while True:`: We enter an infinite loop to listen for incoming messages.
The connection will remain open until the client or server disconnects.
5.  `data = await websocket.receive_text()`: This line waits for a text message to arrive from the client.
You can also use `receive_bytes()` or `receive_json()`.
6.  `await websocket.send_text(...)`: This sends a text message back to the client. You can also use `send_bytes()` or `send_json()`.

The HTML and JavaScript part sets up a simple frontend to connect to the WebSocket, send messages, and display the messages it receives back from the server.

## Using `Depends` and other dependencies

The great thing about FastAPI is that you can use the same dependency injection system you're familiar with for your WebSocket endpoints.

This means you can have path parameters, query parameters, cookies, headers, and dependencies with `Depends`.

Here's an example that uses path parameters, query parameters, and a dependency that checks for a cookie or a query token.

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

In this example:
- The WebSocket path `/items/{item_id}/ws` includes a path parameter `item_id`.
- The function signature includes the path parameter `item_id`, an optional query parameter `q`, and our dependency `cookie_or_token`.
- The `get_cookie_or_token` dependency tries to get a `session` cookie or a `token` query parameter. If neither is present, it raises a `WebSocketException`. This will reject the connection with a specific WebSocket error code.

## Handling Disconnections and Broadcasting

A common use case is a chat application where messages from one client are broadcast to all other connected clients. To do this, you need to maintain a list of active connections.

It's also crucial to handle disconnections properly to remove clients from the list when they leave.

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

Key patterns in this example:
- A `ConnectionManager` class is created to manage the list of active `WebSocket` connections.
- When a client connects, `manager.connect(websocket)` adds them to the list.
- The core logic is wrapped in a `try...except WebSocketDisconnect:` block. When a client disconnects, `websocket.receive_text()` will raise a `WebSocketDisconnect` exception.
- In the `except` block, we remove the client from the list using `manager.disconnect(websocket)` and notify the other clients that they have left.
- The `manager.broadcast()` method iterates over all active connections and sends a message to each one.

## Recap

You've learned how to build powerful real-time applications with FastAPI's WebSockets:
- Use the `@app.websocket()` decorator to create a WebSocket endpoint.
- Manage the connection lifecycle with `websocket.accept()` and by handling `WebSocketDisconnect` exceptions.
- Use `websocket.receive_text()` and `websocket.send_text()` for communication.
- Leverage the full power of FastAPI's dependency injection system with path parameters, query parameters, and `Depends`.
- Manage multiple clients and broadcast messages using a connection manager.

Now you have the tools to add real-time features to your FastAPI applications. A great next step is to learn how to structure larger projects. Check out the guide on [Bigger Applications](./tutorials-advanced-bigger-applications.md).