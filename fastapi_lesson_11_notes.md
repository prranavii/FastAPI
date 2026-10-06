# FastAPI Lesson 11 — APIRouter and Project Structure

## Overview

So far, most of our examples have looked like this:

```text
main.py
```

with all endpoints inside one file.

That is fine for learning.

But imagine an actual application with:

```text
Users
Products
Orders
Authentication
Payments
AI
Admin
```

Putting every endpoint into one `main.py` would become difficult to maintain.

This lesson introduces:

- `APIRouter`
- Route organization
- Modular FastAPI applications
- Basic project structure
- Separation of concerns
- How `include_router()` works

After this lesson, you should understand how a real FastAPI project is organized.

---

# 1. The Problem With One Huge File

Imagine:

```python
app = FastAPI()


@app.get("/users")
def get_users():
    ...


@app.post("/users")
def create_user():
    ...


@app.get("/products")
def get_products():
    ...


@app.post("/products")
def create_product():
    ...


@app.get("/orders")
def get_orders():
    ...


@app.post("/orders")
def create_order():
    ...


@app.post("/login")
def login():
    ...
```

This might work.

But as the project grows, `main.py` becomes huge.

You might eventually have:

```text
500 lines
1000 lines
2000 lines
```

in one file.

That is difficult to understand and maintain.

---

# 2. What Is APIRouter?

FastAPI provides:

```python
APIRouter
```

An `APIRouter` lets us group related routes together.

For example:

```text
User routes
    ↓
users.py

Product routes
    ↓
products.py

Authentication routes
    ↓
auth.py
```

Then the main application combines them.

Conceptually:

```text
                main.py
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     users.py   products.py  auth.py
        |          |          |
      users     products     auth
      routes     routes      routes
```

---

# 3. Basic APIRouter Example

First import:

```python
from fastapi import APIRouter
```

Then:

```python
router = APIRouter()
```

Now instead of:

```python
@app.get("/users")
```

we use:

```python
@router.get("/users")
```

Example:

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/users")
def get_users():
    return {
        "message": "All users"
    }
```

The router contains the route.

But the main FastAPI application still needs to include it.

---

# 4. `include_router()`

In `main.py`:

```python
from fastapi import FastAPI
from users import router

app = FastAPI()

app.include_router(router)
```

Now the routes inside `users.py` become part of the main application.

The architecture is:

```text
main.py
   |
   ↓
include_router()
   |
   ↓
users router
   |
   ↓
/users
```

---

# 5. Project Structure

A simple project can look like:

```text
fastapi-project/
│
├── app/
│   ├── main.py
│   │
│   └── routers/
│       ├── users.py
│       ├── products.py
│       └── auth.py
│
└── .venv/
```

This is already much better than putting every route in one file.

---

# 6. Creating `users.py`

Inside:

```text
app/routers/users.py
```

write:

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/users")
def get_users():
    return {
        "message": "All users"
    }


@router.get("/users/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }
```

Now the user-related routes are together.

---

# 7. Creating `main.py`

Then:

```python
from fastapi import FastAPI
from routers.users import router as users_router

app = FastAPI()

app.include_router(users_router)
```

Now FastAPI knows about the routes in `users.py`.

---

# 8. Why Rename the Router?

You might see:

```python
from routers.users import router as users_router
```

The:

```python
as users_router
```

part gives the imported router a clearer name.

Then:

```python
app.include_router(users_router)
```

is easy to understand.

You could also write:

```python
from routers.users import router

app.include_router(router)
```

but as your application grows, descriptive names can help.

---

# 9. Prefixes

Suppose all user routes start with:

```text
/users
```

Instead of repeating it:

```python
@router.get("/users")
@router.get("/users/{user_id}")
@router.post("/users")
```

we can define a prefix:

```python
router = APIRouter(
    prefix="/users"
)
```

Then:

```python
@router.get("/")
def get_users():
    ...


@router.get("/{user_id}")
def get_user(user_id: int):
    ...


@router.post("/")
def create_user():
    ...
```

The final routes become:

```text
GET  /users/
GET  /users/{user_id}
POST /users/
```

The prefix is automatically added.

---

# 10. Tags

We can also add tags:

```python
router = APIRouter(
    prefix="/users",
    tags=["Users"]
)
```

Example:

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["Users"]
)


@router.get("/")
def get_users():
    return []
