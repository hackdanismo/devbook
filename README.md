# DevBook
A collection of developer notes, snippets and code examples.

+ [Python](#python)
  + [Flask](#flask)
    + [Hello World](#hello-world)
 
## Python

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

When the virtual environment has been created, this will generate a `venv` folder in the project folder.

Once activated, the shell should read something like: `(venv) danieljackson@CH-MBP-DW12345678 flask`.

Once this is done, install `Flask`:

```shell
$ pip3 install Flask

# Once installed, verify the version. Lists out what is installed
$ python -m flask --version
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
