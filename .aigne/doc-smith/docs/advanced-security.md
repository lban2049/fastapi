# Security

Implementing security is a critical aspect of any API. FastAPI provides a set of tools and dependencies that integrate with standard security protocols, making it straightforward to add robust authentication and authorization to your application. This is handled through FastAPI's powerful dependency injection system.

This guide covers several common security schemes, including OAuth2 with JWT tokens, HTTP Basic Authentication, and API Keys.

## Security Flow Overview

A typical authentication flow, like OAuth2 with a bearer token, involves the client first authenticating with credentials to get a token, and then using that token to access protected resources.

```d2
direction: down

"User": { shape: person }
"API Server": {
  shape: package
  grid-columns: 1
  "/token": {label: "Token Endpoint"}
  "/users/me": {label: "Protected Endpoint"}
}

"User" -> "API Server"."/token": "1. Authenticate with credentials" {
  label: "POST /token\n(username, password)"
}
"API Server"."/token" -> "User": "2. Receive Access Token (JWT)"

"User" -> "API Server"."/users/me": "3. Request protected data with token" {
  label: "GET /users/me\n(Authorization: Bearer <token>)"
}
"API Server"."/users/me" -> "User": "4. Receive protected data"
```

## OAuth2 with Password and Bearer Tokens

OAuth2 is a widely used protocol for authorization. The "Password" flow is a common way for a user to directly provide their credentials in exchange for an access token. This token is then sent as a "Bearer" token in the `Authorization` header for subsequent requests.

### First Steps: Creating the Security Scheme

First, you need an instance of `OAuth2PasswordBearer`. This object is a dependency that will require a bearer token in the `Authorization` header.

The `tokenUrl` parameter points to the URL that the client will use to get the token (which we will create later).

```python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

With this, the endpoint `/items/` will require an `Authorization` header with a value like `Bearer your-token-here`. The dependency will return the token as a `str`.

### Get the Current User

Just having the token string isn't enough; you need to verify it and get the corresponding user data. You can create a second dependency, `get_current_user`, that depends on `oauth2_scheme`.

This new dependency will take the token, decode it, and return the user's data.

```python
from typing import Union

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


class User(BaseModel):
    username: str
    email: Union[str, None] = None
    full_name: Union[str, None] = None
    disabled: Union[bool, None] = None


def fake_decode_token(token):
    return User(
        username=token + "fakedecoded", email="john@example.com", full_name="John Doe"
    )


async def get_current_user(token: str = Depends(oauth2_scheme)):
    user = fake_decode_token(token)
    return user


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

Now, the `read_users_me` path operation depends on `get_current_user`. FastAPI will handle calling the dependencies in order: first `oauth2_scheme`, then `get_current_user` with the result. The final result is a `User` object.

### The Token Endpoint

Next, you need to create the `/token` path operation so that clients can send a username and password to get a token.

FastAPI provides `OAuth2PasswordRequestForm` to handle the incoming form data.

```python
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm

# ... (User models and fake_users_db from previous examples)

app = FastAPI()

# ... (get_current_user, etc.)

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    # For now, we are not checking the password, just the username
    # The token is also just the username
    return {"access_token": user.username, "token_type": "bearer"}


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

This endpoint validates the user from a fake database and returns an object containing the `access_token` and `token_type`.

### Using JWT for Tokens

In a real-world application, you should use cryptographically signed tokens like JSON Web Tokens (JWT) instead of simple strings. This ensures that the token data has not been tampered with.

You'll need to install `passlib` for password hashing and `python-jwt` for creating and verifying JWTs.

```bash
pip install "passlib[bcrypt]" python-jwt
```

Here is a more complete example incorporating password hashing and JWT creation:

```python
from datetime import datetime, timedelta, timezone
from typing import Union

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel

# --- Configuration ---
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# --- Models ---
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Union[str, None] = None

# ... (User, UserInDB models)

# --- Hashing & DB ---
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
# ... (fake_users_db with hashed password, verify_password, get_user)

