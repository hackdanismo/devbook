# DevBook
A collection of developer notes, snippets and code examples.

+ [Python](#python)
  + [Functions](#functions)
  + [Flask](#flask)
    + [Hello World](#hello-world)
    + [Route](#route)
    + [Connect Flask to SQLite](#connect-flask-to-sqlite)
+ [Database](#database)
  + [SQLite](#sqlite)
    + [ORM](#orm)
    + [Create Database](#create-database)
    + [Create Database Table](#create-database-table)
 
## Python

### Functions
A `function` in `Python` is a reusable block of code that performs a task.

```python
def function():
  return "This is a function"

# Call the function to execute the code
function()
```

+ `def` - a keyword to define a function
+ `function` - this is the name of the function, does not have to be "function", can be named whatever you wish
+ `return` - send back the value to where the function was called

### Flask
We first need to check if `Python` and `pip` are installed. Open the terminal and enter the command:

```shell
$ python --version
# Or, sometimes:
$ python3 --version

# Check pip version
$ pip --version
# Or, sometimes:
$ pip3 --version
```

The best practice is to create a directory and then create a `virtual environment` so there's no clash with different `python` versions:

```shell
$ mkdir flask-app
$ cd flask-app

# Create the virtual environment
$ python3 -m venv venv

# Activate the virtual environment (Windows)
$ .\venv\Scripts\activate
# Activate the virtual environment (macOS)
$ source venv/bin/activate

# When ready, deactivate the virtual environment once finished
$ deactivate
```

It is worth adding a `.gitignore` file to the project folder to prevent the `venv` folder being added to version control.

```
venv/
.venv/
```

When the virtual environment has been created, this will generate a `venv` folder in the project folder.

Once activated, the shell should read something like: `(venv) danieljackson@CH-MBP-DW12345678 flask`.

Once this is done, install `Flask`:

```shell
$ pip3 install Flask

# Once installed, verify the version. Lists out what is installed
$ python3 -m flask --version
```

We can also export our dependencies and create an `requirements.txt` file so other developers can replicate the environment:

```shell
$ pip3 freeze > requirements.txt
```

The dependencies can then be installed using this `requirements.txt` file:

```shell
$ pip install -r requirements.txt
```

#### Hello World
Inside of our project folder, create a file named `app.py` with the following `Python` code. This should be outside of the `venv` folder:

```python
# Import the Flask class from the flask package
from flask import Flask

# Create an instance of the Flask application
# __name__ tells Flask where to look to resources (like: templates, static files)
app = Flask(__name__)

# Define a route: when a user visits "/" (root URL), run the function below
@app.route("/")
def home():
    # The response sent back to the browser when "/" is requested
    return "Hello, World"

# Run the app only if the script is executed directly
if __name__ == "__main__":
    # Start the development server with debugging enabled
    # Debug mode gives helpful error messages and auto-reloads on changes
    app.run(debug=True)
```

In the terminal, run our code:

```shell
$ python3 app.py
```

The app should be running and can be viewed here: `http://127.0.0.1:5000/` in the browser.

#### Route
In `Flask`, a `route` is a URL path that tells Flask how to handle. Routes are created using the `@app.route()` decorator above a function. That function is called and runs whenever a user visits the route.

The following route would return: `This is the about page` if the user visits: `http://127.0.0.1:5000/about`

```python
@app.route("/about")
def about():
  return "This is the about page"
```

#### Connect Flask to SQLite
Begin by setting up the `SQLite` database in these steps: [SQLite](#sqlite). Once done, move onto the next section.

Update the `app.py` code from earlier to connect to the `SQLite` database:

```python
# Import the Flask class from the flask package
from flask import Flask
# Import the sqlite3 package to connect to SQLite databases
import sqlite3

# Create an instance of the Flask application
# __name__ tells Flask where to look to resources (like: templates, static files)
app = Flask(__name__)

# A helper function to connect the the SQLite database
def get_db_connection():
    # Database file will be created if it doesn't already exist
    conn = sqlite3.connect("app.db")
    # Rows behave like dictionaries
    conn.row_factory = sqlite3.Row
    return conn

# Define a route: when a user visits "/" (root URL), run the function below
@app.route("/")
def home():
    # Fetch all rows from a SQLite database table called "users"
    conn = get_db_connection()
    users = conn.execute("SELECT * FROM users").fetchall()
    # Close the connection to the database
    conn.close()

    # Display user names in plain text (as a test)
    return "<br>".join([user["name"] for user in users])

# Run the app only if the script is executed directly
if __name__ == "__main__":
    # Start the development server with debugging enabled
    # Debug mode gives helpful error messages and auto-reloads on changes
    app.run(debug=True)
```

Run the `app.py` file in the terminal and open the browser here to see the data from the `app.db` SQLite database being displayed: `http://127.0.0.1:5000/`

```shell
$ python3 app.py
```

## Database

### SQLite

#### ORM
`Flask` itself doesn't ship with an `ORM (Object Relational Mapper)` - `Prisma` is an example of an `ORM` used in `JavaScript` - but Flask works very well with `SQLAlchemy` which is a popular `Python` `ORM`. With `SQLAlchemy` `Python` classes are defined instead of writing raw `SQL` to connect and work with databases. `Flask` also has an official extension, `Flask-SQLAlchemy`, that makes using `SQLAlchemy` even easier and this allows connections to: `SQLite`, `MySQL`, `PostgreSQL` and many more databases.

Begin by installing the relevant packages:

```shell
$ pip3 install flask-sqlalchemy
# Optional but recommended for migration later
$ pip3 install Flask-Migrate
```

We can then update our `app.py` code to use the `ORM` or `SQLAlchemy` to connect to the `SQLite` database rather than having to use raw `SQL` queries.

```python
# Import the Flask class from the flask package
from flask import Flask
# Import the SQLAlchemy package to connect to the SQLite database using the ORM
from flask_sqlalchemy import SQLAlchemy
from pathlib import Path

# Create an instance of the Flask application
# __name__ tells Flask where to look to resources (like: templates, static files)
app = Flask(__name__)

# Anchor the SQLite file to this file's directory to avoid "accidental new DB"
BASE_DIR = Path(__file__).resolve().parent
# SQLAlchemy config: points to the dev.db SQLite database in the current folder
app.config["SQLALCHEMY_DATABASE_URI"] = f"sqlite:///{BASE_DIR / 'dev.db'}"
# SQLAlchemy config: disables an event system to save overheads
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)

# ORM model(s)
class Test(db.Model):
    # Matches the existing table name
    __tablename__ = "test"
    # Adjust if the schema differs. Creates the id field in the database table
    id = db.Column(db.Integer, primary_key=True)
    # Adjust if the schema differs. Creates the name field in the databse table
    name = db.Column(db.String, nullable=False)

    def __repr__(self):
        return f"<Test id={self.id} name={self.name!r}>"

# Define a route: when a user visits "/" (root URL), run the function below
@app.route("/")
def home():
    # Fetch all rows from an SQLite database table called "test" using the ORM
    users = Test.query.all()
    # Display the data from inside the table in plain text (as a test)
    return "<br>".join([u.name for u in users])

# Run the app only if the script is executed directly
if __name__ == "__main__":
    # Create tables if they do no exust yet (safe to run repeatedly)
    with app.app_context():
        # Runs inside an app context before the server starts so database is present when the route runs
        db.create_all()

        # Helpful: print the actual DB file being used
        print("Using DB:", db.engine.url.database)

    # Start the development server with debugging enabled
    # Debut mode gives helpful error messsages and auto-reloads on changes
    app.run(debug=True)
```

Run this code as before using to open the running server here: `http://127.0.0.1:5000/` and listing out the data contained within the database:

```shell
$ python3 app.py
```

#### Create Database
Once `SQLite` is installed, create a database in the current folder, if it doesn't exist. A prompt will then appear:

```shell
# Check the SQLite is installed
$ sqlite3 --version

# Create a database named app.db
$ sqlite3 app.db

# Exit SQLite and exit the prompt in the terminal:
$ .exit
```

#### Create Database Table
Once the database has been created, we need to create a database table and add initial content. We use `SQL (Structured Query Language)` for this. Paste this into the terminal when the `sqlite` prompt appears:

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL
);

INSERT INTO users (name) VALUES ('Alice'), ('Bob'), ('Charlie');
.exit
```

This should create an `app.db` file in the project folder.
