<!---
{
  "id": "c1f2cd2b-3ffc-44a8-86b1-111f9d246c10",
  "teaches": "Building Your First RESTful API with FastAPI",
  "depends_on": [
    "a0f6c77d-9645-4e6c-80dc-a80608786266",
    "2d1d315d-bb92-48c0-b19f-19529a45e5ff"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2025-04-07",
  "keywords": ["python", "webtechnology", "REST", "FastAPI"]
}
--->

# Building Your First RESTful API with FastAPI

## Introduction

**FastAPI** is a library for Python 3 that supports the construction of **RESTful APIs**—web services that communicate using standard [HTTP](github.com/STEMgraph/missing) methods. The library reduces boilerplate and encourages the use of Python’s type annotations and asynchronous programming model.

In a FastAPI project, the central object is an instance of the `FastAPI` class, often named `app`. This object serves as the application runtime and receives HTTP requests from the network. A typical FastAPI program defines *endpoints* by associating Python functions with HTTP methods and paths. This association is established using decorators such as `@app.get("/")` or `@app.post("/item")`.

An **endpoint** in this context is a defined location (URI) in the web application that accepts requests using a specific HTTP method. For example, the combination of the path `/item` and the method `POST` defines a unique endpoint. The function attached to this endpoint is responsible for processing the request and producing a response.

Internally, the FastAPI object conforms to the **ASGI** (Asynchronous Server Gateway Interface) specification. ASGI is the standard interface between Python web applications and web servers, supporting both synchronous and asynchronous communication.

Because FastAPI does not include a web server, we use an external ASGI-compatible server to run the application. In this tutorial, we use **Uvicorn**, a lightweight server based on `uvloop` and `httptools`. Uvicorn binds to a TCP/IP port, listens for incoming HTTP requests, parses them into ASGI messages, and passes them to the FastAPI application. The FastAPI app evaluates the request, dispatches it to the correct function (based on the method and path), and generates a response. This response is then returned to Uvicorn, which sends it back to the client.

Other ASGI servers such as Daphne and Hypercorn are also available, but Uvicorn is commonly used with FastAPI due to its simplicity and efficiency.

The following diagram outlines the flow of an HTTP request through the system:

```
Client (HTTP Request)
        |
        v
Uvicorn (ASGI Server) -- parses HTTP, passes ASGI message -->
        |
        v
FastAPI app (ASGI App) -- matches route, processes function -->
        |
        v
Uvicorn (sends HTTP Response)
        |
        v
Client (Receives Response)
```


## Tasks
0. **Ensure a proper `python3` Version**: Run `python3 --version` on your host-machine. If your version of python is older than **3.12** please consider upgrading with your package manager for this exercise:
```bash
sudo apt update
sudo apt install python3.12 python3.12-venv
```

1. **Install FastAPI and Uvicorn**\
   To begin working with FastAPI, you need to install the necessary libraries using Python's package manager. FastAPI itself provides the framework, while Uvicorn is an ASGI server used to run the application.
   Create a new project directory first, start a [virtual environment](https://github.com/STEMgraph/2d1d315d-bb92-48c0-b19f-19529a45e5ff), source it and install the libraries: 

   ```bash
   mkdir ~/my_first_api && cd ~/my_first_api
   python3.12 -m venv fastapi_environment
   source ./fastapi_environment/bin/activate
   pip install fastapi uvicorn
   ```

2. **Minimal construct of a FastAPI app**\
   Create a file named `main.py` and add the following code:

   ```python
   from fastapi import FastAPI

   app = FastAPI()

   @app.get("/")
   def read_root():
       return {"message": "Hello World"}
   ```

   This script creates an `app` instance of the `FastAPI` class and defines a single route (`/`) that responds to HTTP GET requests. The route returns a JSON object with a simple greeting.

3. **Start the FastAPI application using Uvicorn**\
   Run the application with the following command:

   ```bash
   uvicorn main:app --reload
   ```

   Here's what happens:

   - `main` refers to the filename `main.py` (without the `.py` extension). If you called your file differently, you'll have to use the different name here as well.
   - `app` refers to the `FastAPI()` object defined in the script. If you called your object differently, you'll have to use the different name here as well.
   - `--reload` enables automatic reloading of the server when the source code changes—this is useful during development but omitted when running in production.

   Uvicorn starts a lightweight web server that binds to the local address `127.0.0.1` and listens on port `8000` by default. This means the server can be accessed from your local machine using `http://127.0.0.1:8000`.

   - `127.0.0.1` is the loopback interface, i.e., the network interface of your own machine. It restricts access to local connections only.
   - If you used `0.0.0.0` instead, the server would accept requests from all network interfaces, including remote machines (not recommended for development).
   - Port `8000` is an arbitrary, non-privileged port commonly used for web development. It can be changed with the `--port` option.

4. **Call a Hello World GET endpoint with curl**\
   Once the server is running, you can send an HTTP GET request to your root endpoint using [`cURL`](github.com/STEMgraph/e8add8e9-7a67-4b50-af89-6c1ce6558e0d):

   ```bash
   curl http://127.0.0.1:8000/
   ```

   This sends a request to the FastAPI app, which responds with the JSON object `{"message": "Hello World"}`.

5. **Create and call a Hello {name} POST endpoint with curl**\
   Extend your `main.py` with the following code:

   ```python
   from pydantic import BaseModel

   class NameRequest(BaseModel):
       name: str

   @app.post("/hello")
   def greet_name(request: NameRequest):
       return {"message": f"Hello, {request.name}"}
   ```

   This defines a POST endpoint `/hello` that accepts a JSON payload with a `name` field. The `BaseModel` from Pydantic automatically validates the incoming data.

   Test it using:

   ```bash
   curl -X POST http://127.0.0.1:8000/hello \
        -H "Content-Type: application/json" \
        -d '{"name": "Alice"}'
   ```

   The response will be:

   ```json
   {"message": "Hello, Alice"}
   ```

6. **Open the /docs route in a browser**\
   FastAPI automatically provides an interactive API documentation interface based on the OpenAPI specification. Navigate to:

   ```
   http://127.0.0.1:8000/docs
   ```

   This interface lets you test your API by filling out forms and submitting requests directly from the browser.


## Questions

1. What is the purpose of the `FastAPI()` object?
2. What protocol does a webserver use to call a FastAPI route?
3. Why is an external server like Uvicorn necessary?

## Advice

- Use `--reload` during development to automatically restart the server on code changes.
- Keep endpoints small and focused—one function should handle one route.
- Validate incoming data using Pydantic models to avoid manual parsing.
- Start with `curl` for simple testing, then progress to writing automated tests.
- Use the interactive docs (`/docs`) to experiment with your API without writing test scripts.

