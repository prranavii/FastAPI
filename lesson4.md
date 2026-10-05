# FastAPI Learning — Lesson 4
## Receiving Data from the Client

> **Goal:** Understand how a FastAPI application receives information from a client.

In Lesson 3, our API always returned the same response:

```json
{
  "message": "Hello, FastAPI!"
}
```

Real APIs need to receive different data from clients. In this lesson, we learn three important ways:

1. Query parameters
2. Path parameters
3. Request bodies using Pydantic models

---

# 1. What is a client?

A **client** is whoever is using your API.

A client could be:

- A website
- A mobile application
- Another backend
- Postman
- A React frontend
- An AI agent
- A Python program

For example:

```text
React Frontend
      |
      | API Request
      ↓
FastAPI Backend
      |
      | Response
      ↓
React Frontend
```

---

# 2. Query Parameters

A query parameter allows the client to send information through the URL.

Example:

```text
http://127.0.0.1:8000/greet?name=Pranavi
```

Here:

```text
/greet
```

is the endpoint path.

And:

```text
?name=Pranavi
```

is the query parameter.

Breakdown:

```text
?       name       =       Pranavi
↓         ↓        ↓          ↓
start   parameter  equals    value
```

---

# 3. Creating a Query Parameter in FastAPI

Add this to `main.py`:

```python
@app.get("/greet")
def greet(name: str):
    return {"message": f"Hello, {name}!"}
```

`name: str` means the function expects a parameter called `name`, and we expect it to be a string.

---

# 4. How FastAPI receives the value

If the client sends:

```text
/greet?name=Pranavi
```

FastAPI receives:

```text
name = "Pranavi"
```

The function returns:

```json
{
  "message": "Hello, Pranavi!"
}
```

---

# 5. Test the Query Parameter

Start the server:

```bash
uvicorn main:app --reload
```

Open:

```text
http://127.0.0.1:8000/greet?name=Pranavi
```

Expected response:

```json
{
  "message": "Hello, Pranavi!"
}
```

Change `Pranavi` to another name. The response changes without changing the Python code.

---

# 6. Why do we write `: str`?

`str` means string.

Common Python types include:

```python
name: str
age: int
price: float
is_student: bool
```

FastAPI uses these type hints to understand what type of data an endpoint expects.

---

# 7. Required Query Parameters

Consider:

```python
@app.get("/greet")
def greet(name: str):
    return {"message": f"Hello, {name}!"}
```

Because `name` has no default value, it is required.

If the client opens:

```text
http://127.0.0.1:8000/greet
```

the `name` value is missing, so FastAPI returns a validation error.

---

# 8. Multiple Query Parameters

An endpoint can accept multiple query parameters:

```python
@app.get("/profile")
def profile(name: str, age: int):
    return {
        "name": name,
        "age": age
    }
```

Request:

```text
/profile?name=Pranavi&age=21
```

The `&` separates multiple query parameters.

Response:

```json
{
  "name": "Pranavi",
  "age": 21
}
```

---

# 9. Path Parameters

A **path parameter** is part of the URL path.

Example:

```text
/users/25
```

FastAPI code:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }
```

The `{user_id}` tells FastAPI that a value will appear at that position.

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

---

# 10. Path Parameter vs Query Parameter

### Path parameter

```text
/users/25
```

Usually used when identifying a specific resource.

Examples:

```text
/products/100
/orders/567
/books/42
/users/25
```

Think:

> "Give me this particular resource."

### Query parameter

```text
/products?category=shoes
```

Usually useful for filtering, searching, sorting, or providing additional options.

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

# 11. Request Body

Sometimes we need to send larger structured data.

For example:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

This can be sent in the **request body**.

Request bodies are commonly used with POST requests.

---

# 12. Creating a POST Endpoint

```python
@app.post("/users")
def create_user():
    return {
        "message": "User created"
    }
