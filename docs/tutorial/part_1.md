## Let's build a basic Django app with the Github API

Today we are going to build a simple app that consumes the JSON response of a simple GitHub API request. We shall be using the following technologies:

- Python/Django
- Requests
- Pipeline
- NodeJS
- Bower

That is all the technology we shall need. Python and Django will provide the framework for the application. Requests will be used to make a simple request to the Github API. NodeJS and Bower will be used for front end dependency management and Pipeline will do the asset compression.

The Github resources we shall use:

- Github API
- Primer CSS kit
- Octicons

The source code for this tutorial can be found [here](https://github.com/raybesiga/yokogao). By the time we get to the bottom of this, your app should look something like this:

## Getting Started

This tutorial assumes a knowledge of virtual environments. The use of virtualenv and virtualenvwrapper is highly recommended. If you don't have these, you can install them using pip with the following commands:

```
$ pip install virtualenv
$ pip install virtualenvwrapper
```

You should then create a virtual environment. Ours will be called gitit. Get it?

```
$ mkvirtualenv gitit
```

This should activate a virtualenv called Gitit. We shall then install Django, Requests and Pipeline.

```
$ pip install Django==1.8.6
$ pip install requests
$ pip install django-pipeline
```

When you run the "pip freeze" command it should show the following:

```
Django==1.8.6
django-pipeline==1.6.5
requests==2.9.1
wheel==0.24.0
```

Switch to the directory where you intend to create the project and run the command to start a project.

```
$ django-admin startproject gitit
```

This should give us the following directory structure

```
.
├── gitit
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

This is the basic directory structure of a Django project with no apps. We shall now proceed to create directories for our static files and the templates.

```
$ cd gitit
$ mkdir static templates
$ mkdir static/js static/css
```

It now time to run our server just to ensure that everything is fine.

```
$ python manage.py runserver
```
It should run fine, but we need to run migrations to assuage the server gods, so run the migrate command

```
$ python manage.py migrate
```

## Let's create the app

Great, now time to create the app. We shall call it `yokogao` the Japanese word for profile of a face as seen from the side. This may need clarification :)

```
$ python manage.py startapp yokogao
```

Our structure should now have changed to this:

```
.
├── db.sqlite3
├── gitit
│   ├── __init__.py
│   ├── __init__.pyc
│   ├── settings.py
│   ├── settings.pyc
│   ├── urls.py
│   ├── wsgi.py
│   └── wsgi.pyc
├── manage.py
├── static
│   ├── css
│   └── js
├── templates
└── yokogao
    ├── __init__.py
    ├── admin.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py
```
It is now time to customize our settings so as to enable the app work best within the project. Let us add the app to the installed apps

```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # third party apps
    'pipeline',

    # local apps
    'yokogao',
)
```
We should now be ready to create the views that allow us to access the Github API and request the basic information we require for our app. For more about the information returned by the API, use Curl

```
$ curl https://api.github.com/users/raybesiga
```
For a simple Curl tutorial that uses the Github API, check [this one](https://gist.github.com/caspyin/2288960) out. Remember to use your Github username wherever `raybesiga` is used.

Edit the `yokogao/views.py` file to look this way:

```
from django.shortcuts import render, HttpResponse
import requests
import json

def index(request):
    yoko = requests.get('https://api.github.com/users/raybesiga')
    content = yoko.text
    return HttpResponse(content)
