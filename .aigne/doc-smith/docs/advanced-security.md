# Security

Implementing authentication and authorization is a crucial part of most web APIs. FastAPI provides several tools that make it easy to handle security with standard protocols like OAuth2, HTTP Basic/Bearer/Digest Auth, and API Keys.

This guide will walk you through implementing these security schemes with practical, step-by-step examples.

## Overview of Authentication Flow

Most token-based security flows follow a similar pattern: the client authenticates once with credentials (like a username and password) to get a token, and then uses that token for all subsequent requests.

Here's a diagram illustrating a common OAuth2 login and request flow:

```d2
shape: sequence_diagram

Client: Client Application
API: FastAPI Server

Client->API: 1. Request to /token with username & password
API->API: 2. Verify credentials & generate JWT token
API->Client: 3. Return access token

Client->API: 4. Request to /users/me with 'Authorization: Bearer <token>'
API->API: 5. Decode and validate JWT token, identify user
API->Client: 6. Return user data
```

## OAuth2 with Password and Bearer Tokens

OAuth2 is a standard protocol for authorization. The "password flow" is a common way for a user to directly provide their credentials to your application, which then exchanges them for a bearer token.

### First Steps

First, you need an instance of `OAuth2PasswordBearer`. This class is a dependency that provides a `tokenUrl`. This URL is where the client will send the username and password to get a token.

```python
# Source: docs_src/security/tutorial001.py
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

In this example, `Depends(oauth2_scheme)` declares that the `/items/` endpoint depends on the OAuth2 scheme. FastAPI will know that it needs to look for an `Authorization` header with a Bearer token and provide that token to your function as the `token` parameter.

If you open the interactive docs at `/docs`, you will see an "Authorize" button. When you click it, you'll get a popup to enter a username and password. However, we haven't created the `/token` endpoint yet, so this won't work. The key takeaway is that the dependency is already integrated with your API documentation.

### Get the Current User

Just getting the token string isn't very useful. You need to use that token to identify the user making the request. We can create a dependency, `get_current_user`, to handle this.

This dependency will:
1.  Take the token from the `oauth2_scheme` dependency.
2.  Decode the token to get the user information.
3.  Return the user object.

```python
# Source: docs_src/security/tutorial002.py
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

Now, the `read_users_me` path operation depends on `get_current_user`. When a request comes in, FastAPI will call `get_current_user`, which in turn gets the token from the `oauth2_scheme`. The user information is then available in the `current_user` parameter.

### Password Flow and Token Endpoint

Now let's implement the `/token` endpoint. This endpoint will receive the username and password in form data, verify them, and return an access token.

FastAPI provides the `OAuth2PasswordRequestForm` dependency to handle receiving the form data.

We also add a dependency `get_current_active_user` that builds on `get_current_user` to check if the user is active.

```python
# Source: docs_src/security/tutorial003.py
from typing import Union

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

# --- This is a simplified example. For brevity, models and fake DB are not fully shown --- 
# See the full source file for details on fake_users_db, User, UserInDB, etc.

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
}

class User(BaseModel):
    username: str
    email: Union[str, None] = None
    full_name: Union[str, None] = None
    disabled: Union[bool, None] = None

class UserInDB(User):
    hashed_password: str

def fake_hash_password(password: str):
    return "fakehashed" + password

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# ... (get_user and fake_decode_token functions) ...

async def get_current_user(token: str = Depends(oauth2_scheme)):
    # In a real app, this would decode the token and fetch the user
    user_dict = fake_users_db.get(token)
    if not user_dict:
         raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return UserInDB(**user_dict)

async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    return {"access_token": user.username, "token_type": "bearer"}


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

The `/token` endpoint verifies the user and password. If they are correct, it returns a JSON object with `access_token` and `token_type`. In this example, we're just using the username as the token, which is not secure. The next step will fix this.

### Using JWT for Tokens

Instead of returning the username as a token, we should use a standard like JSON Web Tokens (JWT). A JWT contains signed data that can be verified for integrity.

This involves:
1.  **Password Hashing**: Using a library like `passlib` to securely hash and verify passwords.
2.  **JWT Creation**: Creating a JWT with the user's identifier (the `sub` claim) and an expiration time (`exp`).
3.  **JWT Decoding**: In the `get_current_user` dependency, decode the JWT from the `Authorization` header to get the user's identity.

```python
# Source: docs_src/security/tutorial004.py
from datetime import datetime, timedelta, timezone
from typing import Union