```

`@app.post("/users")` tells FastAPI that this endpoint handles POST requests sent to `/users`.

---

# 13. What is Pydantic?

FastAPI commonly uses **Pydantic** to define and validate structured data.

Import `BaseModel`:

```python
from pydantic import BaseModel
```

Create a model:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

Think of this as a template for the data that the API expects.

Expected JSON:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

The model says:

- `name` must be a string.
- `email` must be a string.
- `age` must be an integer.

The model does not automatically save anything to a database. It describes and validates incoming data.

---

# 14. Use the Pydantic Model in the API

```python
@app.post("/users")
def create_user(user: User):
    return {
        "message": "User received",
        "name": user.name,
        "email": user.email,
        "age": user.age
    }
```

The important part is:

```python
user: User
```

This tells FastAPI that the request body should follow the `User` model.

FastAPI and Pydantic work together to parse and validate the incoming data.

---

# 15. Complete Example

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


@app.get("/")
def home():
    return {"message": "Hello, FastAPI!"}


@app.get("/greet")
def greet(name: str):
    return {"message": f"Hello, {name}!"}


@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }


class User(BaseModel):
    name: str
    email: str
    age: int


@app.post("/users")
def create_user(user: User):
    return {
        "message": "User received",
        "name": user.name,
        "email": user.email,
        "age": user.age
    }
```

---

# 16. Test the API with `/docs`

Run:

```bash
uvicorn main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

You should see:

```text
GET  /
GET  /greet
GET  /users/{user_id}
POST /users
```

FastAPI automatically creates interactive API documentation.

---

# 17. Test `GET /greet`

Open:

```text
GET /greet
```

Click **Try it out**.

Enter:

```text
Pranavi
```

Click **Execute**.

Response:

```json
{
  "message": "Hello, Pranavi!"
}
```

---

# 18. Test `GET /users/{user_id}`

Open:

```text
GET /users/{user_id}
```

Click **Try it out**.

Enter:

```text
25
```

Click **Execute**.

Response:

```json
{
  "user_id": 25
}
```

---

# 19. Test `POST /users`

Open:

```text
POST /users
```

Click **Try it out**.

Enter:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

Click **Execute**.

Response:

```json
{
  "message": "User received",
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

Try removing `email`. FastAPI should return a validation error because `email` is required by the `User` model.

---

# 20. The Big Picture

A client can send information to FastAPI in different ways:

```text
                    CLIENT
                       |
          +------------+------------+
          |            |            |
          ↓            ↓            ↓
       Query         Path        Request Body
      Parameter     Parameter
          |            |            |
          ↓            ↓            ↓
 ?name=Pranavi     /users/25    JSON data
          |            |            |
          +------------+------------+
                       |
                       ↓
                    FASTAPI
                       |
                       ↓
                 Your function
                       |
                       ↓
                    Response
```

---

# 21. Quick Comparison

| Type | Example | Common purpose |
|---|---|---|
| Query parameter | `/users?city=Delhi` | Filtering/search/options |
| Path parameter | `/users/25` | Identifying a specific resource |
| Request body | JSON object | Sending structured data |
| Pydantic model | `class User(BaseModel)` | Defining/validating structured data |

---

# 22. Important Things to Remember

### Query parameter

```text
/greet?name=Pranavi
```

Data comes through the URL after `?`.

### Path parameter

```text
/users/25
```

Data is part of the URL path.

### Request body

```json
{
  "name": "Pranavi",
  "age": 21
}
```

Data is sent inside the HTTP request body.

### Pydantic model

```python
class User(BaseModel):
    name: str
    age: int
```

Defines the expected structure and types of incoming data.

### GET

Generally used to retrieve information.

### POST

Generally used to submit data or create something.

---

# 23. Mini Quiz

Try answering these without looking back.

### Question 1

What is the difference between:

```text
/users/10
```

and:

```text
/users?id=10
```

### Question 2

What does this mean?

```python
name: str
```

### Question 3

Why do we use `BaseModel`?

### Question 4

Where is this data being sent?

```json
{
  "name": "Pranavi",
  "age": 21
}
```

### Question 5

Which is more appropriate for retrieving a specific user?

```text
/users/25
```

or:

```text
/users?city=Delhi
```

### Question 6

Why do we commonly use POST when sending structured data to create something?

---

# Key Takeaway

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

- Query parameters
- Path parameters
- Request bodies
- Pydantic models
