# Dependencies and Security

FastAPI includes a powerful but intuitive Dependency Injection system. It's a way to declare things that your application needs to run—like database sessions, authentication credentials, or configuration settings—and have FastAPI take care of providing them to your *path operations*.

This system is not only great for organizing your code and sharing logic, but it's also the foundation for implementing robust authentication and authorization. In this guide, we'll explore how to create dependencies and then use them to secure your API endpoints.

## The Dependency Injection System

At its core, a "dependency" is typically a function (or any other callable) that FastAPI will call before your *path operation function*. The return value of the dependency function is then passed as an argument to your *path operation function*.

### Create a Shared Dependency

Let's start with a simple example. Imagine you have common query parameters like `q`, `skip`, and `limit` that are used in multiple endpoints. Instead of repeating them in every function signature, you can define them once in a dependency.

```python Create a dependency function icon=logos:python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

In this code:
1.  We define a function `common_parameters` that takes the query parameters we want.
2.  In our *path operation functions* (`read_items` and `read_users`), we declare a parameter `commons`.
3.  We set its default value to `Depends(common_parameters)`. 

FastAPI sees `Depends()` and knows it must call `common_parameters` first. It will then take the dictionary returned by `common_parameters` and pass it as the `commons` argument to `read_items` or `read_users`. This keeps your path operation logic clean and avoids code duplication.

### Classes as Dependencies

You can also use a class as a dependency. FastAPI will treat the class itself as a "callable" and create an instance of it.

This is useful for grouping related dependencies or when the dependency needs to maintain some state.

```python Use a class as a dependency icon=logos:python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

Here, FastAPI will create an instance of `CommonQueryParams`, passing the query parameters `q`, `skip`, and `limit` to the `__init__` method. That instance is then passed as the `commons` argument to `read_items`.

A common shortcut is to just use `Depends()` without passing the callable. FastAPI is smart enough to use the type hint (`CommonQueryParams`) as the dependency.

```python Shortcut for class dependencies icon=logos:python
@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
    # ... same logic as before
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

## Authentication and Authorization

Now, let's use the dependency injection system to secure our API. We'll cover two common methods: HTTP Basic Auth and OAuth2 with Bearer Tokens.

### HTTP Basic Authentication

HTTP Basic Auth is a simple authentication scheme built into the HTTP protocol. The client sends a username and password in the `Authorization` header.

FastAPI provides security utilities to make this easy.

```python HTTP Basic Auth Example icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

security = HTTPBasic()


@app.get("/users/me")
def read_current_user(credentials: HTTPBasicCredentials = Depends(security)):
    return {"username": credentials.username, "password": credentials.password}
```

Here, `HTTPBasic()` is a callable class that extracts the username and password from the request. By using `Depends(security)`, we tell FastAPI to run this check. If the `Authorization` header is missing or malformed, FastAPI will automatically return a 401 Unauthorized error.

### OAuth2 with Password and Bearer Tokens

OAuth2 is a more robust and flexible framework for authorization. A common flow is the "Password Flow," where a user exchanges a username and password for a temporary access token. This token is then sent with future requests as a "Bearer Token."

Here's a diagram of the JWT authentication flow:

```d2 JWT Authentication Flow
direction: down
shape: sequence_diagram

User -> FastAPI-App: "1. POST /token with username & password"

FastAPI-App -> DB: "2. Authenticate user"
DB -> FastAPI-App: "3. User record"

FastAPI-App -> FastAPI-App: "4. Create JWT"

FastAPI-App -> User: "5. Return JWT Access Token"

User -> Protected-Endpoint: "6. Request with 'Authorization: Bearer <token>'"

Protected-Endpoint -> Protected-Endpoint: "7. Dependency validates JWT"

Protected-Endpoint -> DB: "8. Fetch current user from DB"

Protected-Endpoint -> User: "9. Return response"

