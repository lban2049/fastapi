# Security Utilities

FastAPI provides a collection of dependency-callable classes to implement various security schemes in your API. These utilities handle the extraction of credentials (like tokens, API keys, or basic auth headers) from the request and integrate with the OpenAPI documentation.

This reference covers the main security utilities available in `fastapi.security`.

---

## API Keys

API key authentication is a common pattern where a secret key is passed in the request. FastAPI provides utilities to extract keys from query parameters, headers, or cookies.

### `APIKeyQuery`

Extracts an API key from a query parameter.

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

query_scheme = APIKeyQuery(name="api_key")


@app.get("/items/")
async def read_items(api_key: str = Depends(query_scheme)):
    return {"api_key": api_key}
```

**Parameters**

| Name | Type | Description |
|---|---|---|
| `name` | `str` | **Required.** The name of the query parameter containing the API key. |
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI documentation. Defaults to the class name. |
| `description` | `Optional[str]` | A description for the security scheme, visible in the OpenAPI documentation. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the key is not found. If `False`, the dependency returns `None`. |

### `APIKeyHeader`

Extracts an API key from a request header.

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

header_scheme = APIKeyHeader(name="x-key")


@app.get("/items/")
async def read_items(key: str = Depends(header_scheme)):
    return {"key": key}
```

**Parameters**

| Name | Type | Description |
|---|---|---|
| `name` | `str` | **Required.** The name of the header containing the API key. |
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI documentation. Defaults to the class name. |
| `description` | `Optional[str]` | A description for the security scheme, visible in the OpenAPI documentation. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the header is missing. If `False`, the dependency returns `None`. |

### `APIKeyCookie`

Extracts an API key from a request cookie.

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

**Parameters**

| Name | Type | Description |
|---|---|---|
| `name` | `str` | **Required.** The name of the cookie containing the API key. |
| `scheme_name` | `Optional[str]` | The security scheme name, visible in the OpenAPI documentation. Defaults to the class name. |
| `description` | `Optional[str]` | A description for the security scheme, visible in the OpenAPI documentation. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the cookie is not found. If `False`, the dependency returns `None`. |

---

## HTTP Authentication

These utilities implement standard HTTP authentication schemes defined in RFC documents, such as Basic, Bearer, and Digest.

### `HTTPBasic`

Implements HTTP Basic authentication. It extracts the `Authorization` header, decodes the Base64 credentials, and returns an `HTTPBasicCredentials` object.

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

**Parameters**

| Name | Type | Description |
|---|---|---|
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. Defaults to the class name. |
| `realm` | `Optional[str]` | The HTTP Basic authentication realm, included in the `WWW-Authenticate` header. |
| `description` | `Optional[str]` | A description for the security scheme in OpenAPI. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the header is invalid or missing. If `False`, the dependency returns `None`. |

### `HTTPBearer`

Implements HTTP Bearer token authentication. It verifies the `Authorization` header starts with "Bearer " and returns an `HTTPAuthorizationCredentials` object.

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

**Parameters**

| Name | Type | Description |
|---|---|---|
| `bearerFormat` | `Optional[str]` | The expected format of the bearer token (e.g., "JWT"), used for OpenAPI documentation. |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. Defaults to the class name. |
| `description` | `Optional[str]` | A description for the security scheme in OpenAPI. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the header is invalid or missing. If `False`, the dependency returns `None`. |

### `HTTPDigest`

Implements HTTP Digest authentication. It verifies the `Authorization` header starts with "Digest " and returns an `HTTPAuthorizationCredentials` object.

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

**Parameters**

| Name | Type | Description |
|---|---|---|
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. Defaults to the class name. |
| `description` | `Optional[str]` | A description for the security scheme in OpenAPI. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the header is invalid or missing. If `False`, the dependency returns `None`. |

### Credentials Models

- **`HTTPBasicCredentials`**: The result of using `HTTPBasic`. It has two attributes:
  - `username` (str): The provided username.
  - `password` (str): The provided password.