```

Tags help organize Swagger/OpenAPI documentation.

In `/docs`, routes can be grouped under:

```text
Users
```

This is very useful in larger APIs.

---

# 11. Complete Users Router

A clean `users.py` might look like:

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["Users"]
)


@router.get("/")
def get_users():
    return {
        "message": "All users"
    }


@router.get("/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }


@router.post("/")
def create_user():
    return {
        "message": "User created"
    }
```

---

# 12. Main Application

Then `main.py`:

```python
from fastapi import FastAPI
from routers.users import router as users_router

app = FastAPI()

app.include_router(users_router)
```

That's the basic pattern.

---

# 13. What `include_router()` Does

This:

```python
app.include_router(users_router)
```

means:

> Add all routes defined inside `users_router` to the main FastAPI application.

Think:

```text
users.py
    |
    ↓
APIRouter
    |
    ↓
include_router()
    |
    ↓
FastAPI application
```

---

# 14. Multiple Routers

Suppose we have:

```text
routers/
├── users.py
├── products.py
└── auth.py
```

Then `main.py` could be:

```python
from fastapi import FastAPI

from routers.users import router as users_router
from routers.products import router as products_router
from routers.auth import router as auth_router


app = FastAPI()

app.include_router(users_router)
app.include_router(products_router)
app.include_router(auth_router)
```

Now the application contains all three groups of routes.

---

# 15. Better Project Structure

As the application becomes more serious, you might see:

```text
app/
│
├── main.py
│
├── routers/
│   ├── users.py
│   ├── products.py
│   └── auth.py
│
├── schemas/
│   ├── user.py
│   └── product.py
│
├── models/
│   ├── user.py
│   └── product.py
│
├── services/
│   ├── user_service.py
│   └── product_service.py
│
├── dependencies/
│   └── auth.py
│
└── database/
    └── connection.py
```

You don't need to memorize every folder.

The important idea is **separation of responsibilities**.

---

# 16. What Each Folder Can Mean

## `main.py`

Application entry point.

Usually contains:

```python
app = FastAPI()
```

and router registration.

---

## `routers/`

Contains API routes/endpoints.

Examples:

```text
users.py
products.py
auth.py
```

---

## `schemas/`

Contains Pydantic models used for API input/output.

Example:

```python
class UserCreate(BaseModel):
    name: str
    email: str
```

---

## `models/`

Often contains database models.

For example, with SQLAlchemy:

```text
User
Product
Order
```

These represent database tables/entities.

---

## `services/`

Contains business logic.

For example:

```text
Create user
Calculate order total
Call an external AI API
Process a payment
```

---

## `dependencies/`

Contains reusable FastAPI dependencies.

Examples:

```text
Authentication
Current user
Database session
Permissions
```

---

## `database/`

Contains database connection/configuration logic.

---

# 17. Why Separate Schemas and Models?

This is an important backend concept.

A Pydantic schema might be:

```python
class UserCreate(BaseModel):
    name: str
    email: str
```

A database model might represent:

```text
users table
```

These are not necessarily the same thing.

Think:

```text
Client/API data
      ↓
Pydantic schema
      ↓
Business logic
      ↓
Database model
      ↓
Database
```

Keeping these concepts separate helps maintain clean architecture.

---

# 18. Separation of Concerns

This phrase is important in interviews.

**Separation of concerns** means:

> Different parts of the application should have different responsibilities.

For example:

```text
Router
→ Handles HTTP requests/routes

Schema
→ Defines API data structure

Service
→ Contains business logic

Model
→ Represents database data

Database
→ Handles persistence
```

Instead of putting everything into one function.

---

# 19. Bad vs Better Architecture

### Everything in one file

```text
main.py
 ├── routes
 ├── validation
 ├── database
 ├── authentication
 ├── business logic
 └── external APIs
```

This becomes difficult to maintain.

### Better

```text
main.py
   ↓
routers
   ↓
services
   ↓
database
```

with:

```text
schemas
dependencies
models
```

organized separately.

---

# 20. APIRouter and AI Backends

This is particularly useful for your AI/backend goals.

Imagine an AI application:

```text
app/
├── main.py
├── routers/
│   ├── chat.py
│   ├── agents.py
│   └── auth.py
├── schemas/
│   ├── chat.py
│   └── agent.py
├── services/
│   ├── llm_service.py
│   ├── rag_service.py
│   └── agent_service.py
└── dependencies/
    └── auth.py
```

Then:

```text
POST /chat
POST /agents
POST /auth/login
```

can each have their own router.

This is a realistic pattern for AI backend applications.

---

# 21. Full Mental Model

Think of a FastAPI project as a company.

