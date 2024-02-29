Ok, so here we are finally going to start playing with some SQL.

What we are going to do is that we are going to add a user table and a complaints table. 
Then we are going to put some canned complaints into the table, so that we can see them. Finally 
we are going to create a user interface to add new complaints.

We'll create an "example" user with username `example` and password `example` as well, but given that our login function
only accepts `admin` as a user, we wont be able to test this (yet).

Ok so here is what you need to do. Cteate a new file `sql.py` in the mysite folder and add the following to it

```python

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

You will create a console, and in your home folder (the folder above `mysite`, run `python mysite/sql.py`. This should create
a database file `sample.db` in this home folder.
