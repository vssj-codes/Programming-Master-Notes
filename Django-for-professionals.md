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
  * [Pitfalls & Gotchas](#pitfalls--gotchas-1)
  * [Interview Angle](#interview-angle-1)
  * [Extension Ideas](#extension-ideas)
  * [Notes](#notes)

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
```
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

### Notes
- One of the most immediate differences between working on a “toy app” in Django and a
production-ready one is the database.
- Django ships with built-in support for five databases: PostgreSQL, MariaDB, MySQL, Oracle, and SQLite
- The Django ORM handles the translation from Python code to SQL configured for each
database automatically for us which is quite amazing if you think about it

``` shell
cd ~/desktop/code
mkdir ch3-postgresql
python3.10 -m venv .venv
source .venv/bin/activate
python3.10 -m pip install django~=4.0.0
django-admin startproject django_project .
python manage.py migrate
python manage.py runserver
# Confirm everything worked by navigating to http://127.0.0.1:8000/

pip freeze > requirements.txt
deactivate
```

- Docker
  - To switch over to Docker first deactivate our virtual environment `deactivate`
```python
# Dockerfile at project level
FROM python:3.10.4-slim-bullseye
# Set environment variables
ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
# Set work directory
WORKDIR /code
# Install dependencies
COPY ./requirements.txt .
RUN pip install -r requirements.txt
# Copy project
COPY . .
  ```
  - .dockerignore
  ```
  .venv 
  .git 
  .gitignore
  ```
- Docker will automatically check if it can use the cached results of previous builds.
- This caching means that the order of a Dockerfile is important for performance reasons. In order to avoid constantly invalidating the cache we start the Dockerfile with commands that are less likely to change while putting commands that are more likely to change, like COPYing the local filesystem, at the end.

```python 
# docker-compose.yml
version: "3.9"
services:
	web:
		build: .
		command: python /code/manage.py runserver 0.0.0.0:8000
		volumes:
		- .:/code
		ports:
		- 8000:8000
```

- Detached mode runs containers in the background, which means we can use a single command line tab without needing a separate one open as well. This saves us from switching back and forth between two command line tabs constantly. The downside is that if/when there is an error, the output won’t always be visible. So if your screen does not match this book at some point, try typing docker-compose logs to see the current output and debug any issues
```shell
docker-compose up -d
```

- Since we’re working within Docker now as opposed to locally we must preface traditional commands with `docker-compose exec <service>` where we specify the name of the service. For example, to create a superuser account instead of typing `python manage.py createsuperuser` the updated command would now look like the line below, using the web service.
``` shell
  docker-compose exec web python manage.py createsuperuser
  ```

