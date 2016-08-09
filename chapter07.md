# Rendering Post objects
Until now, the index page has been empty. Now that we have handled authentication and the backend details for the `Post` model, it's time to give our users something to interact with. We will do this by creating a service that handles retrieving and creating `Post`s and some controllers and directives for handling how the data is displayed.

{video: render-post-object}

## A module for posts
Let's define the posts modules.

Create a file in `static/javascripts/posts` called `posts.module.js` and add the following:

    (function () {
      'use strict';

      angular
        .module('thinkster.posts', [
          'thinkster.posts.controllers',
          'thinkster.posts.directives',
          'thinkster.posts.services'
        ]);

      angular
        .module('thinkster.posts.controllers', []);

      angular
        .module('thinkster.posts.directives', ['ngDialog']);

      angular
        .module('thinkster.posts.services', []);
    })();

{x: posts_module}
Define the `thinkster.posts` module

Remember to add `thinkster.posts` as a dependency of `thinkster` in `thinkster.js`:

    angular
      .module('thinkster', [
        'thinkster.config',
        'thinkster.routes',
        'thinkster.authentication',
        'thinkster.layout',
        'thinkster.posts'
      ]);

{x: posts_module_dep_thinkster}
Add `thinkster.posts` as a dependency of the `thinkster` module

There are two things worth noting about this module.

First, we have created a module named `thinkster.posts.directives`. As you probably guessed, this means we will introduce the concept of directives to our app in this chapter.

Secondly, the `thinkster.posts.directives` module requires the `ngDialog` module. `ngDialog` is included in the boilerplate project and handles the display of modals. We will use a modal in the next chapter when we write the code for creating new posts.

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/posts/posts.module.js' %}"></script>

{x: posts_include_module}
Include `posts.module.js` in `javascripts.html`

## Making a Posts service
Before we can render anything, we need to transport data from the server to the client.

Create a file at `static/javascripts/posts/services/` called `posts.service.js` and add the following:

    /**
    * Posts
    * @namespace thinkster.posts.services
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.posts.services')
        .factory('Posts', Posts);

      Posts.$inject = ['$http'];

      /**
      * @namespace Posts
      * @returns {Factory}
      */
      function Posts($http) {
        var Posts = {
          all: all,
          create: create,
          get: get
        };

        return Posts;

        ////////////////////
        
        /**
        * @name all
        * @desc Get all Posts
        * @returns {Promise}
        * @memberOf thinkster.posts.services.Posts
        */
        function all() {
          return $http.get('/api/v1/posts/');
        }


        /**
        * @name create
        * @desc Create a new Post
        * @param {string} content The content of the new Post
        * @returns {Promise}
        * @memberOf thinkster.posts.services.Posts
        */
        function create(content) {
          return $http.post('/api/v1/posts/', {
            content: content
          });
        }

        /**
         * @name get
         * @desc Get the Posts of a given user
         * @param {string} username The username to get Posts for
         * @returns {Promise}
         * @memberOf thinkster.posts.services.Posts
         */
        function get(username) {
          return $http.get('/api/v1/accounts/' + username + '/posts/');
        }
      }
    })();

{x: posts_service}
Make a new factory called `Posts` in `static/javascripts/posts/services/posts.service.js`

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/posts/services/posts.service.js' %}"></script>

{x: posts_service_include_javascripts}
Include `posts.service.js` in `javascripts.html`

This code should look pretty familiar. It is very similar to the services we created before.

The `Posts` service only has two methods: `all` and `create`.

On the index page, we will use `Posts.all()` to get the list of objects we want to display. We will use `Posts.create()` to let users add their own posts.

## Making an interface for the index page
Create `static/templates/layout/index.html` with the following contents:

    <posts posts="vm.posts" ng-show="vm.posts && vm.posts.length"></posts>

{x: index_template}
Create the index template

We will add a little more later, but not much. Most of what we need will be in the template we create for the posts directive next.

## Making a Snackbar service
In the boilerplate project for this tutorial, we've included SnackbarJS. SnackbarJS is a small JavaScript library that makes showing snackbars (a concept from Google's Material Design) easy. Here, we will create a service to include this functionality in our AngularJS application.

