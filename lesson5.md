# FastAPI Learning — Lesson 5
## Path Parameters vs Query Parameters

### Goal
Understand path parameters, query parameters, optional query parameters, multiple parameters, and FastAPI validation.

---

## 1. Path Parameters

A path parameter is part of the URL structure.

```text
/users/25
```

FastAPI:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }
```

Request:

```text
/users/25
```

Response:

```json
{
  "user_id": 25
}
```

Use path parameters when identifying a specific resource.

Examples:

```text
/users/10
/products/500
/orders/234
/videos/91
```

Think:

> "Give me this particular resource."

---

## 2. Query Parameters

A query parameter is additional information sent through the URL after `?`.

Example:

```text
/users?city=Delhi
```

Here:

```text
/users
```

is the path and:

```text
?city=Delhi
```

is the query parameter.

Query parameters are commonly used for filtering, searching, sorting, pagination, and optional settings.

Examples:

```text
/products?category=shoes
/products?max_price=5000
/users?city=Delhi
/search?query=fastapi
```

Think:

> "Give me resources matching these conditions."

---

## 3. Path vs Query

### Path parameter

```text
/users/25
```

Think:

> "Which one?"

### Query parameter

```text
/users?city=Delhi
```

Think:

> "What conditions?"

This is a useful general rule for learning, although real API designs can vary.

---

## 4. Multiple Query Parameters

```text
/users?city=Delhi&age=21
```

`&` separates query parameters.

FastAPI:

```python
@app.get("/users")
def get_users(city: str, age: int):
    return {
        "city": city,
        "age": age
    }
```

Response:

```json
{
  "city": "Delhi",
  "age": 21
}
```

---

## 5. Optional Query Parameters

Sometimes a query parameter should not be required.

```python
@app.get("/users")
def get_users(city: str | None = None):
    return {
        "city": city
    }
```

```python
city: str | None = None
```

means:

- `city` can be a string.
- `city` can also be `None`.
- `None` is the default value.
- Therefore, `city` is optional.

With the parameter:

```text
/users?city=Delhi
```

Response:

```json
{
  "city": "Delhi"
}
```

Without it:

```text
/users
```

Response:

```json
{
  "city": null
}
```

---

## 6. Multiple Optional Parameters

Example:

```python
@app.get("/products")
def search_products(
    category: str | None = None,
    max_price: float | None = None
):
    return {
        "category": category,
        "max_price": max_price
    }
```

Requests:

```text
/products
```

```text
/products?category=laptop
```

```text
/products?category=laptop&max_price=50000
```

The same endpoint handles all three.

---

## 7. Path and Query Parameters Together

```python
@app.get("/users/{user_id}")
def get_user(
    user_id: int,
    details: bool = False
):
    return {
        "user_id": user_id,
        "details": details
    }
```

Request:

```text
/users/25
```

means:

```text
user_id = 25
details = False
```

Request:

```text
/users/25?details=true
```

means:

```text
user_id = 25
details = true
```

The path identifies the resource and the query parameter controls an option.

---

## 8. How FastAPI Knows Which Is Which

If a parameter appears inside `{}` in the route:

```python
@app.get("/users/{user_id}")
```

it is a path parameter.

If it is a normal function parameter and isn't part of the path:

```python
def get_user(user_id: int, details: bool = False):
```

FastAPI treats `details` as a query parameter.

Therefore:

```text
/users/25?details=true
```

means:

```text
Path:
user_id = 25

Query:
details = true
```

---

## 9. Type Validation

FastAPI validates incoming data according to type hints.

Example:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }
```

This works:

```text
/users/25
```

But:

```text
/users/hello
```

is invalid because `"hello"` is not an integer.

FastAPI returns a validation error.

Think:

```text
Client
   ↓
Incoming data
   ↓
FastAPI validation
   ↓
Valid?
   ↓
Your function
```

---

## 10. Complete Example

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def home():
    return {
        "message": "FastAPI is working!"
    }


@app.get("/users/{user_id}")
def get_user(
    user_id: int,
    details: bool = False
):
    return {
        "user_id": user_id,
        "details": details
    }


@app.get("/products")
def search_products(
    category: str | None = None,
    max_price: float | None = None
):
    return {
        "category": category,
        "max_price": max_price
    }
```

Run:

```bash
uvicorn main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

---

## 11. Test the API

### Test 1

```text
/users/25
```

Expected:

```json
{
  "user_id": 25,
  "details": false
}
```

### Test 2

```text
/users/25?details=true
```

Expected:

```json
{
  "user_id": 25,
  "details": true
}
```

### Test 3

```text
/products
```

Expected:

```json
{
  "category": null,
  "max_price": null
}
```

### Test 4

```text
/products?category=laptop
```

Expected:

```json
{
  "category": "laptop",
  "max_price": null
}
```

### Test 5

```text
/products?category=laptop&max_price=50000
```

Expected:

```json
{
  "category": "laptop",
  "max_price": 50000
}
```

---

## 12. API Request Flow

For:

```text
/products?category=laptop&max_price=50000
```

Think:

```text
Browser / Frontend
        |
        | GET request
        ↓
FastAPI
        |
        | category = laptop
        | max_price = 50000
        ↓
search_products()
        |
        ↓
Response
```

---

## 13. Quick Comparison

| Type | Example | Common purpose |
|---|---|---|
| Path parameter | `/users/25` | Identify a specific resource |
| Query parameter | `/users?city=Delhi` | Filtering/search/options |
| Optional query parameter | `/users?city=Delhi` or `/users` | Optional filters/settings |
| Type hint | `user_id: int` | Tell FastAPI expected data type |

---

## 14. Important Rules

### Path parameter

```text
/users/25
```

The value is part of the URL path.

### Query parameter

```text
/users?city=Delhi
```

The value comes after `?`.

### Multiple query parameters

```text
/users?city=Delhi&age=21
```

Use `&` to separate parameters.

### Optional query parameter

```python
city: str | None = None
```

The client does not have to provide it.

### Type validation

```python
user_id: int
```

FastAPI expects an integer.

---

## 15. Mini Quiz

### Question 1

What is the difference between:

```text
/products/100
```

and:

```text
/products?id=100
```

### Question 2

What does this mean?

```python
city: str | None = None
```

### Question 3

In:

```text
/users/25?details=true
```

which value is the path parameter?

### Question 4

Which part is the query parameter?

```text
/users/25?details=true
```

### Question 5

Why would we make a query parameter optional?

### Question 6

What happens if this endpoint expects:

```python
user_id: int
```

but the client sends:

```text
/users/hello
```

### Question 7 — Challenge

Write an endpoint that handles:

```text
/search?keyword=fastapi&limit=10
```

and returns:

```json
{
  "keyword": "fastapi",
  "limit": 10
}
```

---

## Key Takeaway

The basic request flow is:

```text
Client
   ↓
Sends data
   ↓
FastAPI
   ↓
Receives + validates data
   ↓
Your Python function
   ↓
Returns response
```

FastAPI gives us convenient ways to receive data through:

- Path parameters
- Query parameters
- Request bodies
- Pydantic models
