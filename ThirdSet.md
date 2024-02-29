Clearly you want to be able to have users other than admin if you want there to be complaints from regular users. The
simple way to do this is to allow users to register.

In a usual registration system you would also capture the email and make sure the user exists (and is not a bot) and responds to email.
We wont do this here. Sending email is more complex and is best handled by existing packages and an account at something like sendgrid
or mailgun. If you do any marketing you will learn loads about this.

Here our resistration page will be very similar to our login page. Indeed, we'll log you in when you register. We'll also save your password
in plain text in the database for simplicity, something that is a complete no-no for a production system.

You'll create a `register.html` template:

```html
{% extends 'layout.html' %}

{% block body %}
    <h1>Please Register</h1>
    <br>
    <div class="container">
    <form action="" method="post">
        <input type="text" placeholder="Username" name="username">
         <input type="password" placeholder="Password" name="password">
        <input class="btn btn-default" type="submit" value="Register">
    </form>
    {% for message in get_flashed_messages() %}
        {{ message }}
    {% endfor %}
    </div>
{% endblock %}
```

and a new `register` route in `flask_app.py`:

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
    g.db = connect_db()
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        g.db.execute('INSERT OR IGNORE INTO users (username, password) VALUES (?, ?)',  (username, password,))
        g.db.commit()
        session['logged_in'] = True
        session['username'] = request.form['username']
        flash('You were logged in.')
        return redirect(url_for('index'))
    return render_template('register.html', app_data=app_data)
```

and finally add a "Register link" in the `_navbar.html` include in the `includes` folder...just add 
it at the sample lace as the login for obvious reasons...

```html
<li class="nav-item">
  <a class="nav-link" href="/login">Login</a>
</li>
<li class="nav-item">
  <a class="nav-link" href="/register">Register</a>
</li>
```

And thats it. You have a functioning registration/login system. 

Now there is a bug of too many flashed messages that you should fix, and you should build out the profile page bettar. And oh, atleast
do the right thing with passwords (google up on registering and passwords in flask).
