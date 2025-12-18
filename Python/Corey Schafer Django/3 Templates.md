## Python Django Tutorial: Full-Featured Web App Part 3 - Templates - YouTube

### Key Insights

- **Introduction to Templates**
  - Templates are used to render complex HTML in Django.
  - They help avoid repetitive HTML in views.

- **Setting Up Templates**
  - Create a `templates` directory within the app directory.
  - Inside `templates`, create a subdirectory with the same name as the app.

- **Creating Template Files**
  - Example: `home.html` and `about.html`.
  - Use basic HTML structure in these files.

- **Updating Views to Use Templates**
  - Use the `render` function from `django.shortcuts` to render templates.
  - Example:
    ```python
    from django.shortcuts import render

    def home(request):
        return render(request, 'blog/home.html')
    ```

- **Adding blogConfig to settings.py**
  - Add the blog app configuration to the list of installed apps in `settings.py`.
  - Example:
    ```python
    # settings.py
    INSTALLED_APPS = [
        ...
        'blog.apps.BlogConfig',
        ...
    ]
    ```

- **Passing Data to Templates**
  - Create context data in the views and pass it to the template.
  - Example:
    ```python
    def home(request):
        posts = [
            {
                'author': 'CoreyMS',
                'title': 'Blog Post 1',
                'content': 'First post content',
                'date_posted': 'August 27, 2018'
            },
            {
                'author': 'Jane Doe',
                'title': 'Blog Post 2',
                'content': 'Second post content',
                'date_posted': 'August 28, 2018'
            }
        ]
        context = {
            'posts': posts
        }
        return render(request, 'blog/home.html', context)
    ```

- **Using Template Variables and Loops**
  - Access variables using `{{ variable_name }}`.
  - Use `{% for item in items %}` and `{% endfor %}` for loops.
  - Example:
    ```html
    {% for post in posts %}
      <h1>{{ post.title }}</h1>
      <p>by {{ post.author }} on {{ post.date_posted }}</p>
      <p>{{ post.content }}</p>
    {% endfor %}
    ```

- **Template Inheritance**
  - Create a base template (`base.html`) for common HTML structure.
  - Use `{% block block_name %}{% endblock %}` in `base.html`.
  - Child templates extend the base template using `{% extends 'base.html' %}`.
  - Example `base.html`:
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>{% block title %}My Blog{% endblock %}</title>
    </head>
    <body>
        <header>{% include 'blog/nav.html' %}</header>
        <div class="container">
            {% block content %}{% endblock %}
        </div>
    </body>
    </html>
    ```

- **Adding Bootstrap for Styling**
  - Include Bootstrap CSS and JavaScript in `base.html`.
  - Example:
    ```html
    <head>
        ...
        <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" rel="stylesheet">
    </head>
    <body>
        ...
        <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
    </body>
    ```

- **Using Static Files**
  - Create a `static` directory within the app for CSS and JavaScript files.
  - Load static files in templates using `{% load static %}`.
  - Example:
    ```html
    {% load static %}
    <link href="{% static 'blog/main.css' %}" rel="stylesheet">
    ```

### Necessary Code Snippets

- **Views Using Templates**
  ```python
  from django.shortcuts import render

  def home(request):
      posts = [
          {
              'author': 'CoreyMS',
              'title': 'Blog Post 1',
              'content': 'First post content',
              'date_posted': 'August 27, 2018'
          },
          {
              'author': 'Jane Doe',
              'title': 'Blog Post 2',
              'content': 'Second post content',
              'date_posted': 'August 28, 2018'
          }
      ]
      context = {
          'posts': posts
      }
      return render(request, 'blog/home.html', context)

  def about(request):
      return render(request, 'blog/about.html', {'title': 'About'})
  ```

- **home.html**
  ```html
  {% extends "blog/base.html" %}
  {% block content %}
    {% for post in posts %}
      <article class="media content-section">
        <div class="media-body">
          <div class="article-metadata">
            <a class="mr-2" href="#">{{ post.author }}</a>
            <small class="text-muted">{{ post.date_posted }}</small>
          </div>
          <h2><a class="article-title" href="#">{{ post.title }}</a></h2>
          <p class="article-content">{{ post.content }}</p>
        </div>
      </article>
    {% endfor %}
  {% endblock %}
  ```

- **about.html**
  ```html
  {% extends "blog/base.html" %}
  {% block content %}
    <h1>About Page</h1>
  {% endblock %}
  ```

- **base.html**
  ```html
  <!DOCTYPE html>
  <html>
  <head>
      <title>{% block title %}My Blog{% endblock %}</title>
      <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" rel="stylesheet">
      {% load static %}
      <link href="{% static 'blog/main.css' %}" rel="stylesheet">
  </head>
  <body>
      <header>
          <nav class="navbar navbar-expand-lg navbar-light bg-light">
              <a class="navbar-brand" href="{% url 'blog-home' %}">My Blog</a>
              <div class="collapse navbar-collapse">
                  <ul class="navbar-nav mr-auto">
                      <li class="nav-item">
                          <a class="nav-link" href="{% url 'blog-home' %}">Home</a>
                      </li>
                      <li class="nav-item">
                          <a class="nav-link" href="{% url 'blog-about' %}">About</a>
                      </li>
                  </ul>
              </div>
          </nav>
      </header>
      <div class="container">
          {% block content %}{% endblock %}
      </div>
      <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
      <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
  </body>
  </html>
  ```

- **main.css**
  ```css
  body {
      padding-top: 56px;
  }
  .content-section {
      margin-bottom: 1.5rem;
  }
  .article-metadata {
      margin-bottom: 0.5rem;
  }
  .article-title {
      font-size: 1.25rem;
      font-weight: bold;
      margin-bottom: 0.75rem;
  }
  .article-content {
      white-space: pre-wrap;
  }
  ```