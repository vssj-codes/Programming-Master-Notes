# Table-of-Contents

<!-- toc -->

- [Chapter4-Bookstore](#chapter4-bookstore)
    + [Dockerfile](#dockerfile)
    + [docker-compose.yml](#docker-composeyml)
    + [.dockerignore](#dockerignore)
    + [.gitignore](#gitignore)

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