Open `static/javascripts/utils/services/snackbar.service.js` and add the following:

    /**
    * Snackbar
    * @namespace thinkster.utils.services
    */
    (function ($, _) {
      'use strict';

      angular
        .module('thinkster.utils.services')
        .factory('Snackbar', Snackbar);

      /**
      * @namespace Snackbar
      */
      function Snackbar() {
        /**
        * @name Snackbar
        * @desc The factory to be returned
        */
        var Snackbar = {
          error: error,
          show: show
        };

        return Snackbar;

        ////////////////////
        
        /**
        * @name _snackbar
        * @desc Display a snackbar
        * @param {string} content The content of the snackbar
        * @param {Object} options Options for displaying the snackbar
        */
        function _snackbar(content, options) {
          options = _.extend({ timeout: 3000 }, options);
          options.content = content;

          $.snackbar(options);
        }


        /**
        * @name error
        * @desc Display an error snackbar
        * @param {string} content The content of the snackbar
        * @param {Object} options Options for displaying the snackbar
        * @memberOf thinkster.utils.services.Snackbar
        */
        function error(content, options) {
          _snackbar('Error: ' + content, options);
        }


        /**
        * @name show
        * @desc Display a standard snackbar
        * @param {string} content The content of the snackbar
        * @param {Object} options Options for displaying the snackbar
        * @memberOf thinkster.utils.services.Snackbar
        */
        function show(content, options) {
          _snackbar(content, options);
        }
      }
    })($, _);

{x: angularjs_snackbar_service}
Make a `Snackbar` service

Don't forget to set up your modules. Open `static/javascripts/utils/utils.module.js` and add the following:

    (function () {
      'use strict';

      angular
        .module('thinkster.utils', [
          'thinkster.utils.services'
        ]);

      angular
        .module('thinkster.utils.services', []);
    })();

And make `thinksters.utils` a dependency of `thinkster` in `static/javascripts/thinkster.js`:

    angular
      .module('thinkster', [
        // ...
        'thinkster.utils',
        // ...
      ]);

{x: angularjs_utils_module}
Set up your modules for the `thinkster.utils` module

{x: angularjs_include_utils}
Make `thinkster.utils` a dependency of `thinkster`

The last step for this service is to include the new JavaScript files in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/utils/utils.module.js' %}"></script>
    <script type="text/javascript" src="{% static 'javascripts/utils/services/snackbar.service.js' %}"></script>

{x: include_utils_js}
Include `utils.module.js` and `snackbar.service.js` in `javascripts.html`

## Controlling the index interface with IndexController
Create a file in `static/javascripts/layout/controllers/` called `index.controller.js` and add the following:

    /**
    * IndexController
    * @namespace thinkster.layout.controllers
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.layout.controllers')
        .controller('IndexController', IndexController);

      IndexController.$inject = ['$scope', 'Authentication', 'Posts', 'Snackbar'];

      /**
      * @namespace IndexController
      */
      function IndexController($scope, Authentication, Posts, Snackbar) {
        var vm = this;

        vm.isAuthenticated = Authentication.isAuthenticated();
        vm.posts = [];

        activate();

        /**
        * @name activate
        * @desc Actions to be performed when this controller is instantiated
        * @memberOf thinkster.layout.controllers.IndexController
        */
        function activate() {
          Posts.all().then(postsSuccessFn, postsErrorFn);

          $scope.$on('post.created', function (event, post) {
            vm.posts.unshift(post);
          });

          $scope.$on('post.created.error', function () {
            vm.posts.shift();
          });


          /**
          * @name postsSuccessFn
          * @desc Update posts array on view
          */
          function postsSuccessFn(data, status, headers, config) {
            vm.posts = data.data;
          }


          /**
          * @name postsErrorFn
          * @desc Show snackbar with error
          */
          function postsErrorFn(data, status, headers, config) {
            Snackbar.error(data.error);
          }
        }
      }
    })();

{x: index_controller}
Make a new controller called `IndexController` in `static/javascripts/layout/controllers/index.controller.js`

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/layout/controllers/index.controller.js' %}"></script>

{x: index_controller_include_javascripts}
Include `index.controller.js` in `javascripts.html`

Let's touch on a couple of things here.

    $scope.$on('post.created', function (event, post) {
      vm.posts.unshift(post);
    });

Later, when we get around to creating a new post, we will fire off an event called `post.created` when the user creates a post. By catching this event here, we can add this new post to the front of the `vm.posts` array. This will prevent us from having to make an extra API request to the server for updated data. We will talk about this more shortly, but for now you should know that we do this to increase the *perceived* performance of our application.

    $scope.$on('post.created.error', function () {
      vm.posts.shift();
    });

Analogous to the previous event listener, this one will remove the post at the front of `vm.posts` if the API request returns an error status code.

## Making a route for the index page
With a controller and template in place, we need to set up a route for the index page.

