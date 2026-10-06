# FastAPI Lesson 9 — Response Models

## Overview

In previous lessons, we learned how FastAPI receives and validates data from the client.

For example:

```python
class User(BaseModel):
    name: str
    email: str
    age: int


@app.post("/users")
def create_user(user: User):
    return user
```

Here:

```python
user: User
```

describes the **incoming request data**.

But FastAPI also needs a way to describe and validate the **data going out to the client**.

That is where **response models** come in.

The main concept of this lesson is:

```python
response_model=User
```

---

# 1. What Is a Response Model?

A response model tells FastAPI:

> "This is the structure of data that this endpoint should return."

For example:

```python
@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: int):
    ...
```

The important part is:

```python
response_model=User
```

It defines the expected structure of the response.

---

# 2. Request Model vs Response Model

This distinction is extremely important for interviews.

### Request model

Used for incoming data:

```python
def create_user(user: User):
```

The client sends data to the server.

```text
Client
   |
   | JSON
   ↓
FastAPI
   |
   ↓
User model
```

### Response model

Used for outgoing data:

```python
@app.get("/users", response_model=list[User])
```

The server sends data to the client.

```text
FastAPI
   |
   ↓
User model
   |
   | JSON
   ↓
Client
```

So:

```text
Request model
→ Validates/defines incoming data

Response model
→ Validates/filters/defines outgoing data
```

---

# 3. Basic Example

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    name: str
    email: str
    age: int


@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: int):

    return {
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    }
```

The response should follow:

```python
User
```

which means:

```text
name  → string
email → string
age   → integer
```

---

# 4. Why Do We Need Response Models?

Suppose our backend internally has:

```python
user = {
    "id": 1,
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "age": 21,
    "password": "secret123"
}
```

We should **not** send the password to the frontend.

If we simply return the entire dictionary:

```python
return user
```

we could accidentally expose sensitive information.

A response model allows us to define exactly what should be returned.

---

# 5. Request Model and Response Model Should Sometimes Be Different

Consider a user registration API.

The client may send:

```json
{
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "password": "secret123"
}
```

The request needs a password.

But the response should not contain the password.

So we can create two models.

```python
from pydantic import BaseModel


class UserCreate(BaseModel):
    name: str
    email: str
    password: str


class UserResponse(BaseModel):
    name: str
    email: str
```

Now:

```python
@app.post("/users", response_model=UserResponse)
def create_user(user: UserCreate):

    return {
        "name": user.name,
        "email": user.email,
        "password": user.password
    }
```

FastAPI will use the response model to produce the response structure.

The client receives:

```json
{
    "name": "Pranavi",
    "email": "pranavi@example.com"
}
```

The password is excluded from the response model.

This is an important real-world use case.

---

# 6. Why This Is Important for Security

A backend often contains internal fields that should never be returned.

Examples:

```text
password
password_hash
internal_notes
admin_metadata
database fields
private tokens
internal IDs
```

You don't want to accidentally expose all of these.

A response model creates a defined public API contract.

Think:

```text
Database object
       |
       ↓
Internal data
       |
       ↓
Response model
       |
       ↓
Safe public response
       |
       ↓
Client
```

---

# 7. `response_model=list[User]`

What if an endpoint returns multiple users?

We can write:

```python
@app.get("/users", response_model=list[User])
def get_users():
    return users
```

This means:

> The response should be a list containing User objects.

Example response:

```json
[
    {
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    },
    {
        "name": "Rahul",
        "email": "rahul@example.com",
        "age": 22
    }
]
```

The important part is:

```python
list[User]
```

which means:

```text
List
 ↓
User
 ↓
User
 ↓
User
```

---

# 8. Response Model with CRUD

Our previous CRUD API can be improved.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    name: str
    email: str
    age: int


users = []


@app.post("/users", response_model=User)
def create_user(user: User):

    new_user = {
        "name": user.name,
        "email": user.email,
        "age": user.age
    }

    users.append(new_user)

    return new_user


@app.get("/users", response_model=list[User])
def get_users():
    return users


@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: int):
    ...
```

Now the API clearly describes its responses.

---

# 9. Response Validation

Response models are not only documentation.

FastAPI uses them to validate and serialize the response according to the declared model.

Suppose:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

and:

```python
@app.get("/user", response_model=User)
def get_user():

    return {
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    }
```

This matches the response model.

