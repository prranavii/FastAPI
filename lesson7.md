# FastAPI Lesson 7 — Building a CRUD API

## Overview

In the previous lessons, we learned:

- What APIs are
- HTTP and HTTP methods
- How to create a FastAPI application
- Query parameters
- Path parameters
- Request bodies
- Pydantic models

In this lesson, we combine those concepts to build a complete **CRUD API**.

CRUD is one of the most important concepts in backend development because most applications need to create, read, update, and delete data.

We will first build CRUD **without a database**. We will store data temporarily in a Python list.

This is intentional.

The goal is to understand how CRUD works before introducing databases, SQLAlchemy, and PostgreSQL.

---

# 1. What Does CRUD Mean?

CRUD stands for:

| Letter | Meaning | Example |
|---|---|---|
| C | Create | Create a new user |
| R | Read | Get users |
| U | Update | Change a user's information |
| D | Delete | Remove a user |

Almost every backend application performs these operations.

## Example: Instagram

```text
Create → Upload a post
Read   → View a post
Update → Edit a caption
Delete → Delete a post
```

## Example: E-commerce application

```text
Create → Add a product
Read   → View a product
Update → Change the product price
Delete → Remove a product
```

## Example: Placement Tracker

```text
Create → Add a company/application
Read   → View applications
Update → Change application status
Delete → Remove an application
```

Therefore, understanding CRUD is essential for backend development.

---

# 2. CRUD and HTTP Methods

CRUD operations are commonly mapped to HTTP methods.

```text
CREATE → POST
READ   → GET
UPDATE → PUT
DELETE → DELETE
```

So our API might have:

```text
POST    /users
GET     /users
GET     /users/1
PUT     /users/1
DELETE  /users/1
```

Think about these methods like this:

```text
POST   → Create something new
GET    → Give me existing information
PUT    → Replace/update existing information
DELETE → Remove something
```

---

# 3. What We Are Going to Build

We will build a simple User Management API.

Each user will have:

```text
id
name
email
age
```

Our API will support:

```text
POST    /users
GET     /users
GET     /users/{id}
PUT     /users/{id}
DELETE  /users/{id}
```

The final API will be able to:

1. Create a user
2. Get all users
3. Get one user
4. Update a user
5. Delete a user

---

# 4. Why Are We Not Using a Database Yet?

Normally, a real application stores data in a database.

For example:

```text
FastAPI
   ↓
Database
```

But if we immediately introduce:

- SQL
- SQLite
- PostgreSQL
- SQLAlchemy
- database sessions
- migrations

we would have to learn many concepts at the same time.

Instead, we will temporarily store users in a Python list.

```python
users = []
```

This lets us focus only on understanding CRUD.

Later, we will replace the Python list with a real database.

---

# 5. Project Structure

For this lesson, our project can be very simple:

```text
fastapi-project/
│
├── .venv/
│
└── main.py
```

Our `main.py` will contain the complete CRUD API.

---

# 6. Import FastAPI

Start with:

```python
from fastapi import FastAPI
```

This imports the `FastAPI` class.

Then:

```python
app = FastAPI()
```

This creates our FastAPI application.

So:

```python
from fastapi import FastAPI

app = FastAPI()
```

means:

> Create a FastAPI application that we can add API endpoints to.

---

# 7. Create a Pydantic Model

We need to define what a user should look like.

From Lesson 6, we learned about Pydantic.

Import `BaseModel`:

```python
from pydantic import BaseModel
```

Then create:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

This says that a User must contain:

```text
name  → string
email → string
age   → integer
```

For example, this is valid:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

But something like this is invalid:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": "hello"
}
```

because `age` should be an integer.

---

# 8. Store Users in a Python List

For now:

```python
users = []
```

Initially:

```python
users = []
```

After creating a user, it could become:

```python
users = [
    {
        "id": 1,
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    }
]
```

After another user:

```python
users = [
    {
        "id": 1,
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    },
    {
        "id": 2,
        "name": "Rahul",
        "email": "rahul@example.com",
        "age": 22
    }
]
```

Remember:

> This list is temporary. It is not a database.

---

# 9. CREATE — POST

Our first CRUD operation is Create.

We will use:

```text
POST /users
```

The endpoint:

```python
@app.post("/users")
def create_user(user: User):
    ...
