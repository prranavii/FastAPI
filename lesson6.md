# FastAPI Learning — Lesson 6
# Pydantic Models in FastAPI: Detailed Guide

> **Goal:** Understand Pydantic models deeply enough to confidently use them in FastAPI request bodies.

---

# Table of Contents

1. Why do we need Pydantic?
2. What is Pydantic?
3. What is a Pydantic model?
4. `BaseModel`
5. Your first Pydantic model
6. Connecting Pydantic with FastAPI
7. What happens when JSON reaches FastAPI?
8. Required fields
9. Type validation
10. Optional fields
11. Default values
12. `None` vs a default value
13. Accessing model fields
14. Dictionary vs Pydantic model
15. Nested Pydantic models
16. Pydantic models as contracts
17. Validation errors
18. Automatic API documentation
19. Pydantic does not store data
20. Complete example
21. Request flow
22. AI backend example
23. Common mistakes
24. Practice exercises
25. Lesson 6 quiz
26. Key takeaways

---

# 1. Why Do We Need Pydantic?

Suppose we are creating a user API.

The frontend sends:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

Our backend needs to know:

- Is `name` present?
- Is `name` a string?
- Is `email` present?
- Is `email` a string?
- Is `age` present?
- Is `age` an integer?
- What happens if `age` is missing?
- What happens if `age` contains invalid data?

Without a validation library, we might manually write checks:

```python
if "name" not in data:
    ...

if not isinstance(data["age"], int):
    ...

if "email" not in data:
    ...
```

For a large application, manually checking every field would become repetitive and error-prone.

Pydantic helps us solve this problem.

---

# 2. What Is Pydantic?

Pydantic is a Python library used to:

- Define the structure of data.
- Validate incoming data.
- Parse input data into structured Python objects.
- Provide useful validation errors.
- Work with Python type hints.

A simple way to think about Pydantic:

> **Pydantic is a data validation and data modeling tool.**

You tell Pydantic what your data should look like:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

Pydantic can then check incoming data against that structure.

---

# 3. What Is a Pydantic Model?

A Pydantic model is a Python class that inherits from `BaseModel`.

Example:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    email: str
    age: int
```

This model describes a `User`.

It says:

```text
User
├── name → string
├── email → string
└── age → integer
```

You can think of the model as a blueprint for the data.

---

# 4. What Is `BaseModel`?

We import it:

```python
from pydantic import BaseModel
```

Then:

```python
class User(BaseModel):
```

The `User` class inherits from Pydantic's `BaseModel`.

For now, you do not need to master Python inheritance.

The important idea is:

> Inheriting from `BaseModel` gives our class Pydantic's data validation and modeling behavior.

So:

```python
class User(BaseModel):
```

is not just an ordinary Python class.

It is a Pydantic data model.

---

# 5. Your First Pydantic Model

Create:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    email: str
    age: int
```

Let's understand every part.

## `class User`

```python
class User:
```

creates a Python class called `User`.

## `(BaseModel)`

```python
class User(BaseModel):
```

means the class inherits from Pydantic's `BaseModel`.

## `name: str`

```python
name: str
```

means:

> `name` is expected to be a string.

## `email: str`

```python
email: str
```

means:

> `email` is expected to be a string.

## `age: int`

```python
age: int
```

means:

> `age` is expected to be an integer.

---

# 6. Connecting Pydantic With FastAPI

Now let's connect our model to FastAPI.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    name: str
    email: str
    age: int


@app.post("/users")
def create_user(user: User):
    return {
        "name": user.name,
        "email": user.email,
        "age": user.age
    }
```

The important part is:

```python
def create_user(user: User):
```

This tells FastAPI:

> The request body should contain data matching the `User` Pydantic model.

---

# 7. What Does the Client Send?

The client can send JSON:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

The request is approximately:

```text
POST /users
Content-Type: application/json

{
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "age": 21
}
```

FastAPI receives the request.

Pydantic validates the body.

Then FastAPI passes the validated data to our function.

---

# 8. The Full Flow

The flow is:

```text
Frontend / Client
        |
        | JSON
        ↓
HTTP POST Request
        |
        ↓
FastAPI
        |
        ↓
Pydantic Validation
        |
        +-------- Invalid --------> Validation Error
        |
       Valid
        |
        ↓
Pydantic User Object
        |
        ↓
create_user()
        |
        ↓
Response
```

This is a very important backend concept.

---

# 9. Required Fields

Consider:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

These fields are required:

```text
name
email
age
```

If the client sends:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com"
}
```

then `age` is missing.

The request does not satisfy the model.

FastAPI will return a validation error.

---

# 10. Why Validation Is Useful

Suppose your backend expects:

