# FastAPI Lesson 8 — HTTP Status Codes & Error Handling

## Overview

A real API does not only return successful data. It must also tell the client when a resource is missing, a request is invalid, authentication fails, permission is denied, validation fails, or something goes wrong on the server.

This lesson covers HTTP status codes and FastAPI error handling, especially `HTTPException`.

---

## 1. What Is an HTTP Status Code?

Every HTTP response contains a status code describing the result.

Example:

```text
GET /users/1

200 OK
{
    "id": 1,
    "name": "Pranavi"
}
```

Major categories:

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client error
5xx → Server error
```

For FastAPI interviews, focus mainly on:

```text
200, 201, 204, 400, 401, 403, 404, 422, 500
```

---

## 2. 200 OK

`200 OK` means the request was successful.

Example:

```python
@app.get("/users")
def get_users():
    return users
```

A successful GET commonly returns:

```text
200 OK
```

---

## 3. 201 Created

`201 Created` is commonly used when a new resource has been created.

Example:

```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post(
    "/users",
    status_code=status.HTTP_201_CREATED
)
def create_user(user: User):
    ...
```

You can also write `status_code=201`, but named constants are more readable.

---

## 4. 204 No Content

`204 No Content` means the operation succeeded but there is no response body.

It can be useful for operations such as:

```text
DELETE /users/1
```

when the API does not need to return a message.

---

## 5. 400 Bad Request

`400 Bad Request` generally means the client sent a request that the API cannot accept as a valid request.

Remember:

```text
4xx → Generally a client/request-side problem
```

FastAPI and Pydantic automatically handle many input-validation cases.

---

## 6. 401 Unauthorized

`401 Unauthorized` generally means valid authentication credentials were not provided.

Example:

```text
GET /profile
```

requires a valid token, but the client sends no valid token.

Think:

```text
401 → "Who are you?"
```

Authentication and JWT will be covered later.

---

## 7. 403 Forbidden

`403 Forbidden` means the server knows who the client is, but the client does not have permission to perform the action.

Example:

```text
Admin → Can delete users
Normal user → Cannot delete users
```

Think:

```text
403 → "I know who you are, but you cannot do this."
```

### 401 vs 403

```text
401 → Authentication problem
403 → Authorization/permission problem
```

---

## 8. 404 Not Found

`404 Not Found` is extremely common in CRUD APIs.

Suppose:

```text
GET /users/999
```

but user 999 does not exist.

The API should return:

```text
404 Not Found
```

FastAPI provides `HTTPException` for this.

---

## 9. HTTPException

Import it:

```python
from fastapi import FastAPI, HTTPException
```

Then:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

This tells FastAPI to stop normal endpoint execution and return an HTTP error response.

---

## 10. Why `raise`?

Use:

```python
raise HTTPException(...)
```

not:

```python
return HTTPException(...)
```

`raise` tells Python to stop normal execution and raise an exception.

Example:

```python
if user_not_found:
    raise HTTPException(
        status_code=404,
        detail="User not found"
    )
```

---

## 11. Example — Get One User

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):

    for user in users:

        if user["id"] == user_id:
            return user

    raise HTTPException(
        status_code=404,
        detail="User not found"
    )
```

Flow:

```text
GET /users/1
       ↓
FastAPI extracts user_id
       ↓
Search users
       ↓
Found? ── Yes → return user
   |
   No
   ↓
raise HTTPException
   ↓
404 Not Found
```

---

## 12. Error Response

If:

```text
GET /users/999
```

and the user doesn't exist, FastAPI can return:

```json
{
    "detail": "User not found"
}
```

with:

```text
404 Not Found
```

---

## 13. Using `status` Constants

Instead of:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

you can write:

```python
from fastapi import HTTPException, status

raise HTTPException(
    status_code=status.HTTP_404_NOT_FOUND,
    detail="User not found"
)
```

Common constants include:

```python
status.HTTP_200_OK
status.HTTP_201_CREATED
status.HTTP_400_BAD_REQUEST
status.HTTP_401_UNAUTHORIZED
status.HTTP_403_FORBIDDEN
status.HTTP_404_NOT_FOUND
status.HTTP_422_UNPROCESSABLE_ENTITY
status.HTTP_500_INTERNAL_SERVER_ERROR
```

---

## 14. 422 Unprocessable Entity

You may have already seen `422` in Swagger.

Consider:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

The API expects:

```text
name  → string
email → string
age   → integer
```

If the client sends:

```json
{
    "name": "Pranavi",
    "email": "pranavi@example.com",
    "age": "hello"
}
```

Pydantic detects that `age` is invalid.

FastAPI commonly returns:

```text
422 Unprocessable Entity
```

with validation details.

---

## 15. Automatic Validation

FastAPI uses Pydantic models to validate request data.

For:

```python
class User(BaseModel):
    name: str
    email: str
    age: int
```

and:

```python
@app.post("/users")
def create_user(user: User):
    ...
```

the flow is:

```text
JSON Request
     ↓
FastAPI
     ↓
Pydantic
     ↓
Validation
     ↓
Valid?
   ↙       ↘
 Yes        No
  ↓          ↓
Endpoint   Validation
executes   error
```

You don't need to manually check every field.

---

## 16. 500 Internal Server Error

`500 Internal Server Error` generally means an unexpected problem occurred on the server.

For example:

```python
result = 10 / 0
```

can cause an unexpected Python exception.

Conceptually:

```text
4xx → Problem with request/client
5xx → Problem on server
```

In production, developers should log the actual server error while avoiding exposure of sensitive internal details.

---

## 17. Status Codes to Memorize