But if the returned data does not match the declared response structure, FastAPI can raise a server-side response validation error.

Therefore:

```text
Request validation
→ Protects the application from bad incoming data

Response validation
→ Helps ensure the API returns the expected structure
```

---

# 10. Response Models and Swagger

Response models also improve automatically generated API documentation.

Open:

```text
http://127.0.0.1:8000/docs
```

FastAPI can show the expected response schema.

For example:

```text
GET /users
```

can show:

```text
Response:
[
    User
]
```

and the fields:

```text
name
email
age
```

This is one reason FastAPI's documentation is so useful.

---

# 11. Nested Response Models

Response models can also be nested.

```python
class Address(BaseModel):
    city: str
    pincode: int


class UserResponse(BaseModel):
    name: str
    email: str
    address: Address
```

Then:

```python
@app.get(
    "/users/{user_id}",
    response_model=UserResponse
)
def get_user(user_id: int):
    ...
```

A valid response could be:

```json
{
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "address": {
        "city": "Greater Noida",
        "pincode": 201310
    }
}
```

---

# 12. `response_model` vs Return Type

You may see both:

```python
@app.get("/users", response_model=list[User])
def get_users():
    ...
```

and:

```python
@app.get("/users")
def get_users() -> list[User]:
    ...
```

They are related but serve different purposes.

For FastAPI endpoint responses, `response_model` is especially useful because it explicitly tells FastAPI which model to use for response validation, serialization, filtering, and OpenAPI documentation.

For interview purposes, remember:

```python
response_model=User
```

is the standard FastAPI mechanism for declaring an endpoint's response schema.

---

# 13. Common Interview Question

### What is `response_model` in FastAPI?

A good answer:

> `response_model` defines the expected structure of an endpoint's response. FastAPI uses it for response validation/serialization and for generating OpenAPI documentation. It can also prevent fields not included in the response model from being exposed.

---

# 14. Request Model vs Response Model — Interview Answer

> A request model defines the data the client is allowed or expected to send to the API. A response model defines the structure of data the API sends back to the client. They can be different because the request may contain sensitive or internal input fields that should not appear in the response.

Example:

```python
class UserCreate(BaseModel):
    name: str
    email: str
    password: str


class UserResponse(BaseModel):
    name: str
    email: str
```

---

# 15. Key Takeaways

Remember:

```text
Request model
→ Incoming data

Response model
→ Outgoing data
```

Main syntax:

```python
@app.get(
    "/users",
    response_model=list[User]
)
def get_users():
    return users
```

For a single object:

```python
@app.get(
    "/users/{user_id}",
    response_model=User
)
def get_user(user_id: int):
    ...
```

For security:

```text
Internal object
   ↓
Response model
   ↓
Safe public response
```

---

# 16. Practice

Create:

```python
class UserCreate(BaseModel):
    name: str
    email: str
    password: str


class UserResponse(BaseModel):
    name: str
    email: str
```

Then build:

```text
POST /users
GET /users
```

Make sure the password is accepted during creation but is not included in the response.

---

# 17. Quick Quiz

1. What is a response model?
2. What does `response_model=User` mean?
3. Why might request and response models be different?
4. What does `list[User]` mean?
5. How can response models help prevent accidentally exposing passwords?
6. Where can you see response schemas automatically in FastAPI?
7. What is the difference between request validation and response validation?

### Answers

1. It defines the expected structure of outgoing API data.
2. The endpoint should return data matching the `User` schema.
3. The request may contain fields that should not be returned, such as passwords.
4. A list containing `User` objects.
5. Fields absent from the response model are not part of the public response schema.
6. Swagger/OpenAPI documentation at `/docs`.
7. Request validation checks incoming client data; response validation/serialization checks the outgoing API data.

---

# 18. FastAPI Progress

```text
Lesson 1 → API Fundamentals
Lesson 2 → HTTP + Client + Server
Lesson 3 → First FastAPI App
Lesson 4 → Receiving Data
Lesson 5 → Path + Query Parameters
Lesson 6 → Pydantic Models
Lesson 7 → CRUD
Lesson 8 → Status Codes + Error Handling
Lesson 9 → Response Models
```

Next:

```text
Lesson 10 → Dependency Injection
Lesson 11 → APIRouter + Project Structure
```

After Lesson 11, the interview-focused FastAPI fundamentals will be complete.
