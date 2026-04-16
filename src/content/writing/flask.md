---
title: "Flask"
description: "I'm cooked for computing. (Big thanks to Sonnet 4.6)"
pubDate: "2026-04-16"
---

## 1. Why Flask? (The Big Picture)

Before Flask, writing a dynamic web server in Python meant manually handling raw sockets, parsing HTTP requests, and building HTML strings — messy and error-prone. **Flask is a web framework** that handles all of that boilerplate for you, letting you focus on what your app actually does.

---

## 2. Creating & Running a Flask App

Every Flask app needs these two lines:

```python
import flask
app = flask.Flask(__name__)   # Creates the app

if __name__ == '__main__':
    app.run()                  # Starts the server at http://127.0.0.1:5000/
```

- `__name__` tells Flask where your files are.
- `app.run(port=12345)` changes the port.
- `app.run(debug=True)` enables auto-reload on file changes.

---

## 3. Routing — Mapping URLs to Functions

**Routing** is how Flask decides which Python function to run for each URL. You use the `@app.route()` **decorator** above a function.

```python
@app.route('/')
def home():
    return 'Welcome'

@app.route('/about')
def about():
    return 'About page'
```

### Key routing rules:
| Feature | Example | Notes |
|---|---|---|
| Trailing slash (with `/`) | `@app.route('/page/')` | Matches both `/page` and `/page/` |
| Trailing slash (without `/`) | `@app.route('/page')` | Only matches `/page` exactly |
| Multiple paths → same function | Stack decorators | All three paths run one function |
| Case-sensitive | `/CSS` ≠ `/css` | Always lowercase |

### Variable Routes

You can capture parts of the URL as variables:

```python
@app.route('/greet/<name>/')        # <name> = any string (no slashes)
def greet(name):
    return 'Hello, ' + name

@app.route('/user/<int:id>/')       # <int:id> = digits only, converted to int
def user_profile(id):
    return 'User #' + str(id)
```

Converters: `<str:x>` (default), `<int:x>` (digits only), `<float:x>`.

### Routing by HTTP Method

By default, routes only accept **GET**. To allow POST:

```python
@app.route('/submit/', methods=['POST'])
def submit():
    return 'Received!'
```

---

## 4. Generating URLs with `url_for()`

Instead of hardcoding paths, use `url_for()` to look up the URL for a function by name:

```python
from flask import url_for
url_for('greet', name='Alex')   # Returns '/greet/Alex/'
url_for('home')                  # Returns '/'
```

This is future-proof — if you change a route's path, `url_for()` still works everywhere.

---

## 5. HTTP Responses

Flask functions return responses. By default the status code is `200 OK` and content-type is `text/html`.

```python
# Just a string (status 200 assumed):
return 'Hello!'

# Custom status code:
return ('Error!', 500)

# Custom headers too:
return ('Plain text', 200, {'Content-Type': 'text/plain'})
```

### Redirects

```python
from flask import redirect, url_for

return redirect('http://example.com')          # External redirect
return redirect(url_for('home'))                # Internal redirect
```

---

## 6. Templates (Jinja2)

**Never build HTML by concatenating strings** — it's messy and allows HTML injection attacks. Instead, use **templates**.

Flask uses the **Jinja2** template engine. Templates go in a `templates/` subfolder.

### Basic template (`templates/greet.html`):
```html
<!DOCTYPE html>
<html>
<body>
  <h1>Hello, {{ visitor }}!</h1>
</body>
</html>
```

### Rendering from Python:
```python
from flask import render_template

@app.route('/greet/<n>/')
def greet(name):
    return render_template('greet.html', visitor=name)
```

The keyword argument `visitor=name` passes the Python value `name` into the template as the Jinja2 variable `visitor`.

### Key Jinja2 Syntax:
| Feature | Syntax | Purpose |
|---|---|---|
| Output a value | `{{ variable }}` | Replaces with value, auto-escapes HTML |
| Raw HTML (trusted only) | `{{ variable\|safe }}` | Treats value as HTML |
| Length | `{{ name\|length }}` | Like Python's `len()` |
| If statement | `{% if x %} ... {% endif %}` | Conditional rendering |
| Else / elif | `{% elif x %} ... {% else %}` | As expected |
| For loop | `{% for item in list %} ... {% endfor %}` | Repeat a section |
| URL generation | `{{ url_for('func_name') }}` | Use inside templates |

> ⚠️ Jinja2 is **not Python** — `len()` and many Python functions don't work directly.

### Example — for loop in a template:
```html
<table>
{% for subject in results %}
  <tr>
    <td>{{ subject }}</td>
    <td>{{ results[subject] }}</td>
  </tr>
{% endfor %}
</table>
```

---

## 7. Static Files (CSS, Images)

Put unchanging files (CSS, images) in a `static/` subfolder. Flask automatically serves them at `/static/`.

In your template, reference them using `url_for`:
```html
<link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}">
```

---

## 8. Processing Form Data

### GET Forms
Data appears in the URL (e.g. `/process?name=Alice`). Accessed via `request.args`:

```python
from flask import request

s = request.args['s']   # Get value of field named 's'
```

### POST Forms
Data is hidden from the URL. Use `method="post"` in the HTML form, and access via `request.form`:

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'GET':
        return render_template('form.html')
    s = request.form['s']       # Get POSTed value
    return render_template('result.html', s=s)
```

> Always check `if 's' in request.form` before using it — users can send incomplete data.

### Why use POST instead of GET for forms?
1. Data is hidden from the URL (not visible in browser history)
2. GET shouldn't be used for actions that change server data
3. URLs have length limits — POST handles larger data

---

## 9. File Uploads

The form needs `method="post"` and `enctype="multipart/form-data"`:

```html
<form method="post" enctype="multipart/form-data">
  <input name="photo" type="file">
  <input type="submit">
</form>
```

In Python, uploaded files come from `request.files`:

```python
from flask import request
from werkzeug.utils import secure_filename

photo = request.files['photo']
filename = secure_filename(photo.filename)   # Sanitise the filename!
photo.save('uploads/' + filename)
```

To serve uploaded files back:

```python
from flask import send_from_directory

@app.route('/photos/<filename>')
def get_file(filename):
    return send_from_directory('uploads', filename)
```

> ⚠️ Always use `secure_filename()` — without it, attackers could upload files to dangerous paths like `../../config.py`.

---

## 10. Quick Reference — Flask Objects

| Object/Function | What it does |
|---|---|
| `flask.Flask(__name__)` | Creates the app |
| `app.run()` | Starts the server |
| `@app.route('/path')` | Maps a URL to a function |
| `render_template('file.html', x=val)` | Renders a Jinja2 template |
| `redirect(url)` | Sends the browser to another URL |
| `url_for('function_name')` | Gets the URL for a function |
| `request.method` | `'GET'` or `'POST'` |
| `request.args` | GET query string data (dict) |
| `request.form` | POST form data (dict) |
| `request.files` | Uploaded files (dict) |
| `send_from_directory(folder, file)` | Safely serves a file |
| `secure_filename(name)` | Sanitises an uploaded filename |

---

## 11. Folder Structure

A typical Flask project looks like this:

```
myapp/
├── app.py
├── templates/
│   ├── home.html
│   └── greet.html
├── static/
│   └── styles.css
└── uploads/
    └── (uploaded files go here)
```

---