```

Let's understand it piece by piece.

## `@app.post("/users")`

```python
@app.post("/users")
```

This tells FastAPI:

> When a client sends a POST request to `/users`, execute the function below it.

---

## The function

```python
def create_user(user: User):
```

The parameter:

```python
user: User
```

means:

> The request body should follow the `User` Pydantic model.

For example, the client can send:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

FastAPI passes the validated data to the function.

---

# 10. Creating the New User

We can create a dictionary:

```python
new_user = {
    "id": len(users) + 1,
    "name": user.name,
    "email": user.email,
    "age": user.age
}
```

Let's understand each part.

## Generating the ID

```python
len(users) + 1
```

Suppose:

```python
users = []
```

Then:

```python
len(users)
```

is:

```text
0
```

Therefore:

```python
len(users) + 1
```

is:

```text
1
```

If there is already one user:

```python
users = [
    {...}
]
```

then:

```python
len(users)
```

is:

```text
1
```

and:

```python
len(users) + 1
```

becomes:

```text
2
```

This gives us a simple ID for learning.

Important:

> This is not a production-quality ID-generation strategy. A real database normally generates IDs safely.

---

# 11. Accessing Pydantic Fields

We can access fields using dot notation:

```python
user.name
user.email
user.age
```

For example:

```python
user.name
```

might contain:

```text
"Pranavi"
```

And:

```python
user.age
```

might contain:

```text
21
```

So:

```python
new_user = {
    "id": 1,
    "name": user.name,
    "email": user.email,
    "age": user.age
}
```

could become:

```python
new_user = {
    "id": 1,
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "age": 21
}
```

---

# 12. Add the User to the List

We use:

```python
users.append(new_user)
```

`append()` adds an item to the end of a Python list.

Before:

```python
users = []
```

After:

```python
users = [
    {
        "id": 1,
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    }
]
```

Finally:

```python
return new_user
```

sends the newly created user back to the client.

---

# 13. Complete CREATE Endpoint

```python
@app.post("/users")
def create_user(user: User):

    new_user = {
        "id": len(users) + 1,
        "name": user.name,
        "email": user.email,
        "age": user.age
    }

    users.append(new_user)

    return new_user
```

---

# 14. What Happens During POST?

Suppose the frontend sends:

```http
POST /users
```

with:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

The flow is:

```text
Client
   |
   | POST /users
   | JSON body
   ↓
FastAPI
   |
   ↓
Pydantic validation
   |
   ↓
create_user()
   |
   ↓
Create dictionary
   |
   ↓
users.append()
   |
   ↓