```python
age: int
```

A client sends:

```json
{
  "age": "hello"
}
```

The API should not blindly assume the data is correct.

Pydantic checks the value against the declared type.

This protects your application logic from many forms of malformed input.

Think:

```text
Incoming data
      ↓
Validation
      ↓
Is it valid?
   ↙       ↘
 YES       NO
  ↓         ↓
Function   Error
```

---

# 11. Type Hints

Pydantic models heavily use Python type hints.

Examples:

```python
name: str
age: int
price: float
active: bool
```

Common Python types:

| Type | Example |
|---|---|
| `str` | `"Pranavi"` |
| `int` | `21` |
| `float` | `99.99` |
| `bool` | `True` |
| `list` | `["Python", "FastAPI"]` |
| `dict` | `{"city": "Delhi"}` |

The type declaration tells Pydantic what kind of data the field represents.

---

# 12. Optional Fields

Not every field has to be required.

Example:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
    phone: str | None = None
```

Now:

```text
name  → required
email → required
age   → required
phone → optional
```

The client can send:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

and it can still be valid.

If the client provides:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21,
  "phone": "9876543210"
}
```

that is also valid.

---

# 13. Understanding `str | None`

This:

```python
phone: str | None
```

means the value can be either:

```text
string
```

or:

```text
None
```

For example:

```python
phone = "9876543210"
```

or:

```python
phone = None
```

The `|` is Python's union type syntax.

So:

```python
str | None
```

means:

> string OR None.

---

# 14. Default Values

We can give a field a default value.

Example:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
    country: str = "India"
```

If the client sends:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

then the model can use:

```text
country = "India"
```

because the client didn't provide a country.

---

# 15. Optional vs Default Value

Compare:

```python
phone: str | None = None
```

and:

```python
country: str = "India"
```

### First

```python
phone: str | None = None
```

Means:

> `phone` can be a string or `None`, and if it isn't provided, use `None`.

### Second

```python
country: str = "India"
```

Means:

> `country` is a string, and if it isn't provided, use `"India"`.

The distinction becomes useful when designing precise API schemas.

---

# 16. Accessing Pydantic Fields

Suppose:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

and:

```python
def create_user(user: User):
```

We can access:

```python
user.name
```

```python
user.email
```

```python
user.age
```

For example:

```python
@app.post("/users")
def create_user(user: User):
    return {
        "name": user.name,
        "email": user.email,
        "age": user.age
    }
```

---

# 17. Dictionary vs Pydantic Model

This difference is important.

A normal Python dictionary:

```python
user = {
    "name": "Pranavi",
    "age": 21
}
```

Access values using:

```python
user["name"]
```

A Pydantic model:

```python
user.name
```

So:

```python
user["name"]
```

and:

```python
user.name
```

are different approaches.

When FastAPI gives your endpoint a Pydantic model, you normally access its fields using dot notation.

---

# 18. Pydantic Models Are Structured Objects

A model like:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

represents structured data.

Instead of thinking:

```text
Random JSON
```

think:

```text
User
├── name
├── email
└── age
```

The structure is explicit.

This makes backend code easier to understand.

---

# 19. Nested Pydantic Models

Real applications often contain nested objects.

For example:

```json
{
  "name": "Pranavi",
  "address": {
    "city": "Delhi",
    "pincode": 110001
  }
}
```

We can represent this with two models:

```python
class Address(BaseModel):
    city: str
    pincode: int


class User(BaseModel):
    name: str
    address: Address
```

Now the structure is:

```text
User
├── name
└── address
    ├── city
    └── pincode
```

The `address` field itself is another Pydantic model.

---

# 20. Using Nested Models in FastAPI

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Address(BaseModel):
    city: str
    pincode: int


class User(BaseModel):
    name: str
    address: Address


@app.post("/users")
def create_user(user: User):
    return {
        "name": user.name,
        "city": user.address.city,
        "pincode": user.address.pincode
    }
```

Request:

```json
{
  "name": "Pranavi",
  "address": {
    "city": "Delhi",
    "pincode": 110001
  }
}
```

Response:

```json
{
  "name": "Pranavi",
  "city": "Delhi",
  "pincode": 110001
}
```

---

# 21. Why Nested Models Matter

Imagine a real application.

A user might have:

```text
User
├── name
├── email
├── address
│   ├── city
│   ├── state
│   └── pincode
├── preferences
│   ├── language
│   └── notifications
└── profile
    ├── bio
    └── website
```

Instead of putting everything into one giant class, we can create separate models.

This makes large applications easier to maintain.

---

# 22. Pydantic as a Contract

A Pydantic model can act as a contract between frontend and backend.

Suppose we define:

