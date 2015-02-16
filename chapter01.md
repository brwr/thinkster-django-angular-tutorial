# Extending Django's built-in User model
Django has a built-in `User` model that offers a lot of functionality. The problem with `User` is that the model cannot be extended to include more information. For example, we will be giving our users a tagline to display on their profile. `User` does not have a tagline attribute and we cannot add one ourselves.

`User` inherits from `AbstractBaseUser`. That is where `User` gets most of it's functionality. By creating a new model called `Account` and inheriting from `AbstractBaseUser`, we will get the necessary functionality of `User` (password hashing, session management, etc) and be able to extend `Account` to include extra information, such as a tagline.

In Django, the concept of an "app" is used to organize code in a meaningful way. An app is a module that houses code for models, view, serializers, etc that are all related in some way. By way of example, our first step in building our Django and AngularJS web application will be to create an app called `authentication`. The `authentication` app will contain the code relevent to the `Account` model we just talked about as well as views for logging in, logging out and register.

Make a new app called `authentication` by running the following command:

    $ python manage.py startapp authentication

{x: startapp_authentication}
Make a Django app named `authentication`


## Creating the Account model

{video: account-manager}

To get started, we will create the `Account` model we just talked about.

Open `authentication/models.py` in your favorite text editor and edit it to reflect the following:

    from django.contrib.auth.models import AbstractBaseUser
    from django.db import models

    class Account(AbstractBaseUser):
        email = models.EmailField(unique=True)
        username = models.CharField(max_length=40, unique=True)

        first_name = models.CharField(max_length=40, blank=True)
        last_name = models.CharField(max_length=40, blank=True)
        tagline = models.CharField(max_length=140, blank=True)

        is_admin = models.BooleanField(default=False)

        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)

        objects = AccountManager()

        USERNAME_FIELD = 'email'
        REQUIRED_FIELDS = ['username']

        def __unicode__(self):
            return self.email

        def get_full_name(self):
            return ' '.join([self.first_name, self.last_name])

        def get_short_name(self):
            return self.first_name

{x: create_user_profile_model}
Make a new model in `authentication/models.py` called `Account`

Let's take a closer look at each attribute and method in turn.

    email = models.EmailField(unique=True)

    # ...

    USERNAME_FIELD = 'email'

Django's built-in `User` requires a username. That username is used for logging the user in. By contrast, our application will use the user's email address for this purpose.

To tell Django that we want to treat the email field as the username for this model, we set the `USERNAME_FIELD` attribute to `email`. The field specified by `USERNAME_FIELD` must be unique, so we pass the `unique=True` argument in the email field.

    username = models.CharField(max_length=40, unique=True)

Even though we will log in with our email address, we still want the user to have a username. We need some to display on their posts and profile page. We will also use the username in our URLs, so the username must be unique. To this end, we pass the `unique=True` argument in the username field.

    first_name = models.CharField(max_length=40, blank=True)
    last_name = models.CharField(max_length=40, blank=True)
 
Ideally we should have a more personal way to reference our users. Because we understand that not all users are comfortable giving out their personal details, we make the `first_name` and `last_name` fields optional by passing the `blank=True` argument.

    tagline = models.CharField(max_length=140, blank=True)

As mentioned before, the `tagline` attribute will be displayed on the user's profile. This gives the profile a hint of the user's personally, so it is worth including.

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

The `created_at` field records the time that the `Account` object was created. By passing `auto_now_add=True` to `models.DateTimeField`, we are telling Django that this field should be automatically set when the object is created and non-editable after that.

Similar to `created_at`, `updated_at` is automatically set by Django. The difference between `auto_now_add=True` and `auto_now=True` is that `auto_now=True` causes the field to update each time the object is saved.

    objects = AccountManager()

When you want to get a model instance in Django, you use an expression of the form `Model.objects.get(**kwargs)`. The `objects` attribute here is a `Manager` class whose name typically follows the `<model name>Manager` convention. In our case, we will create an `AccountManager` class. We will do this momentarily.

    REQUIRED_FIELDS = ['username']

We will be displaying the username in multiple places. To this end, having a username is not optional, so we include it in the `REQUIRED_FIELDS` list. Normally the `required=True` argument would accomplish this goal, but because this model is replacing the `User` model, Django requires us to specify required fields in this way.

    def __unicode__(self):
        return self.email

When working in the shell, as we will see shortly, the default string representation of an `Account` object looks something like `<Account: Account>`. Because we will have many different accounts, this is not very helpful. Overwriting `__unicode__()` will change this default behavior. Here we choose to show the user's email instead. The string representation of an account with the email `james@notgoogleplus.com` will now look like `<Account: james@notgoogleplus.com>`.

    def get_full_name(self):
        return ' '.join([self.first_name, self.last_name])

    def get_short_name(self):
        return self.first_name

`get_full_name()` and `get_short_name()` are Django conventions. We won't be using either of these methods, but it is still a good idea to include them to comply with Django conventions.

