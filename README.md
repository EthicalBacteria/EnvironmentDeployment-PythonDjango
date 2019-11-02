# EnvironmentDeployment-PythonDjango
Development environment for Django projects

## Technologies used

Service | Version | Description
------------ | ------------- | -------------
Python | v.3.8.0 | Language
Django | v.2.2.6 | Framework
Psycopg2 | v.2.8.4 | Python-PostgreSQL Database Adapter
Postgresql | v.12 | Database

## Stack Docker

**Dockerfile**  

The Dockerfile defines an application’s image content via one or more build commands that configure that image. Once built, you can run the image in a container. This **Dockerfile** starts with a Python 3 parent image. The parent image is modified by adding a new **code** directory. The parent image is further modified by installing the Python requirements defined in the **requirements.txt** file.
```
FROM python:3.8.0
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

**docker-compose.yml**  

The **docker-compose.yml** file describes the services that make your app. In this example those services are a web server and database. The compose file also describes which Docker images these services use, how they link together, any volumes they might need mounted inside the containers. Finally, the **docker-compose.yml** file describes which ports these services expose.
```
version: '3'

services:
  db:
    image: postgres:latest
    environment:
      - POSTGRES_DB=django
      - POSTGRES_USER=django
      - POSTGRES_PASSWORD=django
    ports:
      - "5432:5432"
    volumes:
      - ./pgdata:/var/lib/postgresql/data

  web:
    build: .
    command: >
      sh -c "sleep 10 &&
             python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
      - pgadmin

  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      - PGADMIN_DEFAULT_EMAIL=pgadmin@mail.com
      - PGADMIN_DEFAULT_PASSWORD=pgadmin
    ports:
      - "8080:80"
    depends_on:
      - db
```

**requirements.txt**  

This file is used by the RUN **pip install -r requirements.txt** command in your **Dockerfile**.
```
Django==2.2.6
psycopg2==2.8.4
```

## Create a Django project

In this step, you create a Django startup project. Before you continue, create a new folder on behalf of your project and then make a **git clone**.

1. Change to the root of your project directory.
2. Create the Django project by running the docker-compose run command as follows.

```
sudo docker-compose run web django-admin startproject app .

```

This instructs Compose to run **django-admin startproject app** in a container, using the **web** service’s image and configuration. Because the **web** image doesn’t exist yet, Compose builds it from the current directory, as specified by the **build: .** line in **docker-compose.yml**.
Once the **web** service image is built, Compose runs it and executes the **django-admin startproject** command in the container. This command instructs Django to create a set of files and directories representing a Django project.


## Configuring the connection to the database

In this section, you set up the database connection for Django.

1. In your project directory, edit the **app/settings.py** file.
2. Replace the **DATABASES = ...** with the following:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'django',
        'USER': 'django',
        'PASSWORD': 'django',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```
These settings are determined by the postgres Docker image specified in **docker-compose.yml**.  

3. Save and close the file.
4. Run the docker-compose up command from the top level directory for your project.

```
MBP-of-EthicalBacteria/:ProjectSetup-Django ethicalbacteria$ docker-compose up
projectsetup-django_db_1 is up-to-date
Creating projectsetup-django_web_1 ... done
Attaching to projectsetup-django_db_1, projectsetup-django_web_1
db_1   | The files belonging to this database system will be owned by user "postgres".
db_1   | This user must also own the server process.
db_1   |
db_1   | The database cluster will be initialized with locale "en_US.utf8".
db_1   | The default database encoding has accordingly been set to "UTF8".
db_1   | The default text search configuration will be set to "english".

. . .

web_1  | November 02, 2019 - 13:42:33
web_1  | Django version 2.2.6, using settings 'app.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
```
At this point, your Django app should be running at port **8000** on your Docker host. On Docker Desktop for Mac and Docker Desktop for Windows, go to **http://localhost:8000** on a web browser to see the Django welcome page. 

![Django It Worked](https://docs.docker.com/compose/images/django-it-worked.png)

On certain platforms (Windows 10), you might need to edit **ALLOWED_HOSTS** inside **settings.py** and add your Docker host name or IP address to the list. For demo purposes, you can set the value to:

```
ALLOWED_HOSTS = ['*']
```

This value is not safe for production usage. Refer to the Django documentation for more information.

5. Finally you have to access the online container to perform the migrations using the commands **docker-compose exec web bash** and **python manage.py migrate**

```
MBP-of-EthicalBacteria/:ProjectSetup-Django ethicalbacteria$ docker-compose exec web bash
root@893b59380946:/code# python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK
root@893b59380946:/code#
```

## Basic commands

1. List running containers.
In another terminal window, list the running Docker processes with the **docker ps** command.

```
MBP-of-EthicalBacteria/:ProjectSetup-Django ethicalbacteria$ docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                    NAMES
d32805de44ea        projectsetup-django_web   "python manage.py ru…"   7 seconds ago       Up 5 seconds        0.0.0.0:8000->8000/tcp   projectsetup-django_web_1
9f76b52375cf        postgres:latest           "docker-entrypoint.s…"   18 seconds ago      Up 16 seconds       0.0.0.0:5432->5432/tcp   projectsetup-django_db_1
```

2. Shut down services and clean up by using either of these methods:
Stop the application by typing Ctrl-C in the same shell in where you started it:
```
^CGracefully stopping... (press Ctrl+C again to force)
Stopping projectsetup-django_web_1 ... done
Stopping projectsetup-django_db_1  ... done
```
Or, for a more elegant shutdown, switch to a different shell, and run docker-compose down from the top level of your Django sample project directory.

```
MBP-of-EthicalBacteria/:ProjectSetup-Django ethicalbacteria$ docker-compose down
Stopping projectsetup-django_web_1 ... done
Stopping projectsetup-django_db_1  ... done
Removing projectsetup-django_web_1 ... done
Removing projectsetup-django_db_1  ... done
Removing network projectsetup-django_default
```

3. Access the online container **docker-compose exec web bash** and use the Django command-line utility with **python manage.py help**

```
root@893b59380946:/code# python manage.py help

Type 'manage.py help <subcommand>' for help on a specific subcommand.

Available subcommands:

[auth]
    changepassword
    createsuperuser

[contenttypes]
    remove_stale_contenttypes

[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver

[sessions]
    clearsessions

[staticfiles]
    collectstatic
    findstatic
    runserver
```

4. To access the dashboard go to this address **http://localhost:8000/admin/**. You must create identifiers with the command **python manage.py createsuperuser**

```
root@893b59380946:/code# python manage.py createsuperuser
Username (leave blank to use 'root'): djangouser
Email address: user@django.com
Password:
Password (again):
Superuser created successfully.
```

5. To access pgadmin **http://localhost:8080** (identifiers in the **Dockerfile** file)  

## Sources
https://docs.docker.com/compose/django/  
https://hub.docker.com/  
https://www.djangoproject.com/  
https://pypi.org/  
