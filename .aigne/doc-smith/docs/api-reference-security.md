# Security

FastAPI provides a collection of tools to handle security and authentication within your API. These utilities are designed to be used as dependencies in your *path operations*, integrating seamlessly with the dependency injection system. They handle extracting credentials from the request and can automatically return the appropriate HTTP errors when credentials are missing or invalid.

For a step-by-step guide on implementing security, please refer to the [Dependencies and Security Tutorial](./tutorials-dependencies-and-security.md).

## API Key Authentication

API key authentication is a common method for securing endpoints. FastAPI provides classes to extract API keys from different parts of the request: query parameters, headers, or cookies.

### APIKeyQuery

`APIKeyQuery` is a dependency class for handling API key authentication via a query parameter.

**Parameters**

<x-field data-name="name" data-type="string" data-required="true" data-desc="The name of the query parameter that holds the API key."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in the OpenAPI documentation."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme, visible in the OpenAPI documentation."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically sends an HTTP 403 error if the key is missing. If false, the dependency returns `None`."></x-field>

**Example**

```python APIKeyQuery Usage icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

query_scheme = APIKeyQuery(name="api_key")


@app.get("/items/")
async def read_items(api_key: str = Depends(query_scheme)):
    return {"api_key": api_key}
```

### APIKeyHeader

`APIKeyHeader` is a dependency class for handling API key authentication via a request header.

**Parameters**

<x-field data-name="name" data-type="string" data-required="true" data-desc="The name of the header that holds the API key (e.g., 'X-API-Key')."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in the OpenAPI documentation."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme, visible in the OpenAPI documentation."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically sends an HTTP 403 error if the key is missing. If false, the dependency returns `None`."></x-field>

**Example**

```python APIKeyHeader Usage icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

header_scheme = APIKeyHeader(name="x-key")


@app.get("/items/")
async def read_items(key: str = Depends(header_scheme)):
    return {"key": key}
```

### APIKeyCookie

`APIKeyCookie` is a dependency class for handling API key authentication via a request cookie.

**Parameters**

<x-field data-name="name" data-type="string" data-required="true" data-desc="The name of the cookie that holds the API key."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in the OpenAPI documentation."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme, visible in the OpenAPI documentation."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically sends an HTTP 403 error if the key is missing. If false, the dependency returns `None`."></x-field>

**Example**

```python APIKeyCookie Usage icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

## HTTP Authentication

FastAPI supports standard HTTP authentication schemes like Basic, Bearer, and Digest.

### HTTPBasic

`HTTPBasic` implements HTTP Basic authentication. It extracts the username and password from the `Authorization` header.

The dependency result is an `HTTPBasicCredentials` object.

**Parameters**

<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in the OpenAPI documentation."></x-field>
<x-field data-name="realm" data-type="string" data-required="false" data-desc="The HTTP Basic authentication realm, included in the 'WWW-Authenticate' header of a 401 response."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme, visible in the OpenAPI documentation."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically sends an HTTP 401 error if credentials are not provided. If false, returns `None`."></x-field>

**Example**

```python HTTPBasic Usage icon=logos:python
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

`HTTPBearer` implements HTTP Bearer token authentication, commonly used with OAuth2.

The dependency result is an `HTTPAuthorizationCredentials` object.

**Parameters**

<x-field data-name="bearerFormat" data-type="string" data-required="false" data-desc="An optional hint to clients about the format of the bearer token (e.g., 'JWT'). Used in OpenAPI documentation."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in the OpenAPI documentation."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme, visible in the OpenAPI documentation."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically sends an error if the token is not provided. If false, returns `None`."></x-field>

**Example**

```python HTTPBearer Usage icon=logos:python
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

`HTTPDigest` implements HTTP Digest authentication.

The dependency result is an `HTTPAuthorizationCredentials` object.

**Parameters**

<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in the OpenAPI documentation."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme, visible in the OpenAPI documentation."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically sends an error if the digest is not provided. If false, returns `None`."></x-field>

**Example**

```python HTTPDigest Usage icon=logos:python
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

### Helper Models

#### HTTPBasicCredentials

A data model containing the decoded `username` and `password` from HTTP Basic auth.

<x-field data-name="username" data-type="string" data-required="true" data-desc="The HTTP Basic username."></x-field>
<x-field data-name="password" data-type="string" data-required="true" data-desc="The HTTP Basic password."></x-field>

#### HTTPAuthorizationCredentials

