# Making a Post model
In this section we will make a new app and create a `Post` model similar to a status on Facebook or a tweet on Twitter. After we create our model we will move on to serializing `Post`s and then we will create a few new endpoints for our API.

{video: making-post-model}

## Making a posts app
First things first: go ahead and create a new app called `posts`.

    $ python manage.py startapp posts

{x: django_app_posts}
Make a new app named `posts`

Remember: whenever you create a new app you have to add it to the `INSTALLED_APPS` setting. Open `thinkster_django_angular_boilerplate/settings.py` and modify it like so:

    INSTALLED_APPS = (
        # ...
        'posts',
    )

## Making the Post model
After you create the `posts` app Django made a new file called `posts/models.py`. Go ahead and open it up and add the following:

from django.db import models

from authentication.models import Account


    class Post(models.Model):
        author = models.ForeignKey(Account)
        content = models.TextField()

        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)

        def __unicode__(self):
            return '{0}'.format(self.content)

{x: django_model_post}
Make a new model called `Post` in `posts/models.py`

Our method of walking through the code line-by-line is working well so far. Why mess with a good thing? Let's do it.

    author = models.ForeignKey(Account)

Because each `Account` can have many `Post` objects, we need to set up a many-to-one relation.

The way to do this in Django is with a `ForeignKey` field to associate each `Post` with a `Account`.

Django is smart enough to know the foreign key we've set up here should be reversible. That is to say, given a `Account`, you should be able to access that user's `Post`s. In Django these `Post` objects can be accessed through `Account.post_set` (not `Account.posts`).

Now that the model exists, don't forget to migrate.

    $ python manage.py makemigrations
    $ python manage.py migrate

{x: django_model_post_migrate}
Make migrations for `Post` and apply them

## Serializing the Post model
Create a new file in `posts/` called `serializers.py` and add the following:

    from rest_framework import serializers

    from authentication.serializers import AccountSerializer
    from posts.models import Post


    class PostSerializer(serializers.ModelSerializer):
        author = AccountSerializer(read_only=True, required=False)

        class Meta:
            model = Post

            fields = ('id', 'author', 'content', 'created_at', 'updated_at')
            read_only_fields = ('id', 'created_at', 'updated_at')

        def get_validation_exclusions(self, *args, **kwargs):
            exclusions = super(PostSerializer, self).get_validation_exclusions()

            return exclusions + ['author']

{x: django_serializer_postserializer}
Make a new serializer called `PostSerializer` in `posts/serializers.py`

There isn't much here that's new, but there is one line in particular I want to look at.

    author = AccountSerializer(read_only=True, required=False)

We explicitly defined a number of fields in our `AccountSerializer` from before, but this definition is a little different.

When serializing a `Post` object, we want to include all of the author's information. Within Django REST Framework, this is known as a nested relationship. Basically, we are serializing the `Account` related to this `Post` and including it in our JSON.

We pass `read_only=True` because we should not be updating an `Account` object with a `PostSerializer`. We also set `required=False` here because we will set the author of this post automatically.

    def get_validation_exclusions(self, *args, **kwargs):
        exclusions = super(PostSerializer, self).get_validation_exclusions()

        return exclusions + ['author']

For the same reason we use `required=False`, we must also add `author` to the list of validations we wish to skip.

## Making API views for Post objects
The next step in creating `Post` objects is adding an API endpoint that will handle performing actions on the `Post` model such as create or update.

Replace the contents of `posts/views.py` with the following:

    from rest_framework import permissions, viewsets
    from rest_framework.response import Response

    from posts.models import Post
    from posts.permissions import IsAuthorOfPost
    from posts.serializers import PostSerializer


    class PostViewSet(viewsets.ModelViewSet):
        queryset = Post.objects.order_by('-created_at')
        serializer_class = PostSerializer

        def get_permissions(self):
            if self.request.method in permissions.SAFE_METHODS:
                return (permissions.AllowAny(),)
            return (permissions.IsAuthenticated(), IsAuthorOfPost(),)

        def perform_create(self, serializer):
            instance = serializer.save(author=self.request.user)

            return super(PostViewSet, self).perform_create(serializer)



    class AccountPostsViewSet(viewsets.ViewSet):
        queryset = Post.objects.select_related('author').all()
        serializer_class = PostSerializer

        def list(self, request, account_username=None):
            queryset = self.queryset.filter(author__username=account_username)
            serializer = self.serializer_class(queryset, many=True)

            return Response(serializer.data)

