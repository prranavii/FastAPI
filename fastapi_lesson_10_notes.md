# FastAPI Lesson 10 — Dependency Injection and `Depends()`

## Overview

Dependency Injection is one of the most important FastAPI-specific concepts for interviews.

The main tool is:

```python
Depends()
```

At first, this can look confusing.

The simplest idea is:

> **Dependency Injection means letting FastAPI provide something that an endpoint needs instead of creating or repeating that thing manually inside every endpoint.**

We will start with simple examples and gradually connect the idea to real backend use cases such as authentication and database sessions.

---

# 1. The Problem Dependency Injection Solves

Imagine several endpoints need the same logic.

For example:

```text
GET /profile
GET /orders
GET /payments
```

All three endpoints may need to check:

```text
Is the user logged in?
```

Without dependency injection, we might repeat the same authentication code in every endpoint.

That becomes difficult to maintain.

Instead, we can create reusable logic once:

```text
Authentication dependency
        ↓
GET /profile
GET /orders
GET /payments
```

This is one of the main reasons `Depends()` exists.

---

# 2. What Is a Dependency?

A dependency is something an endpoint needs before it can perform its job.

Examples:

```text
Current user
Database session
Authentication check
Common query parameters
Permission check
Configuration
Reusable validation logic
```

Think:

```text
Endpoint
   |
   ├── needs current user
   ├── needs database
   └── needs permission check
```

Those needs can be provided as dependencies.

---

# 3. The Basic Syntax

FastAPI provides:

```python
from fastapi import Depends
```

Then:

```python
def get_user():
    return "Pranavi"


@app.get("/profile")
def profile(user = Depends(get_user)):
    return {
        "user": user
    }
```

The important part is:

```python
Depends(get_user)
```

It tells FastAPI:

> Before calling `profile()`, call `get_user()` and provide its result as `user`.

---

# 4. Understanding the Flow

Consider:

```python
def get_user():
    return "Pranavi"


@app.get("/profile")
def profile(user = Depends(get_user)):
    return {
        "user": user
    }
```

The request:

```text
GET /profile
```

flows like:

```text
Client
  ↓
GET /profile
  ↓
FastAPI
  ↓
get_user()
  ↓
"Pranavi"
  ↓
profile(user="Pranavi")
  ↓
Response
```

The endpoint doesn't manually call:

```python
get_user()
```

FastAPI handles it.

---

# 5. Why Is This Called Dependency Injection?

The endpoint depends on `get_user()`.

Instead of doing:

```python
def profile():
    user = get_user()
```

we declare:

```python
def profile(user = Depends(get_user)):
```

FastAPI injects the result.

Conceptually:

```text
Dependency
     ↓
FastAPI
     ↓
Endpoint
```

The endpoint receives what it needs.

---

# 6. A Simple Dependency with Query Parameters

Dependencies can also have their own parameters.

Example:

```python
from fastapi import FastAPI, Depends

app = FastAPI()


def common_parameters(
    skip: int = 0,
    limit: int = 10
):
    return {
        "skip": skip,
        "limit": limit
    }


@app.get("/items")
def get_items(
    params = Depends(common_parameters)
):
    return params
```

A request such as:

```text
GET /items?skip=20&limit=10
```

can result in:

```json
{
    "skip": 20,
    "limit": 10
}
```

FastAPI resolves the dependency and passes the result to the endpoint.

---

# 7. Why Not Just Use a Normal Function?

You might ask:

> Why not simply call `common_parameters()` inside the endpoint?

You could.

But FastAPI dependencies provide useful framework behavior.

They can:

- Be reused across routes
- Receive parameters automatically
- Be combined with other dependencies
- Work with authentication
- Work with database sessions
- Appear in generated API documentation
- Be overridden during testing

So `Depends()` is more than just calling a function.

---

# 8. Dependency for Authentication

This is one of the most important real-world examples.

Suppose we have:

```python
def get_current_user():
    # Check authentication token
    return {
        "id": 1,
        "name": "Pranavi"
    }
```

Then:

```python
@app.get("/profile")
def profile(
    current_user = Depends(get_current_user)
):
    return current_user
```

The endpoint depends on:

```python
get_current_user
```

The flow becomes:

```text
Request
   ↓
Authentication dependency
   ↓
Validate token
   ↓
Get current user
   ↓
Endpoint
   ↓
Response
```