```text
main.py
→ Company headquarters

routers
→ Reception / API entry points

schemas
→ Forms and data contracts

services
→ Employees doing business logic

models
→ Representation of stored data

database
→ Storage room

dependencies
→ Shared support systems
```

Each part has a responsibility.

---

# 22. Common Beginner Mistakes

### Mistake 1 — Forgetting `include_router()`

Creating:

```python
router = APIRouter()
```

is not enough.

You must connect it:

```python
app.include_router(router)
```

---

### Mistake 2 — Confusing `APIRouter` with `FastAPI`

They are related but different.

```python
app = FastAPI()
```

creates the main application.

```python
router = APIRouter()
```

creates a group of routes.

---

### Mistake 3 — Mixing everything into routers

A router should generally focus on handling API routes.

Don't put huge amounts of business logic directly into every route function.

Prefer:

```text
Router
   ↓
Service
```

---

### Mistake 4 — Creating too many folders too early

Project structure should help clarity.

Don't create dozens of folders for a tiny application.

Start simple and organize as complexity grows.

---

# 23. Interview Question — What Is APIRouter?

A strong answer:

> `APIRouter` is FastAPI's mechanism for grouping related routes into separate modules. It helps organize larger applications by separating endpoints such as users, products, authentication, or orders. These routers are registered with the main application using `include_router()`.

---

# 24. Interview Question — Why Use APIRouter?

Good answer:

> `APIRouter` improves modularity and maintainability. Instead of placing every endpoint in `main.py`, related endpoints can be grouped into separate router modules.

---

# 25. Interview Question — What Does `include_router()` Do?

Good answer:

> `include_router()` adds the routes defined in an `APIRouter` to the main FastAPI application.

Example:

```python
app.include_router(users_router)
```

---

# 26. Interview Question — What Is Separation of Concerns?

Good answer:

> Separation of concerns means organizing the application so that different components have different responsibilities. In a FastAPI project, routers can handle HTTP routes, schemas can define API data, services can contain business logic, and models can represent database entities.

---

# 27. Quick Quiz

1. What problem does `APIRouter` solve?
2. What is the difference between `FastAPI()` and `APIRouter()`?
3. What does `include_router()` do?
4. What is a router prefix?
5. What are tags used for?
6. What might go inside a `routers/` directory?
7. What is the purpose of a `schemas/` directory?
8. What is the purpose of a `services/` directory?
9. What is separation of concerns?
10. Why shouldn't a large application put everything in `main.py`?

### Answers

1. It groups related routes and makes large applications modular.
2. `FastAPI()` creates the main application; `APIRouter()` creates a group of routes.
3. It registers the router's routes with the FastAPI application.
4. A common path automatically added to routes in a router.
5. They organize API endpoints in Swagger/OpenAPI documentation.
6. API endpoints/routes.
7. Pydantic request and response models.
8. Business logic and reusable application operations.
9. Giving different parts of the application different responsibilities.
10. It becomes difficult to understand, test, maintain, and extend.

---

# 28. A Small Realistic Example

Project:

```text
app/
│
├── main.py
│
└── routers/
    └── users.py
```

### `routers/users.py`

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["Users"]
)


@router.get("/")
def get_users():
    return []


@router.get("/{user_id}")
def get_user(user_id: int):
    return {
        "user_id": user_id
    }
```

### `main.py`

```python
from fastapi import FastAPI
from routers.users import router as users_router

app = FastAPI()

app.include_router(users_router)
```

Now the API contains:

```text
GET /users/
GET /users/{user_id}
```

---

# 29. FastAPI Progress — Fundamentals Complete

You have now covered:

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
Lesson 10 → Dependency Injection
Lesson 11 → APIRouter + Project Structure
```

## 🎯 FastAPI Interview Fundamentals: COMPLETE

At this point, you have enough FastAPI fundamentals to move from learning concepts to interview practice.

You should now be able to explain:

```text
Client
  ↓
HTTP request
  ↓
FastAPI route
  ↓
Parameters / body
  ↓
Pydantic validation
  ↓
Dependency injection
  ↓
Business logic
  ↓
Response model
  ↓
HTTP response
```

---

# 30. What You Do NOT Need to Learn Right Now

For your current interview preparation, you do not need to immediately go deep into:

```text
WebSockets
Advanced middleware
Background task architecture
Advanced async patterns
Custom exception handlers
Advanced deployment
Distributed systems
Advanced database optimization
```

These can be learned when a specific project or job requires them.

Your next step should be **FastAPI interview questions and practical implementation**, not endlessly adding more FastAPI topics.
