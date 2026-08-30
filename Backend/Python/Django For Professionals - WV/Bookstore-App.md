# Table-of-Contents

<!-- toc -->

- [Chapter4-Bookstore](#chapter4-bookstore)
    + [Dockerfile](#dockerfile)
    + [docker-compose.yml](#docker-composeyml)
    + [.dockerignore](#dockerignore)
    + [.gitignore](#gitignore)
    + [PostgreSQL](#postgresql)
    + [Custom-User-Model](#custom-user-model)

<!-- tocstop -->

---

## Chapter4-Bookstore
```shell
cd ~/desktop/code
mkdir ch4-bookstore
cd ch4-bookstore
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install django~=4.0.0 psycopg2-binary==2.9.3
django-admin startproject django_project .
python manage.py runserver
pip freeze > requirements.txt
```

#### Dockerfile
```python
FROM python:3.10.4-slim-bullseye

ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /ws3

COPY ./requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```
#### docker-compose.yml
```python

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

#### .dockerignore
```python
.venv
.git
.gitignore
```

```shell
git init
```

#### .gitignore
```python
.venv
__pycache__/
db.sqlite3
.DS_Store
```

```shell
docker-compose up -d --build
```

#### PostgreSQL
```python
# django_project/settings.py
DATABASES = {
	"default": {
		"ENGINE": "django.db.backends.postgresql",
		"NAME": "postgres",
		"USER": "postgres",
		"PASSWORD": "postgres",
		"HOST": "db",
		"PORT": 5432,
	}
}
```

#### Custom-User-Model

- You will need to make changes to the built-in User model at some point in your project’s life and if you have not started with a custom user model from the very first migrate command you run, then you’re in for a world of hurt because User is tightly interwoven with the rest of Django internally. It is challenging to switch over to a custom user model mid-project.

There are four steps for adding a custom user model to our project:
1. Create a `CustomUser` model
2. Update `django_project/settings.py`
3. Customize `UserCreationForm` and `UserChangeForm`
4. Add the custom user model to `admin.py`
```shell
docker-compose exec web python manage.py startapp accounts
```

```python
# accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models
class CustomUser(AbstractUser):
	pass
```