Later, when we learn JWT authentication, this pattern will become very important.

---

# 9. Dependency for Database Sessions

Another extremely common use is a database session.

Conceptually:

```python
def get_db():
    db = create_database_session()

    try:
        yield db
    finally:
        db.close()
```

Then:

```python
@app.get("/users")
def get_users(db = Depends(get_db)):
    ...
```

FastAPI handles the dependency for the request.

The flow is:

```text
Request
   ↓
get_db()
   ↓
Database session
   ↓
Endpoint
   ↓
Response
   ↓
Close session
```

You don't need to understand `yield` deeply yet. The important point is that database session management is a common use of dependency injection.

---

# 10. Dependencies Can Be Shared

Suppose these endpoints all require the current user:

```text
GET /profile
GET /orders
GET /payments
```

Instead of repeating authentication code:

```text
GET /profile
   ↓
Depends(get_current_user)

GET /orders
   ↓
Depends(get_current_user)

GET /payments
   ↓
Depends(get_current_user)
```

The authentication logic lives in one reusable place.

If you later change how authentication works, you update the dependency rather than rewriting every endpoint.

---

# 11. Dependencies Can Depend on Other Dependencies

This sounds advanced, but the concept is simple.

For example:

```text
get_current_user
       ↓
get_admin_user
       ↓
admin endpoint
```

You might have:

```python
def get_current_user():
    ...


def get_admin_user(
    user = Depends(get_current_user)
):
    ...
```

Then:

```python
@app.get("/admin")
def admin_panel(
    admin = Depends(get_admin_user)
):
    ...
```

FastAPI can resolve the dependency chain.

Conceptually:

```text
admin endpoint
      ↓
get_admin_user
      ↓
get_current_user
      ↓
user
```

This is called a dependency graph.

---

# 12. Dependency Graph

Imagine:

```text
                Endpoint
                   |
                   ↓
          get_admin_user
                   |
                   ↓
          get_current_user
                   |
                   ↓
           authentication
```

FastAPI resolves these dependencies before executing the endpoint.

This is one reason dependency injection becomes powerful in larger applications.

---

# 13. Dependencies and Request Parameters

A dependency can use:

- Query parameters
- Path parameters
- Headers
- Cookies
- Other dependencies

For example:

```python
def pagination(
    page: int = 1,
    limit: int = 10
):
    return {
        "page": page,
        "limit": limit
    }
```

Then:

```python
@app.get("/products")
def products(
    pagination_data = Depends(pagination)
):
    return pagination_data
```

The dependency itself can receive request data.

---

# 14. `Depends()` Does Not Execute Immediately

This is important.

When you write:

```python
Depends(get_user)
```

you are not manually calling:

```python
get_user()
```

You are telling FastAPI:

> Use `get_user` as a dependency for this parameter.

FastAPI calls it when processing the request.

So:

```python
Depends(get_user)
```

is a declaration of a dependency.

---

# 15. Dependency vs Normal Function

Normal function:

```python
def calculate_total():
    return 100
```

Calling it manually:

```python
total = calculate_total()
```

Dependency:

```python
def get_total():
    return 100


@app.get("/total")
def total(
    value = Depends(get_total)
):
    return value
```

FastAPI manages the dependency.

---

# 16. Dependency Injection and Separation of Concerns

Dependency injection helps keep different responsibilities separate.

Instead of putting everything inside:

```python
@app.get("/profile")
def profile():
    # authenticate
    # connect database
    # check permissions
    # fetch user
    # return response
```

we can separate:

```text
Authentication
      ↓
Dependency

Database
      ↓
Dependency

Permission
      ↓
Dependency

Business logic
      ↓
Endpoint
```

This makes the code easier to maintain.

---

# 17. Common FastAPI Dependency Examples

You should know these examples for interviews:

### Authentication

```python
Depends(get_current_user)
```

### Database

```python
Depends(get_db)
```

### Pagination

```python
Depends(pagination)
```

### Permissions

```python
Depends(require_admin)
```

### Reusable request logic

```python
Depends(common_parameters)
```

---

# 18. `Depends()` with a Class

Dependencies don't have to be simple functions.

FastAPI can also work with classes.

For example:

