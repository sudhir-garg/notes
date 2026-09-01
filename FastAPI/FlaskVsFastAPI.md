# Flask vs FastAPI

Flask and FastAPI are two popular Python web frameworks used for building web applications and APIs.  
This guide compares them feature by feature and includes code examples.

## Table of Contents

- [1. Overview](#1-overview)
- [2. Feature-by-Feature Comparison](#2-feature-by-feature-comparison)
- [3. Routing Example](#3-routing-example)
- [4. Path Parameters](#4-path-parameters)
- [5. Query Parameters](#5-query-parameters)
- [6. Request Validation](#6-request-validation)
- [7. Automatic API Docs](#7-automatic-api-docs)
- [8. Async Support](#8-async-support)
- [9. Dependency Injection](#9-dependency-injection)
- [10. Error Handling](#10-error-handling)
- [11. Bonus: FastMCP and FastAPI Synergy](#11-bonus-fastmcp-and-fastapi-synergy)
- [12. When to Choose Flask](#12-when-to-choose-flask)
- [13. When to Choose FastAPI](#13-when-to-choose-fastapi)
- [14. Quick Recommendation](#14-quick-recommendation)
- [15. Summary](#15-summary)

---

## 1. Overview

### Flask
Flask is a lightweight, flexible, and minimal web framework. It gives you the basics and lets you add extensions as needed.

### FastAPI
FastAPI is a modern framework designed for building APIs quickly. It uses Python type hints for validation, serialization, and automatic documentation.

---

## 2. Feature-by-Feature Comparison

| Feature | Flask | FastAPI |
|---|---|---|
| Framework style | Micro-framework, very flexible | API-first, modern, opinionated |
| Performance | Good | Very fast, especially with async |
| Async support | Limited / extension-based | Built-in async support |
| Input validation | Manual or via extensions | Automatic with Pydantic and type hints |
| API docs | Not built-in | Automatic Swagger / ReDoc |
| Type hints | Optional | Central to the framework |
| Dependency injection | Manual patterns or extensions | Built-in `Depends()` |
| Learning curve | Easier for simple apps | Easy if you know Python typing |
| Best use case | Websites, small apps, prototypes | APIs, microservices, async apps |

---

## 3. Routing Example

### Flask
```python
from flask import Flask

app = Flask(__name__)

@app.route("/hello")
def hello():
    return {"message": "Hello from Flask"}
```

### FastAPI
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello from FastAPI"}
```

---

## 4. Path Parameters

### Flask
```python
from flask import Flask

app = Flask(__name__)

@app.route("/users/<int:user_id>")
def get_user(user_id):
    return {"user_id": user_id}
```

### FastAPI
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

---

## 5. Query Parameters

### Flask
```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/search")
def search():
    q = request.args.get("q", "")
    return {"query": q}
```

### FastAPI
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/search")
def search(q: str = ""):
    return {"query": q}
```

---

## 6. Request Validation

### Flask
Flask does not validate request data automatically. You usually do it yourself or use extensions.

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/items", methods=["POST"])
def create_item():
    data = request.get_json()

    if not data or "name" not in data:
        return jsonify({"error": "name is required"}), 400

    return {"name": data["name"]}
```

### FastAPI
FastAPI validates automatically using type hints and Pydantic models.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str

@app.post("/items")
def create_item(item: Item):
    return item
```

---

## 7. Automatic API Docs

### Flask
Flask does not generate docs automatically. You usually add tools like Flasgger or Flask-RESTX.

### FastAPI
FastAPI automatically provides:
- Swagger UI at `/docs`
- ReDoc at `/redoc`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

---

## 8. Async Support

### Flask
Flask is mostly synchronous. Async usage is possible in newer versions, but it is not the main design focus.

### FastAPI
FastAPI supports async endpoints naturally.

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/slow")
async def slow_endpoint():
    await asyncio.sleep(2)
    return {"message": "done"}
```

---

## 9. Dependency Injection

### Flask
Flask does not have built-in dependency injection like FastAPI, but you can implement dependency-like behavior using decorators, helper functions, or application context.

```python
from flask import Flask, g, jsonify

app = Flask(__name__)

def get_db():
    if "db" not in g:
        g.db = {"connection": "mock-db-connection"}
    return g.db

@app.teardown_appcontext
def close_db(exception=None):
    db = g.pop("db", None)
    if db:
        # Close real DB connection here
        pass

@app.route("/profile")
def profile():
    db = get_db()
    return jsonify({
        "message": "Profile fetched",
        "db": db["connection"]
    })
```

### FastAPI
FastAPI has built-in dependency injection using `Depends()`.

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def get_token():
    return "secret-token"

@app.get("/profile")
def profile(token: str = Depends(get_token)):
    return {"token": token}
```

---

## 10. Error Handling

### Flask
```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Not found"}), 404
```

### FastAPI
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id < 0:
        raise HTTPException(status_code=400, detail="Invalid item ID")
    return {"item_id": item_id}
```

---

## 11. Bonus: FastMCP and FastAPI Synergy

FastAPI works well with MCP-style tooling and backend services because it already provides:
- clean route definitions
- request validation
- async support
- automatic docs

This makes it a strong fit for building APIs that can act as a service layer for agentic or tool-based systems.

### Example: FastAPI endpoint exposing a tool-like action
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class AddRequest(BaseModel):
    a: int
    b: int

@app.post("/add")
def add_numbers(payload: AddRequest):
    return {
        "result": payload.a + payload.b
    }
```

### Example: Simple Flask equivalent
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/add", methods=["POST"])
def add_numbers():
    data = request.get_json()
    a = data.get("a", 0)
    b = data.get("b", 0)

    return jsonify({
        "result": a + b
    })
```

FastAPI is usually a better fit when you want:
- typed inputs and outputs
- strong API contracts
- automatic docs for tooling and clients
- scalable async endpoints

---

## 12. When to Choose Flask

Choose Flask if you want:
- a very lightweight framework
- full control over components
- a traditional web app or simple backend
- an ecosystem that has been around for a long time

---

## 13. When to Choose FastAPI

Choose FastAPI if you want:
- modern API development
- built-in validation
- automatic docs
- async support
- strong use of Python type hints

---

## 14. Quick Recommendation

- Use **Flask** for small apps, learning, or when you want maximum simplicity.
- Use **FastAPI** for APIs, microservices, and modern backend services.

---

## 15. Summary

Flask is flexible and simple.  
FastAPI is modern, fast, and great for APIs.

If your project is API-heavy, FastAPI is usually the better choice.  
If you want minimal setup and a traditional lightweight framework, Flask is a solid pick.