## Making a Manager class for Account
When substituting a customer user model, it is required that you also define a related `Manager` class the overrides the `create_user()` and `create_superuser()` methods.

With `authentication/models.py` still open, add the following class above the `Account` class:

    from django.contrib.auth.models import BaseUserManager


    class AccountManager(BaseUserManager):
        def create_user(self, email, password=None, **kwargs):
            if not email:
                raise ValueError('Users must have a valid email address.')

            if not kwargs.get('username'):
                raise ValueError('Users must have a valid username.')

            account = self.model(
                email=self.normalize_email(email), username=kwargs.get('username')
            )

            account.set_password(password)
            account.save()

            return account

        def create_superuser(self, email, password, **kwargs):
            account = self.create_user(email, password, **kwargs)

            account.is_admin = True
            account.save()

            return account

Like we did with `Account`, let's step through this file line-by-line. We will only cover new information.

    if not email:
        raise ValueError('Users must have a valid email address.')

    if not kwargs.get('username'):
        raise ValueError('Users must have a valid username.')

Because users are required to have both an email and a username, we should raise an error if either of these attributes are missing.

    account = self.model(
        email=self.normalize_email(email), username=kwargs.get('username')
    )

Since we haven't defined a `model` attribute on the `AccountManager` class, `self.model` refers to the `model` attribute of `BaseUserManager`. This defaults to `settings.AUTH_USER_MODEL`, which we will change in just a moment to point to the `Account` class.

    account = self.create_account(email, password, **kwargs)

    account.is_admin = True
    account.save()

Writing the same thing more than once sucks. Instead of copying all of the code from `create_account` and pasting it in `create_superuser`, we simply let `create_user` handle the actual creation. This frees up `create_superuser` to only worry about turning an `Account` into a superuser.

## Changing the Django AUTH_USER_MODEL setting
Even though we have created this `Account` model, the command `python manage.py createsuperuser` (which we will talk more about shortly) still creates `User` objects. This is because, at this point, Django still believes that `User` is the model we want to use for authentication.

To set things straight and start using `Account` as our authentication model, we have to update `settings.AUTH_USER_MODEL`.

Open `thinkster_django_angular_tutorial/settings.py` and add the following to the end of the file:

    AUTH_USER_MODEL = 'authentication.Account'

{x: auth_user_model_setting}
Change `settings.AUTH_USER_MODEL` to use `Account` instead of `User`

This line tells Django to look in the `authentication` app and find a model named `Account`. 

## Installing your first app
In Django, you must explicitly declare which apps are being used. Since we haven't added our `authentication` app to the list of installed apps yet, we will do that now.

Open `thinkster_django_angular_boilerplate/settings.py` and append `'authentication',` to `INSTALLED_APPS` like so:

    INSTALLED_APPS = (
        ...,
        'authentication',
    )

{x: install_authentication_app}
Install the `authentication` app

## Migrating your first app
When Django 1.7 was released, it  was like Christmas in September! Migrations had finally arrived!

Anyone with a background in Rails will find the concept of migrations familiar. In short, migrations handle the SQL needed to update the schema of our database so you don't have to. By way of example, consider the `Account` model we just created. These models need to be stored in the database, but our database doesn't have a table for `Account` objects yet. What do we do? We create our first migration! The migration will handle adding the tables to the database and offer us a way to rollback the changes if we make a mistake.

When you're ready, generate the migrations for the `authentication` app and apply them:

    $ python manage.py makemigrations
    Migrations for 'authentication':
        0001_initial.py:
            - Create model Account
    $ python manage.py migrate
    Operations to perform:
        Synchronize unmigrated apps: rest_framework
        Apply all migrations: admin, authentication, contenttypes, auth, sessions
    Synchronizing apps without migrations:
        Creating tables...
        Installing custom SQL...
        Installing indexes...
    Running migrations:
        Applying authentication.0001_initial... OK


{info}
From now on, the output from migration commands will not be included for brevity.

{x: initial_migration}
Generate the migrations for the `authentication` app and apply them

## Making yourself a superuser
Let's talk more about the `python manage.py createsuperuser` command from a few minutes ago.

Different users have different levels of access in any given application. Some users are admins and can do anything, while some are just regular users whose actions should be limited. In Django, a super user is the highest level of access you can have. Because we want the ability to work will all facets of our application, we will create a super user. That is what `python manage.py createsuperuser` does.

After running the command, Django will prompt you for some information and create an `Account` with superuser access. Go ahead and give it a try.

    $ python manage.py createsuperuser

{x: create_superuser}
Make a new super user `Account`

## Checkpoint
To make sure everything is properly configured, let's take a quick break and open Django's shell:

    $ python manage.py shell

You should see a new prompt: `>>>`. Inside the shell, we can get the `Account` we just created like so:

    >>> from authentication.models import Account
    >>> a = Account.objects.latest('created_at')

If everything went well, you should be able to access the various attributes of your `Account` object:

    >>> a
    >>> a.email
    >>> a.username

{x: get_new_account}
Access the `Account` object you just created