import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel

# --- Constants ---
SECRET_KEY = "a-very-secret-key-that-should-be-in-an-env-var"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# --- Pydantic Models ---
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Union[str, None] = None

# ... (User and UserInDB models) ...

# --- Security Utils ---
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
app = FastAPI()

def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

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
    # In a real app, you would get the user from a database
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# --- Path Operations ---
@app.post("/token")
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()) -> Token:
    # In a real app, authenticate_user would check the password against the hashed one in the DB
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

# ... (other endpoints: /users/me/, /users/me/items/) ...
```

Now, the `/token` endpoint returns a proper JWT, and our dependency `get_current_user` can securely validate it and extract the user's information.

### OAuth2 Scopes for Authorization

Authentication identifies the user, but authorization determines what they are allowed to do. OAuth2 uses "scopes" for this.

You can require specific scopes for a path operation using the `Security` dependency.

```python
# Source: docs_src/security/tutorial005.py
from fastapi import Security
from fastapi.security import SecurityScopes

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"me": "Read information about the current user.", "items": "Read items."},
)

async def get_current_user(
    security_scopes: SecurityScopes, token: str = Depends(oauth2_scheme)
):
    # ... (JWT decoding as before) ...
    # Now, check the scopes from the token payload
    token_data = # ... decode token and get scopes
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": f'Bearer scope="{security_scopes.scope_str}"'},
            )
    return user

async def get_current_active_user(
    current_user: User = Security(get_current_user, scopes=["me"]),
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: User = Security(get_current_active_user, scopes=["items"]),
):
    return [{"item_id": "Foo", "owner": current_user.username}]
```

In this example:
-   `OAuth2PasswordBearer` is initialized with a dictionary of available scopes.
-   The `get_current_user` dependency now accepts a `SecurityScopes` parameter and checks if the token contains the required scopes.
-   The path operation `/users/me/items/` requires the `items` scope by using `Security(get_current_active_user, scopes=["items"])`.

## HTTP Basic Authentication

HTTP Basic Auth is a simpler scheme where the username and password are included directly in the `Authorization` header, Base64-encoded. While less complex than OAuth2, it's generally less secure as credentials are sent with every request.

FastAPI provides the `HTTPBasic` security scheme.

```python
# Source: docs_src/security/tutorial007.py
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

Here, the `get_current_username` dependency uses `HTTPBasic` to extract credentials. It's important to use `secrets.compare_digest` to compare credentials to prevent timing attacks.

## API Keys

API keys are a common way to authenticate server-to-server requests or identify projects. FastAPI supports receiving API keys from headers, query parameters, or cookies.

<x-cards data-columns="3">
  <x-card data-title="API Key in Header" data-icon="lucide:arrow-right-from-line">
    Use the APIKeyHeader class to extract a key from a custom request header. This is a common and recommended method.
  </x-card>
  <x-card data-title="API Key in Query" data-icon="lucide:at-sign">
    Use the APIKeyQuery class to extract a key from a URL query parameter. This is useful for simple scripts or browser testing.
  </x-card>
  <x-card data-title="API Key in Cookie" data-icon="lucide:cookie">
    Use the APIKeyCookie class to extract a key from a browser cookie. This is often used for managing web sessions.
  </x-card>
</x-cards>

Each of these classes acts as a dependency that you can use to protect your endpoints. You would create a dependency function that validates the received key against a database or a list of secrets, like in this example using `APIKeyHeader`:

```python
from fastapi import Depends, FastAPI, HTTPException, Security
from fastapi.security import APIKeyHeader

API_KEY = "my-super-secret-api-key"
API_KEY_NAME = "X-API-KEY"

api_key_header_scheme = APIKeyHeader(name=API_KEY_NAME)

app = FastAPI()

async def get_api_key(api_key_header: str = Security(api_key_header_scheme)):
    if api_key_header == API_KEY:
        return api_key_header
    else:
        raise HTTPException(
            status_code=403,
            detail="Could not validate credentials"
        )

@app.get("/items/")
async def read_items(api_key: str = Depends(get_api_key)):
    return [{"item": "Foo"}, {"item": "Bar"}]
```

---

This guide has covered the main security utilities available in FastAPI. You can now implement robust authentication and authorization for your applications. For more details on the specific classes and their parameters, please see the [Security Utilities API Reference](./api-reference-security.md).

Next, you might want to learn about [Middleware](./advanced-middleware.md) to perform actions on every incoming request.