```

You should note that we import `requests` and `json`. Requests to query the API, and JSON to convert the response returned to a JSON format. You should now create a URLS file at `yokogao/urls.py`

```
from django.conf.urls import url
import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```
Head over to ```gitit/urls.py``` and edit the file so as to use the Yokogao URLs for the app

```
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^', include('yokogao.urls')),
]
```

If you run your server now, you should be able to see this in your browser, albeit without format

```
{
   "login":"raybesiga",
   "id":405697,
   "avatar_url":"https://avatars.githubusercontent.com/u/405697?v=3",
   "gravatar_id":"",
   "url":"https://api.github.com/users/raybesiga",
   "html_url":"https://github.com/raybesiga",
   "followers_url":"https://api.github.com/users/raybesiga/followers",
   "following_url":"https://api.github.com/users/raybesiga/following{/other_user}",
   "gists_url":"https://api.github.com/users/raybesiga/gists{/gist_id}",
   "starred_url":"https://api.github.com/users/raybesiga/starred{/owner}{/repo}",
   "subscriptions_url":"https://api.github.com/users/raybesiga/subscriptions",
   "organizations_url":"https://api.github.com/users/raybesiga/orgs",
   "repos_url":"https://api.github.com/users/raybesiga/repos",
   "events_url":"https://api.github.com/users/raybesiga/events{/privacy}",
   "received_events_url":"https://api.github.com/users/raybesiga/received_events",
   "type":"User",
   "site_admin":false,
   "name":"Ray Besiga",
   "company":"Sparkplug ",
   "blog":"http://raybesiga.com",
   "location":"Kampala, Uganda",
   "email":"raybesiga@gmail.com",
   "hireable":true,
   "bio":null,
   "public_repos":91,
   "public_gists":5,
   "followers":13,
   "following":17,
   "created_at":"2010-09-18T07:51:02Z",
   "updated_at":"2016-02-04T05:30:03Z"
}
```

## Let us build templates

Great, we are now at the step where we can show stuff in the browser.
However, there is business we need to take care of before we can even build our base template.
Time to do some front end stuff. Remember the NodeJS and Bower I mentioned earlier?
Yeah, now is the time to use it. Ensure you have NodeJS and NPM installed on your machine. More on that [here](https://nodejs.org/en/)

We shall install Bower and Yuglify, Pipeline's default JS and CSS compressor.

```
$ npm install -g bower
$ npm install -g yuglify
```
To keep things interesting, we shall use the [Primer](http://primercss.io/) toolkit by Github. It is purposely incomplete an they do say so themselves

<blockquote>Heads up! We love open source, but Primer is unlikely to add new features that are not used in GitHub.com. It's first and foremost our CSS toolkit. We really love to share though, so hopefully that means we're still friends.</blockquote>

Let us proceed to use it in our app, change directory to the `static/css` folder and run the following command

```
$ bower install primer-css --save
```
You should now have a `bower_components` folder therein with the `primer-css` and `octicons` child directories. Time to make changes to the settings file so as to use these assets in our templates.

Change the Templates section of the settings as follows:

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ['templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Add the following for the STATIC to comply

```
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

# Django Pipeline
STATICFILES_STORAGE = 'pipeline.storage.PipelineCachedStorage'

STATICFILES_FINDERS = (
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    'pipeline.finders.PipelineFinder',
    'pipeline.finders.FileSystemFinder',
    'pipeline.finders.AppDirectoriesFinder',
    'pipeline.finders.PipelineFinder',
    'pipeline.finders.CachedFileFinder',
)


PIPELINE_CSS_COMPRESSOR = 'pipeline.compressors.yuglify.YuglifyCompressor'

PIPELINE = {
    'STYLESHEETS': {
        'yokogao': {
            'source_filenames': (
              'css/bower_components/primer-css/css/primer.css',
              'css/bower_components/octicons/octicons/octicons.css',
            ),
            'output_filename': 'css/yokogao.css',
            'extra_context': {
                'media': 'screen,projection',
            },
        },
    }
}
```

Great, the settings are looking good, the stars should be aligned, time to collect our compressed static.

```
$ python manage.py collectstatic
```
The output is long but at the bottom, you should have something similar to this at the bottom:

```
$ 62 static files copied to '/Users/raybesiga/Projects/gitit/static', 64 post-processed.
```

We shall now proceed to use the static in our templates. Change directory to the `templates` folder and create a `base.html` file. Mine looks something like this:

```
{% load pipeline %}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>{% block title %}Basic Django app with the Github API{% endblock %}</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Building with the Github API">
    <meta name="author" content="Ray Besiga">
    {% stylesheet 'yokogao' %}
  </head>
  <body>
    <div id="layout" class="pure-g">
      <div class="sidebar pure-u-1 pure-u-md-1-4">
        <div class="header">
            <h1 class="brand-title">Basic Github App</h1>
            <h2 class="brand-tagline">Using Django, Requests, Bower and Pipeline</h2>
            <nav class="nav">
              <h3>Tools leveraged</h3>
                <ul class="nav-list">
                    <li class="nav-item">Github API</li>,
                    <li class="nav-item">Django</li>,
                    <li class="nav-item">Requests</li>,
                    <li class="nav-item">Bower</li>,
                    <li class="nav-item">Pipeline</li>,
                    <li class="nav-item">Primer CSS</li>,
                    <li class="nav-item">Pure Grid CSS</li>
                </ul>
            </nav>
        </div>
    </div>
    <div class="content pure-u-1 pure-u-md-3-4">
          <div>
          {% block content %}{% endblock %}
          </div>
        </div>
      </div>
    </div>
    {% block extra_scripts %}{% endblock %}
  </body>