- **`HTTPAuthorizationCredentials`**: The result of `HTTPBearer` or `HTTPDigest`. It has two attributes:
  - `scheme` (str): The authentication scheme (e.g., "Bearer").
  - `credentials` (str): The credential string (e.g., the token).

---

## OAuth2

FastAPI provides comprehensive tools for implementing OAuth2 flows.

### `OAuth2PasswordBearer`

A dependency class for the OAuth2 Password Bearer flow. It checks for a valid `Authorization: Bearer <token>` header and returns the token as a string.

**Parameters**

| Name | Type | Description |
|---|---|---|
| `tokenUrl` | `str` | **Required.** The URL of the endpoint that issues the token (e.g., `/token`). |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. |
| `scopes` | `Optional[Dict[str, str]]` | A dictionary of available scopes and their descriptions for OpenAPI. |
| `description` | `Optional[str]` | A description for the security scheme in OpenAPI. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the token is invalid or missing. If `False`, it returns `None`. |
| `refreshUrl` | `Optional[str]` | The URL to refresh an expired token. |

### `OAuth2AuthorizationCodeBearer`

A dependency class for the OAuth2 Authorization Code flow. It also expects an `Authorization: Bearer <token>` header.

**Parameters**

| Name | Type | Description |
|---|---|---|
| `authorizationUrl` | `str` | **Required.** The URL for the authorization endpoint. |
| `tokenUrl` | `str` | **Required.** The URL for the token exchange endpoint. |
| `refreshUrl` | `Optional[str]` | The URL to refresh an expired token. |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. |
| `scopes` | `Optional[Dict[str, str]]` | A dictionary of available scopes and their descriptions for OpenAPI. |
| `description` | `Optional[str]` | A description for the security scheme in OpenAPI. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the token is invalid or missing. If `False`, it returns `None`. |

### `OAuth2PasswordRequestForm` and `OAuth2PasswordRequestFormStrict`

These are dependency classes used in a token-issuing endpoint to receive credentials as form data (`application/x-www-form-urlencoded`).

- `OAuth2PasswordRequestForm`: `grant_type` is optional.
- `OAuth2PasswordRequestFormStrict`: `grant_type` is required to be `"password"`, as per the OAuth2 spec.

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordRequestForm

app = FastAPI()


@app.post("/login")
def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()])
    # In a real app, you would verify form_data.username and form_data.password
    # and then create and return a token.
    return {"access_token": form_data.username, "token_type": "bearer"}
```

The dependency instance will have the following attributes extracted from the form data:
- `grant_type`: The grant type (e.g., "password").
- `username`: The user's name.
- `password`: The user's password.
- `scopes`: A `list[str]` of requested scopes.
- `client_id`: The client ID, if provided.
- `client_secret`: The client secret, if provided.

### `SecurityScopes`

A special dependency used to access the list of scopes required by other security dependencies in the same path operation.

```python
from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token", scopes={"me": "Read information about the current user."}) 

@app.get("/users/me")
async def read_users_me(
    security_scopes: SecurityScopes, token: str = Security(oauth2_scheme, scopes=["me"])
):
    return {"scopes": security_scopes.scopes, "token": token}
```

It provides two attributes:
- `scopes` (`List[str]`): The list of required scopes.
- `scope_str` (`str`): A single string with all scopes separated by spaces.

---

## OpenID Connect

### `OpenIdConnect`

Implements authentication based on an OpenID Connect URL. It extracts the `Authorization` header value.

**Parameters**

| Name | Type | Description |
|---|---|---|
| `openIdConnectUrl` | `str` | **Required.** The discovery URL for the OpenID Connect provider. |
| `scheme_name` | `Optional[str]` | The security scheme name for OpenAPI. |
| `description` | `Optional[str]` | A description for the security scheme in OpenAPI. |
| `auto_error` | `bool` | If `True` (default), an error is raised if the `Authorization` header is missing. If `False`, it returns `None`. |
