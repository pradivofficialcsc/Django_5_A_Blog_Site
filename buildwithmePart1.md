# Main Framework
MVC = Model View Controller

Django Pattern = **MTV** 
Model, Template, View

Model = **Data handler** between database and view

Template = Presentation layer

View = Communicates with the database via the model and **transfers data** to the template.
<hr>

### Architecture: 

Browser -> URL -> view -> model -> DB
DB -> model -> view -> Template -> browser

<hr>

## Project Structure

manage.py = Command line utility to interact with project.

__init__.py = to treat the folder as a module
- asgi = Asynchronous gateway interface
- wsgi = Web server gateway interface
- urls.py = consists of URL patterns
- settings.py = configurations

<hr>

- **STEP 1** : django-admin startproject mysite (this will create the above files)
- **STEP 2** : python3 manage.py migrate
- **STEP 3** : python3 manage.py runserver
- **STEP 4** : python3 manage.py startapp blog

## Application Structure
- \_\_init__.py = to treat the folder as module
- admin.py = django administrations
- apps.py = main config
- migrations = db migrations files
- models.py = data model (important)
- tests.py = testing application
- views.py = application logic

<hr>

- **STEP 5** : create a model in mywebsite/blog/models.py
- **STEP 6** : create datetime fields

<hr>

**TIPS:** The date time method has two methods.

```
publish = models.DateTimeField(default=timezone.now)
```

```
from django.db.models.functions import Now
publish = models.DateTimeField(db_default=Now())
```

- **auto_now_add** : While creating an object
- **auto_now** : while saving an object
<hr>

- **STEP 7** : Defining the sort order for blog posts

<p> Here we will define descending order for blog post </p>

```commandline
class Meta:
    ordering = ['-publish']
```
<p> Define database indexes for model</p>

```
# not supported for MySQL, by default MySQL descending index will be created as normal index

indexes = [
    models.Index(fields=['-publish'])
]
```

- **STEP 7** : Activate the application 

<p> Go to mysite/settings.py, add "blog.apps.BlogConfig"</p>
<p> This is to notify Django that application is active for the project</p>

- **STEP 8** : Save the post as draft before publishing

```commandline
class Post(models.Model):
    class Status(models.TextChoices):
        DRAFT = 'DF', 'Draft'
        PUBLISHED = 'PB', 'Published'
    
    # below the other attributes add the following 
    status = models.CharField(
        max_length=2,
        choices=Status,
        default=Status.DRAFT
    )
```
<p> Enum is used to define choices inside the model class. It is a good practise</p>
<p> This can be used anywhere by importing and used as a reference</p>

```commandline
(InteractiveConsole)
>>> from blog.models import Post
>>> Post.Status.choices
[('DF', 'Draft'), ('PB', 'Published')]
>>> Post.Status.labels
['Draft', 'Published']
>>> Post.Status.values
['DF', 'PB']
>>> Post.Status.names
['DRAFT', 'PUBLISHED']

```

- **STEP 9** : Add a many-to-one relationship
<p> Many users creating a single post, and use authentication framework</p>

```commandline
author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='blog_posts'
```

- **STEP 10** : Create and apply migrations
<p> Migrations will be applied for all applications listed</p>

```commandline
(.venv) pradivgnanaraj@Pradivs-MacBook-Air mysite % python3 manage.py sqlmigrate blog 0001
```
- **STEP 11** : Sync the database
```commandline
(.venv) pradivgnanaraj@Pradivs-MacBook-Air mysite % python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, blog, contenttypes, sessions
Running migrations:
  Applying blog.0001_initial... OK
```
<hr>

## Creating Administration for models.

**Step 1** create super user
 - python3 manage.py createsuperuser
 - python3 manage.py runserver

**Step 2** Add models to admin site
- mysite/blog/admin.py

```commandline
from django.contrib import admin
from .models import Post

\# Registered Models
admin.site.register(Post)
```
**Step 3** : Customizing models (display)

```commandline
from django.contrib import admin
from .models import Post

# Registered Models
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'slug', 'author', 'publish', 'status']

```

**Step 4** Adding facet counts

```commandline
from django.contrib import admin
from .models import Post

# Registered Models
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'slug', 'author', 'publish', 'status']
    list_filter = ['status', 'created', 'publish', 'author']
    search_fields = ['title', 'body']
    prepopulated_fields = { 'slug' : ('title',)}
    raw_id_fields = ['author']
    date_hierarchy = "publish"
    ordering = ['status', 'publish']
    show_facets = admin.ShowFacets.ALWAYS
```