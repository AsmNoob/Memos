Introduction to Django
======================

Start a project
---------------

		django-admin.py startproject mysite

**NOTE**: on a server you should do it in the /var/www/ directory.

What is created:

		mysite/
		    manage.py
		    mysite/
		        __init__.py
		        settings.py
		        urls.py
		        wsgi.py


Setting up your database
------------------------

*The default database manager is Sqlite(included in python)*

The installed apps, may need databases, to initialize them run this:

		python manage.py migrate

This command will look at **INSTALLED_APPS** and create a database table if necessary.

Run your project
----------------

		python manage.py runserver


You can also choose the port and IP : 

	python manage.py runserver IP:PORT or PORT

**NOTE**: Do not use this server other than for development purposes and only restart the server if you add files, changes are automatically loaded.

**NOTE**: If you have a "TIME_ZONE" problem, go in your *settings.py* and change it to 'Europe/YourCity'.

Creating models
---------------

### Applications

To create your app run the following command:

		python manage.py startapp yourAppsName (usually polls)

It creates this: 

		YourAppsName/
		    __init__.py
		    admin.py
		    migrations/
		        __init__.py
		    models.py
		    tests.py
		    views.py

**Writing a Database web app = Define the models**





