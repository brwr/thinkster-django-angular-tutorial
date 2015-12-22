# Serializing the Account Model
The AngularJS application we are going to build will make AJAX requests to the server to get the data it intends to display. Before we can send that data back to the client, we need to format it in a way that the client can understand; in this case, we choose JSON. The process of transforming Django models to JSON is called serialization and that is what we will talk about now.

{video: serialize-account-model}

As the model we want to serialize is called `Account`, the serializer we will create is going to be called `AccountSerializer`.

## Django REST Framework
As part of the boilerplate project you cloned earlier, we have included a project called Django REST Framework. Django REST Framework is a toolkit that provides a number of features common to most web applications, including serializers. We will make use of these features throughout the tutorial to save us both time and frustration. Our first look at Django REST Framework starts here.

## AccountSerializer
Before we write our serializers, let's create a `serializers.py` file inside our `authentication` app:

    $ touch authentication/serializers.py

{x: create_serializers_module}
Create a `serializers.py` file inside the `authentication` app

Open `authentication/serializers.py` and add the following code and imports:

    from django.contrib.auth import update_session_auth_hash

    from rest_framework import serializers

    from authentication.models import Account


    class AccountSerializer(serializers.ModelSerializer):
        password = serializers.CharField(write_only=True, required=False)
        confirm_password = serializers.CharField(write_only=True, required=False)

        class Meta:
            model = Account
            fields = ('id', 'email', 'username', 'created_at', 'updated_at',
                      'first_name', 'last_name', 'tagline', 'password',
                      'confirm_password',)
            read_only_fields = ('created_at', 'updated_at',)

            def create(self, validated_data):
                return Account.objects.create(**validated_data)

            def update(self, instance, validated_data):
                instance.username = validated_data.get('username', instance.username)
                instance.tagline = validated_data.get('tagline', instance.tagline)

                instance.save()

                password = validated_data.get('password', None)
                confirm_password = validated_data.get('confirm_password', None)

                if password and confirm_password and password == confirm_password:
                    instance.set_password(password)
                    instance.save()

                update_session_auth_hash(self.context.get('request'), instance)

                return instance

{x: create_account_serializer}
Make a serializer called `AccountSerializer` in `authentication/serializers.py`

{info}
From here on, we will declare imports that are used in each snippet. These may already be present in the file. If so, they do not need to be added a second time.

Let's take a closer look.

    password = serializers.CharField(write_only=True, required=False)
    confirm_password = serializers.CharField(write_only=True, required=False)

Instead of including `password` in the `fields` tuple, which we will talk about in a few minutes, we explicitly define the field at the top of the `AccountSerializer` class. The reason we do this is so we can pass the `required=False` argument. Each field in `fields` is required, but we don't want to update the user's password unless they provide a new one.

`confirm_password` is similar to `password` and is used only to make sure the user didn't make a typo on accident.

Also note the use of the `write_only=True` argument. The user's password, even in it's hashed and salted form, should not be visible to the client in the AJAX response.

    class Meta:

The `Meta` sub-class defines metadata the serializer requires to operate. We have defined a few common attributes of the `Meta` class here.

    model = Account

Because this serializers inherits from `serializers.ModelSerializer`, it should make sense that we must tell it which model to serialize. Specifying the model creates a guarantee that only attributes of that model or explicitly created fields can be serialized. We will cover serializing model attributes now and explicitly created fields shortly.

    fields = ('id', 'email', 'username', 'created_at', 'updated_at',
              'first_name', 'last_name', 'tagline', 'password',
              'confirm_password',)


The `fields` attribute of the `Meta` class is where we specify which attributes of the `Account` model should be serialized. We must be careful when specifying which fields to serialize because some fields, like `is_superuser`, should not be available to the client for security reasons.

    read_only_fields = ('created_at', 'updated_at',)

If you recall, when we created the `Account` model, we made the `created_at` and `updated_at` fields self-updating. Because of this feature, we add them to a list of fields that should be read-only.

    def create(self, validated_data):
        # ...

    def update(self, instance, validated_data):
        # ...

Earlier we mentioned that we sometimes want to turn JSON into a Python object. This is called deserialization and it is handled by the `.create()` and `.update()` methods. When creating a new object, such as an `Account`, `.create()` is used. When we later update that `Account`, `.update()` is used.

    instance.username = attrs.get('username', instance.username)
    instance.tagline = attrs.get('tagline', instance.tagline)

We will let the user update their username and tagline attributes for now. If these keys are present in the arrays dictionary, we will use the new value. Otherwise, the current value of the `instance` object is used. Here, `instance` is of type `Account`.

    password = validated_data.get('password', None)
    confirm_password = validated_data.get('confirm_password', None)

    if password and confirm_password and password == confirm_password:
        instance.set_password(password)
        instance.save()

Before updating the user's password, we need to confirm they have provided values for both the `password` and `password_confirmation` field. We then check to make sure these two fields have equivelant values.

After we verify that the password should be updated, we much use `Account.set_password()` to perform the update. `Account.set_password()` takes care of storing passwords in a secure way. It is important to note that we must explicitly save the model after updating the password.

{info}
This is a naive implementation of how to validate a password. I would not recommend using this in a real-world system, but for our purposes this does nicely.

    update_session_auth_hash(self.context.get('request'), instance)

When a user's password is updated, their session authentication hash must be explicitly updated. If we don't do this here, the user will not be authenticated on their next request and will have to log in again.

## Checkpoint
By now we should have no problem seeing the serialized JSON of an `Account` object. Open up the Django shell again by running `python manage.py shell` and try typing the following commands:

    >>> from authentication.models import Account
    >>> from authentication.serializers import AccountSerializer
    >>> account = Account.objects.latest('created_at')
    >>> serialized_account = AccountSerializer(account)
    >>> serialized_account.data.get('email')
    >>> serialized_account.data.get('username')

{x: checkpoint_auth_serializers}
Make sure your `AccountSerializer` serializer is working
