# Table-of-Contents

<!-- toc -->

- [Chapter 2: Docker Hello World](#chapter-2-docker-hello-world)
  * [Goal](#goal)
  * [Key Concepts](#key-concepts)
  * [Step-by-Step Coding Notes](#step-by-step-coding-notes)
  * [Pitfalls & Gotchas](#pitfalls--gotchas)
  * [Interview Angle](#interview-angle)
  * [Extension-Ideas](#extension-ideas)
- [Chapter 3: PostgreSQL with Docker](#chapter-3-postgresql-with-docker)
  * [Goal](#goal-1)
  * [Key Concepts](#key-concepts-1)
  * [Step-by-Step Coding Notes](#step-by-step-coding-notes-1)
  * [Pitfalls & Gotchas](#pitfalls--gotchas-1)
  * [Interview Angle](#interview-angle-1)
  * [Extension Ideas](#extension-ideas)

<!-- tocstop -->

---
## Chapter 2: Docker Hello World

### Goal

Set up a simple **Dockerized Python “Hello World” app** to understand containers before using Docker for Django.

---

### Key Concepts

* **Docker**: Tool to containerize applications (isolated environments with dependencies).
* **Image vs Container**:

  * *Image* = blueprint (recipe)
  * *Container* = running instance (actual dish)
* **Dockerfile**: Instructions to build an image.
* **docker-compose.yml**: Define & run multi-container apps.
* **Port mapping**: Expose container ports to host (`8000:8000`).

---

### Step-by-Step Coding Notes

1. **Install Docker**

   * Mac: Docker Desktop
   * Linux: `sudo apt install docker.io`

2. **Create Project Folder**

   ```bash
   mkdir helloworld && cd helloworld
   ```

3. **Create app.py**

   ```python
   print("Hello, World!")
   ```

4. **Create Dockerfile**

   ```dockerfile
   FROM python:3.9-slim

   WORKDIR /code
   COPY . /code

   CMD ["python", "app.py"]
   ```

   * `FROM` → base image (Python 3.9 slim version).
   * `WORKDIR` → working directory inside container.
   * `COPY` → copy files into container.
   * `CMD` → run when container starts.

5. **Build Image**

   ```bash
   docker build -t helloworld:latest .
   ```

6. **Run Container**

   ```bash
   docker run helloworld
   ```

   Output → `Hello, World!`

7. **Optional: Use docker-compose**

   * `docker-compose.yml`

     ```yaml
     version: "3.9"

     services:
       app:
         build: .
         command: python app.py
     ```
   * Run with

     ```bash
     docker-compose up
     ```

---

### Pitfalls & Gotchas

* Forgetting `.` at the end of `docker build` (means “current directory”).
* Containers don’t auto-refresh on code changes → need rebuild.
* Big images → always use slim versions of Python.

---

### Interview Angle

* **Q**: What’s the difference between a Docker image and a Docker container?
* **Q**: Why do we use `docker-compose` instead of plain `docker run`?
* **Q**: What’s the role of a Dockerfile in application deployment?

---

### Extension-Ideas

* Add a `requirements.txt` and install packages inside Docker.
* Expose a Flask/Django app on port `8000` instead of simple `print`.

---
## Chapter 3: PostgreSQL with Docker

### Goal

Run **PostgreSQL inside Docker** and connect it with Django (replacing the default SQLite).

---

### Key Concepts

* **PostgreSQL**: Production-grade relational database (better than SQLite).
* **Environment Variables**: Store DB credentials securely.
* **docker-compose.yml**: Can run multiple services together (Django + Postgres).
* **Volumes**: Persist database data outside container.

---

### Step-by-Step Coding Notes

1. **Update `docker-compose.yml`**

   ```yaml
   version: "3.9"

   services:
     db:
       image: postgres:13
       volumes:
         - postgres_data:/var/lib/postgresql/data/
       environment:
         - POSTGRES_DB=hello_django_dev
         - POSTGRES_USER=hello_django
         - POSTGRES_PASSWORD=hello_django

   volumes:
     postgres_data:
   ```

   * `volumes` → ensures DB data persists even if container restarts.
   * `POSTGRES_*` → sets up DB name, user, and password.

2. **Run Postgres**

   ```bash
   docker-compose up -d db
   ```

   * `-d` → detached mode.
   * Check running containers: `docker ps`.

3. **Verify Database Connection**

   ```bash
   docker-compose exec db psql --username=hello_django --dbname=hello_django_dev
   ```

   Inside psql prompt:

   ```sql
   \l     -- list databases
   \q     -- quit
   ```

4. **Install psycopg2 in Django**

   * Add to `requirements.txt`:

     ```
     psycopg2-binary>=2.8
     ```
   * Rebuild container:

     ```bash
     docker-compose build
     ```

5. **Update Django `settings.py`**

   ```python
   DATABASES = {
       "default": {
           "ENGINE": "django.db.backends.postgresql",
           "NAME": "hello_django_dev",
           "USER": "hello_django",
           "PASSWORD": "hello_django",
           "HOST": "db",
           "PORT": 5432,
       }
   }
   ```

   * Note: `HOST = "db"` matches service name in `docker-compose.yml`.

6. **Run migrations**

   ```bash
   docker-compose exec web python manage.py migrate
   ```

---

### Pitfalls & Gotchas

* Using `localhost` instead of `db` as host (must match docker-compose service name).
* Forgetting to add `psycopg2-binary` → Django won’t talk to Postgres.
* Rebuilding containers (`docker-compose build`) after changing dependencies.

---

### Interview Angle

* **Q**: Why is PostgreSQL preferred over SQLite in production?
* **Q**: What’s the role of Docker volumes for databases?
* **Q**: How does Django know where the DB is running inside Docker?

---

### Extension Ideas

* Add **pgAdmin** (GUI for Postgres) as another Docker service.
* Create multiple databases and test connecting Django to them.
* Try using environment variables via `.env` file instead of hardcoding.
