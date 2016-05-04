# Follow up tutorial

Continuing on the first [tutorial](part_1), we would like to customize
which Github user is used in the API calls. To this end, let's add
the following to settings.py
```
GITHUB_USER = ''
```
In order to keep git clean of any usernames/passwords/etc.. create a
file name ```local_settings.py``` in the same directory as settings.py
Next add this file to the gitignore file
```
echo "local_settings.py" >> .gitignore
```

Add the above mentioned variable in the new file and fill in your
Github username.
This settings file needs to be called by the main settings file. So at
the end of settings.py file add the following.
```python
try:
    from .local_settings import *
except ImportError:
    pass
```
## Updating the view

The ```GITHUB_USER``` variable is now set, and can be used in the views.
Which we now will update. Open the views.py and look for the line
```
yoko = requests.get('https://api.github.com/users/raybesiga')
```
we are going to replace the name with a placeholder, change it like so
```
yoko = requests.get('https://api.github.com/users/%s' % GITHUB_USER)
```
for this to work we need to import the ```GITHUB_USER``` variable from the settings
add to the imports the following
```python
from gitit.settings import GITHUB_USER
```

## I'm missing something

After following the first tutorial and looking at the result, it didn't
look like the preview. Turns out I didn't have ```blog-layout.css```
and ```grids-responsive-min.css```. After adding this to the pipeline
stylesheets, it looked a lot better.
```
PIPELINE = {
    'STYLESHEETS': {
        'yokogao': {
            'source_filenames': (
              'css/bower_components/primer-css/css/primer.css',
              'css/bower_components/octicons/octicons/octicons.css',
              'css/default.css',
              'css/blog-layout.css',
              'css/grids-responsive-min.css'
            ),
            'output_filename': 'css/yokogao.css',
            'extra_context': {
                'media': 'screen,projection',
            },
        },
    }
}
```

## List page for extra info

We now have a page with

- A picture
- A name
- A location
- Public repositories
- Followers
- Following
- Member since
- Website

Let's add a list page for the public repositories, let's start with
creating a new view.
```
def repo_list(request):
    yoko = requests.get('https://api.github.com/users/%s/repos' % GITHUB_USER)
    context = {}
    context['repos'] = yoko.json()

    return render(request, 'yokogao/repo_list.html', context)
```
Get the data from the Github API, create a context dictionary and add
the json representation of the API request.

Next let's add a template for this view.
```
{% extends "base.html" %}
{% block content %}
<h1>Repositories</h1>
<ul>
{% for repo in repos %}
    <li><a href="/repo/{{ repo.name }}">{{ repo.name }}</a></li>
{% endfor %}
</ul>
{% endblock content %}

```
This is basically an unordered list with a link to the detail page of
a repository.
