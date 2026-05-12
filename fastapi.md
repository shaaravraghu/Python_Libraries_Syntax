# FastAPI: The Exhaustive Developer's Reference

## Contents
- [1. Introduction & History](#1-introduction--history)
- [2. Installation & Setup](#2-installation--setup)
- [3. Path Operations](#3-path-operations)
- [4. Request and Response Models](#4-request-and-response-models)
- [5. Dependency Injection](#5-dependency-injection)
- [6. Query, Path, and Body Parameters](#6-query-path-and-body-parameters)
- [7. Validation](#7-validation)
- [8. Error Handling](#8-error-handling)
- [9. Security](#9-security)
- [10. Async and Concurrency](#10-async-and-concurrency)
- [11. Database Integration](#11-database-integration)
- [12. Background Tasks](#12-background-tasks)
- [13. Middleware](#13-middleware)
- [14. WebSockets](#14-websockets)
- [15. Testing](#15-testing)
- [16. Deployment](#16-deployment)
- [17. OpenAPI and Docs](#17-openapi-and-docs)
- [18. Advanced Patterns](#18-advanced-patterns)
- [19. Performance Optimization](#19-performance-optimization)
- [20. Advanced Topics](#20-advanced-topics)

## 1. INTRODUCTION & HISTORY

### What is FastAPI
FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.7+ based on standard Python type hints. Built by Sebastián Ramírez (tiangolo), it was first released in late 2018. It stands on the shoulders of giants: Starlette for the web parts and Pydantic for the data parts.

### Why FastAPI
- **High Performance**: On par with NodeJS and Go (thanks to Starlette and Pydantic). One of the fastest Python frameworks available.
- **Fast to Code**: Increases the speed to develop features by about 200% to 300%.
- **Fewer Bugs**: Reduces about 40% of human (developer) induced errors.
- **Intuitive**: Great editor support. Auto-completion everywhere. Less time debugging.
- **Easy**: Designed to be easy to use and learn. Less time reading docs.
- **Short**: Minimize code duplication. Multiple features from each parameter declaration.
- **Robust**: Get production-ready code. With automatic interactive documentation (Swagger UI and ReDoc).
- **Standards-based**: Based on (and fully compatible with) the open standards for APIs: OpenAPI and JSON Schema.

### Comparison with Flask and Django
| Feature | FastAPI | Flask | Django |
|---------|---------|-------|--------|
| **Type** | Micro-framework | Micro-framework | Full-stack framework |
| **Speed** | Extremely fast (ASGI) | Fast (WSGI, async added later) | Slower (WSGI/ASGI) |
| **Validation** | Built-in (Pydantic) | Extensions (Marshmallow) | Built-in (Forms/DRF) |
| **Docs** | Auto-generated (Swagger/ReDoc) | Extensions (Flasgger) | Extensions (drf-yasg/Spectacular) |
| **Async** | Native | Added in 2.0 | Added in 3.1+ |
| **ORM** | None included | None included | Built-in |

### ASGI vs WSGI
- **WSGI (Web Server Gateway Interface)**: The older Python standard for web applications. It handles requests synchronously. Each request blocks the worker until it finishes.
- **ASGI (Asynchronous Server Gateway Interface)**: The spiritual successor to WSGI, designed to provide a standard interface between async-capable Python web servers, frameworks, and applications. FastAPI is purely ASGI.

### Starlette and Pydantic Overview
- **Starlette**: A lightweight ASGI framework/toolkit, which is ideal for building async web services in Python. FastAPI inherits its routing, middleware, and websocket capabilities from Starlette.
- **Pydantic**: Data validation and settings management using Python type annotations. FastAPI uses it to validate requests, format responses, and generate OpenAPI schemas.

> 💡 **Pro Tip**: Always use the latest version of FastAPI to take advantage of Pydantic v2 performance improvements, which are written in Rust and offer up to a 5x-50x speedup in validation.

> ⚠️ **Common Pitfall**: Trying to use WSGI tools (like synchronous database drivers) in FastAPI without proper `def` vs `async def` handling can severely bottleneck your ASGI event loop.

---

## 2. INSTALLATION & PROJECT SETUP

### Installation
```bash
# Install fastapi and uvicorn (an ASGI server)
pip install fastapi "uvicorn[standard]"
```

### Virtual Environments
Always use a virtual environment for Python projects:
```bash
# Create the environment
python -m venv venv

# Activate it (Windows)
.\venv\Scripts\activate

# Activate it (Unix/MacOS)
source venv/bin/activate
```

### Running Server with Uvicorn
Create a simple `main.py`:
```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    # Return a simple dictionary which is automatically converted to JSON
    return {"Hello": "World"}
```

Run it using uvicorn:
```bash
# uvicorn filename:app_variable --reload
uvicorn main:app --reload
```
- `--reload`: makes the server restart after code changes (development only!).

### Project Structure (Modular vs Single File)
For larger projects, a structured approach is essential:
```text
my_fastapi_project/
├── alembic/              # Database migrations
├── app/                  # Main application package
│   ├── __init__.py
│   ├── main.py           # FastAPI application instance
│   ├── core/             # Configuration, security, etc.
│   │   ├── config.py
│   │   └── security.py
│   ├── models/           # SQLAlchemy database models
│   ├── schemas/          # Pydantic schemas (validation)
│   ├── api/              # Routers and endpoints
│   │   ├── dependencies.py
│   │   └── endpoints/
│   ├── crud/             # Create, Read, Update, Delete logic
│   └── tests/            # Pytest tests
├── .env                  # Environment variables
├── requirements.txt      # Dependencies
└── alembic.ini           # Alembic configuration
```

### Environment Variables (.env, python-dotenv)
Use `pydantic-settings` to manage configuration securely.

```bash
pip install pydantic-settings
```

```python
# app/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My Awesome API"
    admin_email: str
    database_url: str

    class Config:
        env_file = ".env"

# Instantiate settings
settings = Settings()
```

### Dependency Management (requirements.txt / Poetry)
- **requirements.txt**: The classic way. Run `pip freeze > requirements.txt` and `pip install -r requirements.txt`.
- **Poetry**: A modern dependency manager.
  ```bash
  poetry init
  poetry add fastapi "uvicorn[standard]"
  poetry run uvicorn main:app --reload
  ```

> 💡 **Pro Tip**: Use `poetry` or `uv` for modern dependency management. They ensure reproducible builds with lockfiles.

> ⚠️ **Common Pitfall**: Forgetting to add `.env` to `.gitignore`. Never commit secrets to version control!

---

## 3. ROUTING

### Path Operations
FastAPI supports all HTTP methods via decorators on the `app` instance.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items():
    # Handles GET requests
    return [{"name": "Item 1"}, {"name": "Item 2"}]

@app.post("/items/")
async def create_item():
    # Handles POST requests
    return {"message": "Item created"}

@app.put("/items/{item_id}")
async def update_item(item_id: int):
    # Handles PUT requests
    return {"message": f"Item {item_id} updated"}

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    # Handles DELETE requests
    return {"message": f"Item {item_id} deleted"}

@app.patch("/items/{item_id}")
async def patch_item(item_id: int):
    # Handles PATCH requests
    return {"message": f"Item {item_id} patched"}
```

### Path Parameters
Path parameters are defined in the route URL and passed as arguments to the function.

```python
@app.get("/users/{user_id}")
async def read_user(user_id: int): # Type hint ensures 'user_id' is an integer
    return {"user_id": user_id}
```

### Query Parameters
Function parameters that are not in the path are automatically interpreted as query parameters.

```python
@app.get("/search/")
async def search(q: str | None = None, skip: int = 0, limit: int = 10):
    # Usage: /search/?q=fastapi&skip=5&limit=20
    return {"query": q, "skip": skip, "limit": limit}
```

### Request Body
Declare a Pydantic model to parse and validate request bodies automatically.

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
async def create_item(item: Item): # FastAPI reads the body as JSON and maps it to 'Item'
    # 'item' is fully validated and has type hints
    total_price = item.price + (item.tax or 0.0)
    return {"item": item, "total_price": total_price}
```

### Status Codes
Set the default response status code using the `status_code` parameter in the decorator.

```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

### Tags and Metadata
Tags group routes together in the Swagger UI.

```python
@app.get("/users/", tags=["users"], summary="Get all users")
async def get_users():
    """
    Retrieve all users from the database.
    This docstring becomes the endpoint description in OpenAPI.
    """
    return ["user1", "user2"]
```

### APIRouter and Modular Routing
For larger apps, split your routes across multiple files using `APIRouter`.

```python
# app/api/endpoints/users.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/")
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]

# app/main.py
from fastapi import FastAPI
from app.api.endpoints import users

app = FastAPI()

# Mount the router under the '/users' prefix
app.include_router(users.router, prefix="/users", tags=["users"])
```

> 💡 **Pro Tip**: Use `status` module constants (e.g., `status.HTTP_404_NOT_FOUND`) instead of integer literals for readability.

> ⚠️ **Common Pitfall**: Order of routes matters. If you have `/users/{user_id}` before `/users/me`, the `{user_id}` route will catch "me" and try to parse it, causing a validation error if it expects an integer. Always put static paths before dynamic ones.

---

## 4. REQUEST HANDLING

### Request Object
Sometimes you need direct access to the underlying Starlette `Request` object (e.g., for raw client IP or unparsed bodies).

```python
from fastapi import Request

@app.get("/client-ip")
async def get_client_ip(request: Request):
    # Access raw request properties directly
    client_host = request.client.host
    return {"client_ip": client_host}
```

### Body Parsing
You can mix path, query, and body parameters in a single endpoint. FastAPI automatically figures out where each comes from.

```python
from fastapi import Path, Query, Body

@app.put("/items/{item_id}")
async def update_item(
    item_id: int = Path(..., title="The ID of the item to get", ge=1), # Path param with validation
    q: str | None = Query(None, alias="item-query"), # Query param with an alias
    item: Item = Body(..., embed=True) # Body param embedded under an 'item' key
):
    return {"item_id": item_id, "q": q, "item": item}
```

### Form Data
To receive form fields instead of JSON, use `Form`.
*(Note: Requires `python-multipart`)*
```bash
pip install python-multipart
```

```python
from fastapi import Form

@app.post("/login/")
async def login(username: str = Form(...), password: str = Form(...)):
    # Data is received as application/x-www-form-urlencoded
    return {"username": username}
```

### File Uploads (UploadFile, File)
To upload files, use `File` or `UploadFile`. `UploadFile` is preferred because it uses a spooled file (stored in memory up to a limit, then written to disk), and it behaves like an asynchronous file-like object.

```python
from fastapi import File, UploadFile

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(...)):
    # Read file contents asynchronously
    contents = await file.read()
    return {"filename": file.filename, "content_size": len(contents)}
```

### Headers and Cookies
Retrieve headers and cookies directly.

```python
from fastapi import Header, Cookie

@app.get("/headers-and-cookies/")
async def read_info(
    user_agent: str | None = Header(None), # Converts 'User-Agent' header automatically
    session_id: str | None = Cookie(None)
):
    return {"User-Agent": user_agent, "session_id": session_id}
```

### Background Tasks
Run tasks after returning a response.

```python
from fastapi import BackgroundTasks

def write_log(message: str):
    with open("log.txt", "a") as f:
        f.write(message + "\n")

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    # Schedule the task; the API responds immediately
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    return {"message": "Notification scheduled"}
```

> 💡 **Pro Tip**: Use `UploadFile` over `bytes` for file uploads to prevent loading massive files entirely into RAM and crashing your server.

> ⚠️ **Common Pitfall**: If you declare a parameter as a `BaseModel` and another parameter as `Form`, FastAPI will try to read the entire body as form data, leading to conflicts. You cannot mix JSON bodies and Form data in the same request.

---

## 5. RESPONSE HANDLING

### JSONResponse
FastAPI returns JSON by default. You can explicitly use `JSONResponse` to customize headers or status codes dynamically.

```python
from fastapi.responses import JSONResponse

@app.get("/custom-json")
async def custom_json():
    # Return JSON with a custom status code and header
    return JSONResponse(
        content={"message": "Custom JSON response"},
        status_code=418,
        headers={"X-Custom-Header": "value"}
    )
```

### PlainTextResponse
```python
from fastapi.responses import PlainTextResponse

@app.get("/text", response_class=PlainTextResponse)
async def get_text():
    # Returns text/plain
    return "This is pure text, not JSON."
```

### HTMLResponse
```python
from fastapi.responses import HTMLResponse

@app.get("/html", response_class=HTMLResponse)
async def get_html():
    # Returns text/html
    html_content = "<h1>Hello World!</h1>"
    return HTMLResponse(content=html_content, status_code=200)
```

### StreamingResponse
Used for large data, video streaming, or SSE (Server-Sent Events).

```python
from fastapi.responses import StreamingResponse
import time

def fake_data_streamer():
    for i in range(10):
        yield b"some fake data\n"
        time.sleep(0.5) # Synchronous sleep in generator

@app.get("/stream")
async def stream():
    # Stream data chunk by chunk
    return StreamingResponse(fake_data_streamer(), media_type="text/plain")
```

### Custom Responses
You can create your own response classes by inheriting from `Response`.

```python
from fastapi import Response

class XMLResponse(Response):
    media_type = "application/xml"

@app.get("/xml", response_class=XMLResponse)
async def get_xml():
    data = """<?xml version="1.0"?>
    <data><message>Hello XML</message></data>"""
    return XMLResponse(content=data)
```

### Response Models
Define the exact shape of the output data. This filters internal fields (like passwords) and ensures type consistency.

```python
class UserInDB(BaseModel):
    username: str
    password_hash: str
    email: str

class UserOut(BaseModel):
    username: str
    email: str

@app.post("/users/", response_model=UserOut)
async def create_user(user: UserInDB):
    # The API returns 'user', but FastAPI filters out 'password_hash' based on 'UserOut'
    return user
```

> 💡 **Pro Tip**: Use `response_model_exclude_unset=True` in decorators to avoid returning default values that weren't explicitly set in the response, reducing payload size.

> ⚠️ **Common Pitfall**: If your function returns a dictionary that has extra fields not in `response_model`, FastAPI filters them out safely. But if you return an object missing required fields from the `response_model`, FastAPI will raise a 500 Internal Server Error during validation.

---

## 6. PYDANTIC MODELS

### BaseModel Usage
Pydantic is the core of FastAPI's validation.

```python
from pydantic import BaseModel

class Product(BaseModel):
    id: int
    name: str
    in_stock: bool
```

### Field Validation
Use `Field` for extra validation constraints.

```python
from pydantic import Field

class Item(BaseModel):
    name: str = Field(..., min_length=2, max_length=50, description="The name of the item")
    price: float = Field(..., gt=0, description="The price must be greater than zero")
```

### Default Values
```python
class Profile(BaseModel):
    username: str
    bio: str = "Default biography" # Optional because it has a default
```

### Nested Models
Models can contain other models.

```python
class Address(BaseModel):
    street: str
    city: str

class User(BaseModel):
    username: str
    address: Address # Nested model
```

### Optional Fields
Use Python `| None` or `Optional` to denote fields that might be missing.

```python
class UpdateUser(BaseModel):
    email: str | None = None
    age: int | None = None
```

### Validators
In Pydantic v2, use `@field_validator` for custom validation logic.

```python
from pydantic import field_validator

class Account(BaseModel):
    username: str
    
    @field_validator('username')
    @classmethod
    def username_must_be_alphanumeric(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError('must be alphanumeric')
        return v
```

### Custom Validation Logic (Model Validators)
Use `@model_validator` to validate dependencies between fields.

```python
from pydantic import model_validator

class PasswordChange(BaseModel):
    password: str
    confirm_password: str

    @model_validator(mode='after')
    def check_passwords_match(self) -> 'PasswordChange':
        if self.password != self.confirm_password:
            raise ValueError('Passwords do not match')
        return self
```

> 💡 **Pro Tip**: In Pydantic v2, `BaseModel.model_dump()` replaces the old `.dict()` method, and `BaseModel.model_validate()` replaces `.parse_obj()`. Always use the `model_` prefixed methods to avoid conflicts with your own fields.

> ⚠️ **Common Pitfall**: Modifying a list or dict default directly in a class attribute (e.g., `tags: list = []`) causes the mutable default issue. Pydantic handles this gracefully internally, but for complex defaults, use `Field(default_factory=list)`.

---

## 7. DEPENDENCY INJECTION

### Depends()
FastAPI has an immensely powerful but simple Dependency Injection system. It allows you to share logic (database connections, security, caching).

```python
from fastapi import Depends

# A simple dependency function
def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    # 'commons' will contain the result of the 'common_parameters' function
    return commons
```

### Reusable Dependencies
Dependencies can form a tree. Dependencies can depend on other dependencies.

```python
def query_extractor(q: str | None = None):
    return q

def query_or_cookie_extractor(
    q: str = Depends(query_extractor),
    last_query: str | None = Cookie(None)
):
    if not q:
        return last_query
    return q
```

### Class-based Dependencies
Classes can be dependencies too. FastAPI instantiates them automatically.

```python
class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/things/")
async def read_things(commons: CommonQueryParams = Depends()):
    # Depends() without arguments infers the class from the type hint!
    return commons.__dict__
```

### Security Dependencies
Used to enforce authentication across routes.

```python
from fastapi.security import OAuth2PasswordBearer

# Indicates where the client gets the token
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/secure-data/")
async def secure_data(token: str = Depends(oauth2_scheme)):
    # If no token is provided, FastAPI automatically returns 401 Unauthorized
    return {"token": token}
```

### Database Session Injection
A classic use case is injecting a fresh database session per request and cleaning it up afterward.

```python
# SessionLocal is an SQLAlchemy sessionmaker
def get_db():
    db = SessionLocal()
    try:
        # Yield makes this a context-manager-like dependency
        yield db
    finally:
        db.close() # Always close the session after the request

@app.get("/users/")
def get_users(db: Session = Depends(get_db)):
    users = db.query(User).all()
    return users
```

> 💡 **Pro Tip**: If a dependency is computationally expensive but used multiple times in a single request (e.g., in sub-dependencies), use `Depends(func, use_cache=True)` (True is default). FastAPI executes it once and caches the result for that request.

> ⚠️ **Common Pitfall**: If a dependency uses `yield` to manage resources (like closing a DB connection), and an unhandled exception occurs in your endpoint, the code *after* the `yield` will still execute in a finally block, ensuring resources are freed. But you cannot catch endpoint exceptions inside the dependency itself unless you use a specialized setup.

---

## 8. MIDDLEWARE

### What is Middleware?
A "middleware" is a function that works with every request before it is processed by any specific path operation, as well as with every response before returning it.

### Built-in Middleware
FastAPI comes with Starlette middlewares like `CORSMiddleware`, `TrustedHostMiddleware`, `GZipMiddleware`, etc.

```python
from fastapi.middleware.gzip import GZipMiddleware

# Compress responses for payloads larger than 1000 bytes
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

### Custom Middleware
You can write custom middleware to log request times, modify headers, or catch generic errors.

```python
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()
    
    # Process the request and get the response
    response = await call_next(request)
    
    # Modify the response
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    
    return response
```

### Request/Response Lifecycle
1. Request arrives at ASGI server.
2. Custom/built-in Middlewares process the Request (Top to Bottom).
3. Router matches the URL.
4. Dependencies are evaluated.
5. Path Operation (Endpoint) executes.
6. Response returned.
7. Custom/built-in Middlewares process the Response (Bottom to Top).
8. ASGI Server sends response to client.

### CORS Middleware
Cross-Origin Resource Sharing is essential if your frontend is on a different domain/port.

```python
from fastapi.middleware.cors import CORSMiddleware

origins = [
    "http://localhost.tiangolo.com",
    "https://localhost.tiangolo.com",
    "http://localhost",
    "http://localhost:8080",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins, # List of allowed origins
    allow_credentials=True, # Support cookies
    allow_methods=["*"], # Allow all HTTP methods
    allow_headers=["*"], # Allow all headers
)
```

> 💡 **Pro Tip**: Middlewares run on *every* request, even 404s. Keep them extremely lightweight to avoid global performance degradation.

> ⚠️ **Common Pitfall**: If your custom middleware accesses `request.body()`, it consumes the request stream. Subsequent path operations will hang or receive an empty body unless you explicitly store and recreate the body stream. Avoid reading bodies in middleware; use Dependencies instead.

---

## 9. DATABASE INTEGRATION

### SQLAlchemy Setup
FastAPI doesn't force an ORM, but SQLAlchemy is the most common choice.

```bash
pip install sqlalchemy alembic
```

```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
# Use this for Postgres: "postgresql://user:password@postgresserver/db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False} # Only needed for SQLite
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()
```

### Models and Schemas
Keep database models (SQLAlchemy) separate from Pydantic schemas (Validation).

```python
# models.py
from sqlalchemy import Boolean, Column, Integer, String
from database import Base

class DBUser(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    is_active = Column(Boolean, default=True)

# schemas.py
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str

class UserSchema(BaseModel):
    id: int
    email: str
    is_active: bool

    class Config:
        from_attributes = True # Allows Pydantic to read ORM models (formerly orm_mode=True)
```

### CRUD Operations
```python
# crud.py
from sqlalchemy.orm import Session
import models, schemas

def create_user(db: Session, user: schemas.UserCreate):
    db_user = models.DBUser(email=user.email)
    db.add(db_user) # Add to session
    db.commit()     # Commit transaction
    db.refresh(db_user) # Get generated ID
    return db_user
```

### Async Database (Async SQLAlchemy)
SQLAlchemy 2.0 supports native async.

```bash
pip install asyncpg # For Postgres
```

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_async_db():
    async with AsyncSessionLocal() as session:
        yield session

# In endpoint:
@app.get("/users")
async def read_users(db: AsyncSession = Depends(get_async_db)):
    result = await db.execute(select(DBUser))
    return result.scalars().all()
```

### Alembic Migrations
Alembic manages database schema changes.

```bash
alembic init alembic
```
Configure `alembic.ini` and `env.py`, then:
```bash
alembic revision --autogenerate -m "Initial tables"
alembic upgrade head
```

> 💡 **Pro Tip**: Use SQLModel! It's built by the FastAPI creator and combines Pydantic and SQLAlchemy into one class definition, reducing duplication significantly.

> ⚠️ **Common Pitfall**: Doing synchronous DB queries (using standard `psycopg2` or basic SQLite) inside an `async def` endpoint. This blocks the entire event loop! Either use async database drivers, or define your endpoint as a standard `def` (FastAPI runs `def` endpoints in an external threadpool).

---

## 10. AUTHENTICATION & AUTHORIZATION

### OAuth2 Basics
FastAPI provides tools to implement OAuth2 with password flow (JWT tokens) natively, integrating beautifully with the Swagger UI.

### Security Utilities (OAuth2PasswordBearer)
```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from fastapi import Depends

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
```

### Password Hashing (passlib/bcrypt)
```bash
pip install passlib[bcrypt]
```

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)
```

### JWT Authentication
```bash
pip install pyjwt
```

```python
import jwt
from datetime import datetime, timedelta, timezone

SECRET_KEY = "your-super-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# The Login Endpoint
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    # 1. Verify username and password from DB
    user = fake_db.get(form_data.username)
    if not user or not verify_password(form_data.password, user['hashed_password']):
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    
    # 2. Create JWT token
    access_token = create_access_token(data={"sub": user['username']})
    
    # 3. Return it in OAuth2 expected format
    return {"access_token": access_token, "token_type": "bearer"}
```

### Role-Based Access
Protect endpoints using dependencies that parse the JWT token and check roles.

```python
async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except jwt.PyJWTError:
        raise credentials_exception
    user = fake_db.get(username)
    return user

async def get_admin_user(current_user: dict = Depends(get_current_user)):
    if current_user.get("role") != "admin":
        raise HTTPException(status_code=403, detail="Not enough privileges")
    return current_user

@app.delete("/users/{id}")
async def delete_user(id: int, admin_user: dict = Depends(get_admin_user)):
    return {"message": "User deleted"}
```

> 💡 **Pro Tip**: Swagger UI automatically renders an "Authorize" button when you use `OAuth2PasswordBearer`, allowing you to test authenticated endpoints interactively right from the browser.

> ⚠️ **Common Pitfall**: Storing highly sensitive or large data in a JWT token. JWTs are encoded, not encrypted. Anyone can read the payload; they just can't tamper with it without the secret key. Keep payloads small and safe (like `user_id`).

---

## 11. ERROR HANDLING

### HTTPException
To return HTTP errors to the client, raise `HTTPException`.

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items_db:
        # Halts execution and returns a JSON response with status 404
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items_db[item_id]}
```

### Custom Exception Handlers
You can catch custom exceptions globally and format the response however you like.

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )

@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

### Validation Errors
FastAPI automatically raises `RequestValidationError` when Pydantic validation fails. You can override it to change the default validation error format.

```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    # Change default 422 to 400 or modify the body format
    return JSONResponse(
        status_code=422,
        content={"errors": exc.errors(), "body": exc.body},
    )
```

### Global Error Handling
To catch any unhandled 500 error gracefully without exposing stack traces.

```python
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    # Log the exception here
    print(f"Unhandled error: {exc}")
    return JSONResponse(
        status_code=500,
        content={"message": "Internal server error occurred."}
    )
```

> 💡 **Pro Tip**: Always log full stack traces of 500 errors to your centralized logging system (e.g., Sentry) inside your global exception handler.

> ⚠️ **Common Pitfall**: Overusing `HTTPException` inside deep layers of your business logic or DB calls. This couples your business logic to HTTP concerns. Raise standard Python exceptions in core logic, and catch them in the API layer or using `exception_handlers` to convert them to `HTTPException`.

---

## 12. STATIC FILES & TEMPLATES

### Serving Static Files
Use `StaticFiles` to serve images, CSS, and JS.

```python
from fastapi.staticfiles import StaticFiles

# Mount directory "static" to the path "/static"
app.mount("/static", StaticFiles(directory="static"), name="static")

# Usage in browser: http://localhost:8000/static/style.css
```

### Jinja2 Templates
FastAPI works perfectly with Jinja2 for server-side HTML rendering.

```bash
pip install jinja2
```

```python
from fastapi.templating import Jinja2Templates
from fastapi import Request

templates = Jinja2Templates(directory="templates")

@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    # Renders 'templates/item.html'
    return templates.TemplateResponse(
        "item.html", {"request": request, "id": id, "title": "Item Details"}
    )
```

### Rendering HTML
In `templates/item.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
    <!-- Linking static files -->
    <link href="{{ url_for('static', path='/style.css') }}" rel="stylesheet">
</head>
<body>
    <h1>Item ID: {{ id }}</h1>
</body>
</html>
```

### Combining Frontend with FastAPI
While FastAPI is primarily an API framework, you can host built React/Vue/Angular apps by serving their `index.html` via a catch-all route and mounting their static assets.

```python
@app.get("/{full_path:path}")
async def serve_spa(full_path: str):
    # Return React index.html for unknown routes to allow client-side routing
    return FileResponse("frontend/build/index.html")
```

> 💡 **Pro Tip**: Use `url_for` in templates to generate URLs dynamically based on function names. If you change the URL path in the router, the template won't break.

> ⚠️ **Common Pitfall**: Putting static file mounts before dynamic paths that might match. `app.mount` is aggressive. Be careful not to overshadow API routes.

---

## 13. BACKGROUND TASKS

### BackgroundTasks
Run operations after the response has already been sent to the client. Excellent for tasks that take a few seconds but don't require the user to wait.

```python
from fastapi import BackgroundTasks

def write_notification(email: str, message: str):
    with open("log.txt", mode="a") as email_file:
        content = f"notification for {email}: {message}\n"
        email_file.write(content)

@app.post("/send-email/{email}")
async def send_email(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, "Registration successful")
    # Response is immediate; writing happens in background
    return {"message": "Email sent"}
```

### Use Cases
- Sending emails.
- Processing uploaded files (resizing images, compressing video).
- Sending data to external analytics systems.
- Heavy database writes that don't need immediate confirmation.

### Task Queues (Celery Intro)
For heavy processing, `BackgroundTasks` (which run in the same process) are not enough. If the server restarts, tasks are lost. For robust background processing, use Celery with Redis/RabbitMQ.

```python
# With Celery, you define tasks in a separate file/process
# tasks.py
from celery import Celery

celery_app = Celery("worker", broker="redis://localhost:6379/0")

@celery_app.task
def process_heavy_data(data):
    # Do heavy work
    pass

# main.py
from tasks import process_heavy_data

@app.post("/process/")
async def trigger_processing(data: str):
    # Push to Redis queue, return immediately
    process_heavy_data.delay(data)
    return {"message": "Processing started"}
```

> 💡 **Pro Tip**: Dependency injection works with `BackgroundTasks`. You can inject `BackgroundTasks` into a dependency, and the dependency can add tasks that execute after the request.

> ⚠️ **Common Pitfall**: Memory leaks. Because `BackgroundTasks` run in the same asyncio event loop as the main application, a background task that hangs or consumes huge amounts of memory will crash the main server. Use Celery/Arq for heavy lifting.

---

## 14. WEBSOCKETS

### WebSocket Basics
WebSockets enable real-time, two-way communication between the client and server.

```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    # 1. Accept the connection
    await websocket.accept()
    while True:
        try:
            # 2. Wait for incoming message
            data = await websocket.receive_text()
            # 3. Send response back
            await websocket.send_text(f"Message text was: {data}")
        except WebSocketDisconnect:
            break
```

### Real-time Communication & Connection Management
To broadcast messages to multiple users (e.g., a chat app), you need a connection manager to keep track of active websockets.

```python
from fastapi import WebSocket, WebSocketDisconnect

class ConnectionManager:
    def __init__(self):
        self.active_connections: list[WebSocket] = []

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

@app.websocket("/chat/{client_id}")
async def chat_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")
```

> 💡 **Pro Tip**: WebSockets don't inherently have authorization headers like HTTP. Pass tokens via the WebSocket URL path/query parameters, or handle authentication in the first message sent over the socket.

> ⚠️ **Common Pitfall**: Deploying WebSockets requires adjusting your reverse proxy (Nginx, Traefik, HAProxy) to upgrade the connection from HTTP to WS. If you don't configure this, WebSocket connections will drop instantly.

---

## 15. TESTING

### TestClient
FastAPI includes a `TestClient` based on `httpx` for synchronous testing of your API.

```bash
pip install pytest httpx
```

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"Hello": "World"}
```

### pytest Integration
Structure tests in a `tests` directory and use `pytest` fixtures.

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from main import app

@pytest.fixture
def client():
    # Setup
    with TestClient(app) as c:
        yield c
    # Teardown (if needed)

# tests/test_users.py
def test_create_user(client):
    response = client.post("/users/", json={"email": "test@test.com"})
    assert response.status_code == 200
    assert response.json()["email"] == "test@test.com"
```

### Dependency Overrides
When testing, you often want to replace the real database with a test database, or replace external API calls with mocks. FastAPI's `app.dependency_overrides` makes this trivial.

```python
from database import get_db

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

# Override the dependency globally for the app
app.dependency_overrides[get_db] = override_get_db

def test_db_logic():
    response = client.get("/users")
    # Will use the mocked TestingSessionLocal instead of the real database
    assert response.status_code == 200
```

> 💡 **Pro Tip**: If you have heavily async logic or background tasks, consider using `httpx.AsyncClient` alongside `pytest-asyncio` to test your application purely asynchronously.

> ⚠️ **Common Pitfall**: Forgetting to clear `app.dependency_overrides.clear()` after a test. If you don't clear it, your mock will leak into subsequent tests, causing confusing failures.

---

## 16. SECURITY

### CORS (Cross-Origin Resource Sharing)
Already covered in Middleware, but crucially important. Never use `allow_origins=["*"]` in production if your API handles authentication cookies.

### HTTPS Configuration
FastAPI does not handle HTTPS certificates natively. Terminate SSL/TLS at your reverse proxy (e.g., Nginx, Traefik, or an AWS Application Load Balancer).

### Input Validation & Preventing Injections
- **SQL Injection**: Prevented automatically if you use an ORM like SQLAlchemy correctly (don't use string concatenation for queries).
- **XSS (Cross-Site Scripting)**: If returning HTML, use Jinja2 which auto-escapes output.
- **Data Validation**: Pydantic models automatically sanitize and reject unexpected or maliciously formatted JSON.

### Rate Limiting Basics
FastAPI doesn't include rate limiting by default. Use a package like `slowapi`.

```bash
pip install slowapi
```

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.get("/expensive-endpoint")
@limiter.limit("5/minute") # Limit to 5 requests per minute per IP
async def expensive():
    return {"msg": "This is heavy on the server"}
```

> 💡 **Pro Tip**: Use the `secure` flag on cookies (`Cookie(..., secure=True)`) so they are only transmitted over HTTPS.

> ⚠️ **Common Pitfall**: Exposing the `/docs` (Swagger UI) in production without protection can expose your entire API surface area to attackers. In production, you might want to disable it (`app = FastAPI(docs_url=None, redoc_url=None)`) or put it behind basic auth.

---

## 17. DEPLOYMENT

### Uvicorn vs Gunicorn
Uvicorn is an ASGI worker. Gunicorn is a process manager. In production, you run Gunicorn with Uvicorn worker classes to handle multi-processing and process restarts.

```bash
pip install gunicorn uvicorn
```

```bash
# Run 4 worker processes. If one crashes, Gunicorn restarts it.
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker
```

### Docker Setup
Create a `Dockerfile` for easy, reproducible deployments.

```dockerfile
FROM python:3.11-slim

WORKDIR /code

COPY ./requirements.txt /code/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

COPY ./app /code/app

# Start the server using Uvicorn directly (container orchestration manages restarts)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]
```

### Nginx Reverse Proxy
Place Nginx in front of Gunicorn/Uvicorn to handle SSL, static files, and buffering.

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.0:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Deployment Platforms
- **AWS / GCP / Azure**: Deploy via Docker containers using ECS, Cloud Run, or App Service.
- **Render / Railway / Heroku**: Connect your GitHub repo, define a build command, and set the start command to `uvicorn main:app --host 0.0.0.0 --port $PORT`.

> 💡 **Pro Tip**: When running in Docker/Kubernetes, you usually don't need Gunicorn. Run Uvicorn directly with a single worker, and let the orchestrator (Kubernetes/ECS) spin up multiple containers for scaling.

> ⚠️ **Common Pitfall**: Binding Uvicorn to `127.0.0.1` inside a Docker container. The container is an isolated network; you MUST bind to `0.0.0.0` for traffic to reach the app from outside the container.

---

## 18. ASYNC & PERFORMANCE

### Async vs Sync Functions
Understanding the `async def` vs `def` difference is crucial in FastAPI.
- **`async def`**: Runs directly on the main event loop. If you block inside it (e.g., `time.sleep()`, synchronous DB query, `requests.get()`), the *entire server stops responding* to other requests until it finishes.
- **`def`**: FastAPI recognizes this and runs it in an external threadpool. This means if you block inside a `def`, it only blocks that specific thread, and the main server keeps serving other requests.

```python
import time
import asyncio

@app.get("/sync-block")
def sync_block():
    # Safe! FastAPI runs this in a thread. Other requests still work.
    time.sleep(5)
    return {"status": "done"}

@app.get("/async-block")
async def async_block():
    # DISASTER! This blocks the event loop. Server hangs for 5 seconds.
    time.sleep(5) 
    return {"status": "done"}

@app.get("/async-proper")
async def async_proper():
    # Perfect! Returns control to the event loop while waiting.
    await asyncio.sleep(5) 
    return {"status": "done"}
```

### Concurrency vs Parallelism
- **Concurrency**: Dealing with a lot of things at once (e.g., waiting for multiple I/O operations like DB queries or API calls). Handled brilliantly by `async/await`.
- **Parallelism**: Doing a lot of things at once (e.g., heavy CPU math operations). Handled by multiple processes (e.g., Gunicorn workers). Python's GIL prevents true parallelism in a single process.

### Best Async Practices
- Use `httpx` instead of `requests` for making outbound API calls.
- Use `asyncpg` or async SQLAlchemy for database operations.
- Use `aiofiles` for file I/O operations.

> 💡 **Pro Tip**: If you are unsure if a library is async or not, or if it does heavy CPU work (like image processing, machine learning inference), always define your endpoint with `def` to be safe.

> ⚠️ **Common Pitfall**: Using `async def` just because you think "async is faster". `async` is only faster if you are awaiting I/O. If you put synchronous code in an `async def`, it is infinitely slower because it stalls the framework.

---

## 19. PERFORMANCE OPTIMIZATION

### Caching (Redis)
Avoid recomputing heavy requests or querying the DB unnecessarily. Use Redis.

```bash
pip install redis fastapi-cache2[redis]
```

```python
from fastapi_cache import FastAPICache
from fastapi_cache.backends.redis import RedisBackend
from fastapi_cache.decorator import cache
from redis import asyncio as aioredis

@app.on_event("startup")
async def startup():
    redis = aioredis.from_url("redis://localhost", encoding="utf8", decode_responses=True)
    FastAPICache.init(RedisBackend(redis), prefix="fastapi-cache")

@app.get("/expensive-data")
@cache(expire=60) # Cache response for 60 seconds
async def get_expensive_data():
    await asyncio.sleep(2) # Simulate heavy DB query
    return {"data": "This is cached!"}
```

### Database Optimization
- **Connection Pooling**: Use PgBouncer (for Postgres) or configure SQLAlchemy's `pool_size` and `max_overflow`.
- **Lazy vs Eager Loading**: Avoid N+1 query problems in SQLAlchemy by using `joinedload` or `selectinload` for relationships.

```python
from sqlalchemy.orm import joinedload

# Good: Fetches users and their items in a single query
db.query(User).options(joinedload(User.items)).all()
```

### Profiling Tools
Use `py-spy` or `cProfile` to find bottlenecks. For FastAPI middleware profiling:

```bash
pip install pyinstrument
```

```python
from pyinstrument import Profiler
from fastapi.responses import HTMLResponse

@app.middleware("http")
async def profile_request(request: Request, call_next):
    profiler = Profiler(interval=0.001, async_mode="enabled")
    profiler.start()
    response = await call_next(request)
    profiler.stop()
    
    # Save profile to file or print
    with open("profile.html", "w") as f:
        f.write(profiler.output_html())
    return response
```

> 💡 **Pro Tip**: Use ORJSON. It's the fastest Python JSON library. You can set it as your default response class for a massive speedup on large payloads.
`app = FastAPI(default_response_class=ORJSONResponse)`

> ⚠️ **Common Pitfall**: Caching authenticated endpoints without including the user ID in the cache key. User A might see User B's private cached data!

---

## 20. ADVANCED TOPICS

### Custom OpenAPI Docs
Customize the Swagger UI to match your branding.

```python
app = FastAPI(
    title="Chimera Corp API",
    description="This is a very fancy description, with **markdown**.",
    version="2.5.0",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "API Support",
        "url": "http://www.example.com/support",
        "email": "support@example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
)
```

### API Versioning
Instead of putting `/v1/` in every route, group them using APIRouters.

```python
api_v1 = APIRouter(prefix="/v1")
api_v2 = APIRouter(prefix="/v2")

@api_v1.get("/users")
def get_v1(): return {"v": 1}

@api_v2.get("/users")
def get_v2(): return {"v": 2}

app.include_router(api_v1)
app.include_router(api_v2)
```

### GraphQL with FastAPI
Use `Strawberry` to add a GraphQL endpoint directly alongside your REST endpoints.

```bash
pip install strawberry-graphql
```

```python
import strawberry
from strawberry.fastapi import GraphQLRouter

@strawberry.type
class Query:
    @strawberry.field
    def hello(self) -> str:
        return "Hello World"

schema = strawberry.Schema(Query)
graphql_app = GraphQLRouter(schema)

app.include_router(graphql_app, prefix="/graphql")
```

### Message Queues (Kafka/RabbitMQ Intro)
For microservices architectures, FastAPI acts as the ingress, pushing events to a bus.

```python
import pika

def send_message_to_queue(message):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='task_queue')
    channel.basic_publish(exchange='', routing_key='task_queue', body=message)
    connection.close()

@app.post("/trigger-event")
async def trigger_event(payload: dict):
    # Offload event processing entirely
    send_message_to_queue(str(payload))
    return {"status": "event queued"}
```

### Logging Configuration
Proper JSON logging is essential for production observability (ELK stack/Datadog).

```python
import logging
import sys

# Basic setup directing logs to stdout
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[logging.StreamHandler(sys.stdout)]
)
logger = logging.getLogger("fastapi_app")

@app.get("/")
def home():
    logger.info("Home endpoint was accessed")
    return {"status": "ok"}
```

### Third-Party Ecosystem
Leverage these incredible tools to avoid reinventing the wheel:
- **FastAPI Users**: Ready-to-use user management, authentication, registration, forgot password, social logins.
- **SQLModel**: Unifies SQLAlchemy and Pydantic into a single model declaration.
- **Pydantic Settings**: Powerful type-safe environment variable management.

> 💡 **Pro Tip**: Use structured JSON logging (like `structlog`) in production so log aggregators can easily parse and search your application logs.

> ⚠️ **Common Pitfall**: Overcomplicating a monolith into microservices too early. FastAPI scales exceptionally well as a monolith. Don't add Kafka/RabbitMQ unless organizational constraints or extreme traffic demand it.