{x: django_viewset_post}
Make a `PostViewSet` viewset

{x: django_viewset_account_post}
Make an `AccountPostsViewSet` viewset

Do these views look similar? They aren't that different than the ones we made to create `User` objects.

    def perform_create(self, serializer):
        instance = serializer.save(author=self.request.user)

        return super(PostViewSet, self).perform_create(serializer)


`perform_create` is called before the model of this view is saved.

When a `Post` object is created it has to be associated with an author. Making the author type in their own username or id when creating adding a post to the site would be a bad experience, so we handle this association for them with the `perform_create` hook. We simply grab the user associated with this request and make them the author of this `Post`.

    def get_permissions(self):
        if self.request.method in permissions.SAFE_METHODS:
            return (permissions.AllowAny(),)
        return (permissions.IsAuthenticated(), IsAuthorOfPost(),)


Similar to the permissions we used for the `Account` viewset, dangerous HTTP methods require the user be authenticated and authorized to make changes to this `Post`. We will created the `IsAuthorOfPost` permission shortly. If the HTTP method is safe, we allow anyone to access this view.

    class AccountPostsViewSet(viewsets.ViewSet):

This viewset will be used to list the posts associated with a specific `Account`.

    queryset = self.queryset.filter(author__username=account_username)

Here we filter our queryset based on the author's username. The `account_username` argument will be supplied by the router we will create in a few minutes.

## Making the IsAuthorOfPost permission
Create `permissions.py` in the `posts/` directory with the following content:

    from rest_framework import permissions


    class IsAuthorOfPost(permissions.BasePermission):
        def has_object_permission(self, request, view, post):
            if request.user:
                return post.author == request.user
            return False

{x: django_permission_isauthenticatedandownsobject}
Make a new permission called `IsAuthenticatedAndOwnsObject` in `posts/permissions.py`

We will skip the explanation for this. This permission is almost identical to the one we made previously.

## Making an API endpoint for posts
With the views created, it's time to add the endpoints to our API.

Open `thinkster_django_angular_boilerplate/urls.py` and add the following import:

    from posts.views import AccountPostsViewSet, PostViewSet

Now add these lines just above `urlpatterns = patterns(`:

    router.register(r'posts', PostViewSet)

    accounts_router = routers.NestedSimpleRouter(
        router, r'accounts', lookup='account'
    )
    accounts_router.register(r'posts', AccountPostsViewSet)

`accounts_router` provides the nested routing need to access the posts for a specific `Account`. You should also now add `accounts_router` to `urlpatterns` like so:

    urlpatterns = patterns(
      # ...
      
      url(r'^api/v1/', include(router.urls)),
      url(r'^api/v1/', include(accounts_router.urls)),

      # ...
    )

{x: django_url_postviewset}
Make an API endpoint for the `PostViewSet` viewset

{x: django_url_accountpostsviewset}
Make an API endpoint for the `AccountPostsViewSet` viewset

## Checkpoint
At this point, feel free to open up your shell with `python manage.py shell` and play around with creating and serializing `Post` objects.

    >>> from authentication.models import Account
    >>> from posts.models import Post
    >>> from posts.serializers import PostSerializer
    >>> account = Account.objects.latest('created_at')
    >>> post = Post.objects.create(author=account, content='I promise this is not Google Plus!')
    >>> serialized_post = PostSerializer(post)
    >>> serialized_post.data

{x: checkpoint_create_post}
Play around with the `Post` model and `PostSerializer` serializer in Django's shell

We will confirm the views are working at the end of the next section.
