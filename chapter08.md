# Making new posts

{video: make-new-post}

Given that we already have the necessary endpoints in place, the next thing we need to let users make new posts is an interface. We accomplish this by adding a button to the bottom-right corner of the screen. When this button is clicked, a modal shows up asking the user to type in their post.

We only want this button to show up on the index page for now, so open `static/templates/layout/index.html` and add the following snippet to the bottom of the file:

    <a class="btn btn-primary btn-fab btn-raised mdi-content-add btn-add-new-post"
      href="javascript:void(0)"
      ng-show="vm.isAuthenticated"
      ng-dialog="/static/templates/posts/new-post.html"
      ng-dialog-controller="NewPostController as vm"></a>

The anchor tag in this snippet uses the `ngDialog` directive we included as a dependency earlier to show a modal when the user wants to submit a new post.

Because we want the button to be fixed to the bottom-right corner of the screen, we also need to add a new CSS rule.

Open `static/stylesheets/styles.css` and add this rule to the bottom of the file:

    .btn-add-new-post {
      position: fixed;
      bottom: 20px;
      right: 20px;
    }

{x: new_post_modal_button}
Add a button for showing the new post modal

{x: style_new_post_button}
Style the new post button

## An interface for submitting new posts
Now we need to create the form the user will type their new post into. Open `static/templates/posts/new-post.html` and add the following to the bottom of the file:

    <form role="form" ng-submit="vm.submit()">
      <div class="form-group">
        <label for="post__content">New Post</label>
        <textarea class="form-control" 
                  id="post__content" 
                  rows="3" 
                  placeholder="ex. This is my first time posting on Not Google Plus!" 
                  ng-model="vm.content">
        </textarea>
      </div>

      <div class="form-group">
        <button type="submit" class="btn btn-primary">
          Submit
        </button>
      </div>
    </form>

{x: new_post_template}
Make a template for adding new posts in `static/templates/posts/new-post.html`

## Controlling the new post interface with NewPostController
Create `static/javascripts/posts/controllers/new-post.controller.js` with the following content:

    /**
    * NewPostController
    * @namespace thinkster.posts.controllers
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.posts.controllers')
        .controller('NewPostController', NewPostController);

      NewPostController.$inject = ['$rootScope', '$scope', 'Authentication', 'Snackbar', 'Posts'];

      /**
      * @namespace NewPostController
      */
      function NewPostController($rootScope, $scope, Authentication, Snackbar, Posts) {
        var vm = this;

        vm.submit = submit;

        /**
        * @name submit
        * @desc Create a new Post
        * @memberOf thinkster.posts.controllers.NewPostController
        */
        function submit() {
          $rootScope.$broadcast('post.created', {
            content: vm.content,
            author: {
              username: Authentication.getAuthenticatedAccount().username
            }
          });

          $scope.closeThisDialog();

          Posts.create(vm.content).then(createPostSuccessFn, createPostErrorFn);


          /**
          * @name createPostSuccessFn
          * @desc Show snackbar with success message
          */
          function createPostSuccessFn(data, status, headers, config) {
            Snackbar.show('Success! Post created.');
          }

          
          /**
          * @name createPostErrorFn
          * @desc Propogate error event and show snackbar with error message
          */
          function createPostErrorFn(data, status, headers, config) {
            $rootScope.$broadcast('post.created.error');
            Snackbar.error(data.error);
          }
        }
      }
    })();

{x: new_post_controller}
Make a `NewPostController` in `static/javascripts/posts/controllers/new-post.controller.js`

There are a few things going on here that we should talk about.

    $rootScope.$broadcast('post.created', {
      content: $scope.content,
      author: {
        username: Authentication.getAuthenticatedAccount().username
      }
    });

Earlier we set up an event listener in `IndexController` that listened for the `post.created` event and then pushed the new post onto the front of `vm.posts`. Let's look at this a little more closely, as this turns out to be an important feature of rich web applications.

What we are doing here is being *optimistic* that the API response from `Posts.create()` will contain a 200 status code telling us everything went according to plan. This may seem like a bad idea at first. Something could go wrong during the request and then our data is stale. Why don't we just wait for the response?

When I said we are increasing the *perceived* performance of our app, this is what I was talking about. We want the user to *perceive* the response as instant.

The fact of the matter is that this call will rarely fail. There are only two cases where this will reasonably fail: either the user is not authenticated or the server is down.

In the case where the user is not authenticated, they shouldn't be submitting new posts anyways. Consider the error to be a small punishment for the user doing things they shouldn't.

If the server is down, then there is nothing we can do. Unless the user already had the page loaded before the server crashed, they wouldn't be able to see this page anyways.

Other things that could possibly go wrong make up such a small percentage that we are willing to allow a slightly worse experience to make the experience better for the 99.9% of cases where everything is working properly.

Furthermore, the object we pass as the second argument is meant to emulate the response from the server. This is not the best design pattern because it assumes we know what the response will look like. If the response changes, we have to update this code. However, given what we have, this is an acceptable cost.

So what happens when the API call returns an error?

    $rootScope.$broadcast('post.created.error');

If the error callback is triggered, then we will broadcast a new event: `post.created.error`. The event listener we set up earlier will be trigger by this event and remove the post at the front of `vm.posts`. We will also show the error message to the user to let them know what happened.

    $scope.closeThisDialog();

This is a method provided by `ngDialog`. All it does is close the model we have open. It's also worth nothing that `closeThisDialog()` is not stored on the ViewModel, so we must call `$scope.closeThisDialog()` instead of `vm.closeThisDialog()`.

Be sure to include `new-post.controller.js` in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/posts/controllers/new-post.controller.js' %}"></script>

{x: js_include_new_post}
Include `new-post.controller.js` in `javascripts.html`

## Checkpoint
Visit `http://localhost:8000/` and click the + button in the bottom-right corner. Fill out this form to create a new post. You will know everything worked because the new post will be displayed at the top of the page.

{x: checkpoint_new_post}
Create a new `Post` object via the interface you've just created