</html>
```

Please note that we include {% load pipeline %} at the top and then use
{% stylesheet 'yokogao' %} for the output stylesheet.

We shall proceed to create the `templates` folder and the `index.html` file for the app within the `yokogao` folder.

```
$ mkdir yokogao/templates/yokogao
$ cd yokogao/templates/yokogao
$ touch index.html
```

Before we can show data in the `index.html` file, we need to get the context data within the views file. We are going to use the `dateutil` module as it provides powerful extensions to the standard datetime module, available in Python. We shall need it to parse the time string returned for `created_at` into a datetime object. To install dateutil, use pip:

```
$ pip install python-dateutil
```

And make changes to our `yokogao/views.py` file.

```
import dateutil.parser
from django.shortcuts import render
import requests

def index(request):
    yoko = requests.get('https://api.github.com/users/raybesiga')
    context = yoko.json()

    # context has a created_at key with a string value
    # parse the string value into a python date object
    context['created_at'] = dateutil.parser.parse(context['created_at'])
    return render(request, 'yokogao/index.html', context)
```

Then edit the `index.html` file:

```
{% extends "base.html" %}

{% block title %} Your Github Yokogao {% endblock %}

{% block content %}
<div>
    <header class="post-header">
      <h2 class="post-title">Basic Django app with the Github API</h2>
    </header>
       <img class="avatar" src="{{ avatar_url }}" width="300" height="300"
          style="border-radius: 50%;margin-bottom: 15px;margin-top: 15px;">
          <nav class="menu">
            <a class="menu-item" href="#">
            <span class="octicon octicon-person"></span>
            {{ name }}
          </a>
          <a class="menu-item" href="#">
            <span class="octicon octicon-location"></span>
            {{ location }}
          </a>
          <a class="menu-item" href="#">
            <span class="octicon octicon-list-ordered"></span>
            <span class="counter">{{ public_repos }}</span>
            Public Repos
          </a>
          <a class="menu-item" href="#">
            <span class="octicon octicon-bookmark" ></span>
            <span class="counter">{{ followers }}</span>
            Followers
          </a>
          <a class="menu-item" href="#">
            <span class="octicon octicon-eye"></span>
            <span class="counter">{{ following }}</span>
            Following
          </a>
          <a class="menu-item" href="#">
            <span class="octicon octicon-calendar"></span>
            Member for {{ created_at|timesince }}
          </a>
          <a class="menu-item " href="{{ blog }}" target="_blank">
            <span class="octicon octicon-globe"></span>
            Website: {{ blog }}
          </a>
        </nav>
      <h4>Built with <span class="octicon octicon-heart"></span> using the Github API and Primer</h4>
  </section>
</div>
{% endblock %}
```

Now if you run your server, you should have the browser showing your yokogao on the index page.

### Next steps
In the next tutorial, we shall add a form to post any  Github username and return similar informaton, explore options with an asynchronous REST API, and host our app on Heroku. Thanks for taking the time to read and follow through with this. And many thanks to my colleague [Peter Coward](https://github.com/petecoward) for reading through and refactoring my code so it would be palatable. Until next time.
