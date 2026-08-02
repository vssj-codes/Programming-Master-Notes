# Table-of-Contents

<!-- toc -->



<!-- tocstop -->

---

**Setup & Install (6:17 - 7:03)**
1. Create project dir: `mkdir fastapi_blog`
2. Navigate: `cd fastapi_blog`
3. Install FastAPI & Standard: `uv add "fastapi[standard]"` (or `pip install "fastapi[standard]"`)

**Basic App (8:13 - 10:45)**
1. Create `main.py`
2. Import: `from fastapi import FastAPI`
3. Initialize: `app = FastAPI()`
4. Add Route:
   python
   @app.get("/")
   def home():
       return {"message": "Hello World"}
   

**Running the App (10:48 - 12:45)**
1. Start server: `uv run fastapi dev main.py`
2. Visit: `http://127.0.0.1:8000`
3. Auto-Docs: Visit `http://127.0.0.1:8000/docs` (Swagger) or `/redoc`

**Adding Data & HTML (15:27 - 19:15)**
1. Define dummy list of dicts (e.g., `posts = [...]`) at top of `main.py`
2. Create API endpoint for JSON:
   python
   @app.get("/api/posts")
   def get_posts():
       return posts
   
3. Add HTML response:
   * Import: `from fastapi.responses import HTMLResponse`
   * Modify route decorator: `@app.get("/", response_class=HTMLResponse)`
   * Return string with HTML tags.

**Clean Up Docs (19:37 - 21:30)**
1. Stack decorators to map multiple paths: `@app.get("/")` then `@app.get("/posts")` above same function
2. Hide HTML routes from Swagger docs: Add `include_in_schema=False` inside `@app.get` decorator.