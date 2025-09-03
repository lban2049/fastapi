# Security Utilities

FastAPI provides a simple and powerful set of tools to handle security and authentication. These utilities, built on top of the dependency injection system, allow you to easily implement various security schemes like OAuth2, HTTP Basic/Bearer/Digest, and API Keys. They integrate directly with the automated OpenAPI documentation, making your API's security requirements clear and interactive.

This reference guide provides detailed documentation for each security class and utility model available in `fastapi.security`.

```d2
direction: down

"Security Utilities": {
  shape: package
  grid-columns: 2

  "API Key Auth": {
    shape: rectangle
    "APIKeyQuery": {label: "From Query Param"}
    "APIKeyHeader": {label: "From Header"}
    "APIKeyCookie": {label: "From Cookie"}
  }

  "HTTP Auth": {
    shape: rectangle
    "HTTPBasic": {}
    "HTTPBearer": {}
    "HTTPDigest": {}
    "HTTPBasicCredentials": {shape: document}
    "HTTPAuthorizationCredentials": {shape: document}
  }

  "OAuth2": {
    shape: rectangle
    "OAuth2PasswordBearer": {}
    "OAuth2AuthorizationCodeBearer": {}
    "OAuth2PasswordRequestForm": {shape: document}
    "OAuth2PasswordRequestFormStrict": {shape: document}
    "SecurityScopes": {shape: document}
  }

  "OpenID Connect": {
    shape: rectangle
    "OpenIdConnect": {}
  }
}
```

## API Key Authentication

API key authentication can be sourced from query parameters, headers, or cookies.

### APIKeyQuery

Extracts an API key from a query parameter. You create an instance and use it as a dependency.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `name` | `str` | The name of the query parameter for the API key. |
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI docs. |
| `description` | `Optional[str]` | A description for the security scheme in the OpenAPI docs. |
| `auto_error` | `bool` | If `True` (default), raises an HTTP 403 error if the key is missing. If `False`, the dependency returns `None`. |

**Example**

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

query_scheme = APIKeyQuery(name="api_key")


@app.get("/items/")
async def read_items(api_key: str = Depends(query_scheme)):
    return {"api_key": api_key}
```

### APIKeyHeader

Extracts an API key from an HTTP header.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `name` | `str` | The name of the HTTP header for the API key. |
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI docs. |
| `description` | `Optional[str]` | A description for the security scheme in the OpenAPI docs. |
| `auto_error` | `bool` | If `True` (default), raises an HTTP 403 error if the key is missing. If `False`, the dependency returns `None`. |

**Example**

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

header_scheme = APIKeyHeader(name="x-key")


@app.get("/items/")
async def read_items(key: str = Depends(header_scheme)):
    return {"key": key}
```

### APIKeyCookie

Extracts an API key from a request cookie.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `name` | `str` | The name of the cookie for the API key. |
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI docs. |
| `description` | `Optional[str]` | A description for the security scheme in the OpenAPI docs. |
| `auto_error` | `bool` | If `True` (default), raises an HTTP 403 error if the key is missing. If `False`, the dependency returns `None`. |

**Example**

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

## HTTP Authentication

Implements standard HTTP authentication schemes.

### HTTPBasic

Handles HTTP Basic authentication. The dependency result is an `HTTPBasicCredentials` object.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI docs. |
| `realm` | `Optional[str]` | The HTTP Basic authentication realm. |
| `description` | `Optional[str]` | A description for the security scheme in the OpenAPI docs. |
| `auto_error` | `bool` | If `True` (default), raises an error if authentication is not provided. If `False`, returns `None`. |

**Example**

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()


@app.get("/users/me")
def read_current_user(credentials: Annotated[HTTPBasicCredentials, Depends(security)]):
    return {"username": credentials.username, "password": credentials.password}
```

### HTTPBearer

Handles HTTP Bearer token authentication. The dependency result is an `HTTPAuthorizationCredentials` object.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `bearerFormat` | `Optional[str]` | The bearer token format (e.g., 'JWT'), visible in the OpenAPI docs. |
| `scheme_name` | `Optional[str]` | The security scheme name. |
| `description` | `Optional[str]` | A description for the security scheme. |
| `auto_error` | `bool` | If `True` (default), raises an error if the token is missing. If `False`, returns `None`. |

**Example**

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer

app = FastAPI()

security = HTTPBearer()


@app.get("/users/me")
def read_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
):
    return {"scheme": credentials.scheme, "credentials": credentials.credentials}
```

