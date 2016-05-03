# Django app with the Github API

Yokogao is Japaneze for profile of a face as seen from the side.
This is an simple app that consumes the Github API to show information
about a user, it's repositories and issues.

This repository contains a :doc:`tutorial1` on how this application was made.
These documentation is generated using the following command in the
root directory.
```
# build the documentation
mkdocs build

# preview the documentation
mkdocs serve
```
## Run the app

There are a few steps to run before the app can be run.

Create a virtualenv, using virtualenvwrapper.
```
mkvirtualenv yokogao_env

# install the dependencies
pip install -r requirements.txt

# create the database and the structure
./manage.py migrate

# create a local_settings.py file next to settings.py
# it must contain at minimuum
GITHUB_USER = '<Githubusername>'
```

After this, you can run Django's internal webserver with the following
command
```
./manage.py runserver
```