# --- JWT Creation ---
def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# --- Dependency to Get Current User ---
async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except InvalidTokenError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# --- Token Endpoint ---
@app.post("/token", response_model=Token)
async def login_for_access_token(
    form_data: OAuth2PasswordRequestForm = Depends(),
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

# --- Protected Endpoint ---
@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

This implementation correctly authenticates the user, generates a secure JWT, and validates it on protected endpoints.

### OAuth2 Scopes for Authorization

Scopes are used to grant specific permissions to a client. You can define available scopes in `OAuth2PasswordBearer` and then require specific scopes for certain endpoints using the `Security` dependency.

1.  **Define Scopes:**

    ```python
    oauth2_scheme = OAuth2PasswordBearer(
        tokenUrl="token",
        scopes={"me": "Read information about the current user.", "items": "Read items."},
    )
    ```

2.  **Request Scopes in Token Endpoint:** The client can request specific scopes in the `/token` request. You should include these in the JWT.

    ```python
    # In your /token endpoint
    access_token = create_access_token(
        data={"sub": user.username, "scope": " ".join(form_data.scopes)},
        expires_delta=access_token_expires,
    )
    ```

3.  **Check Scopes in Dependency:** The `get_current_user` dependency must be updated to check if the token contains the required scopes for the endpoint being accessed.

    ```python
    from fastapi.security import SecurityScopes

    async def get_current_user(
        security_scopes: SecurityScopes, token: str = Depends(oauth2_scheme)
    ):
        # ... (JWT decoding)
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        scope_str: str = payload.get("scope", "")
        token_scopes = scope_str.split(" ")
        # ... (user fetching)
        for scope in security_scopes.scopes:
            if scope not in token_scopes:
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Not enough permissions",
                    # ...
                )
        return user
    ```

4.  **Require Scopes in Path Operations:** Use `Security` instead of `Depends` to specify required scopes.

    ```python
    from fastapi import Security

    @app.get("/users/me/items/")
    async def read_own_items(
        current_user: User = Security(get_current_active_user, scopes=["items"]),
    ):
        return [{"item_id": "Foo", "owner": current_user.username}]
    ```

## HTTP Basic Authentication

HTTP Basic Auth is a simpler scheme where the username and password are included in the `Authorization` header, Base64-encoded. While simple, it should only be used over HTTPS as the credentials are not encrypted.

FastAPI provides `HTTPBasic` and `HTTPBasicCredentials` for this.

```python
import secrets

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()


def get_current_username(credentials: HTTPBasicCredentials = Depends(security)):
    current_username_bytes = credentials.username.encode("utf8")
    correct_username_bytes = b"stanleyjobson"
    is_correct_username = secrets.compare_digest(
        current_username_bytes, correct_username_bytes
    )
    current_password_bytes = credentials.password.encode("utf8")
    correct_password_bytes = b"swordfish"
    is_correct_password = secrets.compare_digest(
        current_password_bytes, correct_password_bytes
    )
    if not (is_correct_username and is_correct_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Basic"},
        )
    return credentials.username


@app.get("/users/me")
def read_current_user(username: str = Depends(get_current_username)):
    return {"username": username}
```

This example creates a dependency `get_current_username` that receives the credentials, compares them against known values using `secrets.compare_digest` to prevent timing attacks, and raises an exception if they don't match.

## API Key Authentication

API Keys are a common way to grant access to specific clients or services. The key is a single token that the client sends with each request. FastAPI provides helpers to extract API keys from different locations.

<x-cards data-columns="3">
  <x-card data-title="API Key in Query" data-icon="lucide:file-question">
    The API key is passed as a query parameter in the URL.
  </x-card>
  <x-card data-title="API Key in Header" data-icon="lucide:file-terminal">
    The API key is passed in a custom HTTP header.
  </x-card>
  <x-card data-title="API Key in Cookie" data-icon="lucide:cookie">
    The API key is passed in a request cookie.
  </x-card>
</x-cards>

### API Key in a Query

Use `APIKeyQuery` to expect an API key in a query parameter.

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyQuery

app = FastAPI()

api_key_query = APIKeyQuery(name="api-key", auto_error=False)


@app.get("/items/")
async def read_items(api_key: str = Depends(api_key_query)):
    if api_key:
        return {"api_key": api_key}
    else:
        return {"message": "No API Key provided."}
```

### API Key in a Header

Use `APIKeyHeader` to expect an API key in a custom header (e.g., `X-API-Key`).

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyHeader

app = FastAPI()

api_key_header = APIKeyHeader(name="X-API-Key")


@app.get("/items/")
async def read_items(api_key: str = Depends(api_key_header)):
    return {"api_key": api_key}
```

### API Key in a Cookie

Use `APIKeyCookie` to expect an API key in a cookie.

```python
from fastapi import Depends, FastAPI
from fastapi.security import APIKeyCookie

app = FastAPI()

cookie_scheme = APIKeyCookie(name="session")


@app.get("/items/")
async def read_items(session: str = Depends(cookie_scheme)):
    return {"session": session}
```

## Security Policy

Security is taken very seriously. You are encouraged to keep your FastAPI version up-to-date to benefit from the latest features, bug fixes, and security fixes.

### Reporting a Vulnerability

If you believe you have found a security vulnerability, please report it privately by sending an email to **security@tiangolo.com**. Please provide as much detail as possible, including steps to reproduce the issue. Do not discuss potential vulnerabilities publicly until a solution has been found.

---

With these tools, you can implement robust and standard security practices in your FastAPI applications. For more detailed information on the security utilities, you can consult the [API Reference](./api-reference-security.md). To learn about processing requests before they hit your path operations, see the next chapter on [Middleware](./advanced-middleware.md).