Open `static/javascripts/thinkster.routes.js` and add the following route:

    .when('/', {
      controller: 'IndexController',
      controllerAs: 'vm',
      templateUrl: '/static/templates/layout/index.html'
    })

{x: index_route}
Add a route to `thinkster.routes.js` for the `/` path

## Making a directive for displaying Posts
Create `static/javascripts/posts/directives/posts.directive.js` with the following contents:

    /**
    * Posts
    * @namespace thinkster.posts.directives
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.posts.directives')
        .directive('posts', posts);

      /**
      * @namespace Posts
      */
      function posts() {
        /**
        * @name directive
        * @desc The directive to be returned
        * @memberOf thinkster.posts.directives.Posts
        */
        var directive = {
          controller: 'PostsController',
          controllerAs: 'vm',
          restrict: 'E',
          scope: {
            posts: '='
          },
          templateUrl: '/static/templates/posts/posts.html'
        };

        return directive;
      }
    })();

{x: posts_directive}
Make a new directive called `posts` in `static/javascripts/posts/directives/posts.directive.js`

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/posts/directives/posts.directive.js' %}"></script>

{x: posts_directive_include_js}
Include `posts.directive.js` in `javascripts.html`

There are two parts of the directives API that I want to touch on: `scope` and `restrict`.

    scope: {
      posts: '='
    },

`scope` defines the scope of this directive, similar to how `$scope` works for controllers. The difference is that, in a controller, a new scope is implicitly created. For a directive, we have the option of explicitly defining our scopes and that's what we do here.

The second line, `posts: '='` simply means that we want to set `$scope.posts` to the value passed in through the `posts` attribute in the template that we made earlier.

    restrict: 'E',

`restrict` tells Angular how we are allowed to use this directive. In our case, we set the value of `restrict` to `E` (for element) which means Angular should only match the name of our directive with the name of an element: `<posts></posts>`. 

Another common option is `A` (for attribute), which tells Angular to only match the name of the directive with the name of an attribute. `ngDialog` uses this option, as we will see shortly.

## Controller the posts directive with PostsController
The directive we just created requires a controller called `PostsController`. 

Create `static/javascripts/posts/controllers/posts.controller.js` with the following content:

    /**
    * PostsController
    * @namespace thinkster.posts.controllers
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.posts.controllers')
        .controller('PostsController', PostsController);

      PostsController.$inject = ['$scope'];

      /**
      * @namespace PostsController
      */
      function PostsController($scope) {
        var vm = this;

        vm.columns = [];

        activate();


        /**
        * @name activate
        * @desc Actions to be performed when this controller is instantiated
        * @memberOf thinkster.posts.controllers.PostsController
        */
        function activate() {
          $scope.$watchCollection(function () { return $scope.posts; }, render);
          $scope.$watch(function () { return $(window).width(); }, render);
        }
        

        /**
        * @name calculateNumberOfColumns
        * @desc Calculate number of columns based on screen width
        * @returns {Number} The number of columns containing Posts
        * @memberOf thinkster.posts.controllers.PostsControllers
        */
        function calculateNumberOfColumns() {
          var width = $(window).width();

          if (width >= 1200) {
            return 4;
          } else if (width >= 992) {
            return 3;
          } else if (width >= 768) {
            return 2;
          } else {
            return 1;
          }
        }


        /**
        * @name approximateShortestColumn
        * @desc An algorithm for approximating which column is shortest
        * @returns The index of the shortest column
        * @memberOf thinkster.posts.controllers.PostsController
        */
        function approximateShortestColumn() {
          var scores = vm.columns.map(columnMapFn);

          return scores.indexOf(Math.min.apply(this, scores));

          
          /**
          * @name columnMapFn
          * @desc A map function for scoring column heights
          * @returns The approximately normalized height of a given column
          */
          function columnMapFn(column) {
            var lengths = column.map(function (element) {
              return element.content.length;
            });

            return lengths.reduce(sum, 0) * column.length;
          }


          /**
          * @name sum
          * @desc Sums two numbers
          * @params {Number} m The first number to be summed
          * @params {Number} n The second number to be summed
          * @returns The sum of two numbers
          */
          function sum(m, n) {
            return m + n;
          }
        }


        /**
        * @name render
        * @desc Renders Posts into columns of approximately equal height
        * @param {Array} current The current value of `vm.posts`
        * @param {Array} original The value of `vm.posts` before it was updated
        * @memberOf thinkster.posts.controllers.PostsController
        */
        function render(current, original) {
          if (current !== original) {
            vm.columns = [];

            for (var i = 0; i < calculateNumberOfColumns(); ++i) {
              vm.columns.push([]);
            }

            for (var i = 0; i < current.length; ++i) {
              var column = approximateShortestColumn();

              vm.columns[column].push(current[i]);
            }
          }
        }
      }
    })();

