# FastAPI Learning — Lesson 3
## Build Your First FastAPI Application

## What we are building

We will create a small API that returns:

```json
{
  "message": "Hello, FastAPI!"
}
```

FastAPI helps us define API routes and handle requests. A server such as Uvicorn runs the application and listens for incoming requests.

## 1. Create a project folder

Create a folder named `fastapi-learning` and open it in VS Code.

Open **Terminal → New Terminal**.

## 2. Create a virtual environment

Run:

```bash
python -m venv .venv
```

A virtual environment keeps the packages for this project separate from packages used by other projects.

On Windows PowerShell, activate it with:

```powershell
.\.venv\Scripts\Activate.ps1
```

On Windows Command Prompt, use:

```bat
.venv\Scripts\activate.bat
```

If activation works, you will usually see `(.venv)` at the start of your terminal prompt.

## 3. Install FastAPI

Run:

```bash
python -m pip install "fastapi[standard]"
```

- `pip` installs Python packages.
- `fastapi[standard]` installs FastAPI and its standard extras, including tools commonly used to run the app locally.
- Uvicorn is the server we will use to run the app.

## 4. Create `main.py`

Inside the `fastapi-learning` folder, create a file named `main.py`.

Add this code:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello, FastAPI!"}
```

## 5. Understand the code

### Line 1

```python
from fastapi import FastAPI
```

This imports the `FastAPI` class from the FastAPI package so we can use it in our program.

### Line 2

```python
app = FastAPI()
```

This creates a FastAPI application object and stores it in the variable named `app`. The name `app` is a common convention.

### Line 3

```python
@app.get("/")
```

This connects the GET HTTP method and the root URL path `/` to the function below it.

In plain English: when a client sends a GET request to the root path, FastAPI should call the function below.

The `@` syntax introduces a Python decorator. For now, remember that this line registers a route with FastAPI.

### Line 4

```python
def home():
```

This defines a function named `home`. FastAPI calls this function when a request matches the route registered above.

### Line 5

```python
return {"message": "Hello, FastAPI!"}
```

This returns a Python dictionary. FastAPI automatically converts it into a JSON response.

- `"message"` is the key.
- `"Hello, FastAPI!"` is the value.
- `return` sends the result back from the function.

## 6. Run the app

Make sure your terminal is in the project folder, your virtual environment is activated, and `main.py` is saved.

Run:

```bash
uvicorn main:app --reload
```

Meaning:

- `uvicorn`: starts the web server.
- `main`: refers to `main.py` (the `.py` is omitted).
- `app`: refers to the `app` variable inside `main.py`.
- `--reload`: automatically restarts the server when you edit the code.

When the server starts, the terminal usually shows a local address such as `http://127.0.0.1:8000`.

Keep the terminal open while testing.

## 7. Test the API

Open these addresses in your browser while the server is running:

- API endpoint: `http://127.0.0.1:8000/`
- Interactive docs: `http://127.0.0.1:8000/docs`

The root endpoint should display:

```json
{
  "message": "Hello, FastAPI!"
}
```

At `/docs`, find `GET /`, click it, select **Try it out**, and then click **Execute**. The page will show the request and response.

These addresses work on your own computer while the app is running.

## 8. What happens when a request arrives?

1. The browser sends a GET request to `/`.
2. Uvicorn receives the HTTP request and passes it to the FastAPI application.
3. FastAPI matches the GET method and `/` path to the registered route.
4. FastAPI calls `home()`.
5. The function returns a Python dictionary.
6. FastAPI converts the dictionary to JSON and sends the HTTP response back.
7. The browser displays the JSON.

## 9. Common problems

### `uvicorn` is not recognized

Check that the virtual environment is activated. You can also try:

```bash
python -m uvicorn main:app --reload
```

### `Could not import module "main"`

Make sure:
- The file is named exactly `main.py`.
- The terminal is in the folder containing `main.py`.
- The file is saved.

### The browser cannot connect

Make sure the server is still running in the terminal and that you opened the exact local address shown there.

### PowerShell blocks virtual-environment activation

If activation is blocked, use the Command Prompt activation command above, or ask for help troubleshooting the specific error message.

## 10. Practice questions

Try answering these without looking at the lesson:

1. What does `app = FastAPI()` do?
2. What does `@app.get("/")` tell FastAPI?
3. Why is the function named `home()`?
4. What does `return` do in this example?
5. What is the role of Uvicorn?
6. What does `main:app` mean in the run command?
7. What is the difference between `/` and `/docs`?

## Key takeaway

The smallest FastAPI application has three essential ideas:

- Create the app: `app = FastAPI()`
- Register a route: `@app.get("/")`
- Return a response from a function.

Next, we can learn how to accept information from a client, such as a person's name, and return a personalized response.