A data model containing the `scheme` and `credentials` from the `Authorization` header.

<x-field data-name="scheme" data-type="string" data-required="true" data-desc="The authentication scheme (e.g., 'Bearer', 'Digest')."></x-field>
<x-field data-name="credentials" data-type="string" data-required="true" data-desc="The credentials string, such as the token."></x-field>

## OAuth2

FastAPI provides comprehensive support for implementing OAuth2 flows.

### OAuth2PasswordBearer

This is a dependency class that implements the OAuth2 "Password" flow with a Bearer token. It checks for a valid `Authorization: Bearer <token>` header and returns the token as a string.

**Parameters**

<x-field data-name="tokenUrl" data-type="string" data-required="true" data-desc="The URL of the endpoint that issues the token (e.g., '/token')."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in OpenAPI."></x-field>
<x-field data-name="scopes" data-type="Dict[str, str]" data-required="false" data-desc="A dictionary of scope names to descriptions, used in OpenAPI."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically raises an error if the 'Authorization' header is missing or invalid."></x-field>
<x-field data-name="refreshUrl" data-type="string" data-required="false" data-desc="The URL to refresh the token and obtain a new one."></x-field>

### OAuth2AuthorizationCodeBearer

This dependency class implements the OAuth2 "Authorization Code" flow with a Bearer token.

**Parameters**

<x-field data-name="authorizationUrl" data-type="string" data-required="true" data-desc="The URL for the authorization endpoint."></x-field>
<x-field data-name="tokenUrl" data-type="string" data-required="true" data-desc="The URL of the endpoint that issues the token."></x-field>
<x-field data-name="refreshUrl" data-type="string" data-required="false" data-desc="The URL to refresh the token and obtain a new one."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme, used in OpenAPI."></x-field>
<x-field data-name="scopes" data-type="Dict[str, str]" data-required="false" data-desc="A dictionary of scope names to descriptions, used in OpenAPI."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically raises an error if the 'Authorization' header is missing or invalid."></x-field>

### OAuth2PasswordRequestForm & OAuth2PasswordRequestFormStrict

These are dependency classes that parse OAuth2 password flow request bodies sent as form data. They extract `username`, `password`, `scope`, and other fields. `OAuth2PasswordRequestFormStrict` additionally requires the `grant_type` field to be present with the value `"password"`.

**Form Fields**

<x-field data-name="grant_type" data-type="str" data-required="false" data-desc="Required by `OAuth2PasswordRequestFormStrict`. Must be 'password'. Optional for `OAuth2PasswordRequestForm`."></x-field>
<x-field data-name="username" data-type="str" data-required="true" data-desc="The user's username."></x-field>
<x-field data-name="password" data-type="str" data-required="true" data-desc="The user's password."></x-field>
<x-field data-name="scope" data-type="str" data-default="" data-required="false" data-desc="A space-separated string of scopes."></x-field>
<x-field data-name="client_id" data-type="str | None" data-required="false" data-desc="The client ID."></x-field>
<x-field data-name="client_secret" data-type="str | None" data-required="false" data-desc="The client secret."></x-field>

**Example**

```python OAuth2PasswordRequestForm Usage icon=logos:python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordRequestForm

app = FastAPI()


@app.post("/login")
def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()])
    # The form_data object will have attributes like username, password, scopes, etc.
    return {"username": form_data.username, "scopes": form_data.scopes}
```

### SecurityScopes

A special dependency class used to access the list of security scopes required by other dependencies in the same *path operation*.

**Attributes**

<x-field data-name="scopes" data-type="List[str]" data-desc="A list of all scopes required by dependencies."></x-field>
<x-field data-name="scope_str" data-type="str" data-desc="A single string containing all scopes, separated by spaces."></x-field>

## OpenID Connect

### OpenIdConnect

`OpenIdConnect` is a dependency class for handling OpenID Connect authentication. It primarily serves to document the security scheme in OpenAPI.

**Parameters**

<x-field data-name="openIdConnectUrl" data-type="string" data-required="true" data-desc="The OpenID Connect discovery URL."></x-field>
<x-field data-name="scheme_name" data-type="string" data-required="false" data-desc="An optional name for the security scheme."></x-field>
<x-field data-name="description" data-type="string" data-required="false" data-desc="An optional description for the security scheme."></x-field>
<x-field data-name="auto_error" data-type="boolean" data-default="true" data-required="false" data-desc="If true, automatically raises an error if the 'Authorization' header is missing."></x-field>