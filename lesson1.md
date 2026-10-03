
# FastAPI Notes — Lesson 1: APIs and FastAPI Basics

## 1. What is an API?

**API** stands for **Application Programming Interface**.

An API is a defined way for one software system to communicate with another software system.

### Restaurant example

Think of a restaurant:

- **Customer:** Makes a request.
- **Waiter:** Carries the request to the kitchen and brings back the response.
- **Kitchen:** Processes the request.

The waiter is like an API: it provides a communication layer between the customer and the kitchen.

In software:
- The **client** makes a request.
- The **API** provides the interface for communication.
- The **backend/service** processes the request.
- A **response** is returned to the client.

## 2. Why do we need APIs?

Applications often need information or functionality from another system.

Examples:
- A weather app requests weather data from a weather service.
- A shopping website requests product information from its backend.
- A frontend sends login details to a backend.
- A chatbot sends a user's question to an AI service.
- A payment application communicates with a payment provider.

An API provides a defined way to make these requests and receive responses.

## 3. Frontend, backend, database, and API

These terms are related, but they are not the same.

- **Frontend:** The part of an application that users see and interact with.
- **Backend:** Server-side code that handles application logic.
- **Database:** A system used to store and retrieve data.
- **API:** The interface through which software systems communicate.

A typical login flow:

1. A user enters an email and password in the frontend.
2. The frontend sends a request to the backend through an API.
3. The backend checks the credentials using its authentication logic and stored user data.
4. The backend sends a success or failure response.
5. The frontend displays the result.

The API is the communication interface; it is not the same thing as the entire backend.

## 4. What is an API request?

A **request** is a message sent by a client asking a service to do something or provide information.

Examples:
- “Give me the details of student 101.”
- “Create a new account.”
- “Show me the available products.”
- “Answer this question.”

## 5. What is an API response?

A **response** is the result returned by the service.

A response may contain:
- Requested information
- A success message
- A validation error
- An authentication error
- A not-found message

For example, a request for a student who does not exist might receive a “not found” response.

## 6. What is JSON?

**JSON** stands for **JavaScript Object Notation**. It is a common format for exchanging structured data between software systems.

Example:

```json
{
  "id": 101,
  "name": "Rahul",
  "course": "Computer Science"
}
```

In this example:
- `"id"` is a key and `101` is its value.
- `"name"` is a key and `"Rahul"` is its value.
- `"course"` is a key and `"Computer Science"` is its value.

JSON supports strings, numbers, booleans, arrays, objects, and `null`.

## 7. What is FastAPI?

**FastAPI is a Python web framework used to build APIs.**

It helps developers:
- Define API endpoints
- Receive HTTP requests
- Validate incoming data
- Run Python functions to process requests
- Return structured responses
- Generate interactive API documentation

FastAPI is useful for APIs that communicate with frontends, databases, external services, and AI models.

## 8. Is FastAPI itself an API?

Not exactly.

**FastAPI is the framework used to build an API.**

| Term | Meaning |
|---|---|
| Python | The programming language |
| API | The communication interface |
| FastAPI | The framework used to build APIs |
| Backend | The server-side application and its logic |
| Database | A system for storing and retrieving data |

## 9. Python vs. FastAPI

A regular Python function:

```python
def greet():
    return "Hello!"
```

This function runs when another part of the Python program calls it. By itself, it does not create a web endpoint that a browser or frontend can request.

FastAPI can register a Python function as a handler for an HTTP request:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def greet():
    return {"message": "Hello!"}
```

For now, remember:
- **Python** lets us write the function.
- **FastAPI** registers the function to handle a particular API request.
- A **web server** runs the application so clients can reach it.

We will learn each line of this code in a later lesson.

## 10. The basic request-response flow

A common API interaction looks like this:

```text
Client (browser / React app)
           |
           | Request
           v
      FastAPI backend
           |
           | Runs application logic
           v
     Service / database / AI model
           |
           | Result
           v
      FastAPI backend
           |
           | Response
           v
Client (browser / React app)
```

This is a simplified diagram. Not every API request needs a database or an AI model.

## 11. Key points to remember

1. API means Application Programming Interface.
2. An API provides a defined way for software systems to communicate.
3. A request asks for information or an action.
4. A response returns the result.
5. JSON is a common format for exchanging structured data.
6. The frontend is what the user interacts with.
7. The backend processes application logic.
8. FastAPI is a Python framework for building APIs.
9. FastAPI is not the same thing as Python, a database, or the entire backend.
10. FastAPI can expose Python functions as handlers for HTTP requests.

## 12. Quick revision questions

Try answering these without looking at the notes:

1. What does API stand for?
2. What is the purpose of an API?
3. What is the difference between a request and a response?
4. What is JSON used for?
5. What is the difference between a frontend and a backend?
6. What is FastAPI?
7. Is FastAPI a programming language?
8. Does an API always need a database?

### Short answers

1. Application Programming Interface.
2. To provide a defined way for software systems to communicate.
3. A request asks for information or an action; a response returns the result.
4. To represent and exchange structured data.
5. The frontend is the user-facing part; the backend processes server-side logic.
6. A Python framework for building APIs.
7. No. Python is the programming language; FastAPI is a framework.
8. No. Some APIs return data or perform actions without using a database.
