# FastAPI Notes — Lesson 2: Clients, Servers, HTTP, Requests, and Responses

## 1. What are a client and a server?

- **Client:** A program that sends a request. Examples include a browser, React app, mobile app, or another backend.
- **Server:** A program that receives requests and sends responses. It runs the backend application.

A simple flow:

```text
Client
  |
  | Request
  v
Server / FastAPI application
  |
  | Response
  v
Client
```

FastAPI runs as part of the server-side application. It handles requests and connects them to the appropriate Python code.

## 2. What is HTTP?

**HTTP** stands for **Hypertext Transfer Protocol**.

It defines how clients and servers exchange requests and responses over the web.

**HTTPS** is HTTP protected by TLS encryption. It helps protect data while it travels between the client and server.

## 3. What is a URL?

A URL is an address used to locate a resource.

Example:

```text
https://api.example.com/users/25?include_orders=true
```

Parts:
- `https://` — scheme/protocol.
- `api.example.com` — host/domain.
- `/users/25` — path.
- `?include_orders=true` — query string containing a query parameter.

A local FastAPI URL may look like:

```text
http://127.0.0.1:8000/users/25
```

- `127.0.0.1` refers to the local computer.
- `8000` is the port.
- `/users/25` is the path.

## 4. Common HTTP methods

HTTP methods indicate the kind of operation a client is requesting.

| Method | Typical purpose | Example |
|---|---|---|
| GET | Read information | `GET /students` |
| POST | Submit data or create a resource | `POST /students` |
| PUT | Replace a resource representation | `PUT /students/25` |
| PATCH | Partially update a resource | `PATCH /students/25` |
| DELETE | Delete a resource | `DELETE /students/25` |

These are common conventions. The exact behavior depends on how the application implements each endpoint.

### Example: PATCH

Existing student:

```json
{
  "id": 25,
  "name": "Rahul",
  "course": "CSE"
}
```

A request to `PATCH /students/25` could contain:

```json
{
  "course": "AI & ML"
}
```

If the API supports this behavior, it updates the course without changing the name.

**Important:** The HTTP method matters. `GET /students` and `POST /students` are different operations even though the path is the same.

## 5. Parts of an HTTP request

Example:

```http
POST /students HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{
  "name": "Rahul",
  "course": "CSE"
}
```

A request commonly has these parts:

1. **Request line:** Contains the method, path, and protocol version.
2. **Headers:** Provide extra information, such as the content format or authentication credentials.
3. **Body:** May contain submitted data, such as JSON. Not every request needs a body.

You usually do not need to write raw HTTP messages when using FastAPI and an HTTP client, but understanding them helps with debugging.

## 6. Parts of an HTTP response

Example:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 25,
  "name": "Rahul",
  "course": "CSE"
}
```

A response commonly includes:
- **Status code:** Tells the client what happened.
- **Headers:** Provide metadata about the response.
- **Body:** May contain data or a message.

## 7. Common HTTP status codes

| Code | Meaning | Example |
|---|---|---|
| 200 | OK — request succeeded | Student retrieved |
| 201 | Created — resource created | Student added |
| 204 | No Content — success without a response body | Student deleted |
| 400 | Bad Request | Input rejected by application rules |
| 401 | Unauthorized — authentication is missing or invalid | Invalid login token |
| 403 | Forbidden — access is not permitted | User lacks permission |
| 404 | Not Found | Student or route does not exist |
| 405 | Method Not Allowed | Unsupported method for a path |
| 422 | Request content failed validation | Field has the wrong type |
| 500 | Internal Server Error | Unexpected server-side failure |

Status code categories:
- **1xx:** Informational.
- **2xx:** Success.
- **3xx:** Redirection.
- **4xx:** Client-side request or access errors.
- **5xx:** Server-side errors.

Start by remembering 200, 201, 400, 401, 403, 404, 422, and 500.

## 8. How a FastAPI request works

```text
1. Client sends an HTTP request
                |
                v
2. Request contains a method and path;
   it may also contain headers and a body
                |
                v
3. FastAPI matches the route
                |
                v
4. A Python function processes the request
                |
                v
5. FastAPI creates an HTTP response
                |
                v
6. Client receives the response and status code
```

For example, a React frontend can send `POST /students` with JSON. FastAPI can validate that data, call Python code, and return a response.

## 9. Key points to remember

1. A client sends requests; a server handles them and sends responses.
2. HTTP defines how requests and responses are exchanged.
3. HTTPS adds TLS encryption to HTTP communication.
4. A URL includes a host and path, and may include query parameters.
5. HTTP methods indicate the intended operation.
6. Requests may include headers and a body.
7. Responses include a status code and may include headers and a body.
8. Status codes help the client understand the result.
9. FastAPI matches requests to registered path operations and connects them to Python functions.

## 10. Quick revision questions

1. What is the difference between a client and a server?
2. What does HTTP stand for?
3. Why is HTTPS used?
4. In `/users/25`, what is the path?
5. Which method is commonly used to retrieve information?
6. What is the typical difference between PUT and PATCH?
7. What does a request body contain?
8. What does status code 404 mean?
9. What does status code 201 mean?
10. What is the basic flow of a FastAPI request?

### Short answers

1. The client sends a request; the server processes it and returns a response.
2. Hypertext Transfer Protocol.
3. It protects HTTP communication with TLS encryption.
4. `/users/25`.
5. GET.
6. PUT commonly replaces a resource representation; PATCH partially updates it.
7. Submitted data, such as JSON, when a body is used.
8. Not Found.
9. Created.
10. Client request → FastAPI route → Python logic → HTTP response → client.

