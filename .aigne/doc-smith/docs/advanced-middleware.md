# Middleware

Middleware is a function that works with every request before it is processed by any specific *path operation* and also with every response before returning it. It provides a mechanism for hooking into the request and response processing pipeline to perform cross-cutting operations.

Common use cases for middleware include:

*   Adding custom headers to requests or responses.
*   Logging each request.
*   Handling authentication or authorization.
*   Implementing GZip compression.
*   Managing CORS (Cross-Origin Resource Sharing) headers.

Middleware processes requests in the order they are added and processes responses in the reverse order. This can be visualized as layers of an onion that a request must pass through to reach your application code, and then pass back through on its way out.

```d2
direction: down

"Client": {
  shape: person
}

"Middleware Stack": {
  shape: package
  grid-columns: 1

  "Middleware 1 (e.g., GZip)": {
    shape: rectangle
  }
  "Middleware 2 (e.g., CORS)": {
    shape: rectangle
  }
  "Custom Middleware": {
    shape: rectangle
  }
}

"FastAPI Application": {
  shape: rectangle
  "Path Operation Code"
}

"Client" -> "Middleware Stack"."Middleware 1 (e.g., GZip)": "1. Request"

"Middleware Stack"."Middleware 1 (e.g., GZip)" -> "Middleware Stack"."Middleware 2 (e.g., CORS)": "2. Request"
"Middleware Stack"."Middleware 2 (e.g., CORS)" -> "Middleware Stack"."Custom Middleware": "3. Request"
"Middleware Stack"."Custom Middleware" -> "FastAPI Application": "4. Request passed to endpoint"

"FastAPI Application" -> "Middleware Stack"."Custom Middleware": "5. Response from endpoint"
"Middleware Stack"."Custom Middleware" -> "Middleware Stack"."Middleware 2 (e.g., CORS)": "6. Response"
"Middleware Stack"."Middleware 2 (e.g., CORS)" -> "Middleware Stack"."Middleware 1 (e.g., GZip)": "7. Response"

"Middleware Stack"."Middleware 1 (e.g., GZip)" -> "Client": "8. Final Response (e.g., GZipped)"
```

## Creating Custom Middleware

You can create your own middleware using the `@app.middleware("http")` decorator. This function receives the `request` object and a `call_next` function that will receive the `request` as a parameter. `call_next` passes the request to the next middleware or to the path operation.

Here's an example that calculates the processing time for a request and adds it as a custom header `X-Process-Time` to the response.

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

In this example:
1.  The start time is recorded before processing the request.
2.  `await call_next(request)` passes control to the next layer of the application (either another middleware or the actual path operation).
3.  Once the response is generated and returned, the code calculates the total process time.
4.  The calculated time is added to the response headers.
5.  The final modified response is returned.

## Included Middleware

FastAPI includes several useful middleware implementations from Starlette that you can add to your application using `app.add_middleware()`.

### HTTPSRedirectMiddleware

This middleware enforces that all incoming requests must be either `https` or `wss`. If a request arrives with `http` or `ws`, it is redirected to the secure scheme.

```python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

app.add_middleware(HTTPSRedirectMiddleware)


@app.get("/")
async def main():
    return {"message": "Hello World"}

```

### TrustedHostMiddleware

This middleware enforces that all incoming requests have a correctly set `Host` header to protect against HTTP Host header attacks. You must specify a list of allowed hostnames.

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

If a request's `Host` header does not match any of the patterns in `allowed_hosts`, it will receive a 400 Bad Request response.

### GZipMiddleware

This middleware handles GZip compression for responses. If the client supports GZip (`Accept-Encoding` header), responses will be compressed, which can reduce bandwidth usage.

```python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

app.add_middleware(GZipMiddleware, minimum_size=1000, compresslevel=5)


@app.get("/")
async def main():
    # This response will be compressed if its size is > 1000 bytes
    # and the client supports gzip.
    return "somebigcontent" * 200

```

Key parameters:
*   `minimum_size`: Only compress responses that are larger than this number of bytes. Defaults to 500.
*   `compresslevel`: An integer from 0 to 9 specifying the compression level. 9 is slowest and most compressed, 1 is fastest and least compressed. Defaults to 6.

### CORSMiddleware

This middleware handles Cross-Origin Resource Sharing (CORS), which is necessary when a frontend application running on a different domain needs to communicate with your API. It allows you to specify which origins, methods, and headers are permitted.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost",
    "http://localhost:3000",
    "https://your-frontend-domain.com",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins, # Allows specific origins
    allow_credentials=True,
    allow_methods=["*"],      # Allows all methods
    allow_headers=["*"],      # Allows all headers
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

This configuration enables clients from the specified `origins` to make requests to your API.

---

By leveraging middleware, you can add powerful, reusable functionality to your FastAPI application. After setting up your middleware, you might want to explore how to enable real-time communication in your app. See the [WebSockets](./advanced-websockets.md) section to learn more.