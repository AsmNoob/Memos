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
**Models** = Source of your data about your data, the objective is to define your data model in one place and automatically derive things from it.
**Migrations** = History of your models, that django uses to update your database to match the current models.

To create models, simply edit the models.py in your application directory:

*Each model is represented by a class that subclasses django.db.models.Model. Each model has a number of class variables, each of which represents a database field in the model.*

*Each field is represented by an instance of the Field class, it tells django the type of Data it holds.*

		from django.db import models


		class Question(models.Model):
		    question_text = models.CharField(max_length=200)
		    pub_date = models.DateTimeField('date published')


		class Choice(models.Model):
		    question = models.ForeignKey(Question)
		    choice_text = models.CharField(max_length=200)
		    votes = models.IntegerField(default=0)


**How to activate models**:

*This code allows to create a database schema and an API to access Question and Choice objects*

**Note**: We have to add it first to our INSTALLED_APPS () so Django updates it, simply add **'yourAppsName'.

To update the models, run the following command:

		python manage.py makemigrations yourAppsName




