Ok, so here we are finally going to start playing with some SQL.

What we are going to do is that we are going to add a user table and a complaints table. 
Then we are going to put some canned complaints into the table, so that we can see them. Finally 
we are going to create a user interface to add new complaints.

We'll create an "example" user with username `example` and password `example` as well, but given that our login function
only accepts `admin` as a user, we wont be able to test this (yet).

Ok so here is what you need to do. Cteate a new file `sql.py` in the mysite folder and add the following to it

```python
import sqlite3

from datetime import datetime

# Connect to SQLite database (or create it if it doesn't exist)
conn = sqlite3.connect('sample.db')
c = conn.cursor()

# Create 'users' table
c.execute('''
CREATE TABLE IF NOT EXISTS users (
    userid INTEGER PRIMARY KEY,
    username TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL
)
''')

# Create 'complaints' table
c.execute('''
CREATE TABLE IF NOT EXISTS complaints (
    complaint_id INTEGER PRIMARY KEY,
    user_id INTEGER,
    complaint TEXT,
    time DATETIME,
    FOREIGN KEY(user_id) REFERENCES users(userid)
)
''')

# Insert 'admin' user
c.execute('INSERT OR IGNORE INTO users (username, password) VALUES (?, ?)', ('admin', 'admin'))

# Insert 'example' user
c.execute('INSERT OR IGNORE INTO users (username, password) VALUES (?, ?)', ('example', 'example'))

# Get 'example' user id
c.execute('SELECT userid FROM users WHERE username = ?', ('example',))
example_user_id = c.fetchone()[0]

c.execute('SELECT userid FROM users WHERE username = ?', ('admin',))
admin_user_id = c.fetchone()[0]

# Insert example complaints
complaints = [
    (example_user_id, 'Complaint 1 lorem ipsum', datetime.now()),
    (example_user_id, 'Complaint 2 let us run a fast mile', datetime.now()),
    (admin_user_id, 'Complaint 3 let us run a fast mile', datetime.now()),
]

c.executemany('INSERT INTO complaints (user_id, complaint, time) VALUES (?, ?, ?)', complaints)

# Commit the changes and close the connection
conn.commit()
conn.close()

print("Database and tables created successfully with initial data.")
```

Full disclosure: chaptgpt helped me create this file. As long as you can correct chatgpt you should feel free to use it.

You will create a `bash` console, and in your home folder (the folder above `mysite`, run `python mysite/sql.py`. This should create
a database file `sample.db` in this home folder.

Ok, now switch to `service.html` in the templates folder and replace it thus:

```html
{% extends 'layout.html' %}

{% block body %}
    <h1>{{ app_data['description'] }}</h1>
    <h2>Services page</h2>
    <br/>
    <h3>Make a complaint</h3>
    <div class="container">
    <form action="" method="post">
        <textarea rows="10" cols="50" name="complaint"></textarea>
        <input class="btn btn-default" type="submit" value="Complain"/>
    </form>
    </div>
    <br/>
    <h3>Previous Complaints</h3>
    {% for complaint in complaints %}
    <hr/>
    <p>{{ complaint['complaint'] }} </p>
    <strong>Posted at:</strong> {{ complaint['time'] }} <br>
    {% endfor %}
{% endblock %}
```

Notice that here we are providing the additional dictionary `complaints`. Clearly we need to change the service function and
route in the main file. Let us do that.

In `flask_app.py` do the following:

(1) First import the `g` or global object from flask. Also import `sqlite3` and `datetime`:

```python
from flask import g
import sqlite3
from datetime import datetime
```

(2) Create a way to connect to the database:

```python
# connect to database
def connect_db():
    return sqlite3.connect('sample.db')
```

The app runs from your home folder, and this is why we create the `sample.db` there. (I did not know this ahead of time so i experimented a bit to find this out).

Next, add a database in the app config:

```python
app.database = 'sample.db'
```

You could use this in the `connect_db` function above if you like.

Now change the entire `service` route and app to look like this:

```python
@app.route("/service", methods=['GET', 'POST'])
@login_required
def service():
    g.db = connect_db()
    username = session['username']
    cur = g.db.execute('SELECT userid FROM users WHERE username = ?', (username,))
    user_id = cur.fetchone()[0]
    if request.method == 'POST':
        complaint = request.form['complaint']
        thetime = datetime.now()
        g.db.execute('INSERT INTO complaints (user_id, complaint, time) VALUES (?, ?, ?)', (user_id, complaint, thetime,))
        g.db.commit()

    cur2 = g.db.execute('select * from complaints WHERE user_id = ?;', (user_id,))
    complaints = [dict(time=row[3], complaint=row[2]) for row in cur2.fetchall()]
    g.db.close()
    return render_template("service.html", app_data=app_data, complaints=complaints)
```

Ok you are done with all the changes. Save, reload, and enjoy being able to add new complaints as the "admin" user.

In the next step we'll use the `users` table to implement registration and extend this functionality of complaining to all users.