```python
class ChatRequest(BaseModel):
    message: str
    temperature: float
```

The backend expects:

```json
{
  "message": "Explain RAG",
  "temperature": 0.7
}
```

The frontend knows what it should send.

The backend knows what it expects.

Think:

```text
Frontend
    |
    | JSON following the model
    ↓
FastAPI
    |
    | Validate
    ↓
Pydantic Model
    |
    ↓
Business Logic
```

This becomes especially important when multiple developers or services communicate with each other.

---

# 23. Validation Errors

Suppose the model is:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

And the client sends:

```json
{
  "name": "Pranavi"
}
```

The request is missing:

```text
email
age
```

FastAPI will return a validation error.

If the client sends:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": "hello"
}
```

then `age` has the wrong type.

FastAPI/Pydantic will report the invalid field.

---

# 24. FastAPI Automatically Documents the Model

One of FastAPI's most useful features is its automatic documentation.

When we write:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

FastAPI uses this information when generating its API schema.

Run:

```bash
uvicorn main:app --reload
```

Then open:

```text
http://127.0.0.1:8000/docs
```

For:

```text
POST /users
```

you will see an automatically generated request body schema.

Something conceptually like:

```json
{
  "name": "string",
  "email": "string",
  "age": 0
}
```

This is generated from your Pydantic model.

---

# 25. Why Automatic Documentation Is Useful

Imagine your frontend developer asks:

> What JSON should I send to `/users`?

Instead of manually explaining everything, the API documentation shows the expected schema.

The model:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

becomes part of the API's documentation.

This is one reason FastAPI is popular for backend development.

---

# 26. Pydantic Does NOT Store Data

This is extremely important.

Suppose you write:

```python
class User(BaseModel):
    name: str
    email: str
```

You have NOT created:

- A database table
- A permanent user record
- A database connection

Pydantic is primarily about:

```text
Data structure
+
Validation
+
Parsing
```

To permanently store the user, you need a database.

Eventually, our architecture might look like:

```text
Frontend
    ↓
FastAPI
    ↓
Pydantic
    ↓
Business Logic
    ↓
Database
```

We'll learn the database part later.

---

# 27. Complete Basic Example

Here is a clean FastAPI + Pydantic example:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    name: str
    email: str
    age: int
    phone: str | None = None


@app.post("/users")
def create_user(user: User):
    return {
        "name": user.name,
        "email": user.email,
        "age": user.age,
        "phone": user.phone
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

# 28. Test the Endpoint

Send:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

Expected response:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": null
}
```

Why is `phone` null?

Because:

```python
phone: str | None = None
```

makes it optional and gives it a default value of `None`.

---

# 29. Test With Phone

Now send:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21,
  "phone": "9876543210"
}
```

Response:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21,
  "phone": "9876543210"
}
```

---

# 30. AI Backend Example

Pydantic becomes particularly useful when building AI APIs.

Suppose you want to create a chatbot endpoint.

The client sends:

```json
{
  "message": "Explain RAG",
  "model": "llama",
  "temperature": 0.7
}
```

Create:

```python
class ChatRequest(BaseModel):
    message: str
    model: str
    temperature: float
```

Then:

```python
@app.post("/chat")
def chat(request: ChatRequest):
    return {
        "message": request.message,
        "model": request.model,
        "temperature": request.temperature
    }
```

Now your API has a clear contract.

---

# 31. Nested AI Request Example

A more realistic AI request could be:

```json
{
  "message": "Explain RAG",
  "model": "llama",
  "settings": {
    "temperature": 0.7,
    "max_tokens": 500
  }
}
```

Models:

```python
class GenerationSettings(BaseModel):
    temperature: float
    max_tokens: int


class ChatRequest(BaseModel):
    message: str
    model: str
    settings: GenerationSettings
```

This creates:

```text
ChatRequest
├── message
├── model
└── settings
    ├── temperature
    └── max_tokens
```

This kind of structured request will be very common when you start building AI backends.

---

# 32. The Complete Mental Model

When a frontend sends:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

think:

```text
                  CLIENT
                     |
                     | JSON
                     ↓
              HTTP POST Request
                     |
                     ↓
                  FASTAPI
                     |
                     ↓
                PYDANTIC
                     |
             Validate + Parse
                     |
              ┌──────┴──────┐
              ↓             ↓
            Valid         Invalid
              ↓             ↓
        User object       Error
              |
              ↓
       Your Python function
              |
              ↓
           Response
```

This flow is worth remembering.

---

# 33. Common Mistakes

## Mistake 1: Thinking Pydantic is a database

Incorrect:

> "I created a Pydantic model, so my user is stored."

Correct:

> Pydantic validates and structures data. A database stores data permanently.

