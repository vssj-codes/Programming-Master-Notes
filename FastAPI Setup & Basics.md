# Table-of-Contents

<!-- toc -->

- [Setup & Install](#setup--install)
- [Basic App](#basic-app)
- [Running the App](#running-the-app)
- [Adding Data & HTML](#adding-data--html)
  * [Return HTML](#return-html)
- [Clean Up API Docs](#clean-up-api-docs)
  * [Map Multiple Paths to One Function](#map-multiple-paths-to-one-function)
  * [Hide Routes from Swagger](#hide-routes-from-swagger)

<!-- tocstop -->

---



## Setup & Install

1. Create project directory:
    
    ```bash
    mkdir fastapi_blog
    ```
    
2. Navigate into the directory:
    
    ```bash
    cd fastapi_blog
    ```
    
3. Install FastAPI:
    
    ```bash
    uv add "fastapi[standard]"
    ```
    
    Or:
    
    ```bash
    pip install "fastapi[standard]"
    ```
    

## Basic App

1. Create `main.py`
    
2. Import FastAPI:
    
    ```python
    from fastapi import FastAPI
    ```
    
3. Initialize the app:
    
    ```python
    app = FastAPI()
    ```
    
4. Add a route:
    
    ```python
    @app.get("/")
    def home():
        return {"message": "Hello World"}
    ```
    

## Running the App

Start the development server:

```bash
uv run fastapi dev main.py
```

Open:

- App → `http://127.0.0.1:8000`
    
- Swagger Docs → `http://127.0.0.1:8000/docs`
    
- ReDoc → `http://127.0.0.1:8000/redoc`
    

## Adding Data & HTML

Define dummy data at the top of `main.py`:

```python
posts = [
    {"id": 1, "title": "First Post"},
    {"id": 2, "title": "Second Post"}
]
```

Create an API endpoint:

```python
@app.get("/api/posts")
def get_posts():
    return posts
```

### Return HTML

Import:

```python
from fastapi.responses import HTMLResponse
```

Set the response class:

```python
@app.get("/", response_class=HTMLResponse)
def home():
    return "<h1>Hello World</h1>"
```

## Clean Up API Docs

### Map Multiple Paths to One Function

Stack route decorators:

```python
@app.get("/")
@app.get("/posts")
def get_posts():
    return posts
```

### Hide Routes from Swagger

Use `include_in_schema=False`:

```python
@app.get("/", include_in_schema=False)
def home():
    ...
```

This keeps the route working while excluding it from the generated API documentation.