```

Let's build this step by step.

#### Step 1: The Basic Security Scheme

First, we define our security scheme using `OAuth2PasswordBearer`. We must tell it the URL where the client can go to get a token.

```python Basic OAuth2 Setup icon=logos:python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

This simple dependency, `Depends(oauth2_scheme)`, checks for an `Authorization` header, confirms it contains `Bearer` plus a token, and returns the token as a `str`. If not, it raises a 401 error.

#### Step 2: Getting the Current User

Just having the token string isn't very useful. We need a dependency that decodes the token and fetches the corresponding user.

```python Get Current User Dependency icon=logos:python
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

Now, our path operation depends on `get_current_user`. This new dependency in turn depends on `oauth2_scheme`. FastAPI handles this chain automatically. The endpoint gets a full, validated `User` object.

#### Step 3: Real JWTs and Password Hashing

So far, we've used fake token handling. Let's implement the real thing using JSON Web Tokens (JWT) and proper password hashing with `passlib`.

This example is more complete, including a `/token` endpoint to issue tokens.

```python Full JWT Authentication icon=logos:python
from datetime import datetime, timedelta, timezone
from typing import Union

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel

# Configuration
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Password Hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Security Scheme
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()

# --- Pydantic Models --- (User, Token, TokenData, etc.)

# --- Database and Utility Functions --- (fake_users_db, get_user, verify_password, etc.)

# --- Dependency to get current user ---
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

async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# --- Token Endpoint ---
@app.post("/token")
async def login_for_access_token(
    form_data: OAuth2PasswordRequestForm = Depends(),
) -> Token:
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
    return Token(access_token=access_token, token_type="bearer")

# --- Protected Endpoints ---
@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

This is a complete, secure implementation. The `/token` endpoint takes form data (`username`, `password`), verifies it, and returns a signed JWT. The `get_current_user` dependency validates the JWT signature and expiration time before allowing access to protected endpoints.

#### Step 4: Authorization with Scopes

Authentication confirms *who* the user is. Authorization determines *what* they are allowed to do. OAuth2 uses "scopes" for this.

We can define available scopes in our `OAuth2PasswordBearer` scheme and then require specific scopes for certain endpoints using `Security()`.

```python OAuth2 with Scopes icon=logos:python
from fastapi import Depends, FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes
# ... other imports and setup from the previous step

# Update the security scheme with scopes
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"me": "Read information about the current user.", "items": "Read items."},
)

# Update the user dependency to check scopes
async def get_current_user(
    security_scopes: SecurityScopes, token: str = Depends(oauth2_scheme)
):
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
    # ... (JWT decoding logic as before) ...
    # After getting the user and token scopes:
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": authenticate_value},
            )
    return user

# A dependency for an active user (no specific scopes required yet)
async def get_current_active_user(
    current_user: User = Security(get_current_user, scopes=["me"]),
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


# Require the "items" scope for this endpoint
@app.get("/users/me/items/")
async def read_own_items(
    current_user: User = Security(get_current_active_user, scopes=["items"]),
):
    return [{"item_id": "Foo", "owner": current_user.username}]


# This endpoint has no specific scope requirements
@app.get("/status/")
async def read_system_status(current_user: User = Depends(get_current_user)):
    return {"status": "ok"}
```

Key changes:
*   `SecurityScopes` is added as a parameter to the dependency. FastAPI populates this with the scopes required by the endpoint.
*   We use `Security()` instead of `Depends()`. It works the same way but allows us to pass a `scopes` list.
*   The dependency checks if the scopes present in the JWT are sufficient to satisfy the scopes required by the endpoint.

## Summary

FastAPI's Dependency Injection system is a versatile tool for separating concerns, reusing logic, and, most importantly, building secure and robust APIs. By defining dependencies for common parameters, database connections, and user authentication, you can keep your *path operation* logic clean, focused, and easy to test.

You've now learned how to create dependencies and use them to implement everything from simple shared parameters to a full OAuth2 security flow with scopes.

[Next: Advanced User Guide](./tutorials-advanced.md)