### HTTPDigest

Handles HTTP Digest authentication. The dependency result is an `HTTPAuthorizationCredentials` object.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI docs. |
| `description` | `Optional[str]` | A description for the security scheme. |
| `auto_error` | `bool` | If `True` (default), raises an error if the digest is missing. If `False`, returns `None`. |

**Example**

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import HTTPAuthorizationCredentials, HTTPDigest

app = FastAPI()

security = HTTPDigest()


@app.get("/users/me")
def read_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
):
    return {"scheme": credentials.scheme, "credentials": credentials.credentials}
```

### HTTPBasicCredentials

A data model containing the username and password from HTTP Basic auth.

**Attributes**

| Attribute | Type | Description |
|---|---|---|
| `username` | `str` | The HTTP Basic username. |
| `password` | `str` | The HTTP Basic password. |

### HTTPAuthorizationCredentials

A data model containing the scheme and credentials from an `Authorization` header.

**Attributes**

| Attribute | Type | Description |
|---|---|---|
| `scheme` | `str` | The authorization scheme (e.g., 'Bearer', 'Digest'). |
| `credentials` | `str` | The credentials part of the header value. |

## OAuth2

Utilities for implementing OAuth2 flows.

### OAuth2PasswordBearer

Defines an OAuth2 password bearer flow. It extracts the token from the `Authorization` header.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `tokenUrl` | `str` | The URL of the path operation that provides the token (e.g., `/token`). |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. |
| `scopes` | `Optional[Dict[str, str]]` | A dictionary of available scopes and their descriptions. |
| `description` | `Optional[str]` | A description for the security scheme. |
| `auto_error` | `bool` | If `True` (default), raises an error if the token is missing. If `False`, returns `None`. |
| `refreshUrl` | `Optional[str]` | The URL to refresh the token. |

### OAuth2AuthorizationCodeBearer

Defines an OAuth2 authorization code bearer flow. It extracts the token from the `Authorization` header.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `authorizationUrl` | `str` | The URL for the authorization step. |
| `tokenUrl` | `str` | The URL to obtain the token. |
| `refreshUrl` | `Optional[str]` | The URL to refresh the token. |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. |
| `scopes` | `Optional[Dict[str, str]]` | A dictionary of available scopes and their descriptions. |
| `description` | `Optional[str]` | A description for the security scheme. |
| `auto_error` | `bool` | If `True` (default), raises an error if the token is missing. If `False`, returns `None`. |

### OAuth2PasswordRequestForm

A dependency class that captures OAuth2 password flow form data from a request.

**Attributes**

| Attribute | Type | Description |
|---|---|---|
| `grant_type` | `Optional[str]` | Must be 'password'. Permissive, allows `None`. |
| `username` | `str` | The username from the form data. |
| `password` | `str` | The password from the form data. |
| `scopes` | `List[str]` | A list of scopes requested, parsed from a space-separated string. |
| `client_id` | `Optional[str]` | The client ID, if provided in the form. |
| `client_secret` | `Optional[str]` | The client secret, if provided in the form. |

### OAuth2PasswordRequestFormStrict

A stricter version of `OAuth2PasswordRequestForm` that requires the `grant_type` form field to be present with the value `'password'`, as mandated by the OAuth2 specification.

### SecurityScopes

A special dependency class used to get the security scopes required by other dependencies in the same *path operation*.

**Attributes**

| Attribute | Type | Description |
|---|---|---|
| `scopes` | `List[str]` | A list of all scopes required by the dependencies. |
| `scope_str` | `str` | A single string containing all scopes, separated by spaces. |

## OpenID Connect

### OpenIdConnect

Defines OpenID Connect authentication. It extracts the token from the `Authorization` header.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `openIdConnectUrl` | `str` | The OpenID Connect discovery URL. |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. |
| `description` | `Optional[str]` | A description for the security scheme. |
| `auto_error` | `bool` | If `True` (default), raises an error if the token is missing. If `False`, returns `None`. |