---

## Mistake 2: Forgetting `BaseModel`

Incorrect:

```python
class User:
    name: str
```

Correct Pydantic model:

```python
class User(BaseModel):
    name: str
```

---

## Mistake 3: Forgetting the type

Instead of:

```python
class User(BaseModel):
    age
```

write:

```python
class User(BaseModel):
    age: int
```

The type tells Pydantic what kind of data the field represents.

---

## Mistake 4: Confusing request models with database models

A Pydantic model can describe data received by an API.

A database model represents how information is stored in a database.

They can be related, but they serve different purposes.

We'll explore this distinction when we reach databases.

---

# 34. Practice Exercise 1 — Product

Create a Pydantic model:

```python
class Product(BaseModel):
    ...
```

It should contain:

```text
name → string
price → float
quantity → integer
```

Then create:

```text
POST /products
```

that accepts the model.

Expected input:

```json
{
  "name": "Laptop",
  "price": 55000,
  "quantity": 2
}
```

Expected response:

```json
{
  "name": "Laptop",
  "price": 55000,
  "quantity": 2
}
```

---

# 35. Practice Exercise 2 — Student

Create:

```python
class Student(BaseModel):
    ...
```

Required:

```text
name → string
roll_number → integer
course → string
```

Optional:

```text
phone → string or None
```

Then create:

```text
POST /students
```

---

# 36. Practice Exercise 3 — AI Chat

Create:

```python
class ChatRequest(BaseModel):
    ...
```

Fields:

```text
message → string
model → string
temperature → float
```

Create:

```text
POST /chat
```

Expected request:

```json
{
  "message": "Explain RAG",
  "model": "llama",
  "temperature": 0.7
}
```

Return the received values.

---

# 37. Practice Exercise 4 — Nested Model

Create:

```python
class Address(BaseModel):
    city: str
    pincode: int
```

Then:

```python
class User(BaseModel):
    name: str
    address: Address
```

Accept:

```json
{
  "name": "Pranavi",
  "address": {
    "city": "Delhi",
    "pincode": 110001
  }
}
```

Return the user's name, city, and pincode.

---

# 38. Lesson 6 Quiz

Try answering these without looking back.

### Question 1

What problem does Pydantic solve?

### Question 2

What does this code create?

```python
class User(BaseModel):
    name: str
    age: int
```

### Question 3

Why do we write:

```python
user: User
```

inside a FastAPI endpoint?

### Question 4

What happens if the client sends:

```json
{
  "name": "Pranavi"
}
```

when the model requires:

```python
name: str
age: int
```

### Question 5

What does this mean?

```python
phone: str | None = None
```

### Question 6

What is the difference between:

```python
phone: str | None = None
```

and:

```python
country: str = "India"
```

### Question 7

Does a Pydantic model permanently store information?

### Question 8

What is the difference between:

```python
user["name"]
```

and:

```python
user.name
```

when working with a dictionary versus a Pydantic model?

### Question 9

What is a nested Pydantic model?

### Question 10 — Challenge

Create:

```python
class Product(BaseModel):
    name: str
    price: float
    quantity: int
```

Then create:

```text
POST /products
```

that accepts and returns the Product model's data.

---

# 39. Key Takeaways

Remember these points:

### Pydantic

Pydantic is used for:

```text
Data modeling
+
Data validation
+
Data parsing
```

### `BaseModel`

```python
class User(BaseModel):
```

turns a normal Python class into a Pydantic model.

### Type hints

```python
name: str
age: int
price: float
```

describe expected data types.

### Required fields

```python
name: str
```

is required by default.

### Optional field

```python
phone: str | None = None
```

allows the field to be omitted and defaults it to `None`.

### Default value

```python
country: str = "India"
```

uses `"India"` when the client doesn't provide the field.

### Nested models

```python
class User(BaseModel):
    address: Address
```

allow structured objects inside other objects.

### Pydantic is NOT a database

It validates and structures data.

A database is responsible for persistent storage.

---

# 40. The Big Picture So Far

You have now learned:

```text
                    FASTAPI
                       |
       +---------------+---------------+
       |               |               |
       ↓               ↓               ↓
   Path Params    Query Params    Request Body
                                       |
                                       ↓
                                  Pydantic
                                       |
                                       ↓
                                  Validation
                                       |
                                       ↓
                                Python Object
                                       |
                                       ↓
                                Your Function
                                       |
                                       ↓
                                   Response
```

The next major step is to take these concepts and build a **real CRUD API**.

We'll move from:

```text
"Receive data and return it"
```

to:

```text
Create data
Read data
Update data
Delete data
```

After that, we'll introduce a real database so the data doesn't disappear when the server stops.
