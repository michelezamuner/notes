# Flask

## Configuration

Flask uses several terminal commands to scaffold various parts of the application. These commands usually require some environment variable to be set, the most important of which is the location of the Flask application we are working with. To ease setting these variables, we can install `python-dotenv` to handle environment variables from a file. Specifically for Flask, we can add a `.flaskenv` file like:
```
# .flaskenv
FLASK_APP=main.py
```

where `FLASK_APP` points to a Python script exporting the Flask application object.

## Basic routing

In Python, every directory containing a `__init__.py` file, is considered a package. The `__init__.py` file is executed when the package is imported in a script. We can then define an `app` package that will contain the Flask application, by creating an `app` directory in our project, and add a `__init__.py` file to it.

The first thing to do to create a Flask application is creating an instance of the `Flask` class:
```python
# app/__init__.py
from flask import Flask

app = Flask(__name__)
```

The Flask constructor takes a reference to a module, in order to be able to automatically locate files such as templates from the given module. Usually we associate the new Flask application to the current module, by passing its name, which is stored in the Python constant `__name__`.

Next we need to set up routes for our Web application, which in Flask are callbacks for various URLs, defined as simple Python functions. We can define routes into their own module, `routes.py`:
```python
# app/routes.py
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

To define Flask routes we use the `route()` decorator, which is defined in the `Flask` object we created in the main module file. To import this object, we need to import the `app` variable we defined previously in the `__init__.py` file: we do this with the line `from app import app`. At this point we can define a new `index()` function, that should handle all requests to the root endpoint `/`, and to the additional endpoint `/index`. To bind these endpoints to our new `index()` function we just apply the `app.route()` decorator twice, passing the two endpoints.

To load the routes we defined here, we then need to import this module from the main one:
```python
# app/__init__.py
from flask import Flask

app = Flask(__name__)

from app import routes
```

Here we need to put the import after the definition of the `app` variable, because `routes` tries to import the `app` variable itself, that should thus already be defined.

## Templates

Templates in Flask are located inside a `templates` folder, child of the Flask application module, and are written for the Jinja2 template engine. For example:
```html
<!-- app/templates/index.html -->
<html>
    <head>
        {% if title %}
        <title>{{ title }}</title>
        {% else %}
        <title>Welcome!</title>
        {% endif %}
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```

This template will automatically be found by the Flask `render_template` function, which takes the template file name and the list of variable of the view model:
```python
# app/routes.py
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'John'}
    return render_template('index.html', title='Home', user=user)
```

## Configuration

The easiest way to pass values around a Flask application, is to add them to `app.config` like:
```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'my-secret-key'
```

## Web forms

To handle Web forms in Flask we can use the Flask extension Flask-WTF. Flask-WTF needs the `SECRET_KEY` configuration to be available in `app.config` to generate CSRF tokens to protect forms.

In Flask-WTF forms are represented by Python classes:
```python
# app/forms.py
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember me')
    submit = SubmitField('Sign In')
```

A form object instantiated from `LoginForm` can now be directly injected into a template, from where the form fields will be available:
```python
# app/routes.py
# ...
def login():
    form = LoginForm()
    return render_template('login.html', title='Sign In', form=form)
# ...
```
```html
<!-- app/templates/login.html -->
<!-- ... -->
{{ form.hidden_tag() }}
<!-- ... -->
<p>
    {{ form.username.label}}<br>
    {{ form.username(size=32) }}
</p>
<!-- ... -->
```

The `hidden_tag` form method will render an hidden field containing the CSRF token that is defined for this request. Calling the form property as a function will allow the field to be rendered as HTML.

To allow the script to receive posted data, we first need to allow the route to accept the `POST` HTTP method, and then we need to add the code to handle the submitted data:
```python
# app/routes.py
# ...
from flask import render_template, flash, redirect

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for user {}, remember_me={}'.format(
            form.username.data, form.remember_me.data))
        return redirect('/index')
    return render_template('login.html', title='Sign In', form=form)
```

The method `validate_on_submit()` first checks if the current request is `POST` instead of `GET` (which would need to just render the form), and does the back-end validation of the fields, including the CSRF one.

The `flash()` function temporarily stores a message in order for it to be displayed in the Web pages in the future. For a Web page to access flashed messages, it can use the `get_flashed_messages()` function:
```html
<!-- ... -->
{% with messages = get_flashed_messages() %}
{% if messages %}
<ul>
    {% for message in messages %}
    <li>{{ message }}</li>
    {% endfor %}
</ul>
{% endif %}
{% endwith %}
<!-- ... -->
```

The first time flashed messages are requested with `get_flashed_messages()`, they're also removed from the temporary storage, so that only the latest messages are shown.

The validation errors produced during the back-end validation are automatically stored inside each form field, so we can just grab the errors and display them in the form template:
```html
<!-- ... -->
<p>
    {{ form.username.label }}<br>
    {{ form.username(size=32)}}<br>
    {% for error in form.username.errors %}
    <span style="color: red;"[{{ error }}]></span>
    {% endfor %}
</p>
<!-- ... -->
```

## Generate URLs

The Flask function `url_for` automatically generates the correct URLs based on the mapping configuration, which is useful to avoid having to duplicate URL strings around the code:
```html
<a href="{{ url_for('index') }}">Home</a>
```

The argument to `url_for` is the name of the view function that has been associated to the corresponding URL in the routing configuration.

## References
- https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world