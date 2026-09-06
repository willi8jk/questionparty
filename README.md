# questionparty
## Instructions for Building a Django Skeleton Site (for Windows PowerShell)

Tutorial from https://cs-347-f26.github.io/mdn-content/en-us/docs/learn_web_development/extensions/server-side/django/

1. create a new repository:
    1. create a new repository on GitHub
        * add `.gitignore` and choose the Python template
        * create repository and clone to a new local repository
    2. initialize a git repository (locally)
        * copy a Python `.gitignore` template from an [internet source](https://github.com/github/gitignore/blob/main/Python.gitignore) into the root directory

2. copy the following lines into the bottom of `.gitignore` 

        # Text backup files
        *.bak

        # Database
        *.sqlite3

        # Python virtual environment
        .venv/ 

3. create the virtual environment with

        uv init --bare --python 3.14
        uv python pin 3.14

4. add Django as a dependency with `uv add "django"`
    * this will create `.venv/`

5. activate the virtual environment with `.venv\Scripts\Activate.ps1`
    * it can be exited with `deactivate`
        * and re-entered with the initial prompt above
    * it can be deleted `.venv/` to completely remove the environment
        * and recreated with `uv sync` (according to the packages recorded in `uv.lock`)
    * `uv sync` is very useful as it will be used to create a virtual environment based on the project's `uv.lock` to match all dependency versions

6. add Django linter `pylint` with `uv add --dev pylint-django`

7. create a new project ("questionparty_config") with `uv run python -m django startproject questionparty_config .`
    * make sure to include the `.` at the end and that manage.py shows up in the root directory alongside the new directory "questionparty_config"

8. create an application ("catalog") with `uv run python manage.py startapp catalog`
    * make sure this is run in the same directory as manage.py

9. registering an application ("catalog") in the project ("questionparty_config")
    * navigate to `{project}/settings.py` which is `questionparty_config/settings.py` in this example
    * find the `INSTALLED_APPS` list variable and add `catalog.apps.CatalogConfig`
        * this is an application configuration object created for us by default in `/catalog/apps.py/`
        * the formatting is `{application}.{python file}.{application configuration class/object}`

10. specifying the database for the project ("questionparty_config")
    * navigate to `{project}/settings.py`
    * find and change the `DATABASES` dictionary variable to include whatever DB you would like to use
    * SQLite is the default, and willa be used for this example (no changes necessary)

11. also in `{project}/settings.py`, change the `TIME_ZONE` variable to `'America/New_York'` or whatever [tz database time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) desired

12. other options to be aware of (but not change for this example), paraphrased from the instructions
    * `SECRET_KEY` - a key used as part of Django's website security strategy. if this is not protected in development, a different key will need to be used in production (perhaps read from an environment variable or file)
    * `DEBUG` - enables debugging logs to be displayed instead of status code responses
        * `True` is good for development, bad for production (hacking)

13. URL mapper information
    * the project is created with a URL mapper file, `urls.py`
        * it is common practice to defer URL mappings to each application to handle instead of overloading the project's main `urls.py`
    * inside of `urls.py`, the `urlpatterns` list of `path()` functions manages URL mappings
        * each `path()` either associates a URL pattern to a specific view or with another list of URL pattern testing code
        * the route can include a named variable (ex. `catalog/<id>` which will match `catalog/anything` which will pass a variable named `id` with a value of `"anything"` to the view that's mapped)

14. adding our application's url mappings to `urlpatterns`
    * add an import `from django.urls import include`
    * add `path('catalog/', include('catalog.urls')),` to `urlpatterns`

15. redirecting the root URL of the site to `/catalog/`, since this is our only app in the project
    * add an import `from django.views.generic import RedirectView`
    * add `path('', RedirectView.as_view(url='catalog/'))` to `urlpatterns`

16. setting up Django to serve static files
    * this can be useful for the development web server while creating a site
    * add the imports

        from django.conf import settings
        from django.conf.urls.static import static

    * add `+ static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)` to the end of the `urlpatterns` list

    ### `urls.py` updates combined:

        from django.conf import settings
        from django.conf.urls.static import static
        from django.contrib import admin
        from django.urls import path, include
        from django.views.generic import RedirectView

        urlpatterns = [
            path('admin/', admin.site.urls),
            path('catalog/', include('catalog.urls')),
            path('', RedirectView.as_view(url='catalog/')),
        ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)

17. create a new file called `urls.py` **inside** of the application ("catalog") directory (NOT THE PROJECT ("questionparty_config" DIRECTORY)
    * add the following boilerplate to allow future creation of patterns within our application since our project URL mapper will defer to this file for any catalog URLs (per `path('catalog/', include('catalog.urls'))` being added to its urlpatterns)

            from django.urls import path
            from . import views

            urlpatterns = [

            ]

18. run a database migration
    * updates our database to include any models in our installed applications and removes some building warnings
    * as we change our model definitions, Django uses an Object-Relational-Mapper (ORM) to map and track said changes, and it can create database migration scripts (in `{application}/migrtations/`) to automatically migrate the underlying data structure in the database to match the model
    * when the website was orignally created, Django automatically added models for use by the site admin section
    * running the following commands will define tables for those models in our database

            uv run python manage.py makemigrations
            uv run python manage.py migrate

    * `makemigrations` creates (but does not apply) the migrations for all applications installed in the project
        * an application name can be specified to specficially run a migration for a single app
        * gives us a chance to check out code for these migrations BEFORE they are applied
    * `migrate` applies the migrations to the database
        * Django tracks which ones have been added to the current database

19. run the website by calling `uv run python manage.py runserver` (in the same directory as `manage.py`)
    * the site can be viewed on port 8000 on your local host (`127.0.0.1:8000`)
    * it should redirect to `127.0.0.1:8000/catalog` since we specified that in our project `urlpatterns` as `path('', RedirectView.as_view(url='catalog/'))`
    * this page should give a 404 not found error since this tutorial did not setup any pages/urls in application's ("catalog") `urls` module (`urlpatterns` in `{application}/urls.py`)
    * the local server can be stopped with `CTRL+C`
    * and the virtual environment can be deactivated by typing `deactivate` (seen in step 5)