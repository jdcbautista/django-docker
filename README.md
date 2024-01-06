
# Intro to Django, Docker Compose

## Topics Covered
-   Use Docker Compose to simplify setup/teardown
-   Learn to use virtual environments in python
-   Manage packages with pip
-   Create a new Django Project
-   Create a new Django Application
-   Link Django Project with Django Application and PostgreSQL
-   Create Django Models to interact with the Database


## Docker Compose
[Docker Compose](https://www.baeldung.com/ops/docker-compose) is an extra tool available to you to simplify the process of starting and stopping containerized applications.  It's primary purpose is to coordinate starting and running multiple containers at the same time (for example: Nginx, Postgres, and Django), but it is also pretty handy even with just a single container.

To get started, you'll have to define a `docker-compose.yml` file.  Just like a `Dockerfile`, a `docker-compose.yml` describes the infrastructure of your application.  Here's an example we'll use to get a postgres database up:

```yaml
version: '3'
services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=pokedex_db
    ports:
      - '5454:5432'
    volumes: 
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

The important section is `services`.  `services` will contain a description of the container(s) you want to run.  In this case we have a single service `db` (this name is arbitrary).  The `image` field says that we want a postgres:15 image.  This is just like saying `FROM postgres:15` in a Dockerfile.  If we want to pass in any environment variables, we simply list them under `environment`.  Port mapping happens under `ports` (a request to 5454 on the host machine will be forwarded to 5432 in the container).  Lastly, `volumes` says that we want to persist data on the host machine and make it available to the container.  This allows us to create and populate a database, kill and restart the container, and preserve our data.

It's a quirk of docker compose, but if one of your services creates a volume, you have to list it specifically (as you can see at the bottom of the `docker-compose.yml`).

You don't need to become an expert in docker compose, and once you do it a few times, you should start to get the hang of it. 

To run the application you've just described, all you need is:
```bash
docker compose up -d
```

And when you're done:
```bash
docker compose down
```
As long as you run those commands at the same level as your `docker-compose.yml` file.

## Django

You already know how to create a database and interact with it using Python. Today we will use a more sophisticated set of tools called Django to do the same thing.


## Virtual Environments

> Developers have a variety of projects they are working on at any given time, especially if you work for a consultancy. Given the nature of working on greenfield (i.e., brand new) projects and old projects at the same time, versions of software will be all over the place. Your greenfield project may use Python 3 and your client's older code base may use Python 2. How do you protect your computer against getting confused on which versions of software to use? How do you ensure that you have a clean environment to develop from each time? Enter _virtual environments_.

> Virtual environments are containers that wrap around your project to ensure that whatever you install inside your virtual environment stays there / doesn't affect the rest of your computer. Think of it as a placemat at the dining table for babies. Those babies can drool and drop all the food they can on that placemat, but it doesn't affect the rest of your dining table. At the end of the meal, you can throw that placemat away if you wish.

> More specifically, when you set up python on your machine, it was installed to a particular location (check its location with `type python3`), and you also declared a location for that installation of python to store python modules that you install. Lastly, a variable in your system called the `PATH` was adjusted so that when you simply type `python` at the command line, it refers to that version of python you installed, wherever it was installed to.

> When you create a virtual environment, you're installing another copy of python, with its own set of installed modules that are separate from other python modules you've installed previously. The virtual environment also includes an activation script, which adjusts your `PATH` so that when you type `python`, it refers to this new version of python instead. The activation script also defines a function called `deactivate()` that resets your `PATH` back to how it was before. As a convenience, the virtual environment also changes your shell prompt to include the name of the virtual environment, to help you avoid accidentally installing modules into the wrong virtual environment.

Let's create a virtual environment whenever we work with Django so that it doesn't mess up the rest of our machine setups. The 3 commands that you need to remember are:

```bash
# 1. Create your virtual environment
python -m venv <envname>

# 2. Start your virtual environment. Running the script with 'source' applies it to our current shell, instead of a subshell
# Mac:
source <envname>/bin/activate

# Windows:
<envname>/Scripts/activate

# 3. Eject from your virtual environment (once you're done - this is run at the very end of development)
deactivate
```
## Starting a New Django Project


### Installing Django

> Up until now, you may have seen a few built-in python modules used, both in python scripts (re, random, math) and at the command line (http, venv). Now, we're going to use `pip` to install Django, a new python module, that we will use both from the command line, and in our scripts.

```bash
pip install django
```

### Starting a New Django Project

> Now that we've installed Django in our project, let's invoke it at the command line and see what it does.

```bash
python -m django
```

> You should see a menu of options. You'll learn to use several of these options, but we won't need most of them. Don't worry about the warning about django-settings. We'll fix that in a second. Since we don't have a django project yet, let's start by creating one. Let's imagine we're creating an application for a pokedex, so we'll name our project `pokedex_proj`.

```bash
python -m django startproject pokedex_proj . #<--- DON'T MISS THE PERIOD
```

> When we start our project, django will create a folder for the main module of the project, which has the same name. As we progress you'll learn more and more about proper project organization and make other modules, but today we'll just be working with the main module.

> In addition to creating our main module, starting our django project created a file called `manage.py`. Running this file is almost the same as running `python -m django`, except using the manage.py file also loads some settings specific to this project.

> Before we forget, it's important that we record all of the dependencies used in this project, starting with Django. We can do this by using the command `pip freeze`, and redirecting the output to a file.

```bash
pip freeze > requirements.txt
```

> Now when we push this project to github, if another developer pulls it down, they won't receive Django with it. However, they can use the requirements.txt file to install all of the modules used for this project with `pip install -r requirements.txt`.

## Starting Django App

> Django projects are split into many apps (i.e., a project has many apps). Imagine a new _project_ at Amazon where they are selling lots of space on the Moon. That _project_ requires a bunch of different _apps_ in order to run. For example, there might be a billing _app_ to collect money from individuals, a searching _app_ for people to look up lots, a VIP _app_ where they target VIPs, etc. Today, our project will just start with a `pokemon_app` app.

```bash
python manage.py startapp pokemon_app
```

> A quick sidebar - we ran `startproject` earlier and we are now running `startapp`. The difference between these two is that a `project` consists of many `apps`. An `app` can belong to many `projects`.

Next, we need to add the `pokemon_app` app to our `settings.py` file.

```python
## pokedex_proj/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'pokemon_app',
]
```
## Linking PostgreSQL with Django

**Create a database**

> Now our `pokedex_proj` project needs a database to manage all of its data through Django-ORM (NOT OPERATION RISK MANGEMENT BUT OBJECT RELATIONAL MAPPING). We'll be creating a `pokedex_db` database with one table: `pokemon`.

> First we will use docker compose to create our database in PostgreSQL to link onto our Django Project.

docker-compose.yml (at the root of your project):
```yaml
version: '3'
services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=pokedex_db
    ports:
      - '5454:5432'
    volumes: 
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```
> And then tell Django we want to use Postgres as our database instead of the default, SQLite3. Then we will specify the name of the database we want to utilize for this project.

```python
## pokedex_proj/settings.py
## our settings.py file is in the pokedex directory, but if you named your project a different name, then look for that folder name.

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "pokedex_db",
        "USER": "postgres",
        "PASSWORD": "postgres",
        "HOST": "localhost",  
        "PORT": 5454, # This is the port on the host machine (which will be mapped to 5432 in the container)
    }
}
```
Now run:
```bash
docker compose up
```

**Pyscopg3**

> Now we install [psycopg3](https://www.psycopg.org/psycopg3/docs/basic/install.html), the Python library that will help Django talk to Postgres. We won't actually be calling this library directly, it's just a depency of Django-ORM that we need to install in order to turn out Python queries into SQL queries when speaking with Postgres.

```bash
python -m pip install --upgrade pip 
pip install "psycopg[binary]"
```

## Creating a Model

> Models are the foundation of most Django projects. Racing to write Django models without
thinking things through can lead to problems down the road.

> All too frequently we developers rush into adding or modifying models without considering
the ramifications of what we are doing. The quick fix or sloppy “temporary” design decision
that we toss into our codebase now can hurt us in the months or years to come, forcing crazy
workarounds or corrupting existing data.

> So keep this in mind when adding new models in Django or modifying existing ones. Take
your time to think things through, and design your foundation to be as strong and sound as
possible.

Lets create a `Pokemon Model` in our `pokemon_app/models.py` and input some data onto our newly created Data Table.

```python
## pokemon_app/models.py

# models has many different methods we will utilize when creating our Models
from django.db import models

# Create your models here.
# models.Model tell Django this is a Model that should be reflected on our database
class Pokemon(models.Model):
    # CharField is a character field and has a default max length of 255 characters
    name = models.CharField(max_length=255)
    # IntegerField will allow only solid numerical values as input
    level = models.IntegerField(default=1)
```

> We've created a Python class that directly maps to a database table (i.e., a model). Next, let's tell Django to create the necessary code for us to get this table into the database:

```bash
python manage.py makemigrations
```

> A folder was just generated called `migrations`. Look inside there and take a look at the Django code that was generated for us to put our tables into the database. If we were not using an ORM, we would have to write these migrations ourselves, by hand, so let's take a moment to appreciate all the time and effort that Django-ORM is saving us. Next, let's `migrate` our database, so that it reflects the current state of our models.

```bash
python manage.py migrate
```

> We should have a `pokemon_app_pokemon` table in our db. Check it out with:

```bash
docker exec -it  django-pokemon-example-db-1 psql -h localhost -p 5432 -U postgres -d pokedex_db
```
_NOTE: the default name that compose will give to the container is a combination of the root directory, the service name, and the number of service instances_
## Django Console

> While we can interact with our data using Postgres, more often we want to interact with our data using Python. We're going to use a console for our project that will pull in all our Python classes and allow us to query the database directly using Django's ORM.

```bash
python manage.py shell

