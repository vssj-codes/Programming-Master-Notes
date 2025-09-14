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

- PostgreSQL is a database that can be used by almost any programming language. But if you think about it, how does a programming language–and they all vary in some way or another–connect to the database itself?
- The answer is via a database adapter! And that’s what Psycopg is, the most popular database adapter for Python.
-  `docker-compose down`
- Note that there are actually two versions of Pyscopg2 available: pyscopg2 and pyscopg2-binary.
- But since we are committed to Docker we can skip that step and instead just update requirements.txt with the psycopg2-binary package.
- In your text editor open the existing requirements.txt file and add `psycopg2-binary==2.9.3` to the bottom
- In the existing docker-compose.yml file add a new service called db. This means there will be two separate containers running within our Docker host: web for the Django local server and db for our PostgreSQL database
- Docker containers are ephemeral meaning when the container stops running all information is lost.
- The solution is to create a volumes mount called postgres_data and then bind it to a dedicated directory within the container at the location /var/lib/postgresql/data/. The final step is to add a trust authentication to the environment for the db. For large databases with many database users it is recommended to be more explicit with permissions, but this setting is a good choice when there is just one developer.
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
		depends_on:
			- db
	db:
		image: postgres:13
		volumes:
			- postgres_data:/var/lib/postgresql/data/
		environment:
			- "POSTGRES_HOST_AUTH_METHOD=trust"
volumes:
postgres_data:
```

```python
# django_project/settings.py
DATABASES = {
	"default": {
		"ENGINE": "django.db.backends.postgresql",
		"NAME": "postgres",
		"USER": "postgres",
		"PASSWORD": "postgres",
		"HOST": "db", # set in docker-compose.yml
		"PORT": 5432, # default postgres port
	}
}
```

```shell
docker-compose up -d --build
```
- If you refresh the Django welcome page at http://127.0.0.1:8000/ it should work which means Django has successfully connected to PostgreSQL via Docker.
```shell
docker-compose exec web python manage.py migrate 
docker-compose exec web python manage.py createsuperuser
```
- Let’s use postgresqladmin and for testing purposes set the email to `postgresqladmin@email.com` and the password to `testpass123`
- `http://127.0.0.1:8000/admin/ ` and enter in the new superuser log in information.
- Remember to stop our running container with docker-compose down.
```shell
docker-compose down
```