| Code | Meaning | Common use |
|---|---|---|
| 200 | OK | Successful request |
| 201 | Created | Resource created |
| 204 | No Content | Successful operation with no body |
| 400 | Bad Request | Invalid request |
| 401 | Unauthorized | Authentication problem |
| 403 | Forbidden | Permission problem |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | Validation error |
| 500 | Internal Server Error | Unexpected server error |

---

## 18. Improving Our CRUD API

Instead of:

```python
return {"message": "User not found"}
```

use:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

Example:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):

    for user in users:

        if user["id"] == user_id:
            return user

    raise HTTPException(
        status_code=404,
        detail="User not found"
    )
```

This communicates the correct HTTP result to the client.

---

## 19. Improving DELETE

```python
@app.delete("/users/{user_id}")
def delete_user(user_id: int):

    for user in users:

        if user["id"] == user_id:

            users.remove(user)

            return {
                "message": "User deleted"
            }

    raise HTTPException(
        status_code=404,
        detail="User not found"
    )
```

If the user exists:

```text
DELETE /users/1
→ 200 OK
```

If the user doesn't exist:

```text
DELETE /users/999
→ 404 Not Found
```

---

## 20. Using 201 for Create

```python
@app.post(
    "/users",
    status_code=status.HTTP_201_CREATED
)
def create_user(user: User):
    ...
```

A successful creation now returns:

```text
201 Created
```

---

## 21. Complete Example

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    name: str
    email: str
    age: int


users = []


@app.post(
    "/users",
    status_code=status.HTTP_201_CREATED
)
def create_user(user: User):

    new_user = {
        "id": len(users) + 1,
        "name": user.name,
        "email": user.email,
        "age": user.age
    }

    users.append(new_user)

    return new_user


@app.get("/users/{user_id}")
def get_user(user_id: int):

    for user in users:

        if user["id"] == user_id:
            return user

    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="User not found"
    )


@app.delete("/users/{user_id}")
def delete_user(user_id: int):

    for user in users:

        if user["id"] == user_id:

            users.remove(user)

            return {
                "message": "User deleted"
            }

    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="User not found"
    )
```

---

## 22. `return` vs `raise HTTPException`

### `return`

```python
return {
    "message": "User found"
}
```

means:

> Return a normal response.

### `raise HTTPException`

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

means:

> Stop the current operation and return an HTTP error response.

Mental model:

```text
return
  ↓
Normal response

raise HTTPException
  ↓
Error response
```

---

## 23. Common Beginner Mistakes

### Mistake 1 — Returning an error with status 200

Not ideal:

```python
return {
    "message": "User not found"
}
```

Better:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

### Mistake 2 — Using `return HTTPException`

Incorrect:

```python
return HTTPException(
    status_code=404,
    detail="User not found"
)
```

Correct:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

### Mistake 3 — Confusing 401 and 403

Remember:

```text
401 → Authentication
403 → Permission
```

### Mistake 4 — Thinking every error is 500

A bad client request is generally a `4xx` issue.

A `500` indicates an unexpected server-side problem.

---

# 24. Interview Questions

Try answering these without looking at the answers.

### Q1. What is an HTTP status code?

### Q2. What does 200 mean?

### Q3. Why is 201 commonly used for POST?

### Q4. What does 404 mean?

### Q5. What is `HTTPException` in FastAPI?

### Q6. Why do we use `raise HTTPException()`?

### Q7. What is the difference between 401 and 403?

### Q8. What is a 422 response in FastAPI?

### Q9. What is the difference between 400 and 500?

### Q10. How would you return "User not found" from FastAPI?

A good answer:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

---

# 25. Interview Answer — What Is HTTPException?

A concise interview answer:

> `HTTPException` is FastAPI's mechanism for returning an HTTP error response from an endpoint. It allows us to specify a status code and a detail message. We use `raise HTTPException(...)` to stop normal execution and return the appropriate error response.

Example:

```python
raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

---

# 26. Interview Answer — What Happens When Pydantic Validation Fails?

A good answer:

> FastAPI uses Pydantic models to validate request data. If the incoming request does not match the model's expected structure or types, FastAPI automatically generates a validation error response, commonly with status code 422.

Flow:

```text
Request
   ↓
Pydantic model
   ↓
Validation
   ↓
Invalid
   ↓
422 response
```

---

# 27. Interview Answer — 401 vs 403

A strong concise answer:

> `401 Unauthorized` indicates that the client has not provided valid authentication credentials. `403 Forbidden` means the client is authenticated but does not have permission to perform the requested action.

---

# 28. Final Takeaway

Remember:

```text
200 → Success
201 → Created
204 → No Content
400 → Bad Request
401 → Authentication problem
403 → Permission problem
404 → Resource not found
422 → Validation error
500 → Server error
```

Most important FastAPI error-handling pattern:

```python
from fastapi import HTTPException

raise HTTPException(
    status_code=404,
    detail="User not found"
)
```

And:

```text
return
→ normal response

raise HTTPException
→ error response
```

---

# 29. FastAPI Progress So Far

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
       ↓
Lesson 8
HTTP Status Codes + Error Handling
```

You are now very close to completing the interview-focused FastAPI fundamentals.

---

# 30. Next Lesson

## Lesson 9 — Response Models

So far we learned how to validate incoming data:

```python
def create_user(user: User):
```

Next we'll learn how FastAPI controls and validates outgoing data:

```python
@app.get(
    "/users/{user_id}",
    response_model=User
)
```

We will cover:

- What a response model is
- Why response validation matters
- `response_model`
- Request model vs response model
- Hiding sensitive fields
- Structured API responses
- Response models in Swagger/OpenAPI
- Common interview questions

After Lesson 9:

```text
Lesson 10 → Dependency Injection + Depends()
Lesson 11 → APIRouter + Project Structure
```

After these, the core FastAPI fundamentals needed for your interviews will be complete.