```python
class Pagination:
    def __init__(
        self,
        page: int = 1,
        limit: int = 10
    ):
        self.page = page
        self.limit = limit
```

Then:

```python
@app.get("/items")
def get_items(
    pagination = Depends(Pagination)
):
    return {
        "page": pagination.page,
        "limit": pagination.limit
    }
```

You don't need to memorize this pattern yet.

Just know that dependencies can be functions or callable classes.

---

# 19. Dependency Overrides

One useful FastAPI feature is dependency overriding.

This is especially useful during testing.

Suppose production uses:

```python
get_db
```

A test can replace it with a fake database dependency.

Conceptually:

```text
Production
    ↓
Real database

Testing
    ↓
Fake/test database
```

This makes applications easier to test.

You do not need to memorize the exact testing syntax for a fundamentals interview.

The key idea is:

> FastAPI dependencies can be replaced during tests.

---

# 20. Common Beginner Mistakes

### Mistake 1 — Forgetting the import

```python
from fastapi import Depends
```

### Mistake 2 — Calling the dependency yourself

Usually, write:

```python
Depends(get_user)
```

not:

```python
Depends(get_user())
```

You pass the callable to `Depends`.

### Mistake 3 — Thinking dependencies are only for authentication

They are also useful for:

```text
Database sessions
Pagination
Permissions
Reusable validation
Configuration
Common parameters
```

---

# 21. Interview Question — What Is Dependency Injection?

A strong answer:

> Dependency Injection is a design pattern where an endpoint receives the resources or logic it depends on from an external mechanism rather than creating them directly. In FastAPI, `Depends()` is used to declare dependencies, and FastAPI resolves and injects them when processing a request.

---

# 22. Interview Question — What Is `Depends()`?

A good answer:

> `Depends()` is FastAPI's dependency declaration mechanism. It tells FastAPI that a route or another dependency requires a particular callable. FastAPI executes that dependency and injects its result into the parameter.

Example:

```python
def get_current_user():
    return {"id": 1}


@app.get("/profile")
def profile(
    user = Depends(get_current_user)
):
    return user
```

---

# 23. Why Use Dependency Injection?

Main benefits:

```text
Reusable logic
     ↓
Cleaner endpoints
     ↓
Separation of concerns
     ↓
Easier testing
     ↓
Easier maintenance
```

---

# 24. Mental Model

Think of a restaurant.

The endpoint is the chef.

The chef needs ingredients.

Instead of the chef going outside every time to find ingredients, someone provides them.

```text
Endpoint
   ↓
Needs user
   ↓
FastAPI
   ↓
Authentication dependency
   ↓
Provides user
   ↓
Endpoint continues
```

Dependency Injection is basically:

> "Give me what I need to do my job."

---

# 25. Mini Practice

Create:

```python
def get_current_user():
    return {
        "name": "Pranavi",
        "role": "student"
    }
```

Then create:

```python
@app.get("/profile")
def profile(user = Depends(get_current_user)):
    return user
```

Test:

```text
GET /profile
```

Expected response:

```json
{
    "name": "Pranavi",
    "role": "student"
}
```

Then modify the dependency to return:

```python
{
    "name": "Pranavi",
    "role": "admin"
}
```

Notice that the endpoint itself doesn't need to change.

That is one of the benefits of dependency injection.

---

# 26. Quick Quiz

1. What is a dependency?
2. What does `Depends()` do?
3. Why is dependency injection useful?
4. Give two real-world FastAPI dependency examples.
5. Why is authentication commonly implemented as a dependency?
6. Can dependencies depend on other dependencies?
7. Why are dependencies useful for testing?
8. What's the difference between `Depends(get_user)` and `Depends(get_user())`?

### Answers

1. Something an endpoint needs to perform its work.
2. It tells FastAPI to resolve and inject a dependency.
3. It promotes reusable logic, separation of concerns, cleaner code, and easier testing.
4. Database sessions and authentication.
5. Multiple protected endpoints can reuse the same authentication logic.
6. Yes.
7. Dependencies can be replaced or overridden in tests.
8. `Depends(get_user)` passes the callable to FastAPI; `get_user()` calls it immediately and is generally not the intended usage.

---

# 27. FastAPI Progress

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
```

Next:

```text
Lesson 11 → APIRouter + Project Structure
```

After Lesson 11, the interview-focused FastAPI fundamentals will be complete.