Return response
```

The response:

```json
{
  "id": 1,
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

---

# 15. Testing POST Using Swagger UI

Start your server:

```bash
uvicorn main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

Find:

```text
POST /users
```

Click:

```text
Try it out
```

Enter:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

Click:

```text
Execute
```

You should receive something similar to:

```json
{
  "id": 1,
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

---

# 16. READ — Get All Users

Now we want to retrieve all users.

We use:

```text
GET /users
```

Code:

```python
@app.get("/users")
def get_users():
    return users
```

This function simply returns the list.

If:

```python
users = [
    {
        "id": 1,
        "name": "Pranavi",
        "email": "pranavi@example.com",
        "age": 21
    }
]
```

then:

```text
GET /users
```

returns:

```json
[
  {
    "id": 1,
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "age": 21
  }
]
```

---

# 17. Why Is This a GET Request?

GET is used when the client wants to retrieve information.

For example:

```text
GET /users
```

means:

> Give me the users.

We aren't creating anything.

We aren't changing anything.

We aren't deleting anything.

We are simply reading data.

---

# 18. READ — Get One User

Getting all users:

```text
GET /users
```

But what if we want only user ID `1`?

We can use:

```text
GET /users/1
```

This uses a path parameter.

Code:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):

    for user in users:

        if user["id"] == user_id:
            return user

    return {"message": "User not found"}
```

---

# 19. Understanding the Path Parameter

The route:

```python
@app.get("/users/{user_id}")
```

contains:

```text
{user_id}
```

This means that the value will come from the URL.

For example:

```text
/users/1
/users/2
/users/50
```

If the client requests:

```text
/users/2
```

FastAPI puts:

```text
2
```

into:

```python
user_id
```

Because we wrote:

```python
user_id: int
```

FastAPI expects the value to be an integer.

---

# 20. Searching for the User

We use:

```python
for user in users:
```

This means:

> Go through each user in the list.

Then:

```python
if user["id"] == user_id:
```

means:

> Does this user's ID match the ID requested by the client?

For example:

```text
user["id"] = 1
user_id = 1
```

Then:

```text
1 == 1
```

is true.

So:

```python
return user
```

returns that user.

---

# 21. What If the User Does Not Exist?

Suppose the database/list contains:

```text
User 1
User 2
User 3
```

and the client requests:

```text
GET /users/100
```

There is no user with ID 100.

After the loop finishes, we return:

```python
return {"message": "User not found"}
```

So the response becomes:

```json
{
  "message": "User not found"
}
```

In a production API, we will later learn how to return proper HTTP status codes such as `404 Not Found`.

---

# 22. UPDATE — PUT

Now we want to modify an existing user.

We use:

```text
PUT /users/{user_id}
```

For example:

```text
PUT /users/1
```

with:

```json
{
  "name": "Pranavi Jain",
  "email": "pranavi@example.com",
  "age": 22
}
```

Code:

```python
@app.put("/users/{user_id}")
def update_user(user_id: int, updated_user: User):

    for user in users:

        if user["id"] == user_id:

            user["name"] = updated_user.name
            user["email"] = updated_user.email
            user["age"] = updated_user.age

            return user

    return {"message": "User not found"}
```

---

# 23. Understanding the UPDATE Endpoint

There are two sources of information:

### 1. Path parameter

```python
user_id: int
```

This comes from:

```text
/users/1
```

### 2. Request body

```python
updated_user: User
```

This comes from:

```json
{
  "name": "Pranavi Jain",
  "email": "pranavi@example.com",
  "age": 22
}
```

So the request contains both:

```text
URL
↓
user_id = 1

BODY
↓
new user information
```

---

# 24. Updating the Fields

Suppose the existing user is:

```python
{
    "id": 1,
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "age": 21
}
```

The client sends:

```json
{
    "name": "Pranavi Jain",
    "email": "pranavi@example.com",
    "age": 22
}
```

Then:

```python
user["name"] = updated_user.name
```

changes:

```text
Pranavi
```

to:

```text
Pranavi Jain
```

Similarly:

```python
user["age"] = updated_user.age
```

changes:

```text
21
```

to:

```text
22
```

The ID remains unchanged.

---

# 25. DELETE

The final CRUD operation is Delete.

We use:

```text
DELETE /users/{user_id}
```

For example:

```text
DELETE /users/1
```

Code:

```python
@app.delete("/users/{user_id}")
def delete_user(user_id: int):

    for user in users:

        if user["id"] == user_id:

            users.remove(user)

            return {"message": "User deleted"}

    return {"message": "User not found"}
```

---

# 26. Understanding DELETE

Again, we search through the users:

```python
for user in users:
```

Then:

```python
if user["id"] == user_id:
```

If we find the user:

```python
users.remove(user)
```

removes the user from the list.

Then:

```python
return {"message": "User deleted"}
```

sends a response to the client.

---

# 27. Complete CRUD Application

Here is the complete `main.py`:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    name: str
    email: str
    age: int


users = []


# CREATE
@app.post("/users")
def create_user(user: User):

    new_user = {
        "id": len(users) + 1,
        "name": user.name,
        "email": user.email,
        "age": user.age
    }

    users.append(new_user)

    return new_user


# READ ALL
@app.get("/users")
def get_users():
    return users


# READ ONE
@app.get("/users/{user_id}")
def get_user(user_id: int):

    for user in users:

        if user["id"] == user_id:
            return user

    return {"message": "User not found"}


# UPDATE
@app.put("/users/{user_id}")
def update_user(user_id: int, updated_user: User):

    for user in users:

        if user["id"] == user_id:

            user["name"] = updated_user.name
            user["email"] = updated_user.email
            user["age"] = updated_user.age

            return user

    return {"message": "User not found"}


# DELETE
@app.delete("/users/{user_id}")
def delete_user(user_id: int):

    for user in users:

        if user["id"] == user_id:

            users.remove(user)

            return {"message": "User deleted"}

    return {"message": "User not found"}
```

---

# 28. Understanding the Complete Architecture

Our application currently looks like:

```text
                    CLIENT
                       |
                       |
                 HTTP Request
                       |
                       ↓
                 ┌───────────┐
                 │  FastAPI  │
                 └───────────┘
                       |
              ┌────────┼────────┐
              ↓        ↓        ↓
            POST      GET      DELETE
              |        |        |
              └────────┼────────┘
                       ↓
                    users[]
                 Python list
```

For updates:

```text
PUT /users/{id}
        |
        ↓
     FastAPI
        |
        ↓
   Find user
        |
        ↓
 Update fields
```

---

# 29. CRUD Endpoint Summary

| Operation | HTTP Method | Endpoint | Purpose |
|---|---|---|---|
| Create | POST | `/users` | Create a user |
| Read all | GET | `/users` | Get all users |
| Read one | GET | `/users/{id}` | Get one user |
| Update | PUT | `/users/{id}` | Update a user |
| Delete | DELETE | `/users/{id}` | Delete a user |

Memorize this pattern.

It appears everywhere in backend development.

---

# 30. CRUD Request Examples

## CREATE

```http
POST /users
```

Body:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

---

## READ ALL

```http
GET /users
```

No body is required.

---

## READ ONE

```http
GET /users/1
```

No body is required.

---

## UPDATE

```http
PUT /users/1
```

Body:

```json
{
  "name": "Pranavi Jain",
  "email": "pranavi@example.com",
  "age": 22
}
```

---

## DELETE

```http
DELETE /users/1
```

No body is required.

---

# 31. Important Difference: PUT vs POST

These methods have different purposes.

## POST

Usually used to create a new resource.

```text
POST /users
```

Meaning:

> Create a new user.

The server decides the new resource's ID.

---

## PUT

Used to update/replace an existing resource at a known location.

```text
PUT /users/1
```

Meaning:

> Update the user with ID 1.

---

# 32. What About PATCH?

You may also see:

```text
PATCH
```

PATCH is commonly used for partial updates.

Suppose a user has:

```json
{
  "name": "Pranavi",
  "email": "pranavi@example.com",
  "age": 21
}
```

If we only want to change the age:

```json
{
  "age": 22
}
```

PATCH is a natural choice.

Conceptually:

```text
PUT
→ Replace/update the resource representation

PATCH
→ Make a partial change
```

We will study PATCH more carefully later.

For this lesson, focus on:

```text
POST
GET
PUT
DELETE
```

---

# 33. Why Use Pydantic in CRUD?

Without Pydantic, we could accept arbitrary JSON.

That would make our backend less predictable.

With:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

we define the expected structure.

The client must provide:

```text
name
email
age
```

and FastAPI/Pydantic validates the incoming data.

This gives us a clear contract between the client and backend.

---

# 34. CRUD and the Frontend

Imagine a React frontend.

The frontend might have:

```text
Add User
View Users
Edit User
Delete User
```

When the user clicks "Add User":

```text
React
  ↓
POST /users
  ↓
FastAPI
  ↓
Create user
  ↓
Response
  ↓
React updates UI
```

When the user clicks "Delete":

```text
React
  ↓
DELETE /users/1
  ↓
FastAPI
  ↓
Delete user
  ↓
Response
  ↓
React updates UI
```

This is how frontend and backend applications communicate.

---

# 35. Why Our Current API Is Not Production-Ready

Our API works for learning, but there are several problems.

## Problem 1 — Data disappears

We use:

```python
users = []
```

If the server stops, the data disappears.

A real application needs a database.

---

## Problem 2 — ID generation is unsafe

We use:

```python
len(users) + 1
```

Suppose:

```text
User 1
User 2
User 3
```

Then user 2 is deleted.

Now:

```text
User 1
User 3
```

`len(users)` is 2.

A new user might receive:

```text
id = 3
```

But user 3 already exists.

So this approach can create duplicate IDs.

A database normally handles unique IDs properly.

---

## Problem 3 — Error handling

Currently we return:

```json
{
  "message": "User not found"
}
```

A better API should return an appropriate HTTP status code, such as:

```text
404 Not Found
```

We will learn this later.

---

## Problem 4 — Everything is inside one file

Real applications usually separate:

```text
routes
models
database
services
configuration
```

As the application grows, this becomes important.

We will learn proper project structure later.

---

# 36. The Real Architecture We Are Building Toward

Eventually, our backend will look more like:

```text
React / Frontend
        |
        | HTTP
        ↓
     FastAPI
        |
        ↓
     Routes
        |
        ↓
    Service Layer
        |
        ↓
   Database Layer
        |
        ↓
 PostgreSQL / SQLite
```

Later we may also have:

```text
FastAPI
   |
   ├── Authentication
   ├── Database
   ├── External APIs
   ├── AI/LLM APIs
   └── Background tasks
```

This is the foundation for real backend development.

---

# 37. CRUD Flow — Complete Mental Model

Remember this:

```text
                    CLIENT
                       |
                       ↓
                  HTTP Request
                       |
                       ↓
                    FastAPI
                       |
            ┌──────────┼──────────┐
            ↓          ↓          ↓
           POST        GET       DELETE
            |          |          |
            ↓          ↓          ↓
         Create       Read      Delete
            |
            ↓
         Response
```

For update:

```text
CLIENT
  |
  | PUT /users/1
  | JSON body
  ↓
FastAPI
  |
  ↓
Find user 1
  |
  ↓
Update fields
  |
  ↓
Response
```

---

# 38. A Better Way to Think About Endpoints

An endpoint combines:

```text
HTTP METHOD + URL
```

For example:

```text
POST /users
```

is one endpoint.

```text
GET /users
```

is another endpoint.

```text
GET /users/1
```

is another endpoint.

Even though `/users` appears multiple times, the HTTP method changes the operation.

For example:

```text
POST /users
```

means:

> Create a user.

while:

```text
GET /users
```

means:

> Get users.

The method is part of how the endpoint's meaning is defined.

---

# 39. Common Beginner Mistakes

## Mistake 1 — Forgetting `@`

Incorrect:

```python
app.post("/users")
```

Correct:

```python
@app.post("/users")
```

---

## Mistake 2 — Forgetting the Pydantic model

Incorrect:

```python
def create_user(user):
```

Better:

```python
def create_user(user: User):
```

This tells FastAPI what type of request body to expect.

---

## Mistake 3 — Using the wrong HTTP method

For creating:

```text
POST
```

For reading:

```text
GET
```

For updating:

```text
PUT
```

For deleting:

```text
DELETE
```

---

## Mistake 4 — Forgetting the path parameter

Incorrect:

```python
@app.get("/users")
def get_user(user_id: int):
```

This does not match the intended URL.

Correct:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
```

---

## Mistake 5 — Confusing path and body

In:

```python
@app.put("/users/{user_id}")
def update_user(user_id: int, updated_user: User):
```

there are two different sources:

```text
user_id
→ URL

updated_user
→ JSON request body
```

---

# 40. Practice Exercise 1 — Product CRUD

Create a new CRUD API for products.

Your model should be:

```python
class Product(BaseModel):
    name: str
    price: float
    category: str
```

Implement:

```text
POST    /products
GET     /products
GET     /products/{product_id}
PUT     /products/{product_id}
DELETE  /products/{product_id}
```

Do this yourself before looking at your previous User CRUD code.

---

# 41. Practice Exercise 2 — Student CRUD

Create:

```python
class Student(BaseModel):
    name: str
    course: str
    age: int
```

Implement all five CRUD operations.

---

# 42. Practice Exercise 3 — Placement Application CRUD

This one is particularly useful for understanding a real-world application.

Create:

```python
class Application(BaseModel):
    company: str
    role: str
    status: str
```

Example:

```json
{
  "company": "Google",
  "role": "Software Engineer Intern",
  "status": "Applied"
}
```

Implement:

```text
POST    /applications
GET     /applications
GET     /applications/{application_id}
PUT     /applications/{application_id}
DELETE  /applications/{application_id}
```

Possible statuses:

```text
Applied
OA
Interview
Selected
Rejected
```

---

# 43. Mini Quiz

Try answering these without looking above.

### Question 1

What does CRUD stand for?

### Question 2

Which HTTP method is normally used to create data?

### Question 3

Which HTTP method is used to retrieve data?

### Question 4

What endpoint would you use to get user ID 5?

### Question 5

What does this mean?

```python
user_id: int
```

### Question 6

What is the purpose of:

```python
users.append(new_user)
```

### Question 7

What does this do?

```python
users.remove(user)
```

### Question 8

Why are we using a Pydantic model?

### Question 9

What is the difference between:

```text
GET /users
```

and:

```text
GET /users/1
```

### Question 10

Why is a Python list not a suitable permanent database?

---

# 44. Quiz Answers

### Answer 1

CRUD means:

```text
Create
Read
Update
Delete
```

### Answer 2

```text
POST
```

### Answer 3

```text
GET
```

### Answer 4

```text
GET /users/5
```

### Answer 5

It means `user_id` should be an integer.

### Answer 6

It adds `new_user` to the `users` list.

### Answer 7

It removes `user` from the list.

### Answer 8

Pydantic defines and validates the structure and types of incoming data.

### Answer 9

```text
GET /users
→ Get all users

GET /users/1
→ Get user with ID 1
```

### Answer 10

A Python list exists only in the application's memory and data is lost when the application restarts.

---

# 45. Important Concepts to Remember

## CRUD

```text
C → Create
R → Read
U → Update
D → Delete
```

## HTTP mapping

```text
POST   → Create
GET    → Read
PUT    → Update
DELETE → Delete
```

## Collection vs individual resource

```text
/users
```

usually refers to the collection.

```text
/users/1
```

refers to one specific user.

## Pydantic

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

defines the expected request data.

## Temporary storage

```python
users = []
```

is useful for learning but should eventually be replaced by a real database.

---

# 46. What We Have Learned So Far

Our FastAPI journey now looks like:

```text
Lesson 1
API Fundamentals
       ↓
Lesson 2
HTTP + Client + Server
       ↓
Lesson 3
First FastAPI App
       ↓
Lesson 4
Receiving Data
       ↓
Lesson 5
Path + Query Parameters
       ↓
Lesson 6
Pydantic Models
       ↓
Lesson 7
CRUD API
```

We can now build a basic backend that accepts requests and performs operations on data.

---

# 47. What Comes Next?

The biggest missing piece is persistence.

Right now:

```text
FastAPI
   ↓
Python list
```

We want:

```text
FastAPI
   ↓
Database
```

The next major topic is:

# Lesson 8 — Databases with FastAPI

We will start with:

```text
SQLite
```

and then learn:

```text
SQL
↓
SQLite
↓
SQLAlchemy
↓
Database models
↓
CRUD with a database
↓
PostgreSQL
```

Eventually, the application will look like:

```text
Frontend
    |
    ↓
FastAPI
    |
    ↓
SQLAlchemy
    |
    ↓
PostgreSQL
```

This is a much more realistic backend architecture.

---

# Final Takeaway

The most important thing to understand from Lesson 7 is:

> **CRUD is the basic set of operations used to create, retrieve, update, and delete data in an application.**

Remember:

```text
POST    /users
        ↓
      CREATE

GET     /users
        ↓
       READ

GET     /users/{id}
        ↓
   READ ONE USER

PUT     /users/{id}
        ↓
      UPDATE

DELETE  /users/{id}
        ↓
      DELETE
```

And remember that our current list-based implementation is only a learning step.

A real application needs:

```text
FastAPI
   ↓
Database
```

rather than:

```text
FastAPI
   ↓
Python list
```

That database connection is what we will build next.