{x: posts_controller}
Make a new controller called `PostsController` in `static/javascripts/posts/controllers/posts.controller.js`

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/posts/controllers/posts.controller.js' %}"></script>

{x: posts_controller_include_js}
Include `posts.controller.js` in `javascripts.html`

It isn't worth taking the time to step through this controller line-by-line. Suffice it to say that this controller presents an algorithm for ensuring the columns of posts are of approximately equal height.

The only thing worth mentioning here is this line:

    $scope.$watchCollection(function () { return $scope.posts; }, render);

Because we do not have direct access to the ViewModel that `posts` is stored on, we watch `$scope.posts` instead of `vm.posts`. Furthermore, we use `$watchCollection` here because `$scope.posts` is an array. `$watch` watches the object's reference, not it's actual value. `$watchCollection` watches the value of an array for changes. If we used `$watch` here instead of `$watchCollection`, the changes caused by `$scope.posts.shift()` and `$scope.posts.unshift()` would not trigger the watcher.

## Making a template for the posts directive
In our directive we defined a `templateUrl` that doesn't match any of our existing templates. Let's go ahead and make a new one.

Create `static/templates/posts/posts.html` with the following content:

    <div class="row" ng-cloak>
      <div ng-repeat="column in vm.columns">
        <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3">
          <div ng-repeat="post in column">
            <post post="post"></post>
          </div>
        </div>
      </div>

      <div ng-hide="vm.columns && vm.columns.length">
        <div class="col-sm-12 no-posts-here">
          <em>The are no posts here.</em>
        </div>
      </div>
    </div>

{x: posts_template}
Create a template for the `posts` directive

A few things worth noting:

1. We use the `ng-cloak` directive to prevent flashing since this directive will be used on the first page loaded.
2. We will need to create a `post` directive for rendering each individual post.
3. If no posts are present, we render a message informing the user.

## Making a directive for displaying a single Post
In the template for the posts directive, we use another directive called `post`. Let's create that.

Create `static/javascripts/posts/directives/post.directive.js` with the following content:

    /**
    * Post
    * @namespace thinkster.posts.directives
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.posts.directives')
        .directive('post', post);

      /**
      * @namespace Post
      */
      function post() {
        /**
        * @name directive
        * @desc The directive to be returned
        * @memberOf thinkster.posts.directives.Post
        */
        var directive = {
          restrict: 'E',
          scope: {
            post: '='
          },
          templateUrl: '/static/templates/posts/post.html'
        };

        return directive;
      }
    })();

{x: post_directive}
Make a new directive called `post` in `static/javascripts/posts/directives/post.directive.js`

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/posts/directives/post.directive.js' %}"></script>

{x: post_directive_include_js}
Include `post.directive.js` in `javascripts.html`

There is nothing new worth discussing here. This directive is almost identical to the previous one. The only difference is we use a different template.

## Making a template for the post directive
Like we did for the `posts` directive, we now need to make a template for the `post` directive.

Create `static/templates/posts/post.html` with the following content:

    <div class="row">
      <div class="col-sm-12">
        <div class="well">
          <div class="post">
            <div class="post__meta">
              <a href="/+{{ post.author.username }}">
                +{{ post.author.username }}
              </a>
            </div>

            <div class="post__content">
              {{ post.content }}
            </div>
          </div>
        </div>
      </div>
    </div>

{x: post_template}
Create a template for the `post` directive

## Some quick CSS
We want to add a few simple styles to make our posts look better. Open `static/stylesheets/styles.css` and add the following:

    .no-posts-here {
      text-align: center;
    }

    .post {}

    .post .post__meta {
      font-weight: bold;
      text-align: right;
      padding-bottom: 19px;
    }

    .post .post__meta a:hover {
      text-decoration: none;
    }

{x: post_css}
Add some CSS to `static/stylesheets/style.css` to make our posts look better

## Checkpoint
Assuming all is well, you can confirm you're on the right track by loading `http://localhost:8000/` in your browser. You should see the `Post` object you created at the end of the last section!

This also confirms that `PostViewSet` from the last section is working.

{x: checkpoint_render_posts}
Visit `http://localhost:8000/` and confirm the `Post` object you made earlier is shown.
