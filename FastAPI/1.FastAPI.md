# FastAPI Detailed Notes

FastAPI is a modern, high-performance Python web framework for building APIs. It is designed around standard Python type hints and provides automatic validation, serialization, and interactive API documentation.

---

## Table of Contents

- [1. What is FastAPI?](#1-what-is-fastapi)
- [2. Why Use FastAPI?](#2-why-use-fastapi)
- [3. Core Features](#3-core-features)
- [4. How FastAPI Works](#4-how-fastapi-works)
- [5. Project Structure](#5-project-structure)
- [6. Basic App Example](#6-basic-app-example)
- [7. Path Parameters](#7-path-parameters)
- [8. Query Parameters](#8-query-parameters)
- [9. Request Body and Pydantic Models](#9-request-body-and-pydantic-models)
- [10. Validation and Type Hints](#10-validation-and-type-hints)
- [11. Response Models](#11-response-models)
- [12. Dependency Injection](#12-dependency-injection)
- [13. CRUD with FastAPI](#13-crud-with-fastapi)
- [14. Async and Await](#14-async-and-await)
- [15. Error Handling](#15-error-handling)
- [16. Security and Authentication](#16-security-and-authentication)
- [17. Database Integration](#17-database-integration)
- [18. Background Tasks](#18-background-tasks)
- [19. Middleware](#19-middleware)
- [20. CORS](#20-cors)
- [21. Testing FastAPI Apps](#21-testing-fastapi-apps)
- [22. Automatic Documentation](#22-automatic-documentation)
- [23. Production Deployment](#23-production-deployment)
- [24. Best Practices](#24-best-practices)
- [25. Advantages and Limitations](#25-advantages-and-limitations)
- [26. Summary](#26-summary)

---

## 1. What is FastAPI?

FastAPI is a Python framework for building APIs quickly and cleanly. It is built on top of:
- **Starlette** for web handling and async support
- **Pydantic** for data validation and parsing

It is popular because it is:
- fast
- easy to use
- type-safe
- automatically documented

### Simple explanation
Think of FastAPI as a framework that helps you write API code in a way that is:
- readable for humans
- easy for Python to understand
- easy for clients to consume

It reduces a lot of repetitive work such as manual validation, serialization, and API documentation.

---

## 2. Why Use FastAPI?

FastAPI is useful when you want:
- automatic request validation
- clean and readable code
- built-in OpenAPI documentation
- async support
- strong editor support with type hints
- high performance for APIs and microservices

### Why this matters
In traditional frameworks, you often need to write extra code to:
- check if fields exist
- convert strings to integers
- create API docs manually
- handle bad input yourself

FastAPI does much of this automatically, which helps reduce bugs and development time.

---

## 3. Core Features

### Key features
- Fast performance
- Python type hints
- Automatic validation
- Automatic docs
- Dependency injection
- Async support
- Security helpers
- Easy testing

### Common use cases
- REST APIs
- Microservices
- ML model serving
- Backend services
- Internal tools
- Agent/tool servers

### Explanation
FastAPI is especially strong when your application’s main job is to expose data or functionality over HTTP. It is often used when the backend is not rendering HTML pages, but instead serving JSON responses to frontend apps, mobile apps, or other services.

---

## 4. How FastAPI Works

FastAPI uses Python type hints to understand:
- what input is expected
- how to validate it
- how to generate documentation

For example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

Here FastAPI knows `item_id` must be an integer. If a string is passed, it returns a validation error automatically.

### Explanation
When you annotate a parameter like `item_id: int`, FastAPI reads that annotation and uses it in multiple ways:
1. It validates incoming requests.
2. It converts the value into the correct type.
3. It includes the type information in the generated docs.

This is one of the biggest reasons FastAPI feels powerful and concise.

---

## 5. Project Structure

A typical FastAPI project may look like this:

```text
app/
├── main.py
├── routers/
│   ├── users.py
│   └── items.py
├── models/
│   └── schemas.py
├── services/
│   └── item_service.py
├── database/
│   └── connection.py
└── tests/
    └── test_main.py
```

### Suggested organization
- `main.py` for app startup
- `routers/` for route files
- `models/` for schemas
- `services/` for business logic
- `database/` for DB setup
- `tests/` for test cases

### Explanation
As your app grows, keeping everything in one file becomes hard to manage. A clear structure helps you separate concerns:
- routes handle HTTP requests
- services contain application logic
- models define data contracts
- database code handles persistence

This makes the code easier to test, maintain, and extend.

---

## 6. Basic App Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello, FastAPI"}
```

Run it with:

```bash
uvicorn main:app --reload
```

### Explanation
- `FastAPI()` creates the app instance.
- `@app.get("/")` registers a route for HTTP GET requests.
- The function returns a dictionary, which FastAPI converts to JSON automatically.
- `uvicorn` is the ASGI server that runs the app.

---

## 7. Path Parameters

Path parameters are part of the URL.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

### Notes
- `user_id` is extracted from the path
- type hints enforce validation
- FastAPI returns an error if the value is invalid

### Explanation
Path parameters are useful when the resource identity is part of the route. For example:
- `/users/10`
- `/orders/99`
- `/products/123`

FastAPI automatically converts and validates them. If `user_id` is typed as `int`, the framework tries to parse the incoming value into an integer. If parsing fails, the client gets a structured error response.

---

## 8. Query Parameters

Query parameters come after `?` in the URL.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/search")
def search(q: str = "", limit: int = 10):
    return {"query": q, "limit": limit}
```

Example request:

```text
/search?q=fastapi&limit=5
```

### Explanation
Query parameters are often used for:
- filtering
- pagination
- searching
- sorting

Because they are optional by nature, they often have default values. FastAPI also validates them based on the declared type. If `limit` is declared as an integer, passing invalid values will trigger a validation error.

---

## 9. Request Body and Pydantic Models

FastAPI uses Pydantic models for request bodies.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    is_available: bool = True

@app.post("/items")
def create_item(item: Item):
    return item
```

### Benefits
- automatic parsing
- automatic validation
- clean model definitions
- easy JSON serialization

### Explanation
Request bodies are common in POST, PUT, and PATCH requests. Instead of manually reading raw JSON and checking each field, you define a model. FastAPI then:
- parses the JSON body
- validates required fields
- converts values into Python types
- returns readable error messages if data is invalid

This makes the API contract explicit and reliable.

---

## 10. Validation and Type Hints

Validation is one of FastAPI’s strongest features.

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/products")
def list_products(
    page: int = Query(1, ge=1),
    size: int = Query(10, ge=1, le=100)
):
    return {"page": page, "size": size}
```

### Validation examples
- `ge=1` means greater than or equal to 1
- `le=100` means less than or equal to 100

FastAPI rejects invalid input automatically.

### Explanation
Validation rules are important because they protect your application from bad or unexpected input. They also improve your API documentation because the constraints appear in the generated schema. This helps frontend developers and API consumers know exactly what values are acceptable.

---

## 11. Response Models

You can define the shape of responses too.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class UserOut(BaseModel):
    id: int
    name: str

@app.get("/user", response_model=UserOut)
def get_user():
    return {"id": 1, "name": "Alice", "password": "secret"}
```

### Why response models matter
- they filter unwanted fields
- they enforce response consistency
- they improve API documentation

### Explanation
Response models help prevent accidentally returning sensitive or unnecessary fields. In the example above, the function returns a `password` field, but because the response model does not include it, FastAPI excludes it from the final API response. This is very useful for security and clean API design.

---

## 12. Dependency Injection

FastAPI has built-in dependency injection through `Depends`.

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def get_token():
    return "sample-token"

@app.get("/profile")
def profile(token: str = Depends(get_token)):
    return {"token": token}
```

### Common uses
- authentication
- database sessions
- configuration
- reusable business logic
- permission checks

### Example with database dependency

```python
from fastapi import FastAPI, Depends

app = FastAPI()

class FakeDB:
    def query(self):
        return "data from db"

def get_db():
    return FakeDB()

@app.get("/data")
def get_data(db: FakeDB = Depends(get_db)):
    return {"result": db.query()}
```

### Explanation
Dependency injection lets you declare what a route needs, and FastAPI provides it automatically. This makes code cleaner and more testable. For example:
- a database session can be created per request
- an authentication dependency can verify a user before route logic runs
- configuration values can be reused across multiple endpoints

---

## 13. CRUD with FastAPI

### Create
```python
@app.post("/items")
def create_item(item: Item):
    return item
```

### Read
```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

### Update
```python
@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item}
```

### Delete
```python
@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    return {"message": "Item deleted", "item_id": item_id}
```

### Explanation
CRUD represents the most common operations in API development:
- Create: add new data
- Read: fetch data
- Update: modify data
- Delete: remove data

FastAPI makes each of these operations straightforward to implement because of its clear route decorators and typed request handling.

---

## 14. Async and Await

FastAPI supports asynchronous functions naturally.

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/slow")
async def slow_endpoint():
    await asyncio.sleep(2)
    return {"message": "done"}
```

### When to use async
- network calls
- I/O-heavy operations
- calling external services
- database operations with async drivers

### When not to use async
- CPU-heavy work
- simple synchronous logic
- if your libraries are not async-compatible

### Explanation
Async programming helps your app handle more concurrent requests efficiently, especially when waiting on slow operations like APIs or databases. However, async is not magic—if the operation is CPU-heavy, async will not necessarily make it faster. It is most useful when the app spends time waiting rather than computing.

---

## 15. Error Handling

### HTTPException
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id < 0:
        raise HTTPException(status_code=400, detail="Invalid item ID")
    return {"item_id": item_id}
```

### Custom exception handler
```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(
        status_code=400,
        content={"error": str(exc)}
    )
```

### Explanation
Good error handling makes APIs easier to debug and safer to use. FastAPI supports:
- standard HTTP errors
- validation errors
- custom exception handling

This lets you return consistent error responses to clients.

---

## 16. Security and Authentication

FastAPI includes helpers for security.

### OAuth2 password flow example
```python
from fastapi import FastAPI, Depends
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/secure")
def secure_route(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

### Common security patterns
- JWT authentication
- OAuth2
- API keys
- HTTP Basic auth
- role-based access control

### Explanation
Security is a critical part of backend development. FastAPI provides building blocks, but you still need to design how authentication and authorization will work in your application. In many real projects, token verification, user lookup, and permission checks are implemented as reusable dependencies.

---

## 17. Database Integration

FastAPI works with many databases and ORMs.

### SQLAlchemy example
```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session

app = FastAPI()

def get_db():
    db = "database_session"
    try:
        yield db
    finally:
        pass

@app.get("/users")
def list_users(db: Session = Depends(get_db)):
    return {"message": "Users list", "db": db}
```

### Common database tools
- SQLAlchemy
- SQLModel
- Tortoise ORM
- Databases
- asyncpg

### Explanation
FastAPI does not force you to use one specific database library. You can choose synchronous or asynchronous database tools depending on your project needs. The `yield` pattern is often used to create a session, use it for the request, and then clean it up afterward.

---

## 18. Background Tasks

FastAPI supports background work after returning a response.

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def send_email(email: str):
    print(f"Sending email to {email}")

@app.post("/notify")
def notify(background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email, "user@example.com")
    return {"message": "Notification scheduled"}
```

### Use cases
- sending emails
- logging
- cleanup jobs
- analytics events

### Explanation
Background tasks are useful when a job should happen after the user already received a response. This improves responsiveness. For heavy or critical jobs, though, a separate task queue like Celery or RQ may be a better fit.

---

## 19. Middleware

Middleware lets you run logic before and after requests.

```python
from fastapi import FastAPI, Request
import time

app = FastAPI()

@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    response.headers["X-Process-Time"] = str(time.time() - start)
    return response
```

### Uses of middleware
- timing
- logging
- authentication
- compression
- custom headers

### Explanation
Middleware is useful for cross-cutting concerns—things that apply to many or all endpoints. Instead of repeating the same code in every route, you can place it in middleware and keep route handlers focused on business logic.

---

## 20. CORS

CORS is important when frontends call your API from another domain.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Note
For production, avoid `allow_origins=["*"]` unless appropriate.

### Explanation
If your frontend is hosted on a different domain than your backend, the browser may block requests unless CORS is configured. This is common in modern full-stack applications where the frontend and backend are deployed separately.

---

## 21. Testing FastAPI Apps

FastAPI is easy to test with `TestClient`.

```python
from fastapi.testclient import TestClient
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello"}

client = TestClient(app)

def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello"}
```

### Testing benefits
- quick API tests
- integration tests
- easy automation in CI

### Explanation
Testing is important because API behavior should remain stable as code changes. FastAPI’s testing tools make it easy to simulate requests and check responses without running a full production server.

---

## 22. Automatic Documentation

FastAPI automatically generates documentation.

### Docs endpoints
- `/docs` for Swagger UI
- `/redoc` for ReDoc
- `/openapi.json` for the OpenAPI schema

### Why docs are valuable
- easier frontend integration
- easier client generation
- easier debugging
- better API clarity

### Explanation
Documentation often becomes outdated when it is written separately from code. FastAPI solves this by generating docs from the actual route definitions and type hints. This keeps the docs consistent with your implementation.

---

## 23. Production Deployment

### Common deployment options
- Uvicorn
- Gunicorn + Uvicorn workers
- Docker
- Kubernetes
- cloud platforms like AWS, Azure, GCP

### Example command
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

### Production tips
- use environment variables
- enable logging
- configure CORS properly
- separate settings per environment
- use a reverse proxy like Nginx if needed

### Explanation
Development and production are not the same. In production, you usually need:
- proper process management
- security settings
- monitoring
- scalability
- deployment automation

FastAPI integrates well into modern deployment workflows because it is ASGI-based and works smoothly with containerized environments.

---

## 24. Best Practices

- Use Pydantic models for all request and response data
- Keep route handlers small
- Move business logic into services
- Use async only when it gives real benefit
- Validate query/path/body inputs carefully
- Organize routes with `APIRouter`
- Write tests for endpoints
- Use dependency injection for reusable logic
- Keep authentication and authorization separate
- Use proper status codes

### Explanation
Good architecture matters more as the application grows. FastAPI gives you a lot of flexibility, but with flexibility comes responsibility. A clean structure and consistent conventions keep the project maintainable.

---

## 25. Advantages and Limitations

### Advantages
- Very fast
- Easy to learn
- Excellent validation
- Great developer experience
- Automatic docs
- Strong type support
- Good for APIs and microservices

### Limitations
- Not ideal for traditional server-rendered web apps compared to frameworks like Django
- Async can be confusing if your stack is not async-friendly
- Requires thoughtful project structure for large systems

### Explanation
FastAPI is excellent for APIs, but it is not the best framework for every kind of project. If you need lots of built-in admin, ORM integration, and server-rendered pages, another framework might fit better. FastAPI shines when the main goal is building clean, fast, typed APIs.

---

## 26. Summary

FastAPI is a modern Python framework that makes API development fast, clean, and reliable. Its biggest strengths are:
- type hints
- validation
- async support
- automatic docs
- dependency injection

It is a strong choice for:
- REST APIs
- microservices
- backend services
- data and ML APIs
- tool-serving applications

If you want a framework that is simple, modern, and production-friendly for APIs, FastAPI is an excellent choice.

### Final takeaway
FastAPI helps you write less boilerplate and more expressive code. It is especially useful when correctness, scalability, and developer productivity all matter.
