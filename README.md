csesoc-website-v2
=================
## Developer setup for Ubuntu
###Core packages
```bash
$ sudo apt-get install git pip sqlite3 python-ldap libsasl2-dev libldap2-dev python-dev libssl-dev
```

###Setting up virtualenv
This is only necessay if you work on a lot of Python projects. It's mainly to deal with package dependencies. It's not completely necessary, but its a good habit.
```bash
$ sudo apt-get install virtualenv
$ virtualenv <environment_name>
$ . <environment_name>/bin/activate
```
You'll see in your terminal that it has <environment_name> at the start of the line

### Installing packages
Run the following
```bash
$ git clone git@github.com:csesoc/csesoc-website-v2.git
$ cd csesoc-website-v2
$ pip install -r requirements.txt
```

### Database Setup
The database for the website is stored in soc-website.db so we only need to run syncdb
```bash
$ sqlite3 soc-website.db < initial.sql
$ python manage.py syncdb
```

### Local settings
To serve CSS, you need to make a local settings file. Create a file called local_settings.py in the root directory of the project with the following two lines:
```python
STATIC_ROOT = '/path/to/csesoc-website-v2/app/theme/static'
DEBUG=True

```

### Running the server
To start the server, run
```bash
$ python manage.py runserver
```
and go to [localhost:8000](localhost:8000) in your browser. Enjoy!

## Using the CMS front end

### How to log in
If you're not connected to uniwide, anything you type in the log in field will make you log in as admin. If you're on uniwide, you need to make your own users.

### Editing the front end
When you log in as admin, you can see that you can edit things directly from the front end. To add a new page, go to [localhost:8000/admin](localhost:8000/admin). There's a really nice interface which is more or less intiutive to use. Keep in mind that any changes you make here won't be added to the git repo (it saves on your local database).
 
### Features that don't work locally
Some features of the website won't work locally because they require a user key or something similar. The following features are:
* LDAP Authentication
* Teams
* Timetable importer

## Using the Django back end
PSA: Use 4 space SPACE INDENTATION! This is Python! Whitespace is important!

You can do most things through the front end now that we're using Mezzanine, but if you need any complicated features, you still need to use the back end. The easiest way to get start is to go through a [Django tutorial](https://docs.djangoproject.com/en/dev/intro/tutorial01). Otherwise, here's a rough rundown of how to add a new feature to the website.

### Registering the feature
In the app/ folder, add a folder with the app name. In that file, add a file called \_\_init__.py (just leave it empty). Open up settings.py and got to
```python
INSTALLED_APPS = (
    'app.theme',
    'app.sponsors',
    ...
)
```
and add the line:
```python
INSTALLED_APPS = (
    'app.theme',
    'app.<app_name>',
    ...
)
```
### Creating database table
```bash
$ python manage.py schemamigration <app_name> initial --initial
```
This should create a new folder in app/<app_name>/ called migrations and a new file called 0001_initial.py. In that file, you need to add the fields in. It should look something like:
```python
from south.utils import datetime_utils as datetime
from south.db import db
from south.v2 import SchemaMigration
from django.db import models

class Migration(SchemaMigration):
    def forwards(self, orm):
        # Adding model 'Feature'
        db.create_table(u'merch_hoodie', (
            (u'id', self.gf('django.db.models.fields.AutoField')(primary_key=True)),
            ('field1', self.gf('django.db.models.fields.CharField')(max_length=100)),
            ('field2', self.gf('django.db.models.fields.DecimalField')(max_length=8)),
            ('field3', self.gf('django.db.models.fields.EmailField')(max_length=75)),
        ))
        db.send_create_signal(u'feature', ['Feature'])

    def backwards(self, orm):
        # Deleting model 'Feature'
        db.delete_table(u'feature')

    models = {
        u'app.feature': {
            'Meta': {'object_name': 'Feature'},
            'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'field1': ('django.db.models.fields.CharField', [], {'max_length': '100'}),
            'field2': ('django.db.models.fields.DecimalField', [], {'max_length': '8'}),
            'field3': ('django.db.models.fields.EmailField', [], {'max_length': '75'}),
        }
    }

    complete_apps = ['feature']
)
```
PSA: PLEASE SPECIFY A BACKWARDS METHOD
To run the migration, run:
```bash
$ python manage.py migrate <app_name>
```
### Adding models
In the folder app/<app_name>/, add a file called models.py. It should look like this:
```python
from django.db import models

class Feature(models.Model):
   fiedl1 = models.CharField(max_length=100)
   field2 = models.DecimalField(max_length=8)
   field3 = models.EmailField(max_length=75)
```
### Adding views
In the folder app/<app_name>/, add a file called views.py. It should look like this:
```python
from django.shortcuts import render_to_response, redirect
from models import Feature
from django.template import RequestContext

def method(request):
    if request.method == 'POST':
        # Do stuff with POST request
        return render_to_response('feature/post.html', context_instance=RequestContext(request))
    else:
        # Do stuff with GET request
        return redirect('/feature')
```
The first argument of render_to_response points to a template in app/theme/templates/. So make a folder called <app_name> in app/theme/templates/ and add a html template. It should look like:
```html
{% extends 'base.html' %}

{% block meta_title %}Page Title{% endblock %}

{% block main %}

<!-- CONTENT HERE -->

{% endblock %}
```
### Adding routing
Open up urls.py and go to the tuple:
```python
urlpatterns = patterns("",
    url(r'^signin/?$', direct_to_template, {'template': 'auth/login.html'}, name='signin'),
    url(r'^zlogin/?$', 'app.auth.views.signin'),
    ...
```
You need to add a path to which the page can be accessed. Something like:
```python
urlpatterns = patterns("",
    url(r'^signin/?$', direct_to_template, {'template': 'auth/login.html'}, name='signin'),
    url(r'^zlogin/?$', 'app.auth.views.signin'),
    url(r'^featyre/?$', 'app.feature.views.method'),
    ...
```
```

### Adding an admin page:
In the folder app/<app_name>/, add a file called admin.py. It should look like this:
```python
from django.contrib import admin
from app.feature.models import Feature

class FeatureAdmin(admin.ModelAdmin):
    list_filter = ('field1','field2','field3')
    list_display = ('field1','field2','field3')

admin.site.register(Feature, FeatureAdmin)

```
### Adding tests
In the folder app/<app_name>/, add a file called test.py. It should look like this:
```python
from django.test import TestCase

class SimpleTest(TestCase):
    def test_basic_addition(self):
        """
        Tests that 1 + 1 always equals 2.
        """
        self.assertEqual(1 + 1, 2)
```
To run tests, run:
```bash
$ python manage.py test
```