from pokemon_app.models import Pokemon
```

> The shell will allow us to load in our models from Django. Once in the shell, we can create a new student.

```python
pikachu = Pokemon(name = 'Pikachu', level = 12)
pikachu.save()
```

> This should look familiar - the object oriented lessons from weeks 2 and 3 were in preparation for this. Just by writing those two lines of code, we are able to instantiate a pokemon object for us and save it into the database. Under the hood, it's just running:

```sql
INSERT into pokemon (name, level) VALUES ("John Avalos", 12)
```

> Now we can query our database using Python and see our new record.

```python
Pokemon.objects.all()
# Same as SELECT * FROM pokemon;

# We should expect the following:
<QuerySet [<Pokemon: Pokemon object (1)>]>
```

> You should get back a query object. Exit the shell by typing `exit()`. Let's confirm that our new record got saved in our Postgres db.

```bash
docker exec -it  django-pokemon-example-db-1 psql -h localhost -p 5432 -U postgres -d pokedex_db

psql (11.1, server 9.6.3)
Type "help" for help.

pokedex_db=# \d
                          List of relations
 Schema |               Name                |   Type   |     Owner
--------+-----------------------------------+----------+---------------
 ...    |   ...........                     |   ...    |    ........
 public | django_session                    | table    | codingisawesome
 public | pokemon_app_pokemon             | table    | codingisawesome
 public | pokemon_app_pokemon_id_seq      | sequence | codingisawesome

```

> Django creates our database tables with the app name first, followed by the app name. Before we even wrote our own models, there were many built-in tables here, named after the built-in apps in Django, such as `admin`, `auth`, and `sessions`. Also, take note of the `django-migrations` table, which django manages automatically in order to keep track of which migrations have already been run. Our app's table is `attendance_app_student`. Let's query for the records in that table and see what we get.

```bash
pokedex_db=# SELECT * FROM pokemon_app_pokemon;

 id |  name   | level 
----+---------+-------
  1 | Pikachu |    12
(1 row)

pokedex_db=# 
```
## External Resources

- [Django Docs](https://docs.djangoproject.com/en/2.2/)
- [Django Models Intro Docs](https://docs.djangoproject.com/en/2.2/topics/db/models/)
- [Django Queries Cheat Sheet](https://github.com/chrisdl/Django-QuerySet-Cheatsheet)
- [Django Validators Resource](https://docs.djangoproject.com/en/2.2/ref/validators/)
- [Database Diagramer](https://www.quickdatabasediagrams.com/)

## Assignments
[School API 1](https://github.com/echoplatoonew/school-api-1)
