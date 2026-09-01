# Comprehensive Guide to Pydantic (V2)

Pydantic is the most widely used data validation library for Python. It uses type hints to validate data, enforce constraints, and serialize/deserialize complex data structures. This guide covers Pydantic V2, which offers significant performance improvements and a revamped API.

## Table of Contents
1. [Introduction to Pydantic](#1-introduction-to-pydantic)
2. [Installation](#2-installation)
3. [Basic Models and Type Hints](#3-basic-models-and-type-hints)
4. [Data Validation and Type Coercion](#4-data-validation-and-type-coercion)
5. [Field Constraints and Metadata](#5-field-constraints-and-metadata)
6. [Nested Models](#6-nested-models)
7. [Custom Validators](#7-custom-validators)
8. [Serialization and Deserialization](#8-serialization-and-deserialization)
9. [Model Configuration](#9-model-configuration)
10. [Optional Fields](#10-optional-fields)

---

## 1. Introduction to Pydantic

Pydantic enforces type hints at runtime and provides user-friendly errors when data is invalid. It is heavily used in modern Python frameworks like **FastAPI** to handle request/response validation.

**Key Features:**
*   **Powered by type hints:** No new schema definition micro-language to learn.
*   **Speed:** Core validation logic in Pydantic V2 is written in Rust (`pydantic-core`), making it incredibly fast.
*   **JSON Schema:** Models can emit JSON Schema for easy integration with API documentation (like OpenAPI/Swagger).

---

## 2. Installation

Install Pydantic using pip. For standard use:

```bash
pip install pydantic
```

To install Pydantic with email validation support (requires `email-validator`):

```bash
pip install "pydantic[email]"
```

---

## 3. Basic Models and Type Hints

The core building block in Pydantic is the `BaseModel`. You define a model by subclassing `BaseModel` and declaring attributes with type hints.

### Example: A Simple User Model

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str
    is_active: bool = True  # Default value provided

# Creating an instance using keyword arguments
user = User(id=1, username="john_doe")
print(user.id)          # Output: 1
print(user.is_active)   # Output: True (used default)
```

---

## 4. Data Validation and Type Coercion

Pydantic attempts to coerce data to the correct type if possible. If coercion fails, it raises a `ValidationError`.

### Example: Type Coercion and Errors

```python
from pydantic import BaseModel, ValidationError

class Product(BaseModel):
    name: str
    price: float
    stock: int

# Valid data with coercion
# '19.99' (str) becomes 19.99 (float)
# '50' (str) becomes 50 (int)
p1 = Product(name="Laptop", price="19.99", stock="50")
print(p1.price) # 19.99
print(type(p1.price)) # <class 'float'>

# Invalid data
try:
    p2 = Product(name="Phone", price="not-a-number", stock=10)
except ValidationError as e:
    print(e.errors())
```

*Output of ValidationError:*

```json
[
  {
    "type": "float_parsing",
    "loc": ["price"],
    "msg": "Input should be a valid number, unable to parse string as a number",
    "input": "not-a-number"
  }
]
```

---

## 5. Field Constraints and Metadata

The `Field` function allows you to add constraints, descriptions, and alias names to your model attributes.

### Example: String and Number Constraints

```python
from pydantic import BaseModel, Field

class Account(BaseModel):
    username: str = Field(min_length=3, max_length=20, pattern=r'^[a-zA-Z0-9_]+$')
    age: int = Field(ge=18, le=120, description="User age, must be adult")
    
    # Aliases are useful when mapping between Python (snake_case) and JSON (camelCase/kebab-case)
    first_name: str = Field(alias="firstName")

# Creating account (Note the alias used during instantiation)
acc = Account(username="alice_123", age=25, firstName="Alice")
print(acc.first_name) # Alice
```

*Common Field constraints:*
*   **Strings:** `min_length`, `max_length`, `pattern` (regex).
*   **Numbers:** `gt` (greater than), `ge` (greater or equal), `lt` (less than), `le` (less or equal), `multiple_of`.

---

## 6. Nested Models

Pydantic seamlessly handles complex, deeply nested data structures by using models as types inside other models.

### Example: Address inside a User

```python
from typing import List
from pydantic import BaseModel

class Address(BaseModel):
    street: str
    city: str
    zip_code: str

class Customer(BaseModel):
    name: str
    # Nested model
    primary_address: Address
    # List of nested models
    secondary_addresses: List[Address] = []

data = {
    "name": "Jane Smith",
    "primary_address": {
        "street": "123 Main St",
        "city": "Springfield",
        "zip_code": "12345"
    }
}

customer = Customer(**data)
print(customer.primary_address.city) # Springfield
```

---

## 7. Custom Validators

When standard type hints and `Field` constraints aren't enough, you can write custom validation logic using `@field_validator` and `@model_validator`.

### Example: Field and Model Validators

```python
from pydantic import BaseModel, field_validator, model_validator, ValidationError

class Registration(BaseModel):
    username: str
    password: str
    confirm_password: str

    @field_validator('username')
    @classmethod
    def username_must_not_contain_space(cls, v: str) -> str:
        if ' ' in v:
            raise ValueError('Username cannot contain spaces')
        return v.lower() # Can also transform data

    @model_validator(mode='after')
    def check_passwords_match(self) -> 'Registration':
        if self.password != self.confirm_password:
            raise ValueError('Passwords do not match')
        return self

try:
    reg = Registration(username="bad name", password="123", confirm_password="456")
except ValidationError as e:
    print(e.errors())
```

*Note:* In Pydantic V2, `@validator` and `@root_validator` from V1 were replaced with `@field_validator` and `@model_validator`.

---

## 8. Serialization and Deserialization

Pydantic makes it easy to convert between Python dictionary/JSON and Model instances.

### Example: JSON and Dictionary Conversion (V2 syntax)

```python
from pydantic import BaseModel
import datetime

class Event(BaseModel):
    name: str
    timestamp: datetime.datetime

# 1. Deserialization (JSON string to Model)
json_data = '{"name": "Conference", "timestamp": "2026-10-15T14:30:00"}'
event = Event.model_validate_json(json_data)

# 2. Serialization (Model to Dict)
print(event.model_dump())
# {'name': 'Conference', 'timestamp': datetime.datetime(2026, 10, 15, 14, 30)}

# 3. Serialization (Model to JSON string)
print(event.model_dump_json())
# '{"name":"Conference","timestamp":"2026-10-15T14:30:00Z"}'
```

---

## 9. Model Configuration

You can configure model behavior (like stripping whitespace, allowing extra fields, or populating by alias) using `model_config`.

### Example: Allowing Extra Fields and Forbidding Mutations

```python
from pydantic import BaseModel, ConfigDict

class Settings(BaseModel):
    model_config = ConfigDict(
        extra='allow',        # Allows extra fields not defined in the model (default is 'ignore')
        frozen=True,          # Makes the model immutable (cannot reassign attributes)
        str_strip_whitespace=True # Automatically strips leading/trailing whitespaces from strings
    )
    
    app_name: str
    port: int

# Passing an extra field "environment"
settings = Settings(app_name="  MyApp  ", port=8080, environment="Production")

print(settings.app_name)        # "MyApp" (whitespace stripped)
print(settings.environment)     # "Production" (extra field allowed)

# settings.port = 9000          # Raises ValidationError because `frozen=True`
```

---

## 10. Optional Fields

To make a field optional in Pydantic, you must use a type hint like `Optional[T]` or `T | None` (Python 3.10+) **and** assign a default value, typically `None`. If you omit the default value, Pydantic will still require the field to be provided during instantiation (even if you provide `None` explicitly).

### Example: Using `Optional` and `Field`

```python
from typing import Optional
from pydantic import BaseModel, Field

class Profile(BaseModel):
    # Python 3.10+ syntax
    bio: str | None = None
    
    # Using typing.Optional (compatible with older Python versions)
    website: Optional[str] = None
    
    # Making a field optional while still using Field for metadata
    nickname: str | None = Field(default=None, max_length=50, description="Optional nickname")

# You can instantiate without providing optional fields
p1 = Profile()
print(p1.model_dump())
# Output: {'bio': None, 'website': None, 'nickname': None}

# You can also pass None explicitly, or provide valid data
p2 = Profile(bio="Python Dev", nickname=None)
```
