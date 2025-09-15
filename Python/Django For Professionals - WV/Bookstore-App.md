# Table-of-Contents

<!-- toc -->

- [Chapter4-Bookstore](#chapter4-bookstore)
    + [Docker](#docker)

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

#### Docker
```
# Dockerfile

FROM python:3.10.4-slim-bullseye

ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /ws3

COPY ./requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```
