## Giving the user the ability to login

This tutorial is based on one at Real Python, [here](https://realpython.com/introduction-to-flask-part-2-creating-a-login-page/).

`_navbar.html` in `includes` folder is the file that contains the navigation bar. It is included in every page. It is a good place to put links to the main pages of the documentation.

To this we will add:

```html
        <li class="nav-item">
          <a class="nav-link" href="/login">Login</a>
        </li>
```

For this login page we will need to create a `login.html` template, and a flask route `/login`

In the templates folder, create a new file `login.html`, and in in the following:

```html
{% extends 'layout.html' %}

{% block body %}
    <h1>Please login</h1>
    <br>
    <div class="container">
    <form action="" method="post">
        <input type="text" placeholder="Username" name="username" value="{{request.form.username }}">
         <input type="password" placeholder="Password" name="password" value="{{request.form.password }}">
        <input class="btn btn-default" type="submit" value="Login">
    </form>
    {% if error %}
        <p class="error"><strong>Error:</strong> {{ error }}</p>
    {% endif %}
    {% for message in get_flashed_messages() %}
        {{ message }}
    {% endfor %}
    </div>
{% endblock %}
```

At the top of flask_app.py do the following:

```python
from flask import redirect, url_for, request
```

And add the following route:

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != 'admin' or request.form['password'] != 'admin':
            error = 'Invalid Credentials. Please try again.'
        else:
            return redirect(url_for('index'))
    return render_template('login.html', app_data=app_data, error=error)
```

Now click the login link created, and go to the login page. Use the username `admin` and put a junk password in. It will show the error message and take you back to the login page. Then use the correct password `admin` and it will redirect to the index page.

The app is quite crappy: for one, we hardcoded the password, and for two, we are not giving the username at the top once we login. Thirdly, we cant log out.

Also we should be using a database to store the user information, not hardcoding it. Also we want the user to be able to complain using the database, not just login and logout.

## Logging out and tracking user sessions

Add a secret key to the app:

```python
# config
app.secret_key = 'my precious' # tell me you have seen Lord of The Rings
```

Then we'll require that the service page only be shown to logged in users. 

```python
from functools import wraps
# login required decorator
def login_required(f):
    @wraps(f)
    def wrap(*args, **kwargs):
        if 'logged_in' in session:
            return f(*args, **kwargs)
        else:
            flash('You need to login first.')
            return redirect(url_for('login'))
    return wrap
```

and we'll put this decorator on the service route

```python
@app.route("/service")
@login_required
def service():
    return render_template("service.html", app_data=app_data)
```

Lets add a session to login and add a logout route:

Imports:

```python
from flask import session, flash
```

New login function, notice the addition of the session:

```python
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != 'admin' or request.form['password'] != 'admin':
            error = 'Invalid Credentials. Please try again.'
        else:
            session['logged_in'] = True
            flash('You were logged in.')
            return redirect(url_for('index'))
    return render_template('login.html', app_data=app_data, error=error)
```

```python
@app.route('/logout')
@login_required
def logout():
    session.pop('logged_in', None)
    flash('You were logged out.')
    return redirect(url_for('index'))
```

Add the following to `login.html` after the part about the error:

```html
{% for message in get_flashed_messages() %}
        {{ message }}
{% endfor %}
```
We'll also add a logout link to the navbar.

```html
        <li class="nav-item">
          <a class="nav-link" href="/logout">Logout</a>
        </li>
```

## Cleaning up

Now we have a session, and you can log out of it. But I think you will agree that while this is quite
nice, our navigation is quite wonky:

1. If you login from the service page, you are taken to the index page, not the service page.
2. Login and Logout both show up in the navbar, which is not what we want. When we are logged in, we want to see the logout link, and when we are logged out, we want to see the login link.

How do we know we are logged in? We can check the session. This change can me made in the navbar:

```html
        {% if "logged_in" not in session %}
        <li class="nav-item">
          <a class="nav-link" href="/login">Login</a>
        </li>
        {% else %}
        <li class="nav-item">
          <a class="nav-link" href="/logout">Logout</a>
        </li>
        {% endif %}
```

Ok so we now have a nice login and logout system. As a home exercise maybe replace the "login"
in the navbar my the user's name as you see on many websites? Create a new page for the user and a new route for the user where login if required. On that page, show the user's name, and say this is their profile page.

We will fix the service page next, and general